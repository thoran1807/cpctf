### Final Answer

**Flag:** `cpctf{2_m4ny_Qu3sti0n5}`

---

### Problem Understanding

The challenge provides a remote service (`nc 10.4.16.150 7100`) and the source code in `oracle.py`. At first glance, it looks like a simple "encrypt my input" service, but the real task is to recover the hidden `flag` that is appended to our input before encryption. The key detail is that the service uses AES-ECB with a fixed key for the whole session, which makes a chosen-plaintext attack possible.

---

### Thought Process

I started by reading the Python code to understand exactly what was being encrypted. The critical line was:

```python
enc = stream.encrypt(pad(secret + flag, 16)).hex()
```

This means I fully control `secret`, while `flag` is a constant unknown suffix. In ECB mode, identical plaintext blocks encrypt to identical ciphertext blocks under the same key. That is exactly the setup for a byte-at-a-time ECB oracle attack.

I also noticed there is a `for i in range(1000)` query limit. A naive byte-at-a-time attack would likely exceed that if I brute-force 256 guesses per byte. So I ruled out the naive approach and used a batched dictionary method: for each unknown byte position, I submitted many candidate blocks in one request, then mapped encrypted blocks back to candidate bytes. That kept the total query count low and fit comfortably under 1000.

---

### Tools and Techniques Used

- **Linux / Ubuntu (WSL):** `nc`, `printf`, `sed`
- **Python 3:** socket-based exploit script to automate the ECB byte-at-a-time recovery
- **Crypto technique:** AES-ECB chosen-plaintext suffix recovery (byte-at-a-time oracle), optimized with batched candidate blocks

---

### Step-by-Step Solution

#### Step 1 — Initial Inspection

First, I inspected the given source file to confirm encryption mode and data flow.

```bash
sed -n '1,200p' oracle.py
```

Important findings from the code:

- A random key is generated once, then hashed and reused.
- AES is initialized in ECB mode.
- Every query encrypts `pad(secret + flag, 16)`.

This immediately suggested an ECB oracle vulnerability because attacker-controlled bytes are placed directly before the secret flag.

---

#### Step 2 — Locating the Hidden Data

Next, I verified remote behavior and response formatting.

```bash
printf 'a\n' | nc -w 3 10.4.16.150 7100 | sed -n '1,6p'
```

The service printed a banner, asked for a secret, and returned a hex ciphertext line. That confirmed we can repeatedly query an encryption oracle and parse ciphertext reliably.

I then used short probing requests (`""`, `"A"`, `"AA"`, etc.) inside Python to detect when ciphertext length increases by one block. This gave me the unknown suffix length (the flag length) without guessing.

---

#### Step 3 — Extraction and Decoding

I used a Python script to automate recovery:

1. Connect once and keep the same session/key.
2. Infer flag length from ciphertext size growth.
3. For each flag byte:
   - Send a padding prefix to align the next unknown byte at block end.
   - Query target block.
   - Build a candidate dictionary by sending many 16-byte candidate blocks in one request.
   - Match ciphertext block to recover the correct byte.
4. Stop when `}` is recovered.

Core execution command:

```bash
python3 exploit.py
```

(Equivalent logic was run inline in terminal during solving.)

Recovery output showed progressive plaintext reconstruction:

- `cpctf`
- `cpctf{2_m4ny_`
- `cpctf{2_m4ny_Qu3sti0n5}`

No further decoding was needed after extraction because the oracle leaked raw flag bytes through ECB block matching.

---

#### Step 4 — Flag

Final recovered flag:

```text
cpctf{2_m4ny_Qu3sti0n5}
```

The format matches the expected CTF pattern `cpctf{...}`.

---

### Why This Worked

- ECB encrypts blocks independently, so block equality leaks information.
- The service appends a fixed secret (`flag`) to attacker-controlled input.
- By controlling prefix length, the next unknown flag byte can be aligned to a block boundary.
- A dictionary of candidate final bytes produces unique ciphertext blocks for comparison.
- Batching candidates per request made the attack efficient and safe under the 1000-query cap.

---

### Lessons Learned

- Always inspect source first: one line (`secret + flag` under ECB) can reveal the entire attack path.
- Query limits matter; optimize exploit design early instead of brute-forcing blindly.
- ECB is still dangerous in oracle-style APIs even when keys are random and unknown.
- Build tiny validation probes first (prompt parsing, length checks) before writing full automation.
- In crypto CTFs, understanding data layout and block boundaries is often more important than heavy tooling.

---

**Final Flag:** `cpctf{2_m4ny_Qu3sti0n5}`
