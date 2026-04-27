## 📌 How to Prevent File Upload Vulnerabilities (Simple Explanation)

### 🧠 Core idea

File uploads are safe only if the server **strictly controls what gets uploaded, where it goes, and how it is used**.

---

## 🛡️ 1. Use a Whitelist (NOT blacklist)

✔ Allow only safe file types:

* `.jpg`
* `.png`
* `.pdf`

❌ Don’t try to block bad ones like `.php`, `.exe`

👉 Why?
Attackers can invent new bypass tricks, but you can clearly define what is allowed.

---

## 🧼 2. Sanitize file names

Remove dangerous parts like:

```text
../
```

👉 Why?
Without this, attackers can try:

* Directory traversal
* Overwriting system files

---

## 🔁 3. Rename uploaded files

Instead of:

```text
profile.php
```

Use:

```text
a8f3k2x9.jpg
```

👉 Why?

* Prevent file overwrite attacks
* Prevent guessing file paths

---

## ⏳ 4. Validate before saving permanently

✔ First:

* Check file type
* Scan content
* Validate size

✔ Only then:

* Save to final storage

👉 Why?
Prevents malicious files from ever reaching the public server.

---

## 🧰 5. Use trusted frameworks

Instead of writing custom upload logic:

✔ Use:

* Spring (Java)
* Django (Python)
* Laravel (PHP)
* Express middleware

👉 Why?
They already handle:

* validation
* sanitization
* security edge cases

---

## 🧾 One-line Summary

👉 Secure file uploads = **allow only safe files, clean inputs, rename files, validate early, and use trusted frameworks**

---

