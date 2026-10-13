# Changing User Passwords — Explained Simply

## The Core Idea

Changing a password sounds simple: you're already logged in, you type your old password, then your new one twice. Done.

But the **password change page** has a security-critical job:

> **It must verify that the person changing the password is actually the account owner.**

If that verification is weak or bypassable, an attacker can change someone else's password — and take over their account.

---

## What the "Normal" Password Change Flow Looks Like

A **secure** password change page requires:

1. **You must be logged in** — the server knows who you are from your session
2. **You must enter your current password** — proves you know it
3. **You enter the new password twice** — to prevent typos
4. **Server checks the current password matches the logged-in user** — critical step
5. **Server updates the password**

**Key security principle:** The server should **never trust the client** to tell it "who" the user is. It should derive that from the **session cookie** — which the user cannot forge.

---

## 🚨 The Vulnerability: When the Client Controls "Who You Are"

The text says:

> "Password change functionality can be particularly dangerous if it allows an attacker to access it directly without being logged in as the victim user."

And then:

> "For example, if the username is provided in a hidden field, an attacker might be able to edit this value in the request to target arbitrary users."

**What this means in plain English:**

Some poorly built password change forms include the username **as a form field** — hidden in the HTML, like this:

```html
<form action="/change-password" method="POST">
  <input type="hidden" name="username" value="wiener">
  <input type="password" name="current-password">
  <input type="password" name="new-password">
  <input type="password" name="new-password-confirm">
  <button>Change password</button>
</form>
```

The username is sent **from the browser** to the server with each request. The server reads it and uses it to decide **which account to update**.

**The flaw:** The user can **change the hidden field** before submitting. So instead of `username=wiener`, the attacker sends `username=carlos`.

If the server **trusts** that field:
- It looks up `carlos`
- Verifies the "current password" against `carlos`'s password
- If it matches → updates `carlos`'s password

**And the attacker now controls Carlos's password.**

---

## 🎯 The Full Attack

Here's the exact sequence:

### Step 1: Log In As Yourself

The attacker logs into their own account (`wiener:peter`).

### Step 2: Navigate to the Password Change Page

The page shows a form with a **hidden `username` field** set to `wiener`.

### Step 3: Intercept in Burp

The attacker captures the `POST /change-password` request:

```http
POST /change-password HTTP/1.1
Host: victim-site.com
Cookie: session=attacker-session-cookie

username=wiener&current-password=peter&new-password=newpass1&new-password-confirm=newpass1
```

### Step 4: Modify the Username

The attacker changes `username=wiener` to `username=carlos`:

```http
POST /change-password HTTP/1.1
Host: victim-site.com
Cookie: session=attacker-session-cookie

username=carlos&current-password=peter&new-password=newpass1&new-password-confirm=newpass1
```

### Step 5: Send the Request

If the server trusts the client-supplied `username`, it will:
- Look up `carlos`
- Try to verify `current-password=peter` against **Carlos's** password
- If the current password matches → update Carlos's password

**The big IF:** the current password has to match.

Since the attacker typed their own password (`peter`), it won't match Carlos's password — unless the server doesn't check at all, or the attacker brute-forces Carlos's password through this endpoint.

---

## 🎯 Two Ways This Becomes a Full Takeover

### Path 1: Server Doesn't Verify Current Password

Some broken implementations check that the user is logged in, but **never check the current password**. They just accept any new password.

**In this case:**

```http
POST /change-password
username=carlos&new-password=hacked123
```

**→ Carlos's password is now `hacked123`.** Full takeover in one request.

### Path 2: Server Verifies Current Password, But It's Brute-Forceable

The server checks the current password — but:

- **No rate limiting** → attacker can try millions of guesses
- **Server returns different responses** for "wrong password" vs "wrong username" → user enumeration
- **Server leaks timing info** → side-channel attacks

**In this case, the attacker can:**
1. Enumerate valid usernames (see which ones return "wrong password" vs "unknown user")
2. Brute-force the current password for each valid username
3. Once cracked → change the password → full takeover

---

## 🧠 Why This Is So Dangerous

