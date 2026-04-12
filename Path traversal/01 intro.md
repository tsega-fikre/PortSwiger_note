That's an excellent and accurate summary. You've correctly identified the core definition, alternate name, and the severe potential impact.

To build on what you've provided, here’s a slightly more technical breakdown of **how** it works and **why** it's dangerous:

### The Mechanics of the Attack

The vulnerability occurs when an application uses user-supplied input to construct a **file path** without properly validating or sanitizing it. The attacker supplies **path traversal sequences** (like `../` on Unix/Linux or `..\` on Windows) to "climb out" of the intended directory (the *web root*) and access the underlying filesystem.

**Example Scenario:**
An e-commerce site shows product images via a URL parameter:
`https://example.com/loadImage?filename=product.png`

The backend code (bad practice) does this:
`/var/www/app/images/` + `[USER INPUT]`

**The Attack:**
The attacker changes the parameter to:
`https://example.com/loadImage?filename=../../../../etc/passwd`

**Resulting File Path on Server:**
`/var/www/app/images/../../../../etc/passwd` → This resolves to `/etc/passwd`.

If the application reads the file and outputs its contents, the attacker now has the server's list of user accounts.

### Beyond Reading Files (RCE)

You mentioned writing files. This is the critical escalation path to **Remote Code Execution (RCE)** . The most common technique involves:

1. **Log File Poisoning:** The attacker uses Path Traversal to read the server logs (e.g., `/var/log/apache2/access.log`). They then send a normal HTTP request with malicious PHP code inside the **User-Agent** string.
2. **File Inclusion:** The attacker then uses **Local File Inclusion (LFI)** —a closely related vulnerability—to execute the log file as code, thereby running the malicious PHP they injected.

### Types of Defenses (and How Attackers Bypass Them)

Developers often attempt basic security measures, which attackers have learned to circumvent:

| Defense Attempt | Bypass Technique |
| :--- | :--- |
| **Blocking `../`** | Using **Absolute Paths** (e.g., `filename=/etc/passwd`). |
| **Stripping `../` (Non-Recursive)** | **Nested Traversal:** `....//....//etc/passwd`. When the inner `../` is removed, the outer dots combine to form `../` again. |
| **Encoding Checks** | **URL Double Encoding:** `..%252F` (where `%25` decodes to `%`, making `%2F` a slash). |

### Distinction from Local File Inclusion (LFI)

While often used in the same exploit chain, there is a technical distinction:
- **Path/Directory Traversal:** The **Read** or **Write** primitive only. The file is returned as raw data or binary.
- **Local File Inclusion (LFI):** The file is **executed** as code by the server (e.g., a PHP `include()` statement). Path Traversal is the mechanism used to *reach* the file for LFI.