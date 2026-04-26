
---

## 1. Reminder: What’s a static file?

A **static file** is just a file that doesn’t change.  
Examples:  
- `dog.jpg` (a picture)  
- `style.css` (colors and layout instructions)  
- `page.html` (a simple web page)  

When you ask for one, the server finds it on its disk and sends it to you.

---

## 2. What happens when the server gets a request for a file?

Let’s say you type into your browser:  
`https://example.com/images/cat.png`

The server does three main things:

### Step 1: Look at the file’s ending (the extension)

The server looks at the **last part** of the file name after the dot.  
Here, `cat.png` → extension is `.png`

Why? Because the extension tells the server **what kind of file** this probably is.  
- `.jpg` or `.png` → image  
- `.html` → web page  
- `.php` → a script (program)  
- `.txt` → plain text

---

### Step 2: Match extension to a MIME type

The server has a little phonebook inside it that says:  
- `.png` → `image/png`  
- `.jpg` → `image/jpeg`  
- `.html` → `text/html`  
- `.php` → `application/x-httpd-php`

This phonebook is called **MIME type mappings**.

MIME types tell the browser: “This is an image” or “This is a video” or “This is a program’s output.”

---

### Step 3: Decide what to do based on the file type

Now the server asks itself: **Is this file executable or not?**

#### Case A: Non-executable file (image, HTML, text file)

Example: `cat.png` (image)

- The server says: “This is not a program. I will just read the file and send it exactly as it is.”
- It puts the file’s contents into an HTTP response and sends it to your browser.
- Your browser sees the MIME type `image/png` and shows the picture.

**Simple.** No running, no changing. Just a delivery.

---

#### Case B: Executable file (like `.php`) AND server is set up to run it

Example: `calculator.php`

Some servers (like Apache with PHP installed) are configured to run `.php` files as small programs.

What happens:  
1. Server sees `.php` → “Ah, this is a script I should execute.”  
2. Before running it, the server collects information from the request (like what you typed in a form, or data in the web address).  
3. The server runs the PHP script. The script does something (adds numbers, looks up a user, etc.).  
4. The script produces output (like “Result: 14”).  
5. The server wraps that output in an HTTP response and sends it to your browser.  
6. Your browser shows the result.

This is **dynamic** — the output can be different every time.

---

#### Case C: Executable file BUT server is NOT configured to run it

Example: You upload a `.php` file to a server that doesn't have PHP installed or has PHP turned off.

What happens:  
The server says: “I know `.php` sometimes means executable, but I’m not allowed to run it. Now what?”

**Two possibilities:**

1. **The safe way (usually)** → Server returns an error: “403 Forbidden” or “500 Internal Error.”  
   No harm done.

2. **The dangerous misconfiguration** → Some poorly set-up servers will just send the `.php` file as plain text.  
   So instead of running the script, they show you its source code (the actual program).  
   This can leak passwords, database names, or secret logic — big security problem.

Example:  
Request: `https://website.com/secret.php`  
Response: Not the output of the script, but the script itself:  
```php
<?php
$password = "supersecret123";
?>
```  
Now the attacker knows the password.

---

## 3. Why does this matter for file upload vulnerabilities?

When a website lets users upload files, an attacker might upload:
- A `.php` script (executable)
- A `.html` file with malicious JavaScript
- An `.htaccess` file that changes server rules

Then the attacker tries to request that file via the URL.

What happens next depends entirely on how the server handles that file extension:

| File type | Server configured to run it? | What happens |
|-----------|------------------------------|----------------|
| Image (.png) | No (not executable) | File is sent → safe (usually) |
| PHP script (.php) | Yes | Server runs it → attacker takes control |
| PHP script (.php) | No | Either error (safe) OR source code leaked (bad) |
| HTML (.html) | No (not executable, but interpreted by browser) | Browser runs any JavaScript inside → XSS attack |

---

## 4. Summary for someone with zero understanding

- The server looks at a file’s **ending** (like `.jpg` or `.php`) to guess what to do with it.  
- It has a tiny rulebook (MIME types) that says: “.jpg means image, .php means runnable program.”  
- **Images & HTML:** Server just sends them as-is.  
- **Programs like PHP:** Server **runs** them if allowed, or **shows error** if not allowed — but sometimes it mistakenly **shows the program’s secret source code** instead.  
- That last mistake (showing source code) can leak passwords and break the website’s security.

So when an attacker uploads a file, they’re hoping the server will treat it in a dangerous way — either run it or leak it.