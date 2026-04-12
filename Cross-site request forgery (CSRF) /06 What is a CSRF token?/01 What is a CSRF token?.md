
---

# What is a CSRF token?

**A CSRF token is a secret password that changes every time you load a form.**

Unlike your real password (which stays the same), a CSRF token is:
- **Unique** – Different for every user
- **Secret** – The attacker can't guess it
- **Unpredictable** – Randomly generated each time

---

## How it works in plain English:

### Without a CSRF token (vulnerable):
1. You log into your bank
2. Your browser gets a cookie (your ID badge)
3. Attacker tricks your browser into sending a "send money" request
4. Bank sees the cookie and says "OK, it's you" → Money stolen

### With a CSRF token (protected):
1. You log into your bank
2. Bank gives you a cookie **AND** puts a secret one-time code in the form
3. When you submit the form, you must send BOTH the cookie AND the secret code
4. Attacker can trick your browser into sending the cookie, but **cannot know the secret code** (it's hidden in the original page, which the attacker can't read)
5. Bank sees the request missing the correct code → Rejects it

---

## The example you shared (explained):

```html
<form name="change-email-form" action="/my-account/change-email" method="POST">
    <label>Email</label>
    <input type="email" name="email" value="example@normal-website.com">
    <input type="hidden" name="csrf" value="50FaWgdOhi9M9wyna8taR1k3ODOR8d6u">
    <button type='submit'>Update email</button>
</form>
```

**What's happening:**

| Part | Purpose |
|------|---------|
| `email` field | What you want to change your email to |
| `csrf` field (hidden) | The secret one-time token |
| `value="50FaWgd..."` | The actual secret value the server expects |

When you submit:
1. Your browser sends: `email=new@address.com` + `csrf=50FaWgdOhi9M9w...`
2. Server checks: Does the CSRF token match what I stored for this user?
3. If yes → Allow. If no → Reject.

---

## Why the attacker can't bypass it:

The attacker's malicious page can:
- ✅ Make your browser send the cookie (automatic)
- ❌ **Cannot** know the hidden CSRF token value (`50FaWgdOhi9M9w...`)

Why? Because the **Same Origin Policy** prevents the attacker's website from reading content on the bank's website. The token is only visible if you can see the original form page — which the attacker cannot.

---

## Simple analogy:

Imagine your bank account has:
- **Cookie** = Your house key (attacker can trick you into using it)
- **CSRF token** = A different numeric code displayed on a screen inside your house

To transfer money, you need BOTH:
1. The house key (cookie)
2. The current number on the screen (CSRF token)

The attacker can trick you into using your house key, but they can't see the number on the screen inside your house.

---

## Key takeaway:

> A CSRF token turns a single-factor request (cookie only) into a two-factor request (cookie + secret token). The attacker can only control one factor, so the attack fails.