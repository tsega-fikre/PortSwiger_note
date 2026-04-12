Exactly — let’s walk through what that HTML actually does, step by step, in plain language.

---

**What the attacker’s page contains:**

1. **A hidden form**  
   - `action="https://vulnerable-website.com/email/change"` → The form is set to submit a request to the vulnerable website’s “change email” function.  
   - `method="POST"` → It sends the request as a POST (the same type the real website expects).  
   - `<input type="hidden" name="email" value="pwned@evil-user.net" />` → A hidden field that tells the website to change the email to `pwned@evil-user.net`. The user never sees this.

2. **An automatic submit script**  
   - `<script>document.forms[0].submit();</script>` → As soon as the page loads, the form submits itself immediately. No button to click, no warning.

---

**What happens when you (the victim) load that page:**

- Your browser loads the attacker’s HTML page.
- The script runs instantly.
- Your browser sends a POST request to `vulnerable-website.com/email/change` with `email=pwned@evil-user.net`.
- Because you’re still logged into `vulnerable-website.com`, your browser automatically attaches your valid session cookie to that request.
- The vulnerable website sees the cookie, thinks it’s you, and changes your email address to the attacker’s email.

---

**Why this is dangerous:**

- You never clicked anything.
- You never saw the form.
- The request came from *your* browser, with *your* session, so the website trusts it completely.
- Now the attacker can trigger a password reset, which will send the reset link to *their* email address (`pwned@evil-user.net`), giving them full control of your account.

---

**In short:**  
The attacker’s page is like a remote control that uses your browser to press buttons on another website without you knowing.