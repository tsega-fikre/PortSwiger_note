Exactly — you’ve summarized the chain of events perfectly. Let me restate it in the simplest possible way, and then explain that important note at the end.

---

**What happens when the victim visits the attacker’s page:**

1. **Attacker’s page loads** → It automatically sends a hidden request to the vulnerable website (like changing your email).

2. **Browser adds the session cookie** → Because you’re still logged into that website, your browser says, “Oh, this request is going to `vulnerable-website.com` — here’s your ID badge (cookie)” and attaches it automatically.

3. **Vulnerable website trusts the request** → The website sees the cookie, thinks “This is definitely the real user,” and does whatever the request says — like changing your email to the attacker’s address.

No clicks. No warning. No way for you to stop it.

---

**The important note (in plain English):**

You might think CSRF only works with cookies — but it also works with **any automatic credential system**, including:

- **HTTP Basic Authentication**  
  If you’ve logged into a site using a pop-up username/password box, your browser might automatically send those credentials with every request to that site. An attacker could abuse that the same way.

- **Certificate-based authentication**  
  Some systems use digital certificates (like a smart card or file on your computer) to identify you. If your browser automatically uses that certificate when visiting certain sites, a CSRF attack could trick your browser into making requests using that certificate.

---

**The key idea behind CSRF in one sentence:**  
> If a website trusts *anything* that your browser automatically sends (cookies, credentials, certificates), an attacker can trick your browser into misusing that trust.