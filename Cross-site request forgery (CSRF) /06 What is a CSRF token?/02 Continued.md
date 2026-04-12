

## The example (stepping through it):

**The form you saw earlier:**

```html
<input type="hidden" name="csrf" value="50FaWgdOhi9M9wyna8taR1k3ODOR8d6u">
<input type="email" name="email" value="example@normal-website.com">
```

**When you submit, your browser sends:**

```
POST /my-account/change-email HTTP/1.1
Host: normal-website.com

csrf=50FaWgdOhi9M9wyna8taR1k3ODOR8d6u&email=example@normal-website.com
```

**What the server does:**
1. Gets the CSRF token (`50FaWgdOhi9M9w...`)
2. Compares it with the one it stored when it gave you the form
3. If they match → "OK, this request is legitimate"
4. If they don't match → "Reject — this might be a CSRF attack"

---

## Why this stops the attacker:

The attacker wants to send:

```
POST /my-account/change-email HTTP/1.1
Host: normal-website.com

csrf=???&email=hacker@evil.com
```

But they don't know `???`. They can't guess it (it's random and long). They can't read it from the bank's website (Same Origin Policy blocks them). So they can't make a valid request.

**Result:** The server rejects the attack.

---

## The important note (explained simply):

> *"CSRF tokens don't have to be sent as hidden parameters in a POST request."*

**What this means:**

| Where the token can be placed | Example |
|------------------------------|---------|
| Hidden form field (most common) | `<input type="hidden" name="csrf" value="...">` |
| Custom HTTP header | `X-CSRF-Token: 50FaWgdOhi9M9w...` |
| JSON body property | `{"csrf": "50FaWgd...", "email": "..."}` |
| URL query parameter (less secure) | `/change-email?csrf=50FaWgd...&email=...` |

**Why it matters where you put the token:**

- **In a custom header** → Very secure because:
  - Cross-site requests can't easily add custom headers (requires `XMLHttpRequest` + CORS)
  - The attacker's simple `<form>` trick won't work

- **In a URL parameter** → Less secure because:
  - URLs appear in browser history, logs, Referer headers
  - Could leak the token accidentally

---

## Bottom line:

> A CSRF token is a secret handshake between the server and your browser. The attacker can't guess the handshake, so they can't fake a valid request — no matter whether the token is in a form field, a header, or elsewhere.

The **defense works** as long as:
1. The token is **unpredictable** (random)
2. The token is **tied to your session**
3. The attacker **cannot read** the token from another site