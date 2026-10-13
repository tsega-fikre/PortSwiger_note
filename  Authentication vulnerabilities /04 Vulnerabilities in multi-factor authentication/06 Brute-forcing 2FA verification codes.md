# Brute-Forcing 2FA Verification Codes — Explained Simply

## The Core Idea

2FA codes are usually just **4 to 6 digits**. That's a small number of possibilities. If a website doesn't protect against guessing, an attacker can simply **try every possible code** until one works — and get into the account.

---

## How Small Is the Problem?

Let's do the math:

| Code Length | Total Possibilities | Time to Brute-Force (100 requests/sec) |
|-------------|--------------------|-----------------------------------------|
| **4 digits** | 10,000 | ~1.7 minutes |
| **6 digits** | 1,000,000 | ~2.8 hours |

Compare that to a password:
- An 8-character password with mixed letters, numbers, and symbols = **~6 quadrillion possibilities**

A 6-digit code has **6 billion times fewer possibilities** than a decent password. That's why brute-force protection on 2FA codes is **critical**.

**Analogy:** A password is like a safe with billions of combinations. A 6-digit code is like a bike lock with only a million. You can't brute-force the safe, but you can brute-force the bike lock.

---

## Why Websites *Need* to Protect Against This

Websites must implement safeguards, such as:

- **Rate limiting** — only allow a few attempts per minute
- **Account lockout** — lock the account after N failed codes
- **Code expiration** — codes expire after a short time (e.g., 30 seconds or a few minutes)
- **Limited attempts** — invalidate the code after 3-5 wrong tries
- **CAPTCHA** — require human verification after failures

Without these, the 2FA is basically a speed bump, not a wall.

---

## The Weak Defense: Auto-Logout After N Failures

Here's a common (but flawed) defense:

> "If the user enters 3 wrong codes, log them out. They'll have to start over from the password step."

At first glance, this seems reasonable. The attacker only gets 3 guesses per login attempt, right?

**Wrong.** Here's why this doesn't work:

### The Attacker Can Just Log In Again

After being logged out, the attacker simply:

1. Enters the victim's **username + password** again (they already have these — remember, in many of these attacks, the password is already known)
2. Gets to the 2FA page again
3. Tries 3 more codes
4. Gets logged out → repeats

With this loop, the attacker gets **3 guesses per cycle** — and they can run this cycle **thousands of times per hour**.

**Analogy:** A bouncer says "You only get 3 tries to guess the secret word." But every time you fail, he lets you go to the back of the line and try again. You just keep cycling through the line and guessing.

---

## How Attackers Automate This

Manually logging in, entering 3 codes, getting logged out, and repeating is slow. So attackers use tools to automate the whole process.

### 🛠️ Method 1: Burp Intruder with Macros

**Burp Intruder** is a tool that sends many requests automatically. But it has a problem: it can only send *one* request per attempt. The 2FA process has *multiple steps* (login → get 2FA page → submit code → get logged out → repeat).

To solve this, Burp has a feature called **Macros**:

- A **macro** is a recorded sequence of requests
- You record: login → load 2FA page → submit code
- Then Intruder runs the whole macro for each code guess
- Between each attempt, the macro re-logs in and gets a fresh session

This lets an attacker:

- Automate the entire multi-step login process
- Try thousands of codes per hour
- Never get permanently blocked (because each attempt is a "fresh" login)

**Analogy:** It's like a robot that repeatedly walks up to the door, knocks 3 times, gets turned away, walks back, knocks 3 more times — and does it 10,000 times while you sleep.

### 🚀 Method 2: Turbo Intruder

**Turbo Intruder** is a Burp extension that's much faster than regular Intruder.

- It can send **thousands of requests per second** (vs. maybe 10-50 with regular Intruder)
- It supports **custom Python scripts** to handle multi-step flows
- It's designed for **race conditions** and **high-speed attacks**

With Turbo Intruder, an attacker can:

- Try all 1,000,000 possible 6-digit codes in a reasonable time
- Handle the login → 2FA → logout → re-login cycle automatically
- Fully brute-force 2FA on a vulnerable site

**Analogy:** Regular Intruder is like a person knocking on doors one at a time. Turbo Intruder is like a machine gun firing at all the doors at once.

---

## Why Auto-Logout Is "Ineffective in Practice"

The key insight from the source text:

> "This is ineffective in practice because an advanced attacker can even automate this multi-step process."

The defense assumes:

1. The attacker is a human doing this manually
2. Logging out slows them down enough to be impractical

But in reality:

1. The attacker is a script — it doesn't get tired or bored
2. Logging out is just one more step to automate
3. **The password is already known** (that's why they're at the 2FA step)
4. So re-logging in is trivial for the script

**The auto-logout defense only works if the attacker has to re-enter a password they don't have.** Since they already have it, the defense does almost nothing.

---

## What Real Brute-Force Protection Looks Like

Proper defenses are much stronger:

| Defense | How It Helps |
|---------|--------------|
| **Rate limiting per account** | Max 5 code attempts per account per hour |
| **Rate limiting per IP** | Slows down distributed attacks |
| **Permanent lockout** | After N failures, require the user to contact support or reset their password |
| **CAPTCHA on every attempt** | Breaks automation |
| **Longer codes** | 8+ digits dramatically increases possibilities |
| **Time-limited codes** | Codes expire in 30 seconds, invalidating brute-force progress |
| **Invalidate after 3 tries** | Force a fresh code to be sent |
| **Server-side tracking** | Remember failed attempts across sessions (not just per session) |

**The most important one:** Server-side tracking that **survives logout**. If the server remembers "this account has had 50 failed 2FA attempts in the last hour," it can block further attempts regardless of re-logins.

---

## The Real-World Danger

If a site has:

- A predictable 6-digit 2FA code
- No proper rate limiting (only auto-logout after 3 tries)
- A known password (from a breach or phishing)

...then **2FA provides almost no protection**. An attacker can:

- Guess the code in a few hours
- Fully automate the attack
- Log in as the victim

This is a **serious vulnerability** and is exactly what the PortSwigger lab demonstrates.

---

## Summary Table

| Concept | Plain English |
|---------|--------------|
| **The 2FA code** | Only 4-6 digits — small enough to brute-force |
| **Proper defense** | Rate limit + lockout + expiration + server-side tracking |
| **Weak defense** | "Auto-logout after 3 wrong codes" |
| **Why it fails** | Attacker just logs in again and gets 3 more tries |
| **The tools** | Burp Intruder with macros, Turbo Intruder |
| **Why it's dangerous** | With a known password, 2FA becomes a speed bump instead of a wall |
| **The fix** | Track failed attempts **server-side** across sessions |

---

## The Big Takeaway

> **A 2FA code is only as strong as the protection around it.**

A 6-digit code by itself is weak — it's just a million possibilities. It only becomes secure when the website:

- Limits how many times you can try
- Tracks failures across sessions
- Makes automation expensive

Without these, a determined attacker can brute-force the code in hours, fully automate the process, and walk right into any account whose password they already know.

**Simple rule:** If a website logs you out after 3 wrong codes but lets you immediately log back in and try 3 more, that's not protection — that's a speed bump.