────────────────────────────────────────
WRITEUP MODE
────────────────────────────────────────

### Final Answer

**Flag:** `cpctf{686661}` 

---

### Problem Understanding

The challenge provided a single image file (`rainy_day.jpg`) and asked for the pincode of the place where the photo was taken. On the surface, it looks like a normal rainy street scene. However, the actual task is to extract location intelligence from visual clues present in the image.

---

### Thought Process

Initially, I considered whether this might be a typical image forensics challenge involving hidden data (e.g., steganography, metadata, or appended content). However, nothing immediately suggested that direction.

Instead, I focused on visible clues in the image:

- The vehicle number plate started with “KL”, indicating Kerala, India.
- The environment (palm trees, wet roads) also matched a tropical region like Kerala.
- A white statue on a traffic roundabout stood out as a distinctive and non-generic feature.

Generic elements like roads and vehicles are not useful for identification, but unique landmarks (like statues) often are. This led me to focus on the statue as the key clue.

---

### Tools and Techniques Used

- Google Lens: Used to reverse-search the statue from the image  
- Google Search: Used to find the pincode of the identified location  

No Linux or forensic tools were needed since the challenge was visual OSINT-based.

---

### Step-by-Step Solution

#### Step 1 — Initial Inspection

I manually inspected the image instead of running tools like `file` or `strings`, since nothing suggested embedded data.

Key observations:
- Kerala vehicle registration (KL)
- Tropical vegetation
- A roundabout with a white statue

This indicated the challenge likely involved identifying a real-world location.

---

#### Step 2 — Locating the Key Clue

I zoomed into the image and focused on the white statue.

Reasoning:
- Statues are often unique and searchable
- Other elements (cars, roads, trees) are too generic

This made the statue the most promising lead.

---

#### Step 3 — Reverse Image Search

I used Google Lens on the cropped image of the statue.

The search results identified it as:

Statue of Jawaharlal Nehru at Nehru Place, Muvattupuzha

This gave a precise location:
- Muvattupuzha, Kerala, India

---

#### Step 4 — Finding the Pincode

I searched for:

Muvattupuzha pincode

The result was:

686661

---

#### Step 5 — Flag

The required flag format was:

cpctf{686661}

---

### Why This Worked

- The challenge relied on visual OSINT, not hidden data.
- The statue served as a unique identifier, making reverse search effective.
- Regional clues (like “KL” on the number plate) narrowed down the search area.
- Google Lens can map visual objects to real-world locations quickly.
- Once the location was known, the pincode was straightforward to obtain.

---

### Lessons Learned

- Always determine whether a challenge is forensics or OSINT before choosing tools.
- Look for unique landmarks rather than generic objects.
- License plates and environmental context can provide strong geographic hints.
- Reverse image search is extremely useful in CTFs.
- Avoid overcomplicating — sometimes the simplest approach is correct.

---

**Final Flag:** `cpctf{686661}`
