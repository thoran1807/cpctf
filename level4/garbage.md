# Garbage, Garbage, Garbage... — CTF Challenge Writeup

### Final Answer

**Flag:** `cpctf{4nd_l37_7h3r3_b3_R0T}`

---

### Problem Understanding

The challenge gives a long sentence full of noise text and one suspicious structured token near the end: `us_jaspcpgs{4aq_y37_7u3e3_o3_E0G}`. On the surface, it looks like random garbage, but in CTFs that kind of pattern usually means a lightly obfuscated flag. The real task is to identify the encoding method and recover the string in the expected `cpctf{...}` format.

---

### Thought Process

The repeated word "Garbage" suggested misdirection, so I ignored the readable decoy text and focused on the only machine-like artifact: the brace-delimited token. The prefix did not match a normal flag format, but the shape looked close enough to be transformed text. I tested Caesar shifts first because this is a common beginner crypto trick and fast to validate.

When shift 13 (ROT13) was applied, I got `hf_wnfcpctf{4nd_l37_7h3r3_b3_R0T}`. That immediately exposed `cpctf{...}` inside the decoded output, with a small junk prefix (`hf_wnf`) before it. At that point, extraction was straightforward: keep only the valid flag pattern and discard surrounding garbage.

---

### Tools and Techniques Used

- **Linux / Ubuntu (WSL):** `printf`, `grep`, `tr` to isolate the suspicious token and test ROT13 quickly from the terminal
- **Python 3:** optional brute-force Caesar check to confirm that shift 13 is the only meaningful decoding
- **CyberChef / online tools:** not used (terminal workflow was enough)

---

### Step-by-Step Solution

#### Step 1 — Initial Inspection

I first copied the challenge text into a local file so I could inspect it reproducibly from the CLI.

```bash
file challenge.txt
cat challenge.txt
```

What stood out was the token-like segment with braces:

```text
us_jaspcpgs{4aq_y37_7u3e3_o3_E0G}
```

That structure is typical of an encoded flag candidate, so I treated it as the primary target.

---

#### Step 2 — Locating the Hidden Data

I extracted the most suspicious substring directly with a regex.

```bash
grep -oE '[A-Za-z0-9_]+\{[^}]+\}' challenge.txt
```

Output:

```text
us_jaspcpgs{4aq_y37_7u3e3_o3_E0G}
```

This worked because the payload had clear delimiters (`{}`), which are uncommon in random prose but common in CTF flag formats.

---

#### Step 3 — Extraction and Decoding

Next, I applied ROT13 using `tr`.

```bash
echo 'us_jaspcpgs{4aq_y37_7u3e3_o3_E0G}' | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

Output:

```text
hf_wnfcpctf{4nd_l37_7h3r3_b3_R0T}
```

The decoded text contains the valid flag signature `cpctf{...}` but with junk before it. I extracted only the flag pattern:

```bash
echo 'hf_wnfcpctf{4nd_l37_7h3r3_b3_R0T}' | grep -oE 'cpctf\{[^}]+\}'
```

Output:

```text
cpctf{4nd_l37_7h3r3_b3_R0T}
```

No further decoding was required.

---

#### Step 4 — Flag

The recovered flag is:

```text
cpctf{4nd_l37_7h3r3_b3_R0T}
```

It matches the expected format and reads like leetspeak for "and let there be ROT", which is consistent with the challenge theme.

---

### Why This Worked

1. The challenge hid useful data in plain sight by embedding one structured token inside noisy text.
2. Brace-delimited strings are strong indicators of encoded flags, so regex extraction reduced noise immediately.
3. ROT13 is a frequent CTF obfuscation method and is very fast to test.
4. Decoded output revealed a recognizable flag prefix (`cpctf{`), which confirmed we used the right transform.
5. Final extraction with pattern matching removed deliberate garbage bytes/prefix safely.

---

### Lessons Learned

1. In noisy challenge text, isolate structured artifacts first; ignore prose until needed.
2. Try low-cost classic transforms early (ROT13, Caesar, Base64 checks) before complex tooling.
3. Pattern-based extraction (`grep -oE`) is often enough to recover the final answer cleanly.
4. A partially readable decode is still useful; recognizable prefixes like `ctf{` validate direction.
5. Treat CTF text challenges like forensics: reduce data, test hypotheses, keep only validated artifacts.

---

**Final Flag:** `cpctf{4nd_l37_7h3r3_b3_R0T}`
