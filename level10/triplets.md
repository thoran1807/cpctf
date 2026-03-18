### Final Answer

**Flag:** `cpctf{y0u_n3v3r_kn3w_rs4_was_so_easy}`

---

### Problem Understanding

The challenge gives you a Python generator and RSA parameters: `p`, `q`, `n`, `e`, and `c`. At first glance, it looks like normal RSA decryption with two primes. The hidden twist is in the title and source code: they "added one more number," which means the modulus is not `p*q` but `p*q*r`.

---

### Thought Process

I started from the source code because challenge hints are usually embedded in implementation details. In `main.py`, the modulus is explicitly built as `n = p*q*r`, while only `p` and `q` are printed, so the third factor is intentionally hidden in plain sight.

Once that clicked, the attack path became straightforward: if `n` is divisible by `p*q`, then the missing value is `r = n // (p*q)`. After recovering `r`, this becomes standard private-key reconstruction for multi-prime RSA: compute $\varphi(n)$, find `d`, and decrypt `c`.

I ruled out factoring `n` directly because that would be unnecessarily expensive compared to simple division by known factors. I also ruled out padding-based attacks because the challenge uses raw modular exponentiation with integer conversion.

---

### Tools and Techniques Used

- **Linux / Ubuntu (WSL):**
  - `python3` for arithmetic checks and decryption
  - basic file reading (`cat`) to inspect challenge source
- **Python 3:**
  - verify divisibility relation `n % (p*q) == 0`
  - recover hidden prime `r`
  - compute $\varphi(n)$ and modular inverse `d = e^{-1} mod \varphi(n)`
  - decrypt ciphertext and convert integer plaintext to bytes

---

### Step-by-Step Solution

#### Step 1 — Initial Inspection

I inspected the provided source first:

```bash
cd /home/thoran/CPCTF/level_10
cat main.py
```

Key finding from the script:

```python
n = p*q*r
```

This line confirms the modulus is multi-prime RSA, not regular two-prime RSA.

---

#### Step 2 — Locating the Hidden Data

The hidden value is `r`, and it is already embedded in `n`. Since `p` and `q` are known, I checked whether `p*q` divides `n` exactly:

```bash
python3 - <<'PY'
p = 11129421248302716213621154368061472369977977436193110272905628732507690919618767562784943322473801338939203714756047813879696828999563459693654171641039943
q = 8589515261180711920573115614141399991700568002235639267203379568707680312011899778417492092456302023817141043184220845800313824928317607108429371044635561
n = 700128010967189562416557502697811887178816047622099652292758612567741635115707933020859960408359004881347661814718983987794412978927333060835962591183618081170778680181960982724982222331355728601776343990179904110494927020243953416958000020295661527781232411627685984867456546925893191379017588551061324415332557405328369111401299044385183424980476153853169466634503122290295802719638386159752785908063399142476253202235424444611547554396472889660327384416600323

print(n % (p*q))
print(n // (p*q))
PY
```

The remainder is `0`, so `r` is recovered exactly as `n // (p*q)`.

---

#### Step 3 — Extraction and Decoding

After recovering `r`, I reconstructed the private key and decrypted `c`:

```bash
python3 - <<'PY'
from math import gcd

p = 11129421248302716213621154368061472369977977436193110272905628732507690919618767562784943322473801338939203714756047813879696828999563459693654171641039943
q = 8589515261180711920573115614141399991700568002235639267203379568707680312011899778417492092456302023817141043184220845800313824928317607108429371044635561
n = 700128010967189562416557502697811887178816047622099652292758612567741635115707933020859960408359004881347661814718983987794412978927333060835962591183618081170778680181960982724982222331355728601776343990179904110494927020243953416958000020295661527781232411627685984867456546925893191379017588551061324415332557405328369111401299044385183424980476153853169466634503122290295802719638386159752785908063399142476253202235424444611547554396472889660327384416600323
e = 65537
c = 641322175216219311710275597547444613595160629413714574035294230041460640042199229036835740169443315689145719137778899186397099034937576906555029173831974702415973308137881441026827241174913194922674495418702752242113406229191088910544021410809978885162183892280436521492152428682874038043768203634033103741780632545280820676881003387717032240067004481789293735374697390188832954413942866423728941347653845888702847099352088749855048700817193458070033514482801965

r = n // (p*q)
phi = (p-1)*(q-1)*(r-1)
assert gcd(e, phi) == 1

d = pow(e, -1, phi)
m = pow(c, d, n)
pt = m.to_bytes((m.bit_length() + 7) // 8, 'big')
print(pt.decode())
PY
```

Output:

```text
cpctf{y0u_n3v3r_kn3w_rs4_was_so_easy}
```

No extra decoding layer was needed.

---

#### Step 4 — Flag

The final recovered flag is:

```text
cpctf{y0u_n3v3r_kn3w_rs4_was_so_easy}
```

---

### Why This Worked

- The challenge leaked two prime factors directly (`p` and `q`).
- The modulus was constructed as `p*q*r`, so the missing prime was recoverable by exact division.
- Multi-prime RSA still follows the same decryption logic once all factors are known.
- With $\varphi(n)$ available, computing `d` is deterministic.
- The ciphertext was raw RSA integer encryption, so converting the decrypted integer to bytes reveals the flag immediately.

---

### Lessons Learned

- Always read the provided source before attempting hard math; many CTF crypto tasks leak structure in code.
- Small wording hints in problem statements often map to exact implementation details.
- If some RSA factors are known, test divisibility shortcuts before trying generic factoring.
- Keep a quick integrity check (`n % (p*q) == 0`) to avoid wasting time on wrong assumptions.
- In CTF crypto, simplicity often beats complexity when the challenge author intentionally exposes part of the key.

---

**Final Flag:** `cpctf{y0u_n3v3r_kn3w_rs4_was_so_easy}`