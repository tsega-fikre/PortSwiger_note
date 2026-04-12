Let me explain this in the simplest way possible — this is a common and important bypass technique.

---

## The vulnerability in plain English:

Some websites are **lazy with their security checks**.

They check for the CSRF token **only if** you use `POST` (the normal form submission method).  
But if you use `GET` (the method for just requesting a page), they **skip the check entirely**.

---

## What the developer intended:

- POST requests → "This is a form submission that changes data" → Check CSRF token
- GET requests → "This is just someone viewing a page" → No check needed (because GET shouldn't change data)

**The problem:**  
The developer forgot that GET requests **can** change data if the website allows it.

---

## How the attacker abuses this:

**Normal (protected) request using POST:**
```
POST /email/change
csrf=SECRET_TOKEN&email=new@address.com
```
✅ Server checks token → Attack fails because attacker doesn't know `SECRET_TOKEN`

**Attacker switches to GET:**
```
GET /email/change?email=pwned@evil-user.net
```
❌ Server sees GET and thinks "No need to check CSRF token" → Attack succeeds

---

## The attack delivery (self-contained):

Because it's now a GET request, the attacker can use the **simplest delivery method possible** — just an image tag:

```html
<img src="https://vulnerable-website.com/email/change?email=pwned@evil-user.net">
```

When the victim views a page containing this image, their browser:
1. Tries to load the "image"
2. Sends a GET request to that URL
3. Automatically includes their session cookie
4. Server skips CSRF validation (because it's GET)
5. Email address changes

---

## Why this works (the conditions):

| Condition | Status |
|-----------|--------|
| Application has a sensitive action | ✅ Change email |
| Action can be performed with GET | ✅ (bad design) |
| CSRF validation only on POST | ✅ (lazy coding) |
| Session cookie is sent automatically | ✅ |
| No unpredictable parameters needed | ✅ (email is controlled by attacker) |

---

## Real-world impact:

This mistake turns a **secure application** (with CSRF tokens) into a **vulnerable one** — because the developer created a back door (GET requests bypass the token check).

---

## Simple analogy:

Imagine a bank that checks your ID **only if you enter through the front door**:
- POST = Front door → Check ID (CSRF token)
- GET = Side door → No ID check

The attacker just walks through the side door with a stolen badge (session cookie). The bank never checks the CSRF token because "side door visitors don't need ID."

---

## Key takeaway for testers:

> **Always test both methods.** If a website protects POST requests with CSRF tokens, try changing the request to GET. If the action still works and the token check is skipped, you've found a vulnerability.

**How to test:**
1. Capture a POST request with a CSRF token
2. Change `POST` to `GET`
3. Move parameters to the URL (like `?email=hacker@evil.com`)
4. Remove the CSRF token entirely
5. Send the request — if it works, the site is vulnerable