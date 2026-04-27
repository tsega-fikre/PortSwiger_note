
---

# 🔐 Core Idea: Why Obfuscation Works

The whole problem comes from **inconsistency** between different parts of a system:

* 🔍 **Validation layer** (e.g., PHP code checking file name)
* ⚙️ **Storage/processing layer** (filesystem, web server like Apache/Nginx)
* 🧠 **Interpreter** (e.g., PHP engine executing `.php`)

👉 If these layers **interpret the filename differently**, you can bypass restrictions.

---

# 🧩 1. Case Sensitivity Bypass

### Example:

```
exploit.pHp
```

### Why it works:

* Validation: checks only `.php` (lowercase)
* Server: treats `.pHp` = `.php` (case-insensitive)

👉 Result: malicious file slips through.

---

# 🧩 2. Multiple Extensions

### Example:

```
exploit.php.jpg
```

### Different interpretations:

* Validator: sees `.jpg` → ✅ allowed
* Server (Apache misconfig): sees `.php` anywhere → executes

### Important concept:

Some servers process files like this:

* First extension → ignored
* Last extension → used
* OR **any extension triggers handler**

👉 This depends on configuration (very important in real attacks).

---

# 🧩 3. Trailing Characters

### Example:

```
exploit.php.
exploit.php   (with space)
```

### Why it works:

* Validation: sees `.php.` → not `.php`
* OS or server:

  * trims trailing `.` or whitespace
  * becomes `exploit.php`

👉 Now it executes.

---

# 🧩 4. URL Encoding Bypass

### Example:

```
exploit%2Ephp   (%2E = .)
```

### Flow:

1. Validator checks: `"exploit%2Ephp"` → not `.php` ❌
2. Server decodes later:

```
exploit.php
```

👉 Boom — execution possible.

---

### 🔥 Double Encoding (advanced)

```
exploit%252Ephp
```

* First decode → `%2E`
* Second decode → `.`

👉 If validation and server decode at different stages → bypass.

---

# 🧩 5. Null Byte Injection (Very Important)

### Example:

```
exploit.asp%00.jpg
```

### What happens internally:

* `%00` = NULL byte (`\0`)
* In **C/C++**, strings end at NULL

### So:

```
"exploit.asp\0.jpg"
```

* Validator (PHP/Java): sees `.jpg` ✅
* Low-level system (C): sees `exploit.asp` ❗

👉 File treated as `.asp`

---

# 🧩 6. Semicolon Trick

### Example:

```
exploit.asp;.jpg
```

### Why it works:

* Some servers treat `;` as separator
* May execute first part (`exploit.asp`)

👉 Again → mismatch between validation and execution.

---

# 🧩 7. Unicode / Multibyte Bypass

### Example bytes:

```
xC0 x2E
```

### Idea:

* Looks like harmless characters
* After **Unicode normalization** → becomes `.`

👉 So:

```
exploit[weird_unicode]php
→ becomes → exploit.php
```

---

# ⚠️ The Real Root Problem

All these tricks exploit:

## 👉 **Parsing Inconsistency**

Different layers interpret:

| Layer     | Interpretation    |
| --------- | ----------------- |
| Validator | "safe file"       |
| Server    | "executable file" |

---

# 🧠 Real-World Insight (Important for You)

Since you're studying ethical hacking, this is the mindset:

👉 Don’t think:

> "What bypass works?"

👉 Think:

> "Where does interpretation change?"

Ask:

* Is validation case-sensitive?
* Does decoding happen later?
* Does OS normalize filenames?
* Does server use different parsing logic?

---

# 🛡️ Proper Defense (Very Important)

If you were building a secure system:

### ❌ Bad:

* Blacklist `.php`

### ✅ Good:

* Allow only safe extensions (`.jpg`, `.png`)
* Validate **MIME type + file content**
* Rename file completely:

```
user_upload → random_name.jpg
```

* Store outside web root
* Disable execution in upload directory

---

# 🔥 Final Takeaway

All these techniques are not random tricks — they are **exploiting differences in how systems interpret data**.

That’s the mindset that separates:

* Script kiddie ❌
* Real security engineer ✅

---


