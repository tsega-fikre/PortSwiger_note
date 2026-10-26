# XXE via File Upload — Explained Simply

## The Core Idea

You can't always see where XXE lives. Sometimes an app says "upload a picture" — and you think "that's just an image, no XML there." But the **library that processes the image** may accept formats that **use XML** under the hood — like **SVG**.

**SVG is XML.** So if the image library parses SVG and allows external entities, you've got XXE.

**The attack surface is hidden** — but it's real.

---

## Why This Matters

You've learned two XXE vectors:

| Vector | Where You Inject |
|--------|------------------|
| **Classic XXE** | Direct XML request body |
| **XInclude** | A value that gets inserted into server-side XML |
| **File upload XXE** | An uploaded file that is XML-based |

The file upload version is powerful because:

- The app might only advertise "PNG / JPEG only"
- But the underlying library **also accepts SVG**
- You upload a "picture" that is actually an **XML weapon**
- The library parses it → XXE fires

**The developer never thought about XML — the library did it for them.**

---

## What Is SVG?

**SVG** stands for **Scalable Vector Graphics**. It's an image format — but unlike PNG or JPEG, it's **text-based** and **XML-based**.

Here's what an SVG file looks like:

```xml
<?xml version="1.0" standalone="yes"?>
<svg xmlns="http://www.w3.org/2000/svg" width="500" height="500">
  <rect width="100" height="100" fill="red"/>
  <circle cx="50" cy="50" r="40" fill="blue"/>
</svg>
```

**Notice:** It's just XML! Same format. Same parser. Same features — including entities.

**That means you can embed XXE payloads in an SVG.**

---

## 🎯 The Attack

### Step 1: Create a Malicious SVG

Here's an SVG file with an XXE payload embedded:

```xml
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE svg [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<svg xmlns="http://www.w3.org/2000/svg" width="500" height="500">
  <text x="10" y="20" font-size="14">&xxe;</text>
</svg>
```

**What this does:**

1. **Declares a DOCTYPE** with an external entity `xxe` pointing to `/etc/passwd`
2. **Defines a `<text>` element** that contains `&xxe;`
3. When the SVG is rendered, the text element displays the file contents

### Step 2: Upload the File

Upload `malicious.svg` to the target site — the one that "only accepts images."

If the server accepts it (either because it allows SVG, or because the image library does), the server will parse it.

### Step 3: Trigger the Parse

Depending on the app, the parse may happen:

- **On upload** — server processes the file immediately
- **On view** — when you or the app views the image
- **On export** — if the image is included in a PDF or HTML report
- **On scan** — if the server runs image validation / thumbnailing

### Step 4: See the Result

If the response shows the rendered SVG (or the file contents in any way), you'll see `/etc/passwd` contents in place of the text.

**Or** — if the app validates the SVG and returns error messages that include the parsed content, you'll see it there too.

---

## 🧠 Why This Works

The vulnerability is in the **chained assumptions**:

| Assumption | Reality |
|------------|---------|
| "Only images are uploaded" | SVG **is** an image — but it's XML |
| "The upload validation is strong" | It checks MIME type / extension — but might not reject SVG |
| "The image library is safe" | It may parse XML and allow external entities |
| "XXE needs a raw XML request" | XXE just needs **any XML input** — including SVG |

**The chain is:**

```
User uploads SVG
     ↓
App accepts it as an "image"
     ↓
Image library parses SVG as XML
     ↓
XML parser processes external entity
     ↓
File contents are read
     ↓
File contents are returned (in rendering, in errors, or in the file itself)
```

**Any break in the chain stops the attack — but in practice, this chain often holds.**

---

## 🎯 Other XML-Based File Formats

SVG isn't the only vector. Any file format that **contains XML** can be abused:

| Format | Description | Common Use |
|--------|-------------|------------|
| **SVG** | Vector image (XML) | Image uploads, icons |
| **DOCX** | Word document (ZIP of XML) | Document uploads |
| **XLSX** | Excel spreadsheet (ZIP of XML) | Spreadsheet uploads |
| **PPTX** | PowerPoint (ZIP of XML) | Presentation uploads |
| **ODT / ODS** | OpenDocument format (ZIP of XML) | OpenOffice uploads |
| **XPS** | XML Paper Specification | Print/export format |
| **KML** | Keyhole Markup Language | Maps / geodata |
| **SAML** | XML-based SSO assertion | Authentication |
| **RSS / ATOM** | XML feeds | Blog/RSS readers |
| **SOAP** | XML-based web service | API communication |

**DOCX / XLSX / PPTX are especially interesting** — they're ZIP archives containing multiple XML files. If the app **extracts and parses** them on the server (very common!), XXE can be injected via any internal XML file.

---

## 🛠️ XXE via DOCX — A Deeper Example

A `.docx` file is a ZIP containing:

```
word/document.xml
word/styles.xml
[Content_Types].xml
_rels/.rels
...
```

To weaponize it:

1. **Create a normal DOCX file** in Word / LibreOffice
2. **Rename it to `.zip`** and extract it
3. **Edit `word/document.xml`** to include an XXE payload:

   ```xml
   <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
   <!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
   <w:document xmlns:w="...">
     <w:body>
       <w:p><w:r><w:t>&xxe;</w:t></w:r></w:p>
     </w:body>
   </w:document>
   ```

