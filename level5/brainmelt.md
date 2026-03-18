### Final Answer

**Flag:** `cpctf{br41n_fuck3d}`

---

### Problem Understanding

The challenge gives a long encoded-looking string in `brainmelt.txt` and asks for the flag. At first glance, it looks like a normal Base64 decoding task, but that is only the first layer. The hidden task is to recognize that the Base64 output is Brainfuck code and run it correctly to recover the actual flag text.

---

### Thought Process

I started by testing the simplest hypothesis first: if the string is Base64, decoding it should produce something meaningful. The decode did not give readable English, but it did produce a very specific symbol pattern made of `+ - < > [ ] .`, which is a strong fingerprint for Brainfuck.

Once I saw that pattern, I ruled out random binary data and common text encodings. Brainfuck programs are tiny but deliberate, so the right path was to execute it with a small interpreter and inspect the output directly. That output was clean and already in `cpctf{...}` format, so no extra decoding layers were needed.

---

### Tools and Techniques Used

- **Linux / Ubuntu (WSL):**
  - `file` to quickly classify the input artifact
  - `xxd` to inspect raw structure and confirm it is plain text data
  - `base64` to decode the first layer
- **Python 3:**
  - used to execute a minimal Brainfuck interpreter and print program output
- **CyberChef / online tools:**
  - not used

---

### Step-by-Step Solution

#### Step 1 - Initial Inspection

I first treated the challenge input as a file and performed quick triage.

```bash
cd /home/thoran/CPCTF/level_5
file brainmelt.txt
xxd brainmelt.txt | head -40
```

Why this step mattered:

- `file` confirms whether the input is text, binary, archive, or executable.
- `xxd` shows whether the content is structured bytes or plain ASCII characters.
- In this case, the content was printable text that looked like an encoded payload, so decoding was the logical next step.

---

#### Step 2 - Locating the Hidden Data

I then isolated the long encoded token from the file.

```bash
strings brainmelt.txt | grep -E '^[A-Za-z0-9+/=]{80,}$'
```

This worked because Base64 strings are made from a limited character set (`A-Z`, `a-z`, `0-9`, `+`, `/`, `=`). The extracted line matched that pattern and was long enough to be real payload data instead of noise.

---

#### Step 3 - Extraction and Decoding

First decode the token from Base64:

```bash
echo "LS1bLS0tLS0+KzxdPi0tLS4rKysrKysrKysrKysrLi0tLS0tLS0tLS0tLS0uLVstLS0+KzxdPi0tLisrK1stPisrKzxdPisuK1stLS0tLT4rPF0+Li1bLT4rKysrKzxdPi5bLS0tPis8XT4tLS0tLlstLT4rPF0+LS0tLS0uLS0tLi1bLS0tLS0+KzxdPi0tLi0tLS0tLS0tLS0tLS0tLS4rKysrKysrLi1bLS0tPis8XT4tLS4rWy0+KysrPF0+Ky4rKysrKysrKy4tWy0tPis8XT4tLS4tWy0+Kys8XT4uPi0tWy0tPisrKzxdPi4=" | base64 -d
```

Decoded output (trimmed) looked like this:

```text
--[----->+<]>---.+++++++++++++.-------------....
```

That symbol set is Brainfuck. I executed it using Python:

```bash
python3 - << 'PY'
import base64

s = "LS1bLS0tLS0+KzxdPi0tLS4rKysrKysrKysrKysrLi0tLS0tLS0tLS0tLS0uLVstLS0+KzxdPi0tLisrK1stPisrKzxdPisuK1stLS0tLT4rPF0+Li1bLT4rKysrKzxdPi5bLS0tPis8XT4tLS0tLlstLT4rPF0+LS0tLS0uLS0tLi1bLS0tLS0+KzxdPi0tLi0tLS0tLS0tLS0tLS0tLS4rKysrKysrLi1bLS0tPis8XT4tLS4rWy0+KysrPF0+Ky4rKysrKysrKy4tWy0tPis8XT4tLS4tWy0+Kys8XT4uPi0tWy0tPisrKzxdPi4="
code = base64.b64decode(s).decode()
code = ''.join(c for c in code if c in '><+-.,[]')

match = {}
stack = []
for i, c in enumerate(code):
    if c == '[':
        stack.append(i)
    elif c == ']':
        j = stack.pop()
        match[i] = j
        match[j] = i

tape = [0] * 30000
ptr = 0
pc = 0
out = []

while pc < len(code):
    c = code[pc]
    if c == '>':
        ptr += 1
    elif c == '<':
        ptr -= 1
    elif c == '+':
        tape[ptr] = (tape[ptr] + 1) % 256
    elif c == '-':
        tape[ptr] = (tape[ptr] - 1) % 256
    elif c == '.':
        out.append(chr(tape[ptr]))
    elif c == '[' and tape[ptr] == 0:
        pc = match[pc]
    elif c == ']' and tape[ptr] != 0:
        pc = match[pc]
    pc += 1

print(''.join(out))
PY
```

Program output:

```text
cpctf{br41n_fuck3d}
```

No further processing was required.

---

#### Step 4 - Flag

Final flag:

```text
cpctf{br41n_fuck3d}
```

It matches the required `cpctf{...}` format.

---

### Why This Worked

- The challenge used layered encoding, where Base64 was only the outer wrapper.
- Character analysis after Base64 decode revealed a clear Brainfuck signature.
- Brainfuck execution is deterministic, so a minimal interpreter is enough.
- The output directly produced a valid flag format with no ambiguity.

---

### Lessons Learned

- Treat strange text as potentially multi-layered, not single-layer encoded data.
- After each decode step, pause and identify the data type before guessing the next tool.
- Learn visual fingerprints of esoteric languages like Brainfuck; they save time.
- Prefer small reproducible scripts over manual decoding when logic is clear.
- Validate the final result against expected flag format before submitting.

---

**Final Flag:** `cpctf{br41n_fuck3d}`