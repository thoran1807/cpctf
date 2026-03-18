### Final Answer

**Flag:** `cpctf{y0tsub4_h1d_th3_fl4g}`

---

### Problem Understanding

The challenge provided a single file, `yots.jpg`, and asked for a flag in the format `cpctf{...}`. On the surface it looked like a normal steganography task, but the key phrase "she hid something in this image and then broke it" hinted that the file might be intentionally corrupted before or after embedding data. So the real task was not only extraction, but also file repair and layered analysis.

---

### Thought Process

My first goal was to verify whether `yots.jpg` was actually a valid JPEG. The `file` output showed only generic `data`, which is unusual for a normal image and strongly suggested header corruption. When I inspected the first bytes with `xxd`, two details stood out: the JPEG SOI marker was `00 d8` instead of `ff d8`, and `JFIF` had been replaced with `YOTS`.

In the same header region, I noticed a JPEG comment containing a clear clue: `password is y0tsubato`. That was an important sign that steghide was likely involved. I repaired the corrupted header bytes, re-validated the image, and then used the password with `steghide`, which successfully extracted `flag.png`.

At that point, metadata and plain strings did not reveal the flag directly. So I moved to pixel-level analysis and checked least significant bits (LSB). A short Python script reading RGB LSB bits in row-major order produced readable ASCII and recovered the final `cpctf{...}` flag. I ruled out "flag in metadata" and "flag in appended archive" because those checks came back clean.

---

### Tools and Techniques Used

- **Linux / Ubuntu (WSL):** `ls`, `file`, `sha256sum`, `wc`, `xxd`, `strings`, `dd`
- **Stego/forensics tools:** `steghide`, `binwalk`, `exiftool`
- **Python 3:** custom script with Pillow to extract RGB LSB bitstream and decode ASCII
- **No online tools used**

---

### Step-by-Step Solution

#### Step 1 — Initial Inspection

I started by confirming the file state and checking whether the image was valid.

```bash
file yots.jpg
xxd -l 128 yots.jpg
```

What this revealed:
- `file` reported `data`, not `JPEG image data`, so the file signature was not valid.
- The first byte was wrong: `00 d8` instead of JPEG SOI `ff d8`.
- The JFIF identifier looked tampered: `YOTS` instead of `JFIF`.
- A JPEG comment clearly contained: `password is y0tsubato`.

This immediately suggested an intentionally broken JPEG with embedded steghide content.

---

#### Step 2 — Locating the Hidden Data

The hidden-data lead came from header-level inspection rather than blind extraction.

```bash
xxd -l 128 yots.jpg
strings -n 6 yots.jpg | head
```

Why this worked:
- Stego authors often leave clues in JPEG COM segments.
- `xxd` exposed binary structure directly, making corruption and comment markers visible.
- `strings` confirmed the same password clue in printable form.

With both corruption and password identified, the next logical action was repair + extraction.

---

#### Step 3 — Extraction and Decoding

First, I repaired the JPEG header bytes, then extracted the hidden file.

```bash
cp yots.jpg yots_fixed.jpg
printf '\xFF' | dd of=yots_fixed.jpg bs=1 seek=0 count=1 conv=notrunc status=none
printf 'JFIF' | dd of=yots_fixed.jpg bs=1 seek=6 count=4 conv=notrunc status=none

file yots_fixed.jpg
steghide info yots_fixed.jpg -p y0tsubato
steghide extract -sf yots_fixed.jpg -p y0tsubato -f
```

`steghide` reported an embedded file named `flag.png`, which was extracted successfully.

Then I decoded the hidden message inside `flag.png` using RGB LSBs:

```bash
python3 - <<'PY'
from PIL import Image
img = Image.open('flag.png').convert('RGB')
bits = []
for r, g, b in img.getdata():
    bits.extend([r & 1, g & 1, b & 1])
out = bytearray()
for i in range(0, len(bits) - 7, 8):
    v = 0
    for bit in bits[i:i+8]:
        v = (v << 1) | bit
    out.append(v)
print(bytes(out).split(b'\x00', 1)[0].decode('ascii'))
PY
```

This printed the flag directly.

---

#### Step 4 — Flag

The recovered flag is:

```text
cpctf{y0tsub4_h1d_th3_fl4g}
```

It matches the required format `cpctf{...}`.

---

### Why This Worked

- The challenge used **layered obfuscation**: first break the JPEG header, then hide data with steghide.
- Header inspection exposed both the **tampering method** and the **password clue** in one place.
- After extraction, the PNG still carried a second hidden layer in **LSB pixel bits**.
- LSB extraction in straightforward RGB row-major order was enough; no complex traversal was needed.
- Verifying each hypothesis (metadata, archive carving, pixel stego) prevented guesswork and kept the workflow reliable.

---

### Lessons Learned

- If `file` says `data` for an image, inspect signatures before trying random stego tools.
- Read the first bytes (`xxd`) early; challenge authors often place intentional clues in headers/comments.
- In steg CTFs, expect multi-stage hiding: container extraction first, pixel analysis second.
- Automating bit extraction with a short Python script is often faster and more reliable than manual trial.
- Keep decisions evidence-driven: validate each step before moving to the next layer.

---

**Final Flag:** `cpctf{y0tsub4_h1d_th3_fl4g}`