4. **Re-zip the file** and rename it back to `.docx`
5. **Upload it** to the target

If the app extracts and parses the DOCX (e.g., for text extraction, preview, or format conversion), the XXE fires.

---

## 🔍 How to Test for This

### Step 1: Identify File Upload Features

Look for:

- Profile picture uploads
- Document uploads
- Avatar uploads
- Import/export features
- Attachment features

### Step 2: Test What Formats Are Accepted

Upload a normal SVG:

```xml
<?xml version="1.0" standalone="yes"?>
<svg xmlns="http://www.w3.org/2000/svg" width="100" height="100">
  <rect width="100" height="100" fill="red"/>
</svg>
```

If it accepts → SVG processing is on the table.

### Step 3: Try an XXE SVG

Upload:

```xml
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE svg [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<svg xmlns="http://www.w3.org/2000/svg" width="500" height="500">
  <text x="10" y="20" font-size="14">&xxe;</text>
</svg>
```

### Step 4: Look for the Result

Check:

- The **response** to the upload
- The **rendered image** (view it in the app)
- The **export/preview** (if the app converts or displays it)
- **Error messages** (they often reveal parse output)

### Step 5: Escalate

Once confirmed, try:

- `/etc/passwd`, `/etc/hostname`, `/etc/hosts`
- App config files: `/var/www/html/config.php`
- Environment files: `/proc/self/environ`
- SSH keys: `/home/user/.ssh/id_rsa`
- Windows: `C:\Windows\win.ini`, `C:\inetpub\wwwroot\web.config`

---

## 🎯 Why This Is Dangerous in Real Life

| Reason | Why It Matters |
|--------|---------------|
| **Hidden attack surface** | The app looks like "just images" |
| **Bypasses XML-specific defenses** | No raw XML request — no WAF rule for XML |
| **Chains with file processing** | Document converters, preview generators, thumbnail services all parse these files |
| **Hard to detect** | The upload looks like a normal image file |
| **Wide attack surface** | Many formats — DOCX, XLSX, PPTX, SVG, ODT, XPS, KML |
| **Common in enterprise apps** | Document management, email gateways, file storage |

**Real-world impact:** XXE via file upload has been found in:

- Document management systems
- Email attachment scanners
- CMS platforms (WordPress plugins, Drupal modules)
- Cloud file storage apps
- Print/preview services
- PDF generation tools

---

## 🛡️ How to Defend Against XXE via File Upload

| Defense | Why |
|---------|-----|
| **Disable external entities in ALL XML parsers** | The library matters, not the app |
| **Disable XInclude in parsers** | Same idea |
| **Reject DOCTYPE declarations** | Block at the parser level |
| **Whitelist allowed image formats** | Don't accept SVG unless necessary |
| **Re-encode images server-side** | Convert to safe formats (PNG/JPEG) |
| **Use safe image libraries** | Libraries with secure default configs |
| **Sandbox file processing** | Run parsers in isolated environments |
| **Strip XML markup from SVG** | If SVG is needed, sanitize it |
| **Reject DOCX / XLSX if not needed** | Or process them in a sandbox |

**Best practice:** Never let the **file upload** path lead directly to an **XML parser** without sanitization. Convert, sanitize, or reject.

---

## Summary Table

| Concept | Plain English |
|---------|--------------|
| **The vector** | Upload a file that is XML-based (SVG, DOCX, etc.) |
| **The trick** | The app thinks "image" — the parser sees "XML" |
| **The classic format** | SVG — it's an image AND an XML document |
| **The other formats** | DOCX, XLSX, PPTX, ODT, XPS, KML |
| **The mechanism** | XXE payload in the XML → file read / SSRF |
| **The attack surface** | Hidden in "image upload" features |
| **How to test** | Upload a malicious SVG or DOCX, check for file contents in responses |
| **The fix** | Disable external entities in parsers, re-encode images, reject DOCTYPE |

---

## The Big Takeaway

> **"Image upload" doesn't mean "no XML." If the image format is SVG, and the parser is permissive, XXE lives there.**

The pattern:

- The app **expects** PNG/JPEG
- The **library** accepts SVG (or DOCX, XLSX, etc.)
- The parser **allows external entities**
- The result: **file read / SSRF / RCE**

**The lesson:** Attack surface for XXE isn't just "requests that contain XML." It's **any input that a server-side XML parser will eventually process** — including files that look like images.

---

## 🎯 What You've Covered in XXE Now

| # | Topic | Status |
|---|-------|--------|
| 1 | Basic XXE file retrieval | ✅ |
| 2 | XInclude attacks | ✅ |
| 3 | XXE via file upload (SVG, DOCX) | ✅ |
| 4 | Blind XXE | ⏳ |
| 5 | SSRF via XXE | ⏳ |
| 6 | Error-based XXE | ⏳ |
| 7 | XXE with parameter entities | ⏳ |

Ready for **blind XXE** — the hardest but most impressive variant — or do you want to practice any of these on a lab first?