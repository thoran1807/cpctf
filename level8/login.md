### Final Answer

**Flag:** `cpctf{I_g0T_m4d_sk1lLz}`

---

### Problem Understanding

The challenge provided a single file named `login` and described it as a new CAS login page implementation that we should break. On the surface, this sounds like a web authentication problem, but the given artifact is actually a local executable, so the real task is binary analysis. The objective is to recover valid credentials (or the flag-generation path) and extract a flag in the format `cpctf{...}`.

---

### Thought Process

My first check was to identify the file type, because that decides the whole strategy. Once `file` confirmed it was an ELF64 binary, I shifted from web-testing mindset to reverse-engineering mindset.

I then used `strings` to quickly test whether meaningful constants were embedded in plaintext. This immediately revealed several high-value clues: a hardcoded username candidate (`0n3_W4rM`), a suspicious mixed-case token (`zLl1ks_d4m_T0g_I`), and a format string `cpctf{%s}`. That strongly suggested the flag might be derived directly from user-controlled input after passing checks.

At that point, I ruled out heavy dynamic debugging first (like full GDB tracing), because static clues were already strong. I disassembled `main` to confirm the exact logic. The key finding was a loop that reverses the supplied password string and compares the reversed result to `zLl1ks_d4m_T0g_I`. That means the correct input password is simply the reverse of that constant.

Finally, I executed the binary with the recovered username and reconstructed password to validate the full authentication flow and collect the printed flag.

---

### Tools and Techniques Used

- **Linux / Ubuntu (WSL):** `file`, `head`, `xxd`, `strings`, `nm`, `objdump`, `chmod`, direct binary execution.
- **Python 3:** not needed for this challenge.
- **CyberChef / online tools:** not used.

---

### Step-by-Step Solution

#### Step 1 — Initial Inspection

I started by identifying the artifact and checking whether it was text, archive, script, or executable.

```bash
file login
head -c 256 login | xxd -g 1 -u
```

`file` reported:

- ELF 64-bit LSB PIE executable
- dynamically linked
- not stripped

This was important because a non-stripped binary usually preserves symbol names, making static analysis much easier.

---

#### Step 2 — Locating the Hidden Data

Next, I searched for printable strings to quickly identify logic hints and credential material.

```bash
strings -n 6 login | head -n 120
```

This surfaced multiple critical strings:

- `0n3_W4rM`
- `zLl1ks_d4m_T0g_I`
- `Correct!`
- `Welcome back!`
- `cpctf{%s}`

This worked because many binaries store comparison constants and output templates in `.rodata` as plain ASCII. Even without full disassembly, this can reveal authentication logic shape.

---

#### Step 3 — Extraction and Decoding

To verify exactly how the password was checked, I disassembled `main`.

```bash
objdump -d -M intel --disassemble=main login
```

The disassembly showed:

1. Username is compared directly with `0n3_W4rM`.
2. Password input is copied into a heap buffer.
3. A loop reverses that buffer in place.
4. Reversed buffer is compared against `zLl1ks_d4m_T0g_I`.

So the required input password is:

```bash
echo 'zLl1ks_d4m_T0g_I' | rev
```

Result:

- `I_g0T_m4d_sk1lLz`

Then I ran the binary with recovered credentials:

```bash
chmod +x login
./login 0n3_W4rM I_g0T_m4d_sk1lLz
```

And received:

- `Correct!`
- `Welcome back!`
- `cpctf{I_g0T_m4d_sk1lLz}`

---

#### Step 4 — Flag

The final flag is:

```text
cpctf{I_g0T_m4d_sk1lLz}
```

---

### Why This Worked

- The binary stored key constants as readable strings, so quick string extraction gave immediate pivots.
- The executable was not stripped, making function-level disassembly straightforward.
- Authentication logic used reversible obfuscation (string reversal), not cryptographic verification.
- The flag format string `cpctf{%s}` directly embedded user-controlled content after successful checks.
- Static analysis was enough; no exploit memory corruption was required.

---

### Lessons Learned

- Always identify file type first. It prevents wasting time on the wrong attack surface.
- `strings` is often the fastest early triage tool for CTF binaries.
- If a binary is non-stripped, start with `main` disassembly before deep dynamic debugging.
- Obfuscation (reverse/XOR/rotate) is common in beginner-to-mid reverse challenges; verify transformations carefully.
- Confirm recovered logic by actually running the target with derived inputs, not just trusting static interpretation.

---

**Final Flag:** `cpctf{I_g0T_m4d_sk1lLz}`
