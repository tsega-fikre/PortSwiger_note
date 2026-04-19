# Vulnerabilities arising from CORS configuration issues


---

## The setup: CORS is a permission slip

Remember: CORS lets a website say *"It's OK for this other website to read my data."*

It does this by sending back a header like:
```
Access-Control-Allow-Origin: https://trusted-site.com
```

That's like giving `trusted-site.com` a **key to your front door**.

---

## The problem: mistakes in who gets the key

When developers set up CORS, they sometimes make **configuration mistakes** that give the key to the wrong people — or to everyone.

These mistakes create **vulnerabilities** that attackers can exploit.

---

## Common CORS mistakes (the dangerous ones)

### Mistake #1: Reflecting any origin
Some websites take the `Origin` header from the request and copy it into the response.

**What happens:**
1. Attacker's site (`evil.com`) sends request with `Origin: https://evil.com`
2. Vulnerable site responds with `Access-Control-Allow-Origin: https://evil.com`
3. Browser says: *"Oh, evil.com is allowed? Here's the data!"*

**Analogy:**  
A security guard who asks "Who sent you?" and then writes that name on the access list. Anyone can say any name.

---

### Mistake #2: Using `null` origin
Some apps allow `Access-Control-Allow-Origin: null` for local files or special cases.

**Attack:**  
Attacker can trick the browser into making a request with `Origin: null` using a sandboxed iframe.

**Analogy:**  
A club that lets anyone in who says "I have no ID" — forgers love this.

---

### Mistake #3: Allowing `*` with credentials
If a site sets:
```
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

That's like saying: *"Everyone gets a key, and yes, you can use it to open my safe."*

Most browsers actually block this combination, but misconfigurations still happen.

---

### Mistake #4: Weak origin validation
An app tries to allow only `*.company.com` but does bad checking.

**Attack examples:**
- `evilcompany.com` → passes because it ends with `company.com` ❌
- `company.com.evil.net` → passes because it contains `company.com` ❌
- `company.com.attacker.com` → same problem ❌

**Analogy:**  
A bouncer who lets anyone in wearing a badge that *contains* the word "VIP" anywhere — even if it says "NOT VIP".

---

## What can an attacker do with these mistakes?

If an attacker can make your browser send a request to a vulnerable site with a spoofed `Origin` header, and the site reflects it:

1. Attacker's page (`evil.com`) runs JavaScript that fetches `https://vulnerable-bank.com/api/account`
2. Vulnerable bank sees `Origin: https://evil.com` and responds:
   ```
   Access-Control-Allow-Origin: https://evil.com
   Access-Control-Allow-Credentials: true
   ```
3. Browser sends along your bank cookies (because credentials allowed)
4. Browser sees the CORS header and **gives the response to evil.com**
5. Attacker reads your private account data — including balance, transactions, personal info

---

## Real-world impact

| Data stolen | Consequence |
|-------------|-------------|
| API keys | Attacker impersonates your app |
| User tokens | Attacker logs in as you |
| Personal data | Identity theft |
| CSRF tokens | Attacker can now craft forged requests |

---

## Why does this happen?

Developers often:
- Take shortcuts to make things work
- Don't fully understand CORS security boundaries
- Use wildcard `*` for convenience
- Write weak regex patterns for origin validation
- Forget that `Origin` header is controlled by the attacker

---

## Plain English summary

| CORS mistake | What it's like |
|--------------|----------------|
| Reflecting any origin | A guard who lets in anyone whose name you say |
| Allowing `null` origin | Letting people in who claim "I don't exist" |
| Weak domain checking | Accepting any badge that has your company name anywhere on it |
| Wildcard with credentials | Giving everyone a master key to the vault |

**The one-sentence takeaway:**

> CORS vulnerabilities happen when a website hands out "access keys" too freely — either to everyone, to anyone who asks, or based on checks that are easy to fool — allowing attackers to steal your private data from that site.