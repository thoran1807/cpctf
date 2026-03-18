### Final Answer

**Flag:** `cpctf{rs4_1s_fun}`

---

### Problem Understanding

The challenge gives us a standard RSA setup: a large modulus `n`, public exponent `e = 65537`, and ciphertext `c`. At first glance, this looks like normal public-key encryption where decryption should be impossible without the private key. The hint changes everything: "If only there was some place where people saved p and q." That strongly suggests the modulus may already be factored publicly, which would let us rebuild the private key and decrypt the message.

---

### Thought Process

My first check was the provided `main.py` in the challenge folder. It generates two 512-bit primes `p` and `q`, computes `n = p*q`, and encrypts a flag with textbook RSA. There was no padding trick or small-exponent weakness to exploit directly from `c`, so brute force and algebraic shortcuts were not realistic.

The key clue was the hint about a "place where people saved p and q." In CTF crypto, that usually points to FactorDB, where previously factored integers are indexed. If `n` was already in the database, I could recover `p` and `q` immediately, compute Euler's totient, derive `d`, and decrypt in one clean pass.

I ruled out unnecessary paths like trying random RSA attacks (Wiener, Coppersmith, etc.) because there was no evidence for weak key structure. The challenge was intentionally about recognizing the operational weakness: reusing or exposing a factorable modulus.

---

### Tools and Techniques Used

- **Linux / Ubuntu (WSL):**
  - `cat` to inspect the provided Python challenge script
  - Browser/API request to query FactorDB for factorization of `n`
- **Python 3:**
  - Computed `phi(n) = (p-1)*(q-1)`
  - Computed modular inverse `d = e^-1 mod phi(n)`
  - Decrypted with `m = c^d mod n` and converted bytes back to text
- **Online tools:**
  - **FactorDB** to retrieve the prime factors of the provided modulus

---

### Step-by-Step Solution

#### Step 1 — Initial Inspection

I started by reading the provided script to understand what cryptosystem was implemented and whether there was an implementation bug.

```bash
cat main.py
```

The script is straightforward textbook RSA:
- `p` and `q` are generated as 512-bit primes
- `n = p*q`
- `e = 65537`
- plaintext flag is converted to an integer and encrypted with `pow(flag, e, n)`

This told me the challenge was not about parsing files or hidden encodings. It was about recovering the private key material (`p`, `q`) from external leakage.

---

#### Step 2 — Locating the Hidden Data

The hint suggested that `p` and `q` were already "saved" somewhere. I queried FactorDB with the exact `n` value.

```bash
curl "http://factordb.com/api?query=<n>"
```

The response status was `FF` (fully factored) and returned two 154-digit prime factors. That was the critical breakthrough: once `n` is factored, RSA decryption is trivial.

Why this worked: RSA security depends on the hardness of factoring `n`. If someone already factored and published it, the security assumption is gone.

---

#### Step 3 — Extraction and Decoding

With `p` and `q` in hand, I rebuilt the private exponent and decrypted the ciphertext.

```bash
python3 - <<'EOF'
p = 8952581833553607758422434385906933655248639557657548998272471781143910363056397446064443786679680360038546263114609287069848462153554390401132720428294029
q = 9705532396930301862146935914204124231103476346932280099362675611895961398228632604709722757766506937297886324132363485415623328376108264546454261248250289

n = 86889573021724223452823629956282581430966785839271025745185673210984135908879616562838779064993737189566866152297445808849417592631987421141172908742547318692794327427845262007424953103186836209237759425149392375754544303937867677477662641639760905087073552114858125861382655173429827618696881587765476224381
e = 65537
c = 57852569926792480745754709038102276898645720754457377792103227252821302385365375037596158719458221281855582325313562785229162331608241754358584586238089579610798797850062618038313431424911376028924665977269392708262684490686109631081038867172128498053736663727338514730641842638783312448736000637530374980646

phi = (p - 1) * (q - 1)
d = pow(e, -1, phi)
m = pow(c, d, n)
print(m.to_bytes((m.bit_length() + 7) // 8, 'big').decode())
EOF
```

Output:

```text
cpctf{rs4_1s_fun}
```

No additional decoding was needed.

---

#### Step 4 — Flag

The recovered plaintext matches the expected CTF flag format:

```text
cpctf{rs4_1s_fun}
```

---

### Why This Worked

- RSA decryption is easy once `p` and `q` are known.
- The challenge hint explicitly pointed to external factor storage (FactorDB).
- The modulus `n` was already fully factored (`FF`) in the public database.
- From `p` and `q`, we can deterministically derive `phi(n)` and private exponent `d`.
- Textbook RSA math (`m = c^d mod n`) directly recovers the original flag integer.

---

### Lessons Learned

- Always treat challenge hints as technical direction, not flavor text.
- In RSA CTF tasks, check whether `n` is publicly factored before trying advanced attacks.
- Choose attacks based on evidence from key parameters, not random trial-and-error.
- Understand the full RSA key lifecycle: `p,q -> n,phi -> d -> decrypt`.
- Operational leakage (publishing factors) can break strong cryptography instantly.

---

**Final Flag:** `cpctf{rs4_1s_fun}`
