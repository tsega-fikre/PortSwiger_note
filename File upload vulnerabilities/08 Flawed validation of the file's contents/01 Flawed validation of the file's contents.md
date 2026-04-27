# Flawed validation of the file's contents

---

## **Simple explanation**

Imagine you run a library where people can bring in books.  
A rule: only *picture books* (with lots of images) are allowed.

Someone brings in a box labeled **“Picture Book”**.  
If you only check the **label**, you’d say “OK, put it on the shelf.”

But what if it’s actually just a notebook full of text?  
The **label** is wrong.

So instead, you open the box and **look at the pages**.  
If you see no pictures, you reject it.

That’s like a secure server:  
- It doesn't just believe the file’s **label** (Content-Type header like `image/jpeg`).  
- It checks the **actual content** — like “does this file have image dimensions?”

If you try to sneak in a PHP script (which is code, not an image),  
the server checks: “What’s its width and height?”  
The PHP script has none.  
So it rejects it. ✅

---

## **Deep explanation**

### Step 1 – What’s Content-Type?
When a browser or app uploads a file via HTTP (like a form), it sends metadata headers. One is `Content-Type`, e.g.:
- `image/jpeg`
- `image/png`
- `text/html`
- `application/x-php`

That’s just the *client telling the server* what it claims the file is.

### Step 2 – Why is trusting it bad?
A malicious user can easily fake that header.  
They upload `malicious.php` but set `Content-Type: image/jpeg`.  
If the server trusts blindly:
1. It saves the file as `image.jpg` or with a `.jpg` extension.
2. It might even serve it as an image.
3. But if the server executes PHP files (e.g., in an `uploads/` folder), visiting that “image” could run the PHP code, leading to **remote code execution** — a critical security hole.

### Step 3 – How do secure servers verify contents?
Instead of trusting the label, they **validate using file structure**.

Examples:

- **Images**: Try to get dimensions using a library like `getimagesize()` in PHP, or `ImageMagick` in other languages.  
  - For a real JPEG, you get width/height.  
  - For a PHP script, `getimagesize()` returns `false` → reject.

- **PDFs** : Check for `%PDF` header at the start, and `%%EOF` near the end.

- **ZIP files** : Check local file header signature (`PK\x03\x04`).

- **Pure validation**: You might even attempt to process/render a thumbnail after upload. If it fails → reject.

### Step 4 – Why can’t an attacker fake this?
Because valid image formats have a **defined binary structure**.  
Example – a minimal JPEG:
```
FF D8 FF E0 ... (SOI and APP0 markers...)
```
To have valid dimensions, the file must contain SOF (Start of Frame) marker with height & width bytes in correct places.  
A PHP script like `<?php system($_GET['cmd']); ?>` won’t have that structure. No matter how you rename or prepend bytes, you can’t make it a valid image without breaking the PHP code (unless using advanced polyglot tricks, but those are harder and can be detected with deeper checks).

---

## **Key takeaway**
- **Flawed validation** = trusting `Content-Type` header only.  
- **Secure validation** = examining actual file structure to confirm type (e.g., check image dimensions).  
- This prevents *file type confusion attacks*, where a script masquerades as an innocent image, then gets executed.