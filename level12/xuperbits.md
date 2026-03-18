### Final Answer

**Flag:** `cpctf{n4k3d_xoring_1s_n0t_s3cure_ol0l_1t_1s_4ctu4lly_4_m3th0d_0f_3ncrypt10n_w0w_n41s_k00md4n_1s_4ctu4lly_4_g3n1us}`

---

### Problem Understanding

The challenge gives two files: `gen.py` and `encxorlol`. At first glance, the prompt tries to distract with wording about private keys and ECB mode, but this is not an AES/ECB challenge in practice. The real task is to reverse a custom bit-based encoding built from Python's `random` outputs and XOR constraints. So the core objective is to recover the original flag bits from structured leakage written into `encxorlol`.

---

### Thought Process

I started from the source file because custom crypto challenges are usually solvable only after understanding exactly what the author leaked. In `gen.py`, the flag is converted to a big integer, then consumed bit-by-bit (least significant bit first). For every bit, one 180-bit random value is generated and stored in `randoms`; when the bit is `1`, that random value is also appended to `randomsused`.

That gave two important observations:

1. `encxorlol` contains the full `randoms` list, so all per-bit random values are known.
2. It also contains XORs of each consecutive group of 5 elements from `randomsused`, plus `len(randomsused)`.

This means we do not need to break Python RNG or guess any key. We only need to determine which indices in `randoms` were selected as `1` bits, such that their order-respecting groups of five match the published XOR targets. Because each target is a 180-bit XOR and the numbers are random-looking, collisions are extremely unlikely, so each group can be identified uniquely with constrained combination search.

I ruled out brute-forcing the flag directly because the leak is already structured enough for deterministic recovery. I also ruled out trying to recover the RNG seed first, because selecting indices from known random outputs is much simpler than inverting Mersenne Twister from this specific data flow.

---

### Tools and Techniques Used

- **Linux / Ubuntu (WSL):** basic shell commands (`head`, quick one-liners) to inspect files and verify structure.
- **Python 3:** main solver language for parsing the lists (`ast.literal_eval`), matching XOR-of-5 constraints, rebuilding the bitstream, and decoding bytes.
- **CyberChef / online tools:** not used.

---

### Step-by-Step Solution

#### Step 1 — Initial Inspection

I inspected both files to understand the format and generation logic.

```bash
head -3 encxorlol
sed -n '1,220p' gen.py
```

`encxorlol` has exactly three lines: the `randoms` list, the `threerandomxors` list (despite the name, it is XOR of 5 values), and `len(randomsused)`. The code in `gen.py` confirms that each flag bit consumes one random value, and only `1` bits contribute to the grouped XOR constraints. This immediately frames the problem as recovering a hidden subsequence.

---

#### Step 2 — Locating the Hidden Data

Next, I extracted structure metrics from the encoded file to validate assumptions before writing the full solver.

```bash
python3 - <<'PY'
import ast
from pathlib import Path
lines = Path('encxorlol').read_text().splitlines()
R = ast.literal_eval(lines[0])
Y = ast.literal_eval(lines[1])
m = int(lines[2])
print('len(R)=', len(R))
print('len(Y)=', len(Y))
print('m=', m, 'm%5=', m % 5, '5*len(Y)=', 5*len(Y))
PY
```

The output showed `len(R)=911`, `len(Y)=99`, and `m=495`, with `495 = 99*5`. That confirms all selected values are fully covered by complete groups of five, with no leftovers. This property is what makes exact reconstruction possible.

---

#### Step 3 — Extraction and Decoding

I solved each XOR group in order. For each target `Y[g]`, I searched combinations of 5 indices from the current forward window of `R` (enforcing increasing order), found the unique combination whose XOR matched, recorded those indices as selected `1` bits, and moved the pointer past the fifth selected index. After all groups were matched, I converted selected indices back into a bit array, rebuilt the integer in LSB-first order, and decoded bytes.

```bash
python3 - <<'PY'
import ast, itertools
from pathlib import Path

lines = Path('encxorlol').read_text().splitlines()
R = ast.literal_eval(lines[0])
Y = ast.literal_eval(lines[1])

n = len(R)
pos = 0
picked = []

for target in Y:
    found = []
    for W in [10,12,15,20,25,30,35,40,50,60,80,120,200,n]:
        end = min(n, pos + W)
        if end - pos < 5:
            break
        for comb in itertools.combinations(range(pos, end), 5):
            if R[comb[0]] ^ R[comb[1]] ^ R[comb[2]] ^ R[comb[3]] ^ R[comb[4]] == target:
                found.append(comb)
        if found:
            break

    # In this challenge, every group resolved uniquely.
    c = sorted(found)[0]
    picked.extend(c)
    pos = c[-1] + 1

bits = [0] * n
for i in picked:
    bits[i] = 1

value = 0
for i, b in enumerate(bits):
    if b:
        value |= (1 << i)

flag = value.to_bytes((value.bit_length() + 7) // 8, 'big').decode('latin1')
print(flag)
PY
```

This prints the full flag string directly. One terminal rendering split part of the text visually across lines due to display width, but the underlying string is continuous and valid.

---

#### Step 4 — Flag

Final recovered value (format checked):

```text
cpctf{n4k3d_xoring_1s_n0t_s3cure_ol0l_1t_1s_4ctu4lly_4_m3th0d_0f_3ncrypt10n_w0w_n41s_k00md4n_1s_4ctu4lly_4_g3n1us}
```

---

### Why This Worked

- The challenge leaks all per-bit random values in order (`randoms`), so the unknown is only the selection mask.
- The mask is constrained by 99 high-entropy equations, each XORing exactly 5 selected values in sequence.
- Grouping is deterministic and complete (`len(randomsused)` is divisible by 5), so there is no ambiguous tail case.
- 180-bit random numbers make accidental XOR collisions rare, so each group is effectively unique in practice.
- Once selected indices are known, reconstructing the original integer and bytes is straightforward.

---

### Lessons Learned

- In custom crypto challenges, always read the generator code first; many “hard” problems become deterministic after leakage analysis.
- Distinguish between “random-looking” output and actual secrecy. Here, randomness existed, but key structure was overexposed.
- Check list sizes and divisibility early. Small arithmetic checks often reveal whether a reconstruction path is feasible.
- Use constraints from data generation order (monotonic indices here) to reduce search space massively.
- If terminal output looks broken, verify raw string representation before assuming decoding failed.

---

**Final Flag:** `cpctf{n4k3d_xoring_1s_n0t_s3cure_ol0l_1t_1s_4ctu4lly_4_m3th0d_0f_3ncrypt10n_w0w_n41s_k00md4n_1s_4ctu4lly_4_g3n1us}`