| Attack Aspect | Why It's Serious |
|--------------|------------------|
| **No login required** | Attacker doesn't need to be logged in as the victim |
| **Username is client-controlled** | Attacker can target any user |
| **Combines enumeration + brute force** | Attacker can find + break accounts |
| **No rate limiting** | Unlimited guesses per second |
| **Bypasses normal login protections** | Login may be rate-limited; password change may not |
| **Direct takeover** | Once password is changed, the attacker is in |

---

## 🎯 What to Look For

When you're testing a password change feature, look for these red flags:

| Red Flag | What It Means |
|----------|--------------|
| **Username in a hidden field** | Client controls which account is targeted |
| **User ID in a hidden field** | Same issue — just with ID instead of username |
| **No CSRF token** | Another way to trigger resets on behalf of the victim |
| **Different responses for different usernames** | User enumeration |
| **No rate limiting on current-password checks** | Brute-forceable |
| **Password change works without a session cookie** | No authentication required |
| **Cookie/session not validated before change** | Session fixation / hijack risk |

---

## 🛠️ How to Test (Manual Approach)

### Step 1: Log In As Yourself

Use your own account (`wiener:peter`).

### Step 2: Go Through the Password Change Flow Normally

Capture the `POST` request in Burp.

### Step 3: Look for User-Controlled Values

Check the request body for:
- `username=wiener` — is it there?
- `user_id=123` — is there an ID?
- `email=wiener@...` — is the email in the form?

**These are the fields to attack.**

### Step 4: Try to Change Another User's Password

Modify the username in the request to a victim (e.g., `carlos`).

- If the server accepts the change → vulnerability
- If it rejects it → either the server validates against the session, or the current password check failed

### Step 5: Test User Enumeration

Send requests with different usernames and observe the response:

| Username | Response | Meaning |
|----------|----------|---------|
| `wiener` | "Password updated" (or valid response) | Known user |
| `carlos` | "Incorrect password" | Known user, wrong password |
| `nonexistent` | "Unknown user" or different response | User doesn't exist |
| `admin` | Different timing | Possibly exists |

**If responses differ → user enumeration works.**

### Step 6: Try Without a Session

Remove the `session` cookie and send the request.

- If the server **still** processes it → huge vulnerability
- If the server rejects it → it's at least checking authentication

### Step 7: Try Without Current Password

Remove `current-password` from the request.

- If the server **still** updates the password → massive vulnerability

---

## 🛡️ How Password Change Should Be Implemented

| Requirement | Why |
|-------------|-----|
| **Server derives the user from the session** | Never trust client-supplied `username` or `user_id` |
| **Verify current password** | Proves the person knows it |
| **Rate-limit current password checks** | Prevents brute force |
| **Same response for all failures** | Prevents user enumeration |
| **Require re-authentication** | Ask for the password again even if the user is "logged in" |
| **CSRF token** | Prevents cross-site attacks |
| **Invalidate existing sessions** | Password change kills all other sessions |
| **Email notification** | Let the user know the password changed |

---

## Summary Table

| Concept | Plain English |
|---------|--------------|
| **Password change page** | Similar to login — verifies user, then updates |
| **The vulnerability** | Client-controlled `username` field → attacker picks target |
| **Why it works** | Server trusts what the browser sends |
| **Two attack paths** | (1) No password check, (2) Password check but brute-forceable |
| **Real impact** | Account takeover without knowing the victim's password |
| **How to test** | Look for hidden fields, test different usernames, check responses |
| **Fix** | Server derives user from session; verify current password; rate-limit |

---

## The Big Takeaway

> **The password change page is a "logged-in" version of the login page — and it must be just as careful.**

If the server:
- **Trusts the client** to say who they are → vulnerability
- **Skips the current password check** → vulnerability
- **Doesn't rate-limit** → brute-forceable
- **Returns different responses** → user enumeration

Any of these flaws turns the password change page into a **one-shot account takeover tool**.

---

## 🎯 What's Coming Next

In the follow-up labs, you'll see:

1. **Password change without authentication** — the page is reachable without logging in
2. **Client-controlled username** — hidden fields that can be changed
3. **User enumeration + brute-force via password change** — combining the two
4. **No current password check** — the server lets you change the password without verifying the old one

Ready to look at the next section, or want to explore this vulnerability class with a lab?