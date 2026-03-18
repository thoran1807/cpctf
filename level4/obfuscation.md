# CTF Challenge: Obfuscation Reversal (memfrob)

### Final Answer

**Flag:** `cpctf{m3mfr0b_1s_th3_be5t_3ncrypti0n}`

---

### Problem Understanding

The challenge provides two files: a binary executable (`chall`) and an obfuscated text file (`flag`). The hint mentions that while reading man pages, something interesting was discovered. The task is to understand how the flag file is obfuscated and reverse the process to recover the flag in the format `cpctf{...}`.

The real hidden task isn't simply finding or decoding data—it's recognizing a specific C library function commonly discussed in Unix documentation and understanding its cryptographic weakness.

---

### Thought Process

The first instinct was to examine the binary to understand what it does. When `strings` revealed references to a function called `memfrob`—along with a comment explicitly mentioning discovering this function—it became clear that this was the intended attack vector.

`memfrob` is a GNU C library function that performs XOR operations with a fixed constant (0x2A). It's famous in security contexts precisely because it's weak obfuscation—the fixed key makes it trivial to reverse. The problem statement mentioning "man pages" was a direct hint: `memfrob` is documented in the Linux man pages.

Given that the binary used this function and the flag file contained garbled text, the logical conclusion was that the flag file had been processed with `memfrob`. Since XOR is symmetric (XORing twice with the same value returns the original), applying the same operation would decrypt the flag.

---

### Tools and Techniques Used

- **`file`** — identified that `chall` is an ELF 64-bit executable
- **`strings`** — extracted human-readable strings from the binary, revealing references to `memfrob`
- **`chmod`** — made the binary executable
- **Python 3** — used to XOR the flag file contents with the `memfrob` constant (0x2A)

---

### Step-by-Step Solution

#### Step 1 — Initial Inspection

The first step was to determine what type of files we were working with:

```bash
file /home/thoran/CPCTF/level4_1/chall
```

Output: `ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked...`

This immediately identified `chall` as a compiled binary. The other file was readable text:

```bash
cat /home/thoran/CPCTF/level4_1/flag
```

Output: `IZI^LQGGLXHuYu^BuHO^uDIXSZ^CDW`

The flag file contained what looked like garbage characters—clearly obfuscated. The binary was the key to understanding how it was obfuscated.

---

#### Step 2 — Locating the Hidden Data (Function References)

To understand the binary's purpose, I extracted all readable strings:

```bash
strings /home/thoran/CPCTF/level4_1/chall
```

This output a long list of library functions and debugging information. Among the noise, three things stood out:

1. A reference to `memfrob`—a function that shouldn't appear in most programs
2. A comment split across multiple strings: `"I just found out about this function called memfrob."`
3. A reference to the `flag` file and error message: `"Flag file not found."`

This was a breadcrumb trail. The binary was explicitly telling us that `memfrob` was relevant. The fact that the source code comment appeared in the binary (due to the binary being unstripped) confirmed this was intentional on the challenge author's part.

---

#### Step 3 — Understanding memfrob

`memfrob` is a GNU C function documented in the man pages:

```bash
man memfrob
```

The key detail: `memfrob` XORs memory with the byte value 0x2A (42 in decimal). XOR is symmetric—if you XOR data with 0x2A twice, you get the original data back. This is actually quite weak protection, but it's sometimes used for trivial obfuscation.

---

#### Step 4 — Decryption

Since we now knew the obfuscation method, decryption was straightforward:

```bash
python3 << 'EOF'
with open('/home/thoran/CPCTF/level4_1/flag', 'r') as f:
    obfuscated = f.read().strip()

# memfrob XORs with 0x2A
deobfuscated = ''.join(chr(ord(c) ^ 0x2A) for c in obfuscated)
print("Obfuscated:", obfuscated)
print("Deobfuscated:", deobfuscated)
EOF
```

Output:
```
Obfuscated: IZI^LQGGLXHuYu^BuHO^uDIXSZ^CDW
Deobfuscated: cpctf{m3mfr0b_1s_th3_be5t_3ncrypti0n}
```

The flag appeared immediately, correctly formatted and readable.

---

### Why This Worked

**1. Explicit Hints in Strings**
The binary wasn't truly obfuscated—it contained explicit references to `memfrob`. This is typical of educational CTF challenges where the goal is to test your ability to recognize functions and understand their security implications, not to hide information in binary code.

**2. Man Pages Connection**
The problem statement mentioning "man pages" was a direct pointer. `memfrob` is documented there, and searching the man pages would quickly reveal what the function does.

**3. XOR is Reversible**
XOR has a mathematical property: A XOR B XOR B = A. This means applying `memfrob` twice returns the original. Even without code access, recognizing the weak encryption was the key insight.

**4. The Challenge is Educational**
The lesson isn't "here's an impossible binary"—it's "here's a function you might find in the wild, and here's why you should be skeptical of it as security."

---

### Lessons Learned

**1. String Extraction is Powerful**
Many binaries, especially in CTF challenges, contain hints in their readable strings. Before attempting complex reverse engineering, always run `strings` first. It often reveals the solution directly.

**2. Know Your Standard Library**
Recognition of standard library functions (whether from C, Python, or other languages) is crucial. `memfrob` isn't obscure—it's in the official man pages. Familiarity with common functions and their security properties is invaluable.

**3. XOR is Not Encryption**
XOR with a fixed key is not cryptographically secure. While it can obfuscate data, it's trivially reversible. This is a foundational principle in cryptography: a single, repeated key is not sufficient for security.

**4. Problem Statements Contain Clues**
"I was reading man pages and found something interesting" isn't filler—it's a direct hint. Always read problem statements carefully for this kind of guidance. CTF authors rarely waste words.

**5. Understand the Tooling**
The combination of `file`, `strings`, and basic Python XOR knowledge solved this challenge entirely. Having a mental toolkit of quick analysis commands (file type checking, string extraction, basic encoding/decoding) is essential for reversing problems.

---

**Final Flag:** `cpctf{m3mfr0b_1s_th3_be5t_3ncrypti0n}`
