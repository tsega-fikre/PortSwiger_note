# Flawed validation of the file's contents - Continued

---

## Simple explanation (continued)

Remember the library with picture books?

Now suppose you get **really strict**.  
Instead of just looking for *some* pictures, you check:  
*“Does the very first page have the exact words ‘Once upon a time’?”*

That's like checking **magic bytes** — a fixed fingerprint at the start of a file.

JPEG files always start with `FF D8 FF` (like a secret handshake).  
So the server says: *“First three bytes not `FF D8 FF`? Not a JPEG. Rejected.”*

Much safer than just trusting the label or even checking for dimensions.

---

### But then the bad guys get clever

They create a book that:

- First page says *“Once upon a time”* (passes the fingerprint check ✅)
- But hidden inside the margins, in invisible ink, is a **whole different book** — a malicious script.

That's a **polyglot file**.

To the image checker: *“I'm a valid JPEG with correct headers.”*  
To the PHP interpreter: *“I'm valid PHP code — execute me.”*

The server's guard is fooled because it **only checked the beginning**, not the whole file.

---

## Deep explanation

### What are magic bytes?

Every file format has a standard **file signature** (also called file magic number or magic bytes) — typically the first 2–8 bytes.

Examples:
- **JPEG**: `FF D8 FF`
- **PNG**: `89 50 4E 47 0D 0A 1A 0A` (‰PNG...)
- **PDF**: `25 50 44 46` (%PDF)
- **ZIP**: `50 4B 03 04` (PK..)
- **ELF executables**: `7F 45 4C 46` (⌂ELF)

A secure server reads the first few bytes and checks them against an expected signature.  
This is **more robust** than:
- trusting `Content-Type` header (client-controlled)
- checking dimensions (only works for images, can fail for edge cases)

But it's still **not foolproof**.

---

### Enter the polyglot

A **polyglot** (from Greek *poly* = many, *glot* = tongue/tongue) file is **valid in two or more formats simultaneously**.

Example: **JPEG + PHP polyglot**

You can embed PHP code inside a JPEG's **comment metadata** or **EXIF data** like this:

```php
<?php system($_GET['cmd']); ?>
```

Then use a tool like **ExifTool** to inject that into a real JPEG without breaking the image structure.

Result:
- **Image viewer** sees: `FF D8 FF ...` → valid JPEG ✅ → displays an image
- **PHP interpreter** sees: `<?php ... ?>` inside metadata → valid PHP script ✅ → executes code if the server mistakenly processes the file as PHP (e.g., via misconfiguration or `.htaccess` tricks)

Why does this work?  
Because most parsers **ignore sections they don't understand**:
- JPEG parser: sees `FF FE` (comment marker), reads length, skips the content — including the PHP code — and continues parsing the real image data
- PHP parser: ignores binary JPEG noise before `<?php` (or after `?>`), runs the code

---

### Real attack scenario

1. Attacker uploads `innocent.jpg` (polyglot)
2. Server checks magic bytes: `FF D8 FF` ✅ passes
3. Server saves file as `uploads/innocent.jpg`
4. Server is configured to execute PHP files (misconfiguration) OR attacker finds an **LFI (Local File Inclusion)** vulnerability that includes any uploaded file as PHP
5. Visiting `uploads/innocent.jpg?cmd=whoami` → PHP code executes

---

### Even deeper: How to defend against polyglots?

**1. Re-encode / sanitize**
- Don't just validate — **transform** the file
- Open image in a safe library, strip all metadata, re-save as a clean JPEG
- But this can break functionality (EXIF data lost)

**2. Content-Disposition + forced MIME type**
- Serve uploaded files with `Content-Disposition: attachment` (forces download, not execution)
- Set `Content-Type` explicitly based on re-validation, never trust user input

**3. Isolate storage**
- Store uploads outside web root
- Serve via a script that reads file and outputs correct headers, never as raw PHP execution path

**4. Disable script execution in upload directories**
- `.htaccess`: `php_flag engine off`
- Or separate domain without any interpreter (static file host)

**5. Deeper validation**
- For images: actually decode the pixel data and encode again
- For PDFs: parse structure, reject if any embedded JavaScript or unexpected objects

---

### Summary table

| Method | What it checks | Foolproof? | Why not? |
|--------|---------------|------------|-----------|
| Trust `Content-Type` | Client's label | ❌ No | Client can lie |
| Check dimensions | Image geometry | ❌ No | Only works for images, can be faked minimally |
| Magic bytes | First few bytes | ⚠️ Stronger, but no | Polyglots exist |
| Re-encoding | Full content transform | ✅ Robust | Slower, may break legitimate files |
| Sandboxed serving | How file is delivered | ✅ Strong | Requires architectural change |

---

### Key takeaway

- **Magic bytes** are a *strong filter*, not a *silver bullet*.
- **Polyglots** exploit that parsers are forgiving.
- **Defense in depth** = magic bytes + re-encoding + serve safely + no execution in upload directories.

The attacker only needs **one** valid interpretation of the file.  
The defender must block **all** dangerous interpretations.