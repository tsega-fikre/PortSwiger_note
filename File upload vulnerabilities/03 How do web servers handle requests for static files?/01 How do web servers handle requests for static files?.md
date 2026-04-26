

---

## 1. What’s a web server?

Think of a **web server** as a **giant filing cabinet** in a room.  
When you visit a website, your computer (the “client”) sends a request to that filing cabinet, like:  
> “Give me the file `home.html`.”

The server’s job is to find that file and send it back to you.

---

## 2. What are “static files”?

**Static files** are files that don’t change.  
Examples:  
- `picture.jpg` (an image)  
- `style.css` (instructions for colors & layout)  
- `script.js` (code that runs in your browser)  

When you request these, the server just **picks the file from its disk** and sends it exactly as it is.  
No thinking, no changing — just give the file.

---

## 3. How does the server know where the file is?

In the **old days**, websites were just folders on a computer, like:  

```
website/
  home.html
  images/
    logo.jpg
  css/
    style.css
```

If you asked for `https://example.com/images/logo.jpg`, the server would:  
1. Look in the `website/` folder (the “root”).  
2. Go into the `images/` folder.  
3. Grab `logo.jpg`.  
4. Send it to you.

**Simple rule:** The web address (URL) after the domain name maps **directly** to a file path on the server.  

Example:  
URL path: `/images/cat.png` → Server looks for `website/images/cat.png`

This is called **1:1 mapping** — one URL path maps exactly to one file location.

---

## 4. Example step-by-step

Imagine a visitor opens a website in their browser.

1. **Browser:** “Please give me `https://shop.com/style.css`”  
2. **Server (filing cabinet):**  
   - Starts at the website’s main folder, say `/var/www/shop/`  
   - Sees `/style.css` in the URL.  
   - Looks for file `/var/www/shop/style.css`  
   - Finds it.  
3. **Server:** Reads the file and sends it back exactly as is.  
4. **Browser:** Uses that CSS file to make the page pretty.

That’s it. No programs run, no data is changed. Just raw file reading.

---

## 5. What’s changed today?

Nowadays, most websites are **dynamic** — the page changes depending on who you are, the time, or what you click.  
Example: Facebook’s URL `facebook.com/profile` is **not** a real file `profile.html` on their disk. Instead, a program (like PHP, Python, or Java) runs, looks up your info from a database, and **creates** the page on the fly.

But even on modern sites, **static files** are still there for things like:  
- Logos, icons, photos, CSS, JavaScript, PDFs, downloads.

And servers still handle them the **old way** — by mapping URL → file path.

---

## 6. Why does this matter for security?

Because of this 1:1 mapping, if an attacker can **upload a file** to the server (like a malicious `script.php`), and then guess where that file is stored, they can request it directly via a URL.  

For example:  
1. Attacker uploads `evil.php` to a photo-sharing website.  
2. Server saves it in `/var/www/uploads/evil.php`  
3. Attacker visits `https://website.com/uploads/evil.php`  
4. The server, following the 1:1 mapping rule, runs that file if it’s executable.  
5. Attacker takes over the website.

So the simplicity of **static file handling** is exactly what makes **file upload vulnerabilities** so dangerous.

---

## Summary for someone with zero knowledge:

- A web server is like a clerk with a filing cabinet.  
- For static files (images, CSS, etc.), the clerk simply looks at the URL, opens the cabinet at that exact spot, and hands over the file.  
- That rule is very simple — and very powerful.  
- If an attacker can put a bad file inside the cabinet, they can later ask the clerk for it by URL, and the clerk will happily hand it over — or worse, run it if it’s a program.