# Bypassing Two-Factor Authentication — Explained Simply

## The Core Idea

Sometimes, a website's 2FA is built so poorly that you can **skip it entirely**. You never even have to enter a code.

This isn't about guessing the code or intercepting it — it's about the website **forgetting to check** whether you actually completed step 2.

---

## How a Normal 2FA Login Flow Works

Here's how a well-built 2FA login should work:

1. You enter your **username + password**
2. The website checks them → if correct, you move to step 2
3. You enter the **6-digit code** from your phone
4. The website checks the code → **only then** are you fully logged in
5. You can now access your account pages (like `/my-account`)

**Key point:** Until step 4 succeeds, you should NOT be able to access any logged-in pages.

---

## The Flaw: The "Half-Logged-In" State

Here's where things break. On many poorly built sites, after step 2 (password check), the website **already considers you logged in** — it just *asks* for the code on a separate page as a courtesy.

**Analogy:**

Imagine a nightclub with two bouncers:
- **Bouncer 1** (password check): Checks your ID at the front door
- **Bouncer 2** (2FA check): Asks for a secret handshake inside

On a **well-built** club, if you don't know the handshake, Bouncer 2 kicks you out.

On a **poorly built** club, Bouncer 1 lets you in, and Bouncer 2 is just... standing in a hallway. If you walk past him without stopping — or find another door into the VIP area — nobody stops you.

---

## How the Bypass Actually Works

### Step 1: Enter Your Password
You log in normally with your username and password. The website says:

> "Great! Now enter your 2FA code."

You're shown a page like `/login2` or `/2fa` asking for the code.

### Step 2: Don't Enter the Code — Just Skip the Page

Here's the trick: **you're already logged in at this point.** The website gave you a session cookie when you passed the password check. It's just *asking* you to enter the code before proceeding.

So instead of entering a code, you **manually navigate to a page that should require full login**, such as:

```
https://victim-site.com/my-account
```

Or:

```
https://victim-site.com/admin
```

Or whatever "logged-in only" page exists.

### Step 3: See If It Works

If the website is poorly built, **it will show you the page** — no code required.

Because the site never actually verified that you completed step 2, you've effectively **bypassed 2FA entirely**.

---

## Why This Happens (The Developer's Mistake)

The developer made a subtle but critical error:

- **What they should do:** Track a flag like `2fa_completed = false` in your session, and **block every protected page** until that flag becomes `true`.
- **What they actually do:** Mark you as "logged in" after the password check, but only *display* the 2FA page as a step in the UI. If you skip the UI, you skip the check.

**Analogy:** The developer put a "Do Not Enter" sign on the VIP door — but didn't actually lock it.

---

## Real-World Example

Imagine a banking website:

1. You log in with your password ✅
2. It shows you a 2FA page
3. You ignore it and type `bank.com/transfer-money` into the URL bar
4. The page loads — and you can transfer money without ever entering a code

That's a **critical vulnerability**. It means 2FA provides zero protection.

---

## What to Test (For Pentesters)

If you're testing a site with 2FA, try these things after completing only the password step:

| Test | What to Try |
|------|-------------|
| **Direct URL access** | Go to `/my-account`, `/admin`, `/settings`, etc. |
| **Change the URL** | If 2FA is at `/login2`, try going to `/login3` or `/home` |
| **Use the API** | Sometimes the mobile API doesn't check 2FA even when the web does |
| **Check cookies** | See if a session cookie is already set before you enter the code |
| **Go back and forward** | Some sites break if you navigate back then forward |

If any of these work, **2FA is bypassed**.

---

## Why This Is So Dangerous

When 2FA is bypassable, it means:

- A stolen password is **enough** to break in — 2FA gives no protection
- The user *thinks* they're protected, but they're not
- The attacker doesn't need to intercept codes, swap SIMs, or guess anything
- Detection is hard — the login looks "normal" to the server

**Analogy:** You paid for a home security system, but the alarm doesn't actually trigger — it just *looks* like it's armed.

---

## Summary Table

| Concept | Plain English |
|---------|--------------|
| **Normal 2FA flow** | Password → 2FA code → full access |
| **The flaw** | Website marks you as logged in after password, before code |
| **The bypass** | After entering password, skip the 2FA page and go directly to a protected page |
| **Why it works** | The site never verifies that step 2 was completed |
| **The fix** | Track `2fa_completed` in the session and block all protected pages until it's true |
| **The takeaway** | 2FA is only as strong as its implementation — a skipped check = no protection |

---

## The Big Takeaway

> **2FA must be enforced, not just displayed.**

Showing a user a 2FA page is **not the same** as requiring them to pass it. If the website doesn't actively block access to protected pages until the code is verified, the entire second factor is meaningless.

A determined attacker with a stolen password can walk right past the second door — because it was never locked in the first place.