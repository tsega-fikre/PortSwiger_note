# Resetting User Passwords — Explained Simply

## The Core Problem

A user forgets their password. They can't log in. So the website needs a way to let them **prove they are who they say they are** — without using the password (because they've forgotten it).

That's a weird situation. The entire point of authentication is to prove your identity, and now the user **can't do that**.

So the website has to fall back on **other ways** to verify identity. Every one of those ways is a potential vulnerability.

---

## Why This Feature Is Inherently Dangerous

Here's the key quote from the text:

> "The password reset functionality is inherently dangerous and needs to be implemented securely."

**Why "inherently dangerous"?**

Because you're building a **backdoor** into every account. Normally, you need a password to get in. Now, there's an **alternate path** that skips the password entirely. That path had better be bulletproof — or attackers will just use it instead of guessing passwords.

**Analogy:**
Your house has a front door with a strong lock. Great. But you also have a **spare key hidden under the doormat** for when you forget your keys. If an attacker finds that spare key, the front-door lock doesn't matter. The spare key is the weak link.

Password reset is the spare key.

---

## The Common Implementations (From Best to Worst)

The text says:

> "There are a few different ways that this feature is commonly implemented, with varying degrees of vulnerability."

Let's look at them, from the most secure to the most dangerous:

---

### 🟢 1. Emailed Reset Token (Standard, Good When Done Right)

**How it works:**
1. User enters their email
2. Server generates a **random, single-use token**
3. Server emails a link: `https://site.com/reset?token=a8f3e9c1...`
4. User clicks → server verifies token → user sets new password

**Why it's good:**
- Token is random → cannot be guessed
- Token expires after some time → limits damage if leaked
- Token is single-use → can't be replayed
- The attacker needs **access to the email** to abuse it

**Where it fails:**
- If the token is **predictable** (e.g., derived from username, timestamp)
- If the token is **never invalidated** after use
- If the reset link is leaked (via Referer header, browser history, etc.)
- If the email itself can be hijacked

---

### 🟡 2. Email-Based Confirmation Code

**How it works:**
1. User enters their email
2. Server emails a **6-digit code**
3. User enters the code + new password
4. Server checks code → resets password

**Why it's okay:**
- Simpler UX than clicking a link
- Still requires email access

**Where it fails:**
- **No rate limiting** → 6-digit code = 1 million possibilities = brute-forceable
- **Code never expires** → any valid code works forever
- **Code is predictable** → some sites use `md5(email)` truncated to 6 digits

**The big risk:** if the site doesn't rate-limit the code submission, an attacker can brute-force 1,000,000 codes in hours.

---

### 🟠 3. Security Questions

**How it works:**
1. User enters their email
2. Site shows security questions: "What was your first pet's name?"
3. User answers correctly → can reset password

