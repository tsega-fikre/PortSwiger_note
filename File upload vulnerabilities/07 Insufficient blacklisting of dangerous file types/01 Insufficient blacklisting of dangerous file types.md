# **Insufficient Blacklisting of Dangerous File Types**.

---

## The Big Picture

When a website wants to stop users from uploading harmful files, one idea is:  
> “Let’s make a list of bad file extensions and block them.”

For example:  
Block `.php`, `.exe`, `.js`, etc.

This is called a **blacklist** — a list of things that are **not allowed**.

But this approach has a **major flaw**: You can’t think of every possible bad extension. Attackers know extensions you forgot.

---

## Step 1: What is a file extension?

A file extension is the part after the last dot in a filename.  
It tells the computer (or server) what kind of file it is.

Examples:
- `photo.jpg` → `.jpg` = image
- `document.pdf` → `.pdf` = document
- `script.php` → `.php` = PHP script (dangerous if uploaded by a hacker)

If a hacker uploads `evil.php`, the server sees `.php` and might run it as code.

---

## Step 2: How blacklisting tries to help

The website’s upload filter says:

> “I will check the file extension. If it is `.php`, I will reject the upload.”

Seems smart… but here’s the problem.

---

## Step 3: Why blacklisting is flawed

### Analogy: Banned words in a chat room

Imagine a chat room that bans the word **“cat”** because it’s offensive.  
A user types: `c a t` with spaces.  
Or `CaT`.  
Or `kat` (if the offensive word was “cat” but they use a variant).

You can never ban **every possible way** to say something offensive.

Same with file extensions.

---

## Step 4: Real example with PHP

The server blocks `.php`.  
But PHP can also be executed from these extensions:

| Extension | Works? |
|-----------|--------|
| `.php` | Blocked ✅ |
| `.php5` | Works! ❌ Not blocked |
| `.php4` | Works! ❌ Not blocked |
| `.php3` | Works! ❌ Not blocked |
| `.phps` | Sometimes works |
| `.phtml` | Sometimes works |
| `.phar` | PHP archive — can execute |

The hacker tries `evil.php5`.  
Server: “Hmm, `.php5` isn’t on my blacklist. Allowed!”  
Then the server executes it. **Hack successful.**

---

## Step 5: Other dangerous extensions people forget

Not just PHP. Here are others that can run code:

| Extension | What it does |
|-----------|---------------|
| `.shtml` | Runs server-side includes (can execute commands) |
| `.asp` | Runs ASP code on Windows servers |
| `.aspx` | Newer ASP.NET |
| `.jsp` | Runs Java code |
| `.pl` | Perl script |
| `.py` | Python script |
| `.cgi` | Any CGI script |
| `.hta` | Runs HTML + scripts in Windows |

A blacklist might block `.asp` but forget `.aspx`.  
Block `.pl` but forget `.pm` (Perl module).  
Block `.jsp` but forget `.jspx`.

---

## Step 6: Why this is a never-ending problem

New file extensions appear.  
Old ones come back.  
Servers are configured differently.

On **Server A**, `.php5` runs.  
On **Server B**, `.php5` does nothing.  
The attacker doesn’t know — they just try.

Also, servers can be configured to recognize **anything** as PHP:

```
evil.php.fake
evil.php.jpg
evil.php;.jpg
```

Sometimes the server ignores everything after `.php` and still runs it.

---

## Step 7: The real lesson

> **Blacklisting is weak because it assumes you know all bad extensions. You don’t.**

Better approach: **Whitelisting** (allow only good extensions).

Whitelist example:  
> “Only allow `.jpg`, `.png`, `.gif`, `.pdf` — reject everything else.”

Now the hacker can’t sneak in `.php5`, `.shtml`, or anything else — because it’s not on the **allowed list**.

---

## Summary in one sentence

> Blacklisting dangerous file extensions fails because attackers know many alternative extensions that servers might still execute, so the only safe approach is to whitelist only the harmless extensions you explicitly allow.