# Common defences against CSRF

---

## 1. CSRF Tokens (Most Effective)

**What it is:**  
A secret, one-time code that the server gives your browser when you load a page (like a form). When you submit the form, you must send that code back.

**How it stops CSRF:**  
- The attacker's page doesn't know your secret token (it's different for each user and each session)
- When the fake request arrives without the token (or with a wrong one), the server rejects it

**Analogy:**  
Like a bank requiring both your ID badge (cookie) AND a one-time PIN from a text message. The attacker can steal your badge but not your PIN.

**Why it works:**  
The attacker can't guess or steal the token because of the **Same Origin Policy** (they can't read your browser's memory or the page content from another site).

---

## 2. SameSite Cookies (Browser Defense)

**What it is:**  
A setting on cookies that tells your browser: *"Only send this cookie if the request comes from the same website, not from another site."*

**Three levels:**
- `SameSite=Lax` (default since 2021 in Chrome) → Cookie sent only for top-level navigation (clicking a link) but NOT for POST requests from other sites
- `SameSite=Strict` → Cookie never sent for cross-site requests (most secure)
- `SameSite=None` → Cookie always sent (most vulnerable)

**How it stops CSRF:**  
If the bank's session cookie has `SameSite=Lax`, your browser won't attach it when the attacker's page tries to submit a POST request. Without the cookie, the server doesn't recognize you, so the attack fails.

**Analogy:**  
Your ID badge has a rule: *"Only works if you enter through the front door (direct visit). Doesn't work if someone else brings you in through a side window."*

---

## 3. Referer-based Validation (Least Effective)

**What it is:**  
The server checks the `Referer` header (which says *"where did this request come from?"*).

**How it tries to stop CSRF:**  
The server only allows requests where `Referer` matches its own domain (e.g., `bank.com`). Requests from `evil.com` get blocked.

**Why it's weaker:**  
- Some browsers don't always send the Referer header
- Referer can sometimes be stripped or spoofed
- Attackers can sometimes bypass it using `<meta>` tags or other tricks

**Analogy:**  
Like a bouncer asking *"Who sent you?"* — but if you arrive without a referral card (missing header) or with a fake one, they might still let you in.

---

## Comparison Table:

| Defense | How it works | Strength | Bypass difficulty |
|---------|--------------|----------|-------------------|
| **CSRF Token** | Secret value in each request | Very strong | Very hard (requires XSS) |
| **SameSite Cookie** | Browser restricts cookie sending | Strong (Lax/Strict) | Depends on browser settings |
| **Referer Validation** | Checks where request came from | Weak | Often possible |

---

## Bottom line for attackers (and defenders):

- **CSRF tokens** → Hardest to bypass. You'd need another vulnerability (like XSS) to steal the token.
- **SameSite cookies** → Modern browsers make this a strong defense. You might need to find GET-based actions or misconfigured `SameSite=None`.
- **Referer validation** → Often bypassable. Look for missing Referer headers or weak checks (e.g., only checks if string *contains* domain).