**Why it's weak:**
- Answers are often **public info** (mother's maiden name, city of birth)
- Answers are often **guessable** (pet names — "Fluffy", "Rex")
- Answers are often **forgettable** — user gives a different answer next time
- Answers are **stored as plain text** in many cases
- Often **no rate limit** → attacker can brute-force answers
- **User enumeration** — different behavior for known vs. unknown emails

**Real-world impact:** Security questions are so weak that many security experts recommend **never using them**.

---

### 🔴 4. Reset Link Sent to a "New" Email (Very Dangerous)

**How it works:**
1. User enters their email
2. Site emails a link to "confirm" and "change" the email associated with the account
3. If the user clicks → password reset is sent to the **new** email

**Why it's terrible:**
- The "new" email is **attacker-controlled** in some flawed implementations
- No verification that the new email belongs to the user
- **Complete account takeover** in one step

**The classic attack:** attacker enters the victim's email + their own email. Site sends reset link to attacker's email. Game over.

---

### 🔴 5. Direct Password Reset Without Verification (Worst)

**How it works:**
1. User enters their email
2. Site shows a form: "Enter your new password"
3. User submits → password is changed

**Why it's a disaster:**
- **Anyone** who knows a victim's email can reset their password
- No verification whatsoever

**This is only theoretical** — no real site does this. But it's the "ceiling" of bad implementations.

---

## 🎯 The Recurring Themes of Vulnerable Password Reset

Across all these implementations, the same weaknesses appear:

| Weakness | What It Enables |
|----------|-----------------|
| **Predictable tokens** | Attacker can forge reset links |
| **No expiration** | Old leaked links work forever |
| **No single-use** | One valid link → infinite resets |
| **Brute-forceable codes** | Try all 6-digit codes |
| **User enumeration** | Learn which emails are registered |
| **Host header injection** | Reset link sent to attacker's domain |
| **Referer leakage** | Reset link leaks via third-party sites |
| **Weak security questions** | Answers are public info |
| **No rate limiting** | Unlimited guessing |
| **No CSRF protection** | Reset initiated from another site |

Each of these is a vulnerability class on its own. The next sections (password reset broken logic, password reset poisoning) will focus on the most exploitable ones.

---

## 🧠 Why Developers Get This Wrong

| Reason | Result |
|--------|--------|
| **"It's just a password reset, how bad can it be?"** | Underestimation of risk |
| **"Nobody attacks the reset page"** | Skipped security review |
| **"We copied this from a tutorial"** | Copied known-bad patterns |
| **"The token is random enough"** | Weak randomness (e.g., `Math.random()` in JS) |
| **"Only the user sees the link"** | Ignored Referer leakage, caching, logs |
| **"We don't need to expire it"** | Links leak, get shared, get logged |
| **"The code expires in 1 hour"** | 1 hour = 3,600 seconds = 3,600 guesses per IP per hour — plenty |

---

## 🛡️ How Password Reset Should Be Implemented

| Requirement | Why |
|-------------|-----|
| **Random, high-entropy token** | Cannot be guessed or forged |
| **Short expiration** | Leaked tokens have a limited window |
| **Single-use** | Every reset requires a new token |
| **Bound to user** | Token is useless without the right user context |
| **Sent to verified email only** | Reset link goes where the account owner can see it |
| **No sensitive data in URL** | Avoid Referer leakage |
| **Rate-limited** | Slow down brute-force attempts |
| **Same response for known/unknown emails** | Prevent user enumeration |
| **Invalidate existing sessions** | Password change kills all sessions |
| **Require re-authentication** | For account setting changes, ask for the password again |

---

## Summary Table

| Concept | Plain English |
|---------|--------------|
| **Password reset = backdoor** | A way in that skips the password |
| **"Inherently dangerous"** | Any flaw here = full account takeover |
| **Best implementation** | Random, expiring, single-use emailed token |
| **Weak implementation** | Security questions, brute-forceable codes |
| **Worst implementation** | Direct reset without verification |
| **Common mistakes** | Predictable tokens, no expiration, no rate limit |
| **Core lesson** | Password reset must be **as secure as login** — not weaker |

---

## The Big Takeaway

> **Password reset is a "backdoor" you deliberately install into every account. If it's weaker than the front door, attackers will just use the backdoor.**

Since the entire purpose of authentication is to prove identity, and password reset exists specifically to **bypass the primary proof**, it must be designed as carefully as (or more carefully than) the login page itself.

The recurring pattern in every vulnerability class is the same:

- **Predictable inputs → forged tokens**
- **Missing expiration → replay attacks**
- **User-controlled parameters → tampering**
- **Missing rate limits → brute force**

---

## 🎯 What's Coming Next

In the following lessons, you'll see these specific vulnerability classes in action:

1. **Password reset broken logic** — tampering with the reset request parameters (e.g., changing the username)
2. **Password reset poisoning via middleware** — manipulating the Host header so the reset link points to the attacker's domain
3. **Password reset poisoning via dangling markup** — injecting HTML into the reset email itself
4. **Weak tokens** — brute-forcing predictable reset tokens
5. **Brute-forcing reset codes** — brute-forcing short verification codes

Each of these is a **one-shot account takeover** in the wrong hands.

---

Ready to dive into the first one — **password reset broken logic** — or do you want to review any of these concepts in more depth first?