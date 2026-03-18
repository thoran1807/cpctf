### Final Answer

**Flag:** `cpctf{Isnt_it_magic}`

---

### Problem Understanding

The challenge gives one file named `magic` and a story-like hint: Kishor went to the 58th shop on magical DLF street and forgot the way back. At first, this looks like a normal binary challenge, but the real task is to notice deliberate file tampering and then decode the program output correctly. The hint is not decorative: "DLF" points to a broken ELF header, and "58th shop" points to Base58.

---

### Thought Process

I started with file triage instead of execution because unknown binaries often fail for reasons that become obvious in metadata or headers. The first useful clue appeared immediately in hex: the binary began with `7f 44 4c 46` (".DLF") instead of `7f 45 4c 46` (".ELF"). That strongly suggested a one-byte anti-execution trick.

After that, I ruled out deeper reversing first (no need for disassembly yet) and tested the simplest hypothesis: patch one byte and run the binary. The repaired file executed and printed a token that looked like Base58 text. The challenge text already mentioned "58th shop," so decoding in Base58 was the most evidence-based next step. That decode produced a valid CTF flag directly.

---

### Tools and Techniques Used

- **Linux / Ubuntu (WSL):**
  - `file` to classify the artifact
  - `xxd` to inspect the magic bytes in the header
  - `strings` to quickly inspect printable content
  - `dd` to patch one byte without changing file size/layout
  - `chmod` + direct execution to run the repaired binary
- **Python 3:**
  - Used to decode the output token from Base58 into plaintext
- **CyberChef / online tools:**
  - Not used

---

### Step-by-Step Solution

#### Step 1 - Initial Inspection

First, I inspected the file type and raw header.

```bash
cd /home/thoran/CPCTF/level_3
file magic
xxd -l 128 magic
```

What this revealed:

- `file magic` returned generic `data`, not an executable type.
- `xxd` showed the header starts with `.DLF` (`7f 44 4c 46`).
- A valid ELF binary should start with `.ELF` (`7f 45 4c 46`).

This mismatch explained why execution failed with `Exec format error` and indicated intentional header corruption.

---

#### Step 2 - Locating the Hidden Data

I patched only the incorrect byte and executed the repaired file.

```bash
cp magic magic_fixed
printf '\x45' | dd of=magic_fixed bs=1 seek=1 count=1 conv=notrunc
chmod +x magic_fixed
./magic_fixed
```

The program output was:

```text
2PMKviXitTqsQrgvzWRVQmUqQewN
```

Why this mattered:

- The output looked like Base58 (character set excludes confusing symbols like `0`, `O`, `I`, `l`).
- The prompt clue "58th shop" reinforced Base58 as the intended decoding layer.

---

#### Step 3 - Extraction and Decoding

I decoded the value using Python with the standard Base58 alphabet.

```bash
python3 - << 'PY'
s='2PMKviXitTqsQrgvzWRVQmUqQewN'
alphabet='123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz'
num=0
for ch in s:
    num = num*58 + alphabet.index(ch)

out=bytearray()
while num>0:
    out.append(num & 0xff)
    num >>= 8
out=bytes(reversed(out))

pad=0
for c in s:
    if c=='1':
        pad += 1
    else:
        break
out = b'\x00'*pad + out

print(out.decode())
PY
```

Decoded output:

```text
cpctf{Isnt_it_magic}
```

No additional decoding or cleanup was needed.

---

#### Step 4 - Flag

The final flag is:

```text
cpctf{Isnt_it_magic}
```

It matches the expected `cpctf{...}` format.

---

### Why This Worked

- The challenge used a minimal but effective binary tamper: one wrong ELF header byte.
- Header inspection is fast and immediately exposed that tamper.
- Repairing the header restored intended execution behavior.
- The runtime output was encoded, and the prompt gave the decoding hint (`58` -> Base58).
- Combining static clues (header) and narrative clues (58th shop) led to a deterministic solve path.

---

### Lessons Learned

- Always do quick triage (`file`, `xxd`, `strings`) before deep reverse engineering.
- If a binary throws format errors, validate magic bytes early.
- In CTFs, story text often contains exact technical hints, not filler.
- Prefer smallest-possible patching when validating a hypothesis.
- Use clues to prioritize likely encodings (Base58 here) instead of brute-forcing everything.

---

**Final Flag:** `cpctf{Isnt_it_magic}`
