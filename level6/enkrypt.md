### Final Answer

**Flag:** `cpctf{r5a_5h0u1d_b3_4v01d3d_4t_411_c05t5}`

---

### Problem Understanding

The challenge provides a Python script that generates RSA keys with `e = 5`, then encrypts a short text message directly as an integer using `c = m^e mod n`. Alongside that script, we are given concrete public values `n`, `e`, and `c`, and asked to recover the flag. On the surface this looks like normal RSA, but the hidden task is to spot that this is textbook RSA without padding, which is unsafe for short messages and small public exponents.

---

### Thought Process

My first check was the encryption flow in `main.py`: convert plaintext bytes to an integer and apply raw modular exponentiation. There was no OAEP, PKCS#1 v1.5 padding, or any randomness. That immediately suggested a classic low-exponent attack.

Because `e = 5` and the plaintext is short (a CTF flag sentence), I considered the condition `m^5 < n`. If true, then RSA encryption does not wrap modulo `n`, so ciphertext is literally `c = m^5`. In that case, recovering `m` is just an integer fifth root of `c`, no factorization needed.

I ruled out heavy approaches like factoring `n` or trying side channels, because this weakness is much simpler and matches the challenge hint about “breaking RSA.” Once I got an exact fifth root and decoded bytes cleanly into ASCII text, the flag was directly visible.

---

### Tools and Techniques Used

- **Linux / Ubuntu:**
  - `cat` to inspect the provided script quickly
  - shell + `python3` for direct cryptanalysis execution
- **Python 3:**
  - big-integer arithmetic
  - binary search to compute an exact integer fifth root
  - hex/bytes decoding to convert the recovered integer into readable text
- **CyberChef / online tools:**
  - Not required for this challenge

---

### Step-by-Step Solution

#### Step 1 — Initial Inspection

I started by reading the provided script to understand how encryption was implemented.

```bash
cd /home/thoran/CPCTF/level_6
cat main.py
```

The relevant logic was:

```python
key = RSA.generate(4096, e=5)
m = int(binascii.hexlify(msg.encode()), 16)
c = pow(m, key.e, key.n)
```

This confirmed textbook RSA with a small exponent and no padding. That is important because secure RSA encryption should use randomized padding (such as OAEP). Without it, structural attacks become possible.

---

#### Step 2 — Locating the Hidden Data

In this challenge, the “hidden data” is not in a file footer or metadata; it is hidden in plain sight inside the math. The key observation is that short plaintext plus small `e` can make `m^e` smaller than `n`.

That means:

- RSA equation: `c = m^e mod n`
- If `m^e < n`, then modular reduction does nothing
- So ciphertext is exactly `c = m^e`

At that point, the target becomes finding an exact integer fifth root of `c`.

---

#### Step 3 — Extraction and Decoding

I used Python to compute the integer fifth root and decode the resulting integer as bytes.

```bash
python3 - << 'PY'
c = 77079255473347455288225847541329478054660339231217815305701324893250377045466546134894689116587146216425214893321346033573340374952321729350620768471452090473415813358829966284905115399672311503759907955522407936628505898408930012889924865829995626779070685475597829604648289299915617713143034552720907566470043379609176287345593948132897809291714796693552889746048811993457741292137568135600946906920529751073976384145310044916443361783577349736206243345264096444313182110834807113726188242527013616018292463874654283129898216888610120635298443021643669763358868746946714797217812663425310522465343511044433121715041334030120196925543732414795004740665968172034909174581644958341196834267217431271625893337012920175960012168543421346835963135373351619122233018903144271133907579024532548822546293706384655995953430527785271518477940656232641961790901070884107094085939868063038618496981187720002026345963388842616404235908071780092448639782969584810291777047392841131256532449061489463316500888541998542373439495022307421529041871501

# Integer 5th root via binary search
lo, hi = 0, 1
while hi**5 <= c:
    hi *= 2
while lo + 1 < hi:
    mid = (lo + hi) // 2
    p = mid**5
    if p == c:
        lo = mid
        break
    if p < c:
        lo = mid
    else:
        hi = mid

m = lo
print("exact root:", m**5 == c)

h = hex(m)[2:]
if len(h) % 2:
    h = "0" + h
msg = bytes.fromhex(h).decode()
print(msg)
PY
```

Output showed an exact root and decoded plaintext:

```text
welcome to cpctf!
your super secret flag is: cpctf{r5a_5h0u1d_b3_4v01d3d_4t_411_c05t5}
```

No further transformation was needed.

---

#### Step 4 — Flag

The extracted flag is:

```text
cpctf{r5a_5h0u1d_b3_4v01d3d_4t_411_c05t5}
```

It matches the expected CTF format.

---

### Why This Worked

- The scheme used raw RSA encryption with no padding.
- Public exponent `e = 5` is small, increasing risk for short messages.
- The plaintext integer was small enough that `m^5 < n`.
- Because of that, `c` equaled `m^5` exactly, not a wrapped modular value.
- Taking an integer fifth root directly recovered the plaintext.

---

### Lessons Learned

- Never use textbook RSA for encryption; always use OAEP (or a modern hybrid encryption design).
- Small public exponents are not automatically unsafe, but become dangerous when padding is missing.
- Before attempting expensive attacks (like factoring), always check for structural weaknesses.
- In CTF crypto, reading the source carefully often reveals the intended vulnerability quickly.
- When ciphertext comes from deterministic math, exact integer-root checks are a fast and high-value test.

---

**Final Flag:** `cpctf{r5a_5h0u1d_b3_4v01d3d_4t_411_c05t5}`
