# **continued section** 

---

## Where we left off

We just saw that a secure server will **not** execute an uploaded script. Instead, it serves the script as plain text (which is harmless but can leak the source code).

Now this new section explains:
1. Why serving as plain text is still a little risky
2. How an attacker might bypass these protections
3. Two important technical details (filenames and reverse proxies)

---

## 1. Serving as plain text can leak source code (but still stops web shells)

### What is “leaking source code”?

When the server shows the PHP script as plain text, the attacker can **read** the code inside.

Example:  
Instead of running:

```php
<?php echo system($_GET['command']); ?>
```

The server just shows that exact text to the attacker.

**Why is that bad?**  
The attacker can now see **how your application works** — database passwords, hidden logic, security flaws in the code itself.

**Why is it still good?**  
The attacker **cannot** use the file as a **web shell**.

### What is a “web shell”?

A web shell is a script that lets an attacker run commands on your server through their browser.  
For example, visiting:

```
https://victim.com/upload/evil.php?command=delete_all
```

If `evil.php` is executed, it deletes everything.  
If it’s shown as plain text → no deletion → **no web shell**.

> So plain text = source code leak (bad) but no remote control (good).

---

## 2. Different directories have different rules

This is a **critical** point.

### Analogy

Imagine a school:
- **Student upload folder** → Very strict. No running any files. Teachers check everything.
- **Teacher’s office folder** → Less strict. Teachers can run files they trust.

On a web server:
- **`/user_uploads/`** → Strict: No execution allowed.
- **`/scripts/`** or **`/cgi-bin/`** → Execution allowed because the server assumes those files are safe (written by the developer, not users).

### The attacker’s trick

If the attacker can **upload their evil script to `/scripts/` instead of `/user_uploads/`** — the server **will execute it**.

So the attacker asks:  
> “How can I make the server save my file in a different folder?”

### How the server decides where to save a file

> **Tip from the text:**  
> Web servers often use the `filename` field in `multipart/form-data` requests to decide the name and location.

### What does that mean?

When you upload a file via a web form, the browser sends something like:

```
Content-Disposition: form-data; name="file"; filename="../../scripts/evil.php"
```

The server sees `filename="../../scripts/evil.php"` and might think:  
> “Oh, I should save this file as `evil.php` inside the `scripts` folder.”

If the server doesn’t check properly, the attacker can **traverse directories** (`../` means “go up one folder”) to sneak the script into an executable directory.

---

## 3. Even if you block that — reverse proxies add another layer of complexity

### What is a reverse proxy?

You type `normal-website.com` in your browser.  
That **domain name** might not actually be a single server. It could be a **load balancer** or **reverse proxy** that forwards your request to one of several hidden servers behind it.

### Why does this matter for security?

Because each of those hidden servers might be **configured differently**.

Example:
- **Proxy server** (the one you talk to) → Has strict rules: “No executing files in uploads.”
- **Backend server #2** (where your file actually goes) → Has weak rules: “Oh, this file ends in `.php`? Execute it!”

The attacker sends a request to `normal-website.com` (the proxy). The proxy forwards it to a backend server that is **less secure**.  
The attacker never knows there are multiple servers — they just see that sometimes their script works and sometimes it doesn’t.

---

## Summary table for easy memory

| Concept | Simple explanation |
|---------|--------------------|
| **Leaking source code** | Server shows your script as text — attacker can read it but not run it. |
| **Web shell** | A script that gives remote control. Plain text prevents this. |
| **Different directories** | Upload folder = no execution. Scripts folder = execution allowed. |
| **Directory traversal** | Using `../../` in filename to sneak file into safe-to-execute folder. |
| **Reverse proxy** | A middleman server. Backend servers behind it may have different, weaker security rules. |

---

## Final one-sentence summary

> Even if the main server blocks script execution in user uploads, attackers can try to upload to another folder (using filename tricks) or exploit different rules on hidden backend servers — so security must be consistent everywhere.