Perfect — let’s walk through that exact example in plain language.

---

**The normal way you change your email:**

1. You go to `vulnerable-website.com` and log in.
2. You go to the “change email” page and type in your new email address.
3. Your browser sends this request to the website:

```
POST /email/change
email = wiener@normal-user.com
```

And it also automatically includes your session cookie (your ID badge).

The website sees the cookie, knows it’s you, and changes the email address.

---

**How the attacker abuses this with CSRF:**

The attacker wants to change your email to *their* email address (`hacker@evil.com`).

1. The attacker creates a fake webpage or email with a hidden form that looks like this (invisible to you):

```html
<form action="https://vulnerable-website.com/email/change" method="POST">
    <input type="hidden" name="email" value="hacker@evil.com">
</form>
<script>document.forms[0].submit();</script>
```

2. They trick you into visiting that page while you’re still logged into the vulnerable website.
3. Your browser loads the page, runs the script, and sends the request:

```
POST /email/change
email = hacker@evil.com
Cookie: session=yvthwsztyeQkAPzeQ5gHgTvlyxHfsAfE
```

4. The website sees your valid session cookie, thinks *you* requested the email change, and updates your account to `hacker@evil.com`.

---

**Why this works (matching your earlier conditions):**

| Condition | How it applies here |
|-----------|----------------------|
| **Relevant action** | Changing email is useful to the attacker — now they can request a password reset and take over your account. |
| **Cookie-based session only** | The website only checks the session cookie. No extra security like a one-time token or CAPTCHA. |
| **No unpredictable parameters** | The only parameter needed is `email=hacker@evil.com`. The attacker knows exactly what to put there. |

---

**The result:**  
You never clicked “change email,” but the attacker silently changed it. Then they reset your password, lock you out, and control your account.