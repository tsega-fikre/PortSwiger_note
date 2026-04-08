You've perfectly captured the core reason why these vulnerabilities are so common: **it's not that developers are unaware of the danger, but that the gap between *intended* security and *actual* security is often filled with flawed logic.**

To expand on your points and provide a more granular breakdown, we can categorize the root causes of file upload vulnerabilities into four main areas: **Flawed Validation Logic**, **Inconsistent Enforcement**, **Information Disclosure**, and **Insecure Server Configuration**.

Here’s a deeper look at how these vulnerabilities arise:

### 1. Flawed Validation Logic (The "Blacklist" Mentality)

As you noted, trying to block "bad" things is inherently risky. Attackers are constantly looking for something that isn't on the list.

- **Extension Blacklisting:** A developer might block `.php`, but forget to block the less common `.php5`, `.phtml`, or `.phar` extensions. On some servers, these are still executed as PHP code.
- **MIME Type Reliance:** Checking the `Content-Type` header (e.g., `image/jpeg`) is a common mistake because this header is sent by the client and can be changed with a simple intercepting proxy.
- **Superficial Content Checks:** A developer might check if the file contains the word `<?php` to block PHP files. An attacker can simply obfuscate the code (e.g., using `<?= system($_GET['cmd']); ?>` if short tags are enabled, or using base64 encoding) to bypass this.
- **Double Extensions:** The server might check the *last* extension in the filename. If the code checks for `.jpg` and the file is named `shell.php.jpg`, it might pass validation. Whether this is dangerous depends entirely on the web server's parsing rules (Apache might treat it as PHP based on the `.php` part, while Nginx might treat it as a static image).

### 2. Inconsistent Enforcement (The Parsing Discrepancy)

This is a sophisticated category you alluded to. The vulnerability arises when the validation logic and the underlying filesystem/webserver "see" the filename differently.

- **Null Byte Injection:** In older versions of PHP (5.3.* and below), the file system was vulnerable to null bytes. If an attacker uploaded `shell.php%00.jpg`, the validation check might see `.jpg` and allow it. However, when the operating system goes to save the file, it interprets the null byte (`%00`) as the end of the string, truncating it and saving the file as `shell.php`.
- **Unicode Overflows/Case Sensitivity:** If the blacklist blocks `.asp` but the server is configured on a case-insensitive filesystem (like Windows), an attacker could upload `shell.ASP` or `shell.Asp` to bypass the filter.
- **Trailing Characters:** Some validation strips trailing spaces or dots. If an attacker uploads `shell.php. . . .`, the filter might see the trailing dots and allow it. However, when Windows saves the file, it automatically strips those trailing spaces/dots, leaving `shell.php`.

### 3. Information Disclosure (Leaking the Battle Plan)

Sometimes, the vulnerability isn't in the upload function itself, but in what the upload function reveals to the attacker.

- **Path Disclosure:** If an upload error message reveals the full server path (e.g., `/var/www/secret-project/uploads/`), the attacker now knows exactly where to place files in a path traversal attack.
- **"File Exists" Errors:** If the application tells the user "This file already exists," an attacker can use this as a boolean check to enumerate existing files on the server (e.g., trying to upload `/etc/passwd` to see if it exists).
- **Client-Side Source Code:** If the validation is done entirely in JavaScript, viewing the page source reveals the exact logic (the allowed extensions, the regex patterns) to the attacker, allowing them to tailor their malicious file specifically to bypass it.

### 4. Insecure Server Configuration (The Environment is the Enemy)

Sometimes the file uploaded is perfectly valid (e.g., a real `image.jpg`), but the server's configuration makes it dangerous.

- **Misconfigured Web Server:** The server might be configured to execute arbitrary files in the upload directory. For example, an Apache server might have `mod_php` enabled and a directive like `AddHandler php7-script .jpg`. In this case, uploading a valid `image.jpg` that has PHP code embedded in its metadata (an image polyglot) would result in code execution.
- **Shared Hosting Confusion:** An attacker on a shared host might be able to bypass directory restrictions by uploading a `.htaccess` file (if allowed) to the upload directory. This file can reconfigure Apache on the fly, instructing it to treat `.jpg` files as executable scripts, turning a previously safe image upload into a critical vulnerability.

### Summary of the Root Cause

In essence, file upload vulnerabilities arise from a failure to answer the "Three Questions of Trust":

1.  **Is it the right *kind* of file?** (Failing to validate content thoroughly).
2.  **Is it the right *size* of file?** (Failing to limit resources).
3.  **Will it be handled *safely* once stored?** (Failing to separate storage from execution).

When developers answer these questions with "trust the user" or "trust the filename extension," rather than "trust our thorough server-side validation and configuration," they introduce the gaps that attackers exploit.