### Final Answer

**Flag:** `cpctf{th15_15_50_5ad}`

---

### Problem Understanding

The challenge gave a single file, `roads.scn`, with the hint: "Who knows where the road leads to?" and a required output format of `cpctf{...}`. At first glance, it looks like a normal OpenTTD scenario file, but the actual task is not to play the map in-game. The real objective is to inspect the file format, extract hidden structured data, and decode how the road layout itself encodes text.

---

### Thought Process

My first check was whether the flag was directly visible in plain strings. It was not. The string output looked mostly high-entropy, which usually means either compression or embedded binary structures.

The header bytes immediately showed `OTTX` and an `xz` signature, so the next logical step was to decompress the payload and inspect the internal chunk structure. After decompression, the content clearly matched OpenTTD save/scenario chunk data (`GLOG`, `MAPT`, `MAP5`, `CITY`, `SIGN`, etc.).

I then tested the obvious places (`SIGN`, `GOAL`, `CITY`) expecting prewritten clue text, but these records were empty. That ruled out metadata-based hiding and pointed to map-layer encoding.

From there, I focused on tile arrays (`MAPT`, `MAP5`, `MAP7`, `MAP8`) and isolated the road tile class. Once the coordinates were interpreted in the correct orientation, the road pattern produced readable leetspeak text. OCR gave multiple noisy variants, so I used high-resolution rendering and final human verification to remove ambiguity in the tail. That gave a stable, valid flag.

---

### Tools and Techniques Used

- **Linux / Ubuntu (WSL):**
  - `file` for quick type identification
  - `xxd` for magic-byte/header inspection
  - `strings` for fast plaintext clue hunting
  - `binwalk` and `7z` for embedded stream/container confirmation
  - `dd` + `xz -dc` for payload extraction/decompression
- **Python 3:**
  - Parsed OpenTTD chunks and table structures
  - Enumerated chunk tags/sizes and analyzed map layers
  - Rendered tile masks and transformed coordinate views
- **Image/OCR utilities:**
  - `convert` (ImageMagick) for thresholding/upscaling/cropping
  - `tesseract` for OCR-assisted reading of encoded map text

---

### Step-by-Step Solution

#### Step 1 — Initial Inspection

I started with standard triage to understand what `roads.scn` actually is.

```bash
file roads.scn
xxd -g 1 -l 64 roads.scn
strings -n 6 roads.scn | head -n 40
```

The first bytes were `4f 54 54 58` (`OTTX`) and shortly after `fd 37 7a 58 5a` (XZ signature), which strongly indicated a wrapped compressed payload. This explained why direct `strings` output looked random and why plain grep for `cpctf{` failed.

---

#### Step 2 — Locating the Hidden Data

Next, I validated the embedded stream and its offset.

```bash
binwalk roads.scn
7z l roads.scn
```

Both tools agreed there was an XZ stream starting at byte offset `8`, containing a decompressed object named `roads`. This gave a precise extraction path: skip the first 8 bytes, then decompress.

Why this worked: OpenTTD save/scenario containers often use wrappers (`OTTX`, `OTTZ`, etc.) around compressed chunk data. Signature-based tools are ideal here because they identify compression boundaries without guessing.

---

#### Step 3 — Extraction and Decoding

I extracted and decompressed the payload, then analyzed internal chunk data.

```bash
dd if=roads.scn bs=1 skip=8 status=none | xz -dc > roads.dec
```

After this, I parsed chunks and found:
- `SIGN`, `GOAL`, `CITY` records were empty (no obvious text clue)
- large raw map chunks (`MAPT`, `MAP5`, etc.) were populated

So the clue had to be geometric. I used Python to isolate road tiles (`MAPT == 32`), project coordinates in the correct orientation, and render a binary text image from the road layout. OCR on that rendered image produced noisy but consistent leetspeak patterns around:

- `cpctf{th15_15_50_...}`

The final suffix was ambiguous in OCR-only output, so I generated zoomed/cropped images and confirmed manually.

---

#### Step 4 — Flag

The final verified flag is:

```
cpctf{th15_15_50_5ad}
```

This matches the required `cpctf{...}` format.

---

### Why This Worked

1. The key clue was format-level, not string-level: `OTTX` + XZ indicated compressed structured data.
2. Chunk parsing narrowed effort quickly: metadata chunks were empty, so attention shifted to map layers.
3. The challenge hint (“where the road leads”) was literal: roads were the encoding medium.
4. Correct coordinate interpretation was critical; one orientation looked like noise, another revealed text-like structure.
5. OCR helped but was not blindly trusted; final validation used high-resolution rendering and manual confirmation.

---

### Lessons Learned

1. Always inspect file headers first; magic bytes often save hours of blind guessing.
2. If direct `strings` fails, assume compression/encoding before assuming encryption.
3. In game/map challenges, visual structures (tiles, paths, coordinates) can carry payloads even when metadata is empty.
4. OCR is useful for acceleration, but noisy outputs must be cross-validated.
5. Keep extraction reproducible: offset, decompression command, and parsing logic should be explicit and scriptable.

---

**Final Flag:** `cpctf{th15_15_50_5ad}`
