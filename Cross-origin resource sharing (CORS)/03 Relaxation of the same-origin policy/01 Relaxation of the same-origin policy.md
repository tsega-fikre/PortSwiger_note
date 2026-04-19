# Relaxation of the same-origin policy


---

## The problem: same-origin policy is too strict

Remember the library study rooms analogy?  
The same-origin policy says: *"You cannot look at papers in another room."*

That's great for security, but what if you **legitimately need** to look?

Real-world examples:

- `maps.google.com` needs to read your location from `accounts.google.com`
- `my-blog.com` wants to show your recent tweets from `twitter.com`
- `company.com` has an API at `api.company.com` (different subdomain)

Without relaxing the policy, **none of this works**.

---

## The old, dangerous way to relax it

Before CORS, websites used hacks like:

- **JSONP** (injecting a `<script>` tag to sneak data in)
- **postMessage** (messy and error-prone)
- **Setting `document.domain`** (lowers security too much)

These worked but were like cutting a hole in the wall instead of using a door.  
Anybody could crawl through.

---

## The smart solution: CORS

**CORS (Cross-Origin Resource Sharing)** is a controlled, formal way to relax the same-origin policy.

Think of it as installing a **secure intercom system** between rooms:

> "Room A, you may open Room B's door, but only if Room B says it's OK."

---

## How CORS works (the handshake)

When `site-A.com` tries to fetch data from `site-B.com`, here's what happens:

### Step 1: The request
`site-A.com` asks for `https://site-B.com/api/data`

### Step 2: The browser checks with `site-B`
The browser adds an `Origin` header:
```
Origin: https://site-A.com
```

### Step 3: `site-B` responds with rules
`site-B` sends back headers like:
```
Access-Control-Allow-Origin: https://site-A.com
Access-Control-Allow-Credentials: true
```

### Step 4: Browser decides
- If `site-B` says "site-A is allowed" → browser shares the data ✅
- If not → browser blocks the response ❌

---

## The special pre-flight check (for dangerous requests)

For requests that could cause damage (like DELETE or PUT with custom headers), the browser first sends a **pre-flight** `OPTIONS` request:

> "Hey `site-B`, I'm about to send a DELETE request from `site-A`. Is that OK?"

Only if `site-B` says yes does the real request go through.

---

## What the CORS headers mean

| Header | What it does |
|--------|---------------|
| `Access-Control-Allow-Origin: *` | Anyone can access (dangerous with credentials) |
| `Access-Control-Allow-Origin: https://trusted.com` | Only `trusted.com` can access |
| `Access-Control-Allow-Credentials: true` | Allow cookies/auth to be sent |
| `Access-Control-Allow-Methods: GET, POST` | Which HTTP methods are allowed |

---

## Plain English analogy

**Without CORS:**  
Your apartment has a rule: *"No one from other apartments can see inside your living room."*  
Very safe, but you can't let your neighbor borrow a cup of sugar by looking at your kitchen.

**With CORS:**  
You install a peephole. You can look at who's knocking and say:  
*"Oh, it's my neighbor. I'll open the door just for them."*  
Everyone else still sees a closed door.

---

## The one-sentence takeaway

> The same-origin policy is like a strict "no entry" rule, but CORS is a polite handshake that lets websites say: *"I trust this specific other website — let them in."*

It's a controlled relaxation — not breaking down the walls, but installing secure doors with locks you control.