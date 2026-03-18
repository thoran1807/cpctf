### Final Answer

**Flag:** `cpctf{ev1l_fl0at1ng_p01nt_bi7_l3v3l_h4ck1n6_w1thou7_pr0fan1ty}`

---

### Problem Understanding

The challenge provides a file named `flag` and a hint about an "evil floating point bit level hacker" modifying the real flag. On the surface, it looks like a plain text file containing many large numbers, which could be mistaken for random data or big integers from crypto. The real task is to recognize that these numbers are not meant to be read directly, but interpreted as bit patterns of floating-point values and converted back to text.

---

### Thought Process

I started with quick file triage instead of guessing an encoding. The first clue was that `file` reported ordinary ASCII text, but the content was a single long line of huge decimal values separated by spaces. That pattern did not look like base64, hex, or compressed data.

The challenge title strongly suggested IEEE-754 manipulation, so I tested the idea that each decimal token might be a 64-bit integer representing the raw bits of a `double`. If this hypothesis was correct, decoding each value as a float should produce values close to printable ASCII code points.

I ruled out heavy approaches like brute-forcing transformations, OCR, steganography, or binary carving because the data already had a clean and structured numeric pattern. Once the float-bit interpretation produced readable characters immediately, the solution path was confirmed.

---

### Tools and Techniques Used

- **Linux / Ubuntu (WSL):**
  - `ls -l` to inspect file size and basic context
  - `file` to identify the top-level file type
  - `xxd` to validate raw byte structure
  - `strings` to confirm token pattern in the payload
- **Python 3:**
  - parse space-separated integers
  - reinterpret each integer as IEEE-754 `double` bits using `struct.pack` / `struct.unpack`
  - convert decoded numeric values to characters
- **CyberChef / online tools:**
  - not used

---

### Step-by-Step Solution

#### Step 1 — Initial Inspection

I first checked what kind of file I was dealing with and whether there were obvious signatures:

```bash
cd /home/thoran/CPCTF/level_11
file flag
xxd -g 1 -l 256 flag
```

`file` reported ASCII text with one very long line and no line terminator. The hex view showed digits and spaces only, which confirmed the content was intentionally numeric text rather than binary file data.

---

#### Step 2 — Locating the Hidden Data

Next, I inspected the printable content directly:

```bash
strings -n 4 flag | head -n 5
```

This surfaced one long sequence of large decimal integers like:

```text
4636666922610458624 4637581716284768256 4636666922610458624 ...
```

This worked because the whole payload is already printable text; the hidden part is semantic, not visibility-based. The meaningful data is in how each number should be interpreted.

---

#### Step 3 — Extraction and Decoding

I decoded each integer as a 64-bit IEEE-754 double bit pattern, then mapped each float value to an ASCII character:

```bash
python3 - << 'PY'
import struct

with open('/home/thoran/CPCTF/level_11/flag', 'r') as f:
    nums = [int(x) for x in f.read().strip().split()]

# Last value is a 0 terminator in this challenge file
nums = nums[:-1]

decoded = []
for n in nums:
    v = struct.unpack('<d', struct.pack('<Q', n))[0]
    decoded.append(chr(round(v)))

print(''.join(decoded))
PY
```

Output:

```text
cpctf{ev1l_fl0at1ng_p01nt_bi7_l3v3l_h4ck1n6_w1thou7_pr0fan1ty}
```

No additional decoding was required.

---

#### Step 4 — Flag

The recovered flag is:

```text
cpctf{ev1l_fl0at1ng_p01nt_bi7_l3v3l_h4ck1n6_w1thou7_pr0fan1ty}
```

---

### Why This Worked

- The challenge data was structured as decimal-encoded 64-bit words, not random text.
- The problem hint explicitly pointed to floating-point bit-level manipulation.
- IEEE-754 reinterpretation converts raw bit patterns into numeric values without changing bits.
- Those decoded values matched ASCII code points directly, so plaintext recovery was deterministic.
- A trailing `0` acted as a terminator and could be safely ignored.

---

### Lessons Learned

- Start with lightweight triage (`file`, `xxd`, `strings`) before attempting complex decoding.
- Challenge titles often reveal the exact data representation trick.
- When you see repeated large integers, test reinterpretation attacks (bit-casts) before brute force.
- In CTF misc tasks, "hidden" frequently means "wrong interpretation," not encryption.
- A short Python script with `struct` is often enough to solve bit-level format puzzles cleanly.

---

**Final Flag:** `cpctf{ev1l_fl0at1ng_p01nt_bi7_l3v3l_h4ck1n6_w1thou7_pr0fan1ty}`
