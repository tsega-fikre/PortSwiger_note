Excellent! You've moved from SSRF to another critical vulnerability class. Let me explain file upload vulnerabilities in the simplest possible way — like you're teaching someone who knows nothing about security.

---

## The Big Idea (ONE sentence)

> **File upload vulnerabilities happen when a website lets you upload a dangerous file (like a hacker script) but thinks it's a safe file (like a cat picture).**

---

## A simple story to start

**Imagine you're at airport security.**

Normally, you can only bring:
- Water bottles (small ones)
- Laptops
- Phones

But what if the security guard just **glances** at your bag and says: *"Looks fine, go ahead"* — without actually checking?

You could bring:
- A knife
- Fireworks
- Anything dangerous

**That's a file upload vulnerability.**  
The website is the security guard. It's supposed to check files, but it does a bad job.

---

## How normal file uploads work (safe)

When you upload a profile picture to Facebook or Twitter:

1. You click "Upload Image"
2. You select `cat.jpg` from your computer
3. The server checks:
   - Is this really an image? ✅
   - Is it under 5MB? ✅
   - Does the filename end in `.jpg`? ✅
4. The server saves it safely
5. You see your cat picture

**Everything is fine.**

---

## How vulnerable file uploads work (dangerous)

Now imagine a **badly coded** website:

1. You click "Upload Profile Picture"
2. You select `cat.jpg` (but you secretly renamed a hacker script to `cat.jpg`)
3. The server checks:
   - Does the filename end in `.jpg`? ✅ (yes, because you renamed it)
   - Is the file size small? ✅
4. The server saves it WITHOUT checking what's *inside*
5. You visit `website.com/uploads/cat.jpg`
6. Instead of a picture, the server runs your hacker script

**Now you control the website.**

---

## What are "dangerous files"?

| Safe files | Dangerous files |
|------------|----------------|
| `.jpg`, `.png` (images) | `.php` (server-side script) |
| `.txt` (text) | `.jsp`, `.asp`, `.aspx` (web scripts) |
| `.pdf` (documents) | `.exe` (executable program) |
| `.mp4` (videos) | `.js` (JavaScript that server runs) |
| `.csv` (spreadsheets) | `.html` (with malicious JavaScript) |

The danger depends on **how the server handles** the file.

---

## Two ways file uploads cause damage

### Way 1: The upload itself is the attack

Even before you open the file, the upload process can cause harm:

| What you upload | What happens |
|----------------|--------------|
| `../../config.php` (filename with path traversal) | Server saves file outside intended folder, overwriting important files |
| `huge-file.jpg` (500GB file) | Server runs out of storage space (denial of service) |
| `virus.exe` | Antivirus might crash when scanning it |
| `file.xml` with malicious content | Server parses XML and gets hacked (XXE attack) |

### Way 2: You trigger the file later

This is more common. You:

1. Upload a hacker script (renamed as `cat.jpg`)
2. Visit `website.com/uploads/cat.jpg`
3. The server executes your script instead of displaying an image
4. You can now delete files, steal data, or take over the server

---

## Real-world example (simplified)

**Company:** A small website that lets artists upload their portfolio images.

**What developers thought:** *"We'll only allow JPEG and PNG files."*

**How they checked:** They looked at the filename extension only.  
If filename ends in `.jpg` → accept.

**What an attacker did:**
1. Created a file called `hack.php` (contains server takeover script)
2. Renamed it to `hack.php.jpg`
3. Uploaded it as their profile picture
4. The server saw `.jpg` at the end → accepted it
5. Attacker visited `website.com/uploads/hack.php.jpg`
6. The server saw `.php` in the middle and executed the script
7. **Attacker now controls the entire website**

---

## Why is this hidden attack surface?

| Normal file upload (safe) | Vulnerable file upload |
|---------------------------|------------------------|
| Website checks file contents | Website only checks filename |
| Website renames file to something safe | Website keeps your original filename |
| Website stores files outside web root | Website stores files inside accessible folder |
| Website limits file size strictly | Website allows huge or tiny files |

The vulnerability is **invisible** because the upload button looks normal. You don't know the server is doing a bad job checking files.

---

## Common mistakes that cause vulnerabilities

### Mistake 1: Only checking the filename extension
```javascript
if (filename.endsWith(".jpg")) {
    accept(); // DANGER! File could be hack.php.jpg
}
```

### Mistake 2: Only checking the MIME type (what the browser says)
Browser says: `Content-Type: image/jpeg`  
But the file could actually be `hack.php` with a fake label.  
**Servers that trust the browser are vulnerable.**

### Mistake 3: Storing files inside the web root
If files go to `https://website.com/uploads/`, anyone can access them directly.
Instead, files should go to `/home/private/uploads/` (not accessible by URL).

### Mistake 4: Not renaming uploaded files
If you upload `malicious.php`, the server saves it as `malicious.php`.  
Safe servers rename to something like `user_123_abc123.tmp` (random, no extension).

### Mistake 5: Allowing double extensions
`hack.php.jpg` might bypass `.jpg` check but still execute as `.php`

---

