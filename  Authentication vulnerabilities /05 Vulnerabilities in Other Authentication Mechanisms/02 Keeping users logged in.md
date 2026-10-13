# Keeping Users Logged In — Explained Simply

## What Is "Stay Logged In"?

When you log into a website, you usually get logged out when you close your browser. But most sites offer a checkbox:

> ☑️ **Remember me** or ☑️ **Keep me logged in**

If you tick it, you stay logged in even after closing the browser — sometimes for weeks or months. That's convenient, but it also means there's a **persistent key to your account** sitting somewhere on your computer.

If an attacker can **steal or forge that key**, they get into your account without ever needing your password.

---

## How It Usually Works (The Simple Version)

1. You tick "Remember me" and log in
2. Server generates a **"remember me" token** — a random-looking string
3. Server sends it to your browser inside a **persistent cookie**
4. Every time you visit the site, your browser sends that cookie automatically
5. Server sees the cookie → "Ah, this is John, let them in" → **no password needed**

**The cookie IS your login.** If someone else gets a copy of it, they get your account.

---

## The Core Security Rule

Since the cookie bypasses the entire login process, it should be:

- **Long** (at least 32 random characters)
- **Random** (cryptographically secure)
- **Unguessable** (no pattern an attacker can reverse-engineer)
- **Single-use per session** (rotate on use — an advanced technique)
- **Expirable** (so stolen cookies don't work forever)

**If the token is predictable, the whole security model collapses.**

---

## ❌ The Bad Way (What Vulnerable Sites Do)

Instead of generating a random token, some sites build the cookie from **values that the user already knows or can find out**:

### Common Bad Patterns

| Pattern | Example |
|---------|---------|
| **Username + timestamp** | `john.doe:1697200000` |
| **MD5 of username** | `5d41402abc4b2a76b9719d911017c592` |
| **Base64 of "username:password"** | `am9objpzZWNyZXQ=` |
| **Hash of username + static secret** | `sha1("john" + "company_salt")` |
| **Username + password hash** | `john:5f4dcc3b5aa765d61d8327deb882cf99` |

These are **all predictable** because they're built from data the attacker can guess, or learn.

---

## 🚨 Why "Attacker Can Create Their Own Account" Is the Key

This is the crucial insight:

> "This approach is particularly dangerous if an attacker is able to create their own account because they can study their own cookie and potentially deduce how it is generated."

**What this means in plain English:**

If you can sign up for the site yourself, you can:

1. Log in with **your own account**
2. Tick "Remember me"
3. Look at your own cookie — e.g., `remember_me=am9objoxNjk3MjAwMDAwMA==`
4. Decode it → discover it's `john:1697200000`
5. Realize it's just `username:timestamp`

Now you know the **formula**. You just need to figure out what the victim's cookie would look like.

**Analogy:**
A brand-new employee gets a keycard on their first day. If they realize the keycard number is just `employeeID + startDate`, they can figure out the keycard number of the CEO — and print a fake card to walk into the executive suite.

---

## 🔓 "Encrypting" the Cookie — Three Levels of Bad

The text warns about three different "protections" that fail:

### 🟥 Level 1: Base64 Encoding

The site thinks:

> "Our cookie is encrypted! Nobody can read it!"

Reality: Base64 is **encoding**, not encryption. It's just a way of representing data in a text-safe format. Anyone can decode it instantly.

**Example:**
```
Cookie: remember_me=am9objoxNjk3MjAwMDAwMA==
```
Decoded:
```
john:1697200000
```

**Tools to decode:** `base64 -d`, Burp Decoder, CyberChef — takes 2 seconds.

**Verdict:** Base64 provides **zero** security. It's not even hiding anything.

### 🟧 Level 2: A Simple Hash (No Salt)

The site thinks:

> "We hash the cookie! You can't reverse a hash!"

Reality: Hashing prevents **direct** reversal, but hashing isn't magic. If the attacker can guess the inputs, they can just hash their guesses and compare.

**Example:**
```
Cookie: remember_me=5d41402abc4b2a76b9719d911017c592
```

If the site generates this by `MD5(username)`, an attacker can:

- Hash `john` → get hash A
- Hash `admin` → get hash B
- Hash `victim` → get hash C
- Compare with the cookie

Or run a **wordlist** through MD5 and match every hash in seconds.

**Verdict:** Unhashed, unsalted hashes are **trivially brute-forced**.

### 🟨 Level 3: A Hash With a Weak/Static Salt

The site thinks:

> "We hash AND salt it! Now it's bulletproof!"

Reality: If the salt is:

- **Static** (same for every user — like `"company_name_2024"`)
- **Well-known** (from a leaked config, an error message, or public code)
- **Included in the cookie itself** (defeats the purpose)

…then the attacker just includes the salt in their guesses.

**Example:**
```
Cookie: remember_me=MD5("john" + "company_secret_2024")
```

Attacker discovers the salt somehow → tries:
```
MD5("admin" + "company_secret_2024")
MD5("victim" + "company_secret_2024")
MD5("carlos" + "company_secret_2024")
```

Or, since they can make their own account, they compare:
```
Their cookie = MD5("john" + "company_secret_2024")
```

They can't reverse it, but they can **brute-force the inputs** to match it.

**Verdict:** Static or weak salts offer no real protection.

### 🟩 Level 4: Secure Random Token (The Right Way)

The site generates a **cryptographically random** token — 32+ bytes of true randomness:

```
Cookie: remember_me=Kf8Hq2mNp7Xw3LvYt9DsA6bCjRzE1uIg
```

This has:
- No relationship to username, password, or any known value
- No pattern to deduce
- So much entropy that brute-forcing is mathematically infeasible

**Verdict:** Secure — assuming it's also invalidated when the user logs out or changes password.

---

## 🎯 The Attack Workflow

Here's how an attacker exploits a predictable "remember me" cookie:

| Step | Action |
|------|--------|
| **1** | Create your own account on the site |
| **2** | Log in with "Remember me" enabled |
| **3** | Copy your cookie value |
| **4** | Decode or analyze it (Base64, hash, etc.) |
| **5** | Identify the pattern (`username:X`, `MD5(username)`, etc.) |
| **6** | Figure out how to generate the value for **any** username |
| **7** | Target the victim — e.g., try `carlos`, `admin`, `victim` |
| **8** | Craft a cookie with the victim's username and any guessed timestamp/hash |
| **9** | Send it in a request → server accepts it → **account hijacked** |

**The killer detail:** This attack **bypasses login attempt limits** because you're not submitting passwords — you're submitting cookies. If the site doesn't rate-limit cookie guesses, you can try as many as you want.

---

## 🔍 Real-World Examples

This isn't theoretical. Real cases include:

- **Companies using MD5 of email as the "remember me" token** — attackers precompute MD5s of common emails and match cookies
- **Sites that Base64 "encrypt" `user_id:expiry`** — attackers forge cookies for any user ID
- **Apps that hash only the username** — attackers can guess the cookie for `admin` in one request

---

## 🧠 Why Developers Get This Wrong

| Mistake | Why It Happens |
|---------|---------------|
| **Base64 = "encryption"** | Confusion between encoding and encryption |
| **Hashing without a salt** | Assumption that "hashes can't be reversed" |
| **Static salt** | "We add a secret" — but the secret isn't secret |
| **Predictable inputs** | "It's easier to compute than random" |
| **Same token forever** | "We give them a token, they keep using it" |
| **Not invalidating on logout** | "We just delete the cookie from their browser" (but the server doesn't) |

Every one of these is a real-world vulnerability class that appears in bug bounty reports.

---

## ✅ How a "Remember Me" System Should Be Built

| Requirement | Why |
|-------------|-----|
| **Random 32+ byte token** | Cannot be guessed or forged |
| **Stored server-side** (as a hash) | Database stores `hash(token)` — cookie holds the token itself |
| **Bound to the user** | Server looks up the token → finds the user |
| **Single-use or rotatable** | Optionally rotate on each request to prevent replay |
| **Expiration** | e.g., 30 days |
| **Invalidated on logout** | Server deletes the token record |
| **Invalidated on password change** | Changing password kills all other sessions |
| **Rate-limited** | Slow down cookie guessing |

---

## Summary Table

| Concept | Plain English |
|---------|--------------|
| **"Remember me" cookie** | A persistent token that bypasses login |
| **The flaw** | Token is predictable (built from username, timestamp, etc.) |
| **The enabler** | Attacker can create their own account and study their own cookie |
| **Common bad "encryptions"** | Base64 (no security), unsalted hashes, static salts |
| **The attack** | Forge a cookie for the victim's username → full account takeover |
| **Why it's dangerous** | Bypasses login attempt limits and password requirements |
| **The fix** | Use cryptographically random, server-validated tokens |

---

## The Big Takeaway

> **A "remember me" cookie is a master key to your account. If it's predictable, everyone can forge it.**

The pattern repeats across all authentication features:

- Developers think "nobody will guess this" — and they're wrong
- "Encryption" doesn't help if it's really just Base64 or a hash of guessable input
- The presence of **user-controlled input** (like a username) inside the cookie is a giant red flag

**The golden rule:** A "remember me" token must be as random and unpredictable as a strong password — because if it isn't, it's a **backdoor into every account on the site**.

Ready for the next topic (password reset broken logic, password reset poisoning), or do you want to explore this "remember me" vulnerability deeper with a lab walkthrough?