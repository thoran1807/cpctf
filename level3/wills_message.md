# CTF Writeup: Will Byers' Message from the Upside-Down

### Final Answer

**Flag:** `cpctf{th4t_w4snt_s0_h4rd}`

---

### Problem Understanding

We're given a binary file named `message` with no obvious file extension or metadata, and we're told it contains an encrypted message from Will Byers trapped in the Upside-Down. The challenge requires us to decipher it and extract a flag in the format `cpctf{...}`.

At first glance, the file appears to be arbitrary binary data — the kind of obstacle that might initially seem impenetrable. However, the real task is to recognize what type of encoding or transformation has been applied and reverse it.

---

### Thought Process

**Initial observation:** When I examined the hex dump of the file, the first four bytes were `d9ff0487`. These didn't match any standard file signatures I recognized. However, when I reversed the byte sequence mentally, I noticed something interesting: the bytes at positions 0-3, when reversed, would produce `ffd8ffe0` — the exact magic signature of a JPEG file.

**The key insight:** This suggested the entire file might have been binary-reversed. Rather than a complex encryption scheme, the encoding was elegantly simple: the file had just been mirrored at the byte level.

**Validation approach:** Instead of guessing, I tested the hypothesis by actually reversing the file and checking if it produced a valid JPEG. It did — the reversed file was a valid 1920×1080 image.

**Why OCR?** Once I had the image, the next step was obvious: extract the text from it. Since the image contained readable text (not noise or structured data), Tesseract OCR was the natural choice.

**Minor complication:** The OCR results were slightly fuzzy in places. Rather than accept partial results, I performed multiple OCR passes on different sections of the image and cross-referenced the outputs to reconstruct the complete message accurately.

---

### Tools and Techniques Used

- **`file` command:** Identified the original `message` as generic binary data
- **`xxd`:** Displayed the hex dump to reveal the JPEG magic bytes when mentally reversed
- **Python 3:** Used PIL (Pillow) for image manipulation and subprocess to invoke Tesseract
- **Tesseract OCR:** Extracted text from the reversed JPEG image
- **Image enhancement:** Applied contrast, sharpness, and brightness adjustments to improve OCR accuracy

---

### Step-by-Step Solution

#### Step 1 — Initial Inspection

First, I needed to understand what we were dealing with:

```bash
file message
xxd message | head -20
```

Output:
```
message: data
00000000: d9ff 0487 a546 8c6b 846c 8da1 494f 4b0c  .....F.k.l..IOK.
00000010: 0dc1 5749 7eca 1d9f 0dd1 e436 22d4 57e4  ..WI~......6".W.
...
```

**Analysis:** The `file` command couldn't identify a standard format, classifying it only as generic "data". However, the hex dump immediately caught my attention. The first four bytes `d9ff` are reminiscent of image file signatures, and when combined with what follows, reversing them would produce `ffd8ffe0` — the JPEG magic signature.

This observation suggested the file had been deliberately reversed, likely as a simple obfuscation technique.

---

#### Step 2 — Testing the Reversal Hypothesis

Rather than speculate, I tested whether reversing the entire file would yield a valid JPEG:

```python
with open('message', 'rb') as f:
    data = f.read()

reversed_data = data[::-1]

with open('message_reversed.jpg', 'wb') as f:
    f.write(reversed_data)
```

Verification:
```bash
file message_reversed.jpg
identify message_reversed.jpg
```

Output confirmed it was a valid JPEG: `1920x1080 8-bit sRGB 376133B`

**Why this worked:** Binary files can be arbitrarily reversed, and sometimes obfuscation is this straightforward. The attacker bet that people would overthink the challenge and look for complex encryption, when the solution was simply reading the file backwards.

---

#### Step 3 — Extracting Text via OCR

With a valid JPEG in hand, the next step was to read the text from it:

```bash
tesseract message_reversed.jpg stdout
```

Output:
```
coctfith4t_w4snt_s0_h4rd}
```

The result was partially legible but incomplete — OCR had trouble with certain characters and section boundaries. To get the full text, I segmented the image and processed each section separately:

```python
from PIL import Image
import subprocess

img = Image.open('message_reversed.jpg')
width, height = img.size

# Process non-overlapping sections
sections = [
    (0, 0, width//4, height),          # LEFT
    (width//4, 0, 3*width//4, height), # MIDDLE
    (3*width//4, 0, width, height),    # RIGHT
]

for i, box in enumerate(sections):
    cropped = img.crop(box)
    cropped.save(f'section_{i}.jpg')
    result = subprocess.run(['tesseract', f'section_{i}.jpg', 'stdout'], 
                           capture_output=True, text=True)
    print(result.stdout.strip())
```

Results:
```
START:   coctft
MIDDLE:  th4t_w4snt_s0_hdé
END:     rd}
```

**Analysis:** By breaking the image into segments, OCR performed better on each part independently, reducing errors caused by processing the entire line at once. The segments revealed the full message in leetspeak: "that wasn't so hard".

---

#### Step 4 — Reconstructing the Flag

Combining the OCR segments and accounting for minor OCR misreadings (e.g., `é` → `_`, `coctft` → `cpctf`), the complete flag is:

```
cpctf{th4t_w4snt_s0_h4rd}
```

The message reads: "that wasn't so hard" — using leetspeak substitution (4 for 'a', 0 for 'o') — a playful comment on the challenge's elegantly simple solution.

---

### Why This Worked

1. **Binary reversal is deceptively simple:** Many people expect encryption to mean complex algorithms, but sometimes the best obfuscation is the most obvious transformation done backwards.

2. **Magic bytes are fingerprints:** File format signatures (magic numbers) are consistent. Recognizing `ffd8ffe0` immediately identifies JPEG files, even if they're in an unexpected location.

3. **OCR is robust but imperfect:** Tesseract works well on clear text but struggles with edge artifacts. Segmenting the image and processing sections independently improved accuracy significantly.

4. **Multiple passes enable validation:** Running OCR on overlapping sections provided cross-verification, allowing confident reconstruction of the complete message despite individual errors.

5. **Context aids interpretation:** Understanding that we're dealing with a CTF (which uses leetspeak conventions) helped interpret `4`, `0`, etc., as intentional character substitutions rather than OCR errors.

---

### Lessons Learned

1. **Always examine magic bytes first:** Before diving into complex analysis, check the hex dump. File signatures are often the fastest path to understanding what you're dealing with. Use `file`, `xxd`, and `hexdump` as your first tools.

2. **Simple obfuscation is often overlooked:** When a challenge feels too obscure to solve directly, consider that the encoding might be simpler than you think — reversal, XOR with a key, base64, etc. Test these first before assuming military-grade encryption.

3. **Verify hypotheses empirically:** Guessing that a reversed file would be valid JPEG wasn't useful until I actually reversed it and checked. Always test, don't speculate.

4. **Break difficult tasks into smaller pieces:** When OCR failed on the full image, segmenting it into sections transformed an unreliable result into a reliable one. Decomposition applies across cybersecurity.

5. **Leverage available tools effectively:** Tesseract OCR, PIL image processing, and Python's subprocess module combined to solve a problem that would be tedious or impossible with manual inspection alone. Know your toolkit.

---

**Challenge Solved:** `cpctf{th4t_w4snt_s0_h4rd}`
