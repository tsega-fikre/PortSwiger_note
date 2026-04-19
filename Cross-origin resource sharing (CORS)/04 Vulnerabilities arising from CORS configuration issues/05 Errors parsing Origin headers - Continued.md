# Errors parsing Origin headers - Continued


---

## The Big Picture (Analogy First)

Imagine you work at a company called **Normal-Website Inc.** You have a **VIP lounge** that only certain people can enter.

You tell the security guard:  
*"Let in anyone whose badge ends with `@normal-website.com`."*

**Mistake #1 (suffix matching)**:  
An attacker makes a badge that says `@hackersnormal-website.com`.  
The guard looks at the end: `normal-website.com` ✅ → "Come in!"

But that's NOT your domain — it just **ends with** your domain name.

---

**Mistake #2 (prefix matching)**:  
You change the rule: *"Let in anyone whose badge starts with `normal-website.com`."*

Attacker makes a badge: `normal-website.com.evil.net`.  
Guard looks at the start: `normal-website.com` ✅ → "Come in!"

Again, NOT your domain — it just **starts with** your domain name.

---

## Real Technical Example #1: Suffix Matching (Ends With)

### The vulnerable server's broken rule:
> "Allow any domain that **ends with** `normal-website.com`"

So these would be allowed:
- `api.normal-website.com` ✅ (subdomain — fine, it's yours)
- `blog.normal-website.com` ✅ (also yours)
- `normal-website.com` ✅ (main domain)

### How the attacker exploits it:

**Step 1:** Attacker registers this domain:  
`hackersnormal-website.com`  
(Notice: no dot before `normal-website.com` — it's one word)

**Step 2:** Attacker hosts malicious page at `https://hackersnormal-website.com/attack.html`

**Step 3:** Victim visits that page while logged into `normal-website.com`

**Step 4:** Malicious script tries to fetch data from `https://normal-website.com/api/private-data`

**Step 5:** Browser sends request with `Origin: https://hackersnormal-website.com`

**Step 6:** Server checks: "Does `hackersnormal-website.com` **end with** `normal-website.com`?"  
Let's see:  
`hackersnormal-website.com` → last 19 characters?  
`normal-website.com` is 19 chars. Does it match?  

`hackersnormal-website.com`
`................normal-website.com` ← Yes, the end matches!

**Step 7:** Server reflects:  
`Access-Control-Allow-Origin: https://hackersnormal-website.com`

**Step 8:** Browser says: "The page from `hackersnormal-website.com` is allowed to read the response from `normal-website.com`" → **Data stolen**

### Why this is confusing:
`hackersnormal-website.com` is a **completely different domain** from `normal-website.com`.  
But the broken "ends with" check treats it as valid because the last part looks the same.

---

## Real Technical Example #2: Prefix Matching (Starts With)

### The vulnerable server's broken rule:
> "Allow any domain that **starts with** `normal-website.com`"

So these would be allowed:
- `normal-website.com` ✅ (main domain)
- `normal-website.com.api.com` ❓ (this is dangerous)

### How the attacker exploits it:

**Step 1:** Attacker registers this domain:  
`normal-website.com.evil-user.net`  
(Note: It literally starts with `normal-website.com` followed by a dot)

**Step 2:** Attacker hosts page at `https://normal-website.com.evil-user.net/attack.html`

**Step 3:** Victim visits while logged into `normal-website.com`

**Step 4:** Malicious script requests `https://normal-website.com/api/private-data`

**Step 5:** Browser sends `Origin: https://normal-website.com.evil-user.net`

**Step 6:** Server checks: "Does `normal-website.com.evil-user.net` **start with** `normal-website.com`?"  
Yes:  
`normal-website.com.evil-user.net`  
`normal-website.com` ← matches at the beginning!

**Step 7:** Server reflects the attacker's origin → Access granted → Data stolen

### Why this is confusing:
`normal-website.com.evil-user.net` is **NOT** owned by Normal-Website Inc.  
Anyone can register `normal-website.com.evil-user.net` because the dot separates it.  
It's like registering `google.com.microsoft.com` — Microsoft doesn't own that.

---

## Side-by-Side Comparison (Very Clear)

| **Mistake Type** | **Server's Broken Rule** | **Attacker's Domain** | **Why It Works** |
|------------------|--------------------------|----------------------|-------------------|
| Suffix (ends with) | Domain must end with `normal-website.com` | `hackersnormal-website.com` | Ends with `normal-website.com` (no dot needed) |
| Prefix (starts with) | Domain must start with `normal-website.com` | `normal-website.com.evil.net` | Starts with `normal-website.com` |

---

## What the Server *Should* Have Done

**Correct rule:**  
> "Allow **exactly** `normal-website.com` and any domain that is a **subdomain** (has a dot before the name)."

So allowed:
- `normal-website.com` ✅
- `api.normal-website.com` ✅ (subdomain — note the dot before normal)
- `blog.normal-website.com` ✅

Not allowed:
- `hackersnormal-website.com` ❌ (no dot before `normal-website.com`)
- `normal-website.com.evil.net` ❌ (not a subdomain of `normal-website.com`)

### Correct implementation (pseudocode):
```python
def is_allowed(origin):
    # Remove https://
    domain = origin.replace("https://", "").replace("http://", "")
    
    # Check exact match
    if domain == "normal-website.com":
        return True
    
    # Check subdomain (ends with .normal-website.com)
    if domain.endswith(".normal-website.com"):
        return True
    
    return False
```

---

## Real-World Analogy to Remember Forever

**Bad suffix rule (ends with):**  
"I'll let anyone whose last name ends with 'Smith'."  
Attacker changes name to "FakeSmith" → allowed. ❌

**Bad prefix rule (starts with):**  
"I'll let anyone whose first name starts with 'Dr.'"  
Attacker becomes "Dr. Evil" → allowed. ❌

**Correct rule:**  
"I'll let only people on this exact list: John Smith, Jane Smith, and anyone with 'Smith' after a space in their last name."  
Attacker "FakeSmith" → denied. ✅  
Attacker "Dr. Evil" → denied. ✅

---

## The One-Sentence Takeaway

> **Matching by "starts with" or "ends with" lets attackers create domains that look like yours but aren't — always match exact domain names or proper subdomains with a dot.**

If you remember nothing else:  
`hackersnormal-website.com` ≠ `normal-website.com`  
`normal-website.com.evil.net` ≠ `normal-website.com`