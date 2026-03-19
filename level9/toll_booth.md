### Final Answer

**Flag:** `cpctf{28.666,77.216}`

---

### Problem Understanding

The challenge provided a video file (`batman.mp4`) showing CCTV footage of a toll booth. The task was to determine the exact geographic coordinates (latitude and longitude) of the location where the video was recorded.

At first glance, the video appears ordinary surveillance footage. However, the real task is an OSINT (Open Source Intelligence) problem — identifying a real-world location from visual clues in the footage.

---

### Thought Process

The key observation was that the footage showed a toll booth with distinctive structural features. Since direct identification using the full frame was difficult due to obstructions like trees, I focused on isolating the most identifiable part — the toll structure itself.

Using Google Lens on the full image produced noisy results. Narrowing the search to just the toll booth significantly improved relevance. While reviewing the results, I noticed a recurring phrase: “View other cams in :”, which is a known signature of the Insecam website.

This was a critical pivot point. Instead of searching generally, I refined the query to include “insecam”, which led directly to a matching live camera feed.

---

### Tools and Techniques Used

- **Google Lens:** Used for reverse image search and visual matching
- **Web browser:** For OSINT-based investigation and validation
- **Insecam:** Identified as a source of publicly accessible CCTV feeds

---

### Step-by-Step Solution

#### Step 1 — Initial Inspection

The provided file was a video (`batman.mp4`) showing CCTV footage of a toll booth.

Initial approach:
- Visually inspect the video
- Identify unique structures or landmarks

Observation:
- A toll booth structure was clearly visible
- Surroundings (trees, weather) added noise and reduced clarity for direct search

---

#### Step 2 — Narrowing Down the Search

Instead of searching the entire frame, I focused only on the toll booth structure.

Using Google Lens on the cropped/targeted region:
- Produced multiple similar toll booth results
- However, no exact match initially

While scanning results, I noticed a recurring phrase:
- “View other cams in :”

This phrase is strongly associated with the Insecam website, which hosts publicly accessible CCTV feeds.

---

#### Step 3 — Targeted OSINT Pivot

I refined the search query by adding the keyword:
```
<toll booth> + insecam
```

This led to a specific page:
```
http://insecam.org/en/view/890651/
```

This page contained:
- The exact same camera feed as in the challenge video
- Metadata including:
  - Country: India
  - City: Delhi
  - Latitude: 28.666670
  - Longitude: 77.216670

---

#### Step 4 — Extracting the Flag

The challenge required coordinates rounded to three decimal places.

Rounding:
- Latitude: 28.666670 → 28.666
- Longitude: 77.216670 → 77.216

Final flag:
```
cpctf{28.666,77.216}
```

---

### Why This Worked

- The video contained real-world infrastructure, enabling OSINT techniques
- Google Lens helped identify visually similar structures
- The phrase “View other cams in :” acted as a unique fingerprint for Insecam
- Pivoting from generic search to platform-specific search drastically reduced noise
- Insecam provided precise metadata, eliminating guesswork

---

### Lessons Learned

- Always isolate the most distinctive visual element before running reverse image searches
- Small textual clues (like UI phrases) can be powerful OSINT indicators
- Platform-specific fingerprints can significantly narrow down search space
- OSINT challenges often reward observation more than brute-force searching
- When results are noisy, refine queries incrementally instead of switching tools immediately

---

**Final Flag:** `cpctf{28.666,77.216}`
