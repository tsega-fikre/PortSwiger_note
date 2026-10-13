# Flawed Two-Factor Verification Logic — Explained Simply

## The Core Idea

Sometimes 2FA is broken because the website **forgets who you are** between step 1 and step 2. It relies on a **cookie that the user can change** to decide which account you're logging into.

If you can change that cookie to someone else's username, you can **log into their account** — without ever knowing their password.

---

## How a Normal 2FA Flow *Should* Work

A properly built 2FA system needs to remember **who you are** throughout the entire process. Here's the correct flow:

1. You enter your username + password → server verifies
2. Server creates a session that says: **"This is user carlos, waiting for 2FA"**
3. You enter the 2FA code → server checks it against *that same session*
4. Server confirms → you're logged in as carlos

The key: **The server decides who you are. You don't get to tell it.**

---

## How the Broken Flow Works (The Vulnerability)

Here's the flawed version from the example:

### Step 1: You Log In With Your Own Credentials

```
POST /login-steps/first
username=carlos&password=qwerty
```

The server checks the password → correct → responds with a cookie:

```
HTTP/1.1 200 OK
Set-Cookie: account=carlos
```

**Notice what just happened:** The server told your browser "You are carlos" by giving you a cookie. But this cookie is **just a plain text value** — and your browser sends it back on every subsequent request.

### Step 2: You're Asked for the 2FA Code

Your browser loads the 2FA page:

```
GET /login-steps/second
Cookie: account=carlos
```

The page says: "Enter your 6-digit code."

### Step 3: You Submit the Code

```
POST /login-steps/second
Cookie: account=carlos
verification-code=123456
```

The server looks at the `account` cookie, sees `carlos`, and checks the code against **carlos's account**.

---

## 🚨 The Attack: Change the Cookie

Here's where it gets dangerous. The `account` cookie is **client-controlled**. Your browser sends it — but **you can change it before sending**.

### The Attack Flow

**Step 1:** You log in with **your own** account:

```
POST /login-steps/first
username=wiener&password=peter
```

Server responds with:

```
Set-Cookie: account=wiener
```

Now you're on the 2FA page — but you don't care about your own account.

**Step 2:** Before submitting the code, you **change the cookie** from `wiener` to `victim-user` (the person you want to break into):

```
POST /login-steps/second
Cookie: account=victim-user     ← CHANGED!
verification-code=123456
```

**Step 3:** If the code is correct (or you brute-force it), the server thinks:

> "Hmm, the account cookie says `victim-user`. Let me check if the code matches victim-user's account. Yes! Let them in."

**Result:** You are now **logged in as victim-user** — even though you never knew their password.

---

## Why This Is So Dangerous

### 💀 Reason 1: No Password Needed

You logged in with **your own** credentials, but the server let you switch to **anyone's account** by changing a cookie. You never even had to guess the victim's password.

### 💀 Reason 2: Brute-Force the Code

The 2FA code is usually just 6 digits — only **1,000,000 possibilities**. If the website doesn't rate-limit the code submissions, you can brute-force it in minutes.

And since you can submit codes for **any username** you want, you can:

- Brute-force codes for **victim-user-1**
- Brute-force codes for **victim-user-2**
- Brute-force codes for **victim-user-3**
- ...and so on

Once you crack one, you're in.

### 💀 Reason 3: The Server Is Trusting the Wrong Thing

The server is asking **the attacker**: "Who are you trying to log in as?"

That's like a bank teller asking a robber: "Which account would you like to access today?" — and then handing over the money.

---

## Analogy Time

Imagine a two-step security process at a building:

1. **Step 1:** Guard checks your ID at the front desk and gives you a **sticky note** that says "Visitor: John"
2. **Step 2:** A second guard asks for your secret code, then looks at your sticky note to decide who to let in

Now imagine you can **rewrite the sticky note** before showing it to the second guard. You write "Visitor: The CEO" — and the second guard lets you into the CEO's office.

The second guard **trusted the sticky note** instead of verifying your identity himself. That's the flaw.

---

## What the Correct Implementation Looks Like

The server should **never** trust the client to say which account is logging in. Instead:

1. After step 1, the server creates a **server-side session** tied to the user
2. The session stores: "This session is waiting for 2FA for user carlos"
3. The client only gets an **opaque session ID** (a random string) — not the username itself
4. When the client submits the code, the server looks up the session **on its own side** to see which user this is for
5. The client **cannot** change which user the session belongs to

**The fix in one sentence:** The server must decide who you are — not the client.

---

## Summary Table

| Concept | Plain English |
|---------|--------------|
| **Normal 2FA** | Server remembers who you are from step 1 to step 2 |
| **The flaw** | Server uses a **client-controlled cookie** to decide which account you're logging into |
| **The attack** | Log in as yourself → change the cookie to the victim's username → brute-force the code |
| **Why it works** | The server trusts the cookie value instead of a secure server-side session |
| **The danger** | You can log into **any account** without knowing any password |
| **The fix** | Store the user identity **server-side** in a session the client can't modify |

---

## The Big Takeaway

> **Never trust the client to tell the server who they are.**

Authentication must be based on **server-side state**, not on cookies or parameters the user can freely change. If a website lets you say "I'm logging in as this person" via a cookie, then 2FA is basically useless — anyone can pretend to be anyone.

**The golden rule of authentication:** The server decides who you are. You just prove it.


![[Pasted image 20261012210453.png]]