## Simple testing strategy (for beginners)

### Step 1: Find a file upload feature
Look for:
- Profile picture upload
- Document upload
- Resume upload
- Attachment upload
- Import file

### Step 2: Try uploading a harmless test file
Upload a real `.txt` file that says "Hello". Does it work? Note the URL where it's stored.

### Step 3: Try dangerous extensions
Create a file called `test.php` with this content:
```php
<?php echo "HACKED"; ?>
```

Upload it. If it's blocked → good. If it uploads → try to visit it.

### Step 4: Try bypass techniques
If `.php` is blocked, try:
- `test.php.jpg`
- `test.PHP` (uppercase)
- `test.php%00.jpg` (null byte trick)
- `test.phtml` (alternative PHP extension)
- `test.php5` (another alternative)

### Step 5: Check if you can access the uploaded file
If you know the upload folder (sometimes `/uploads/`, `/images/`, `/files/`), try visiting:
```
https://website.com/uploads/test.php
https://website.com/images/test.php.jpg
https://website.com/files/test.PHP
```

If you see "HACKED" → You found remote code execution!

---

## What happens when you succeed?

**Remote Code Execution (RCE)** = You can run commands on the server.

Example: You upload a file `shell.php` with:
```php
<?php system($_GET['cmd']); ?>
```

Then visit:
```
https://website.com/uploads/shell.php?cmd=whoami
```

The server shows: `www-data` (the user running the website)

Now you can run any command:
```
?cmd=cat /etc/passwd
?cmd=ls -la
?cmd=rm -rf *
```

**You own the server.**

---

## Even without RCE, file uploads are dangerous

| Attack | How it works |
|--------|--------------|
| **Overwrite files** | Upload `../../config.php` to replace website's configuration |
| **Steal sessions** | Upload `../../../sessions/sess_abc123` to get other users' login tokens |
| **XSS (Cross-Site Scripting)** | Upload `image.svg` with JavaScript inside. When viewed, it steals cookies |
| **Phishing** | Upload `login.html` that looks like the real site, trick users into entering passwords |
| **Denial of Service** | Upload 10,000 files or 1 huge file to fill server storage |

---

## Real-world example: ImageMagick (famous vulnerability)

**What happened:** Many websites use ImageMagick to resize uploaded images.

**The vulnerability:** An attacker could upload a specially crafted image that had hidden commands inside.

**The result:** When ImageMagick processed the image, it ran the attacker's commands on the server.

**The lesson:** Even "safe" file types like JPEG can be dangerous if the software processing them has bugs.

---

## Your mental checklist

Whenever you see a file upload, ask:

- [ ] Can I upload `.php`, `.jsp`, `.asp`, `.aspx` files?
- [ ] Can I upload `.php.jpg` (double extension)?
- [ ] Can I upload files with path traversal (`../../config.php`)?
- [ ] Can I upload an extremely large file (denial of service)?
- [ ] Can I upload an extremely small file (0 bytes, cause crashes)?
- [ ] Can I upload a file with the same name as an existing file?
- [ ] Can I access the uploaded file directly via URL?
- [ ] Does the server process the file (resize, convert, scan) in a risky way?

If you answer YES to any → Potential vulnerability.

---

## Quick comparison: Safe vs. Vulnerable upload

| Safe implementation | Vulnerable implementation |
|---------------------|---------------------------|
| Checks file contents (magic bytes), not just extension | Only checks filename extension |
| Renames file to random string | Keeps original filename |
| Stores outside web root | Stores inside `/uploads/` |
| Limits file type to specific list | Allows any extension except a few |
| Scans for malware | No scanning |
| Limits file size strictly | Large or no size limit |
| Removes metadata (EXIF) | Keeps everything |

---

## How developers SHOULD do it (your defense)

If you ever build a website:

1. **Check file contents** using magic bytes (not just extension)
2. **Rename every uploaded file** to `random_string.jpg` (remove original name)
3. **Store files outside web root** (so no direct URL access)
4. **Serve files through a script** that checks permissions before serving
5. **Use a whitelist** of allowed extensions (`.jpg`, `.png`, `.gif` only — not `.php`, `.html`)
6. **Set a small file size limit** (2-5 MB)
7. **Scan for malware** if possible

---

## Your mastery check

After reading this, you should answer:

1. **Q:** Can renaming `virus.exe` to `virus.jpg` make it safe?  
   **A:** NO! The server might still execute it if it checks only the extension.

2. **Q:** Is a PDF file always safe?  
   **A:** NO! PDFs can contain JavaScript, embedded files, and malicious actions.

3. **Q:** If the server blocks `.php`, how can you bypass?  
   **A:** Try `.php.jpg`, `.phtml`, `.php5`, `.PHP` (uppercase), or `file.php%00.jpg`

4. **Q:** What's the worst that can happen?  
   **A:** Remote Code Execution — you take full control of the server.

---

## Final simple rule

> **Never trust a file just because it has a "safe" name or the user says it's an image. Always check what's actually inside.**

The file upload button is a door. If the website doesn't check who's coming through, anything can walk in — including a hacker disguised as a cat picture.