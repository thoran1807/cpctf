### Final Answer

**Flag:** `cpctf{p4p3r_sc1ss0r_r0cky0u}`

---

### Problem Understanding

The challenge gives two files: a Python script (`rockme.py`) and an encrypted blob (`enc`).
At first glance, this looks like "strong AES encryption," but the real task is not breaking AES itself. The hidden weakness is key management: the AES key is derived from a human password, and that password can be guessed with a dictionary attack.

---

### Thought Process

I started by reading the Python code instead of touching the ciphertext first, because crypto challenge logic is usually in the script. The script showed AES-ECB decryption with `key = SHA256(password)`, and no salt, no key stretching, and no authentication. That immediately suggested an offline password attack: if I can try many passwords locally, I can test candidates very quickly.

I ruled out "cryptanalysis" paths (like attacking AES mode structure) because AES-ECB is still cryptographically strong against direct key recovery. The practical weakness here was the password itself, not the AES algorithm. So the correct strategy was to get a good password list and test candidates until decrypted output matched the expected `cpctf{...}` format.

---

### Tools and Techniques Used

- **Linux / Ubuntu (WSL):**
  - `file`, `wc`, `xxd` to inspect ciphertext format and size
  - `curl` to download a common password wordlist
- **Python 3:**
  - scripted AES-ECB decryption attempts over a dictionary
  - `hashlib.sha256` for key derivation matching challenge logic
  - regex check for `cpctf{...}` to detect valid plaintext
- **Package installation:**
  - `sudo apt-get install -y python3-pycryptodome`

---

### Step-by-Step Solution

#### Step 1 — Initial Inspection

I first inspected the ciphertext and the script to understand the exact decryption pipeline.

```bash
cd /home/thoran/CPCTF/level_8
file enc
wc -c enc
xxd -g 1 -l 128 enc
cat rockme.py
```

What this showed:
- `enc` is raw binary data, 32 bytes long (exactly two AES blocks)
- `rockme.py` decrypts using AES-ECB
- key is `sha256(password).digest()`
- no IV, no salt, no PBKDF2/bcrypt/scrypt, no integrity check

This is a textbook setup for fast offline password guessing.

---

#### Step 2 — Locating the Hidden Data

The useful "hidden data" here is not embedded metadata inside `enc`; it is the unknown password that generated the AES key. So the key step was preparing a realistic password corpus.

```bash
cd /home/thoran/CPCTF/level_8
curl -L --fail -o top100k.txt \
  https://raw.githubusercontent.com/danielmiessler/SecLists/master/Passwords/Common-Credentials/10k-most-common.txt
wc -l top100k.txt
head -n 5 top100k.txt
```

Why this worked:
- the challenge title hints overconfidence in AES, which often means weak password usage
- common-password lists are highly effective for CTF password-derived-key challenges

---

#### Step 3 — Extraction and Decoding

I ran a Python script that:
1. reads each password candidate
2. computes `SHA-256(password)`
3. decrypts `enc` with AES-ECB
4. checks if plaintext contains `cpctf{...}`

```bash
cd /home/thoran/CPCTF/level_8
/usr/bin/python3 - <<'PY'
import hashlib
import re
from Cryptodome.Cipher import AES

enc = open('enc', 'rb').read()
pat = re.compile(r'cpctf\{[^\n\r\t\x00]{1,100}\}', re.I)

with open('top100k.txt', 'r', encoding='utf-8', errors='ignore') as f:
    for line in f:
        password = line.strip()
        if not password:
            continue
        key = hashlib.sha256(password.encode()).digest()
        pt = AES.new(key, AES.MODE_ECB).decrypt(enc)
        try:
            s = pt.decode('utf-8')
        except UnicodeDecodeError:
            continue
        if pat.search(s):
            print('password =', password)
            print('plaintext =', repr(s))
            break
PY
```

Result:
- recovered password: `iloveyou2`
- decrypted text: `cpctf{p4p3r_sc1ss0r_r0cky0u}\x04\x04\x04\x04`

The trailing bytes are valid PKCS#7 padding, so I removed padding to get the clean flag.

```bash
cd /home/thoran/CPCTF/level_8
/usr/bin/python3 - <<'PY'
import hashlib
from Cryptodome.Cipher import AES
from Cryptodome.Util.Padding import unpad

enc = open('enc', 'rb').read()
password = 'iloveyou2'
key = hashlib.sha256(password.encode()).digest()
pt = AES.new(key, AES.MODE_ECB).decrypt(enc)
print(unpad(pt, 16).decode())
PY
```

Output:

```text
cpctf{p4p3r_sc1ss0r_r0cky0u}
```

---

#### Step 4 — Flag

The recovered plaintext matches the expected flag format:

```text
cpctf{p4p3r_sc1ss0r_r0cky0u}
```

---

### Why This Worked

- AES itself was not broken; the password-derived key was weak.
- `SHA-256(password)` without salt or stretching allows very fast offline guessing.
- ECB mode gave deterministic block encryption, but the practical break here was dictionary recovery of the key.
- The challenge plaintext had a recognizable format (`cpctf{...}`), making candidate validation easy.
- Once the correct password was found, decryption was immediate and deterministic.

---

### Lessons Learned

- In crypto CTFs, inspect the implementation first; design mistakes often beat heavy math attacks.
- "Strong algorithm" does not help if key generation is weak.
- Always use salted, slow KDFs (PBKDF2, scrypt, Argon2) for password-based encryption.
- Build fast validation checks (like expected flag regex) to automate large candidate testing.
- Distinguish between algorithm security and operational security; most real breaks are operational.

---

**Final Flag:** `cpctf{p4p3r_sc1ss0r_r0cky0u}`
