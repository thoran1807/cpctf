### Final Answer

**Flag:** `cpctf{l0l_1dk_h0w_t0_mak3_fl4g5}`

---

### Problem Understanding

The challenge provides two files in level_9: `gen.py` and `enclol`. On the surface, it looks like a custom encryption scheme with many large random numbers, so it feels like a hard brute-force problem. The real task is to read the generator logic carefully and use the leaked structure (`randoms`, grouped OR values, and total sum) to reconstruct the original plaintext bits directly. In short, this is not about breaking randomness; it is about exploiting side-channel leakage in the format.

---

### Thought Process

I started with the code, not the ciphertext, because custom crypto challenges usually fail in implementation details. In `gen.py`, each bit of the flag decides whether one 180-bit random value is added to `randsum`. That is already a subset-sum structure. However, the script also writes `threerandomors`, which are bitwise ORs of selected random values in chunks of three. That is a strong leak.

The key observation was this: if a random number `x` was part of a leaked OR target `t`, then `x | t = t` must hold. I tested this relation across all random values for each OR group, and it produced exactly three matches per group (and one match for the last group). This meant I could recover the exact selected random entries, then map their indices back to bit positions of the original integer and decode it to text.

I ruled out generic brute force and advanced cryptanalysis because the leakage made direct reconstruction possible and verifiable with the published `randsum`.

---

### Tools and Techniques Used

- **Linux / Ubuntu (WSL):**
  - `file` to identify `enclol` format
  - `wc -c` to check payload size
  - `xxd` to inspect raw bytes quickly
- **Python 3:**
  - parsing serialized lists from `enclol`
  - checking OR-subset relation `x | t == t`
  - reconstructing selected indices, verifying `randsum`, and converting recovered bitset back to bytes

---

### Step-by-Step Solution

#### Step 1 — Initial Inspection

I first checked the encrypted file and generator script.

```bash
cd /home/thoran/CPCTF/level_9
file enclol
wc -c enclol
xxd -l 256 enclol
cat gen.py
```

What this showed:
- `enclol` is text-like serialized data, not opaque binary ciphertext
- it contains three lines: a full random list, grouped OR leaks, and a final sum
- `gen.py` seeds Python RNG with the flag integer, then conditionally accumulates random terms when a bit is `1`

This immediately suggested a recoverable subset-selection problem rather than a one-way encryption primitive.

---

#### Step 2 — Locating the Hidden Data

The useful hidden signal was in `threerandomors`. For each OR target `t`, I searched all random values `x` that satisfy:

```text
x | t == t
```

I validated this with a quick parser:

```bash
cd /home/thoran/CPCTF/level_9
python3 - <<'PY'
import ast
with open('enclol','rb') as f:
    a,b,c = f.read().decode().splitlines()
R = ast.literal_eval(a)
T = ast.literal_eval(b)
for k,t in enumerate(T):
    cand = [i for i,x in enumerate(R) if (x | t) == t]
    print(k, len(cand), cand)
PY
```

Why this worked:
- OR can only keep or set bits, never unset them
- if `x` contributes to `t`, all `1` bits in `x` must also be `1` in `t`
- in this dataset, each group produced unique candidates (three per group, one in the last group), so the selected items were directly recoverable

---

#### Step 3 — Extraction and Decoding

After identifying selected indices, I rebuilt the message integer by setting those bit positions, then decoded to bytes.

```bash
cd /home/thoran/CPCTF/level_9
python3 - <<'PY'
import ast

with open('enclol','rb') as f:
    a,b,c = f.read().decode().splitlines()

R = ast.literal_eval(a)
T = ast.literal_eval(b)
S = int(c)

selected = []
for t in T:
    cand = [i for i,x in enumerate(R) if (x | t) == t]
    selected.extend(cand)

idx = sorted(set(selected))

# integrity check: recovered selected terms must match leaked total sum
assert sum(R[i] for i in idx) == S

m = 0
for i in idx:
    m |= (1 << i)

pt = m.to_bytes((m.bit_length() + 7) // 8, 'big')
print(pt.decode())
PY
```

Output:

```text
cpctf{l0l_1dk_h0w_t0_mak3_fl4g5}
```

No additional decoding was required.

---

#### Step 4 — Flag

The recovered plaintext matches the expected CTF format:

```text
cpctf{l0l_1dk_h0w_t0_mak3_fl4g5}
```

---

### Why This Worked

- The challenge leaked far more than a secure encryptor should: full random terms, grouped ORs, and exact selected-term sum.
- The OR leakage allowed direct identification of selected terms via the relation `x | t == t`.
- Grouping selected terms in triples preserved enough structure to recover each chunk with almost no ambiguity.
- Recovered indices are exactly the original bit positions of the flag integer.
- `randsum` provided a clean integrity check, confirming the reconstruction before decoding.

---

### Lessons Learned

- Always read the generation code first in custom-crypto CTFs; implementation leakage is often the real weakness.
- If a scheme exposes intermediate values, test algebraic relations (`OR`, `XOR`, sums) before attempting brute force.
- Treat “encryption” outputs as data formats, not black boxes; serialization choices can reveal secrets.
- Build verification steps into your solve flow (here, matching `randsum`) to avoid false positives.
- Randomness does not imply security when structural side channels leak selection information.

---

**Final Flag:** `cpctf{l0l_1dk_h0w_t0_mak3_fl4g5}`