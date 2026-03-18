### Final Answer

**Flag:** `cpctf{17_w0n7_G37_34513R}`

---

### Problem Understanding

The challenge gives a single executable file, `a.out`, with a hint that says reversing is involved ("reversing things is the stuff of gods" written backwards). On the surface, this looks like a normal Linux binary, but the actual task is to inspect what data is embedded inside it and recover the flag. The key question is not "how do I run it," but "what does the binary reveal through static analysis?"

---

### Thought Process

My first goal was to classify the file before doing anything risky. If the file had been packed, encrypted, or stripped heavily, I would have needed deeper reverse engineering tools.

After checking the type, I noticed the binary was **not stripped**, which usually means easier static inspection. That immediately suggested trying `strings` as a fast win, because CTF binaries often store human-readable messages or even flags directly in read-only sections.

`strings` quickly surfaced a full flag-looking token in `cpctf{...}` format. At that point, instead of stopping immediately, I verified it from the `.rodata` section using `objdump` to confirm the string really exists in the binary data and was not a misleading artifact.

I ruled out dynamic debugging because there was no sign of obfuscation, encoding layers, or runtime-only construction of the flag.

---

### Tools and Techniques Used

- **Linux / Ubuntu (WSL):** `ls`, `file`, `strings`, `objdump`
- **Python 3:** Not needed for this challenge
- **CyberChef / online tools:** Not used

---

### Step-by-Step Solution

#### Step 1 — Initial Inspection

I started by checking what files were present and identifying the exact file type.

```bash
ls -la
file a.out a.out:Zone.Identifier
```

Key findings:

- `a.out` is an **ELF 64-bit PIE executable** for x86-64.
- It is **not stripped**, so symbol and string recovery is easier.
- `a.out:Zone.Identifier` is plain text metadata (from Windows download provenance), not the challenge payload.

This told me the main target is the ELF binary itself, and static inspection is a strong first path.

---

#### Step 2 — Locating the Hidden Data

Next, I extracted printable strings from the binary.

```bash
strings -n 4 a.out | head -n 200
```

This returned a very important line:

```text
This was very easy aye: cpctf{17_w0n7_G37_34513R}
```

Why this worked: `strings` scans binary bytes for printable ASCII sequences. If developers hardcode messages in C (for example in `printf`), those values often land in `.rodata` and can be recovered directly.

---

#### Step 3 — Extraction and Decoding

No additional encoding (like Base64) was used here, so decoding was not necessary. I still validated the discovered flag from the binary's read-only data section.

```bash
objdump -s -j .rodata a.out | head -n 40
```

Relevant output showed the flag bytes directly in `.rodata`, including:

```text
2020 63706374 667b3137 5f77306e 375f4733
2030 375f3334 35313352 7d00
```

These hex bytes map to:

```text
cpctf{17_w0n7_G37_34513R}
```

So the `strings` result was confirmed as authentic.

---

#### Step 4 — Flag

Final recovered flag:

```text
cpctf{17_w0n7_G37_34513R}
```

It matches the expected `cpctf{...}` challenge format.

---

### Why This Worked

- The binary was not stripped or obfuscated, so static indicators remained visible.
- Hardcoded strings were stored in `.rodata`, which `strings` can read easily.
- The challenge hint pointed toward reversing/static inspection, not exploitation.
- Verification with `objdump` removed ambiguity and confirmed exact bytes.
- No runtime decryption path existed, so dynamic analysis was unnecessary.

---

### Lessons Learned

- Always start with lightweight triage (`file`, `strings`) before opening heavy reverse tools.
- If a flag appears in output, verify it from another source (like `.rodata`) to avoid false positives.
- Non-stripped binaries are often intentionally beginner-friendly in entry-level CTF reversing tasks.
- Metadata side files (like `Zone.Identifier`) are useful context but usually not the core payload.
- Keep your workflow evidence-based: every command should answer a specific question.

---

**Final Flag:** `cpctf{17_w0n7_G37_34513R}`
