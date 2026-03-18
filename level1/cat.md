### Final Answer

**Flag:** `cpctf{c4ts_4r3_cut3}`

---

### Problem Understanding

You're given a single JPEG file (`cat.jpg`) with no obvious instructions—just the filename itself. The challenge is a steganography problem: the flag isn't displayed visibly in the image, but hidden somewhere within the file. In CTF steganography challenges, data often hides in file metadata, trailing bytes, or encoded within pixel values. The task is to locate and extract it.

---

### Thought Process

When faced with a mystery file, the standard approach is to treat it as a black box and examine its structure systematically. JPEGs are binary files with various internal sections, and flags in CTF challenges frequently appear in less-obvious places—like the tail end of the file after the image data proper. My first instinct was to check both the file metadata (EXIF headers) and the raw trailing bytes, since many beginners hide plaintext flags there. A quick hexdump of the tail end confirmed the suspicion: readable ASCII text appeared near the end of the file, and within that text was the flag string itself. No decoding or complex extraction required—just systematic inspection.

---

### Tools and Techniques Used

- **Windows PowerShell:** `Get-Item`, `Get-Content` with `-Encoding Byte` for viewing file metadata and raw bytes
- **Format-Hex:** to display the file header in hexadecimal format
- **ASCII decoding:** to convert the trailing bytes back to readable text

---

### Step-by-Step Solution

#### Step 1 — Initial Inspection

First, I listed the directory to confirm the file existed and then gathered basic file properties.

```powershell
Get-Item .\cat.jpg | Select-Object Name,Length,CreationTime,LastWriteTime | Format-List
```

**Output:**
```
Name          : cat.jpg
Length        : 265827 bytes
CreationTime  : 17-03-2026 00:45:56
LastWriteTime : 17-03-2026 00:40:26
```

The file is reasonably large (~266 KB), which suggests it's a full image rather than a tiny placeholder. The size is typical for a standard JPEG photo.

---

#### Step 2 — Examining File Structure

Next, I checked the file header to confirm it was a valid JPEG and to identify any obvious metadata.

```powershell
Get-Content .\cat.jpg -Encoding Byte -TotalCount 32 | Format-Hex
```

**Output:**
```
00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F
FF D8 FF E1 00 6C 45 78 69 66 00 00 49 49 2A 00
```

The header `FF D8 FF E1` confirms this is a JPEG file with EXIF metadata. The `45 78 69 66` bytes spell out "Exif" in ASCII—standard JPEG structure. This told me the file was legitimate and properly formatted.

---

#### Step 3 — Searching the File Tail

The key insight was checking the very end of the file, where hidden data often resides in steganography challenges. I extracted and decoded the last 256 bytes as ASCII text.

```powershell
[System.Text.Encoding]::ASCII.GetString((Get-Content .\cat.jpg -Encoding Byte -ReadCount 0)[-256..-1])
```

**Output (last line):**
```
cpctf{c4ts_4r3_cut3}
```

There it was—the flag sitting plainly in the ASCII-decodable tail of the file, mixed with binary garbage and non-printable characters around it.

---

#### Step 4 — Flag Validation

The extracted flag matches the required format: `cpctf{...}` with recognizable leetspeak substitutions (`4` for 'a', `3` for 'e'). The message itself is a playful reference to the filename—"cats are cute"—which confirms this is the intended solution.

```
cpctf{c4ts_4r3_cut3}
```

---

### Why This Worked

1. **File structure awareness:** Understanding that JPEGs contain multiple sections (header, EXIF, image data, trailing bytes) allowed me to target the tail end rather than randomly searching.

2. **Tail-byte analysis is a common first technique:** In steganography, especially for beginner challenges, appending plaintext after the image data is the simplest hiding method because many image viewers only read up to the end of the official image structure.

3. **Readable ASCII in binary:** The flag was encoded as plaintext, not encrypted or encoded in a complex format. Filtering for ASCII text is therefore an effective first-pass technique.

4. **No false complexity:** The challenge didn't require specialized steganography tools (like `binwalk` or `steghide`). Sometimes the simplest approach—just look at the raw bytes—is correct.

---

### Lessons Learned

1. **Always examine file structure first:** Use `xxd`, `hexdump`, or equivalent tools to see how data is organized. This reveals hiding spots naturally.

2. **Check multiple vantage points:** Don't stop at the header. Examine the beginning, middle, and especially the end of files. Attackers often append data.

3. **ASCII patterns are telltale:** Readable text in binary files stands out instantly in a hex dump. Train your eye to spot it.

4. **Steganography ≠ encryption:** A hidden flag isn't necessarily encrypted. Many CTF challenges rely on obscurity, not cryptography. Start simple.

5. **Filename hints matter:** In this case, `cat.jpg` foreshadowed the flag's content. Always read the challenge description and filenames carefully—they're often intentional clues.

---

**Final Flag:** `cpctf{c4ts_4r3_cut3}`
