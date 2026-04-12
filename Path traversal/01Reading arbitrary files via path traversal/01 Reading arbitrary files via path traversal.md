That's the classic textbook example from the PortSwigger Web Security Academy—it perfectly illustrates the core mechanic. You've laid out the Unix case clearly, and including the Windows `..\` variant is an important detail that many overviews miss.

To extend that example with the **real-world context** that usually follows in that lesson, here's what comes next:

### Common Obstacles & Bypasses in That Scenario

The example assumes zero defenses. In practice, the application might do **one** of these three things, each with a known bypass:

**1. Stripping "../" (but only once)**
If the app removes `../` to "sanitize" the input, an attacker just nests them.
- **Payload:** `....//....//etc/passwd`
- **Result:** The inner `../` is removed, leaving the outer `../` intact.

**2. Requiring a Specific File Extension**
The app might append `.png` to the filename automatically.
- **Payload (Null Byte Injection - Legacy PHP):** `../../../etc/passwd%00.png`
- **Explanation:** The `%00` (Null Byte) terminates the string in older PHP versions, so the filesystem stops reading at `passwd` and ignores the appended `.png`. *(Note: This is fixed in modern PHP > 5.3.4)*.

**3. Requiring the Path to Start with Expected Folder**
The app checks if the path *starts* with `/var/www/images/`.
- **Payload:** `/var/www/images/../../../etc/passwd`
- **Explanation:** The check passes because the string begins correctly, but the traversal still resolves back to root.

### The Ultimate Endgame: Reading Source Code

The PortSwigger lab hints at this, but it's worth stressing: `/etc/passwd` is a **proof of concept**. In a real engagement, the attacker doesn't stop there. The next step is almost always:
`filename=../../../var/www/html/config.php`

Since PHP files are **executed** by the server, a direct request to `config.php` via a web browser only shows the *output* (usually a blank page). However, a **Path Traversal** vulnerability reads the **source code** of that file. This is how attackers steal database passwords and API keys.