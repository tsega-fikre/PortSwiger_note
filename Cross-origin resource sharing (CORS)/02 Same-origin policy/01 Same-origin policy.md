# Same-Origin Policy (SOP)

---

## The core idea: websites should mind their own business

Imagine you're in a library with private study rooms.  
Each website is like a separate study room:

- `gmail.com` is Room A
- `your-bank.com` is Room B
- `evil-site.com` is Room C

The **Same-Origin Policy** is a rule that says:

> "A person in Room A cannot reach into Room B and steal papers. They also cannot look through Room B's window to read what's on the desk."

---

## What does "same origin" mean?

Two websites have the **same origin** if three things match:

1. **Protocol** (`http` vs `https`)
2. **Domain** (`google.com` vs `evil.com`)
3. **Port** (`:80` vs `:8080`)

Example:

| Website A | Website B | Same origin? |
|-----------|-----------|---------------|
| `https://bank.com` | `https://bank.com/login` | ✅ Yes (same domain) |
| `https://bank.com` | `http://bank.com` | ❌ No (different protocol) |
| `https://bank.com` | `https://evil.com` | ❌ No (different domain) |

---

## The key rule (most people get this wrong)

Here's the surprising part:

> The same-origin policy **allows** your bank to *send* a request to another website, but it **blocks** your bank from *reading* the response.

Wait, what?

---

## Let me explain with an example

You're on `evil-site.com`.  
Evil site tries to fetch your private emails from `gmail.com` using JavaScript:

```javascript
fetch('https://gmail.com/inbox')
```

What happens?

1. **Request is sent** ✅  
   Your browser actually sends the request to Gmail, including your Gmail cookies.

2. **Response is blocked** ❌  
   Gmail replies with your emails, but the browser says:  
   *"Nope, `evil-site.com` is not allowed to read this response."*

So the evil site sees nothing — just an error message.

---

## Why is this brilliant?

This design prevents **data theft** but still allows **basic interactions**.

For example:
- An ad on `news.com` can *send* a request to `ad-tracker.com` to count a view.  
  (It doesn't need to read the response.)
- But `news.com` cannot *read* your private Facebook messages, even if it sends a request for them.

---

## The one big exception

**Links and form submissions** are always allowed across origins.  
That's why CSRF (cross-site request forgery) works — you can *send* a "transfer $1000" request to your bank from a malicious site.  
The bank will process it because your cookie is sent along.  
But the malicious site can't *read* the "transfer successful" response.

---

## Plain English summary

| What happens | Allowed? |
|--------------|----------|
| Site A sends a request to Site B | ✅ Yes |
| Site A reads the response from Site B | ❌ No (unless Site B says OK via CORS) |
| Site A embeds an image from Site B | ✅ Yes (can't read the image data though) |
| Site A submits a form to Site B | ✅ Yes (this is how CSRF works) |

---

## The one-sentence takeaway

> The same-origin policy lets websites **talk to** other websites but not **eavesdrop on** their conversations.

It's like being able to knock on your neighbor's door (send a request) but not being allowed to open their mail when they slide it under the door (read the response).

That's why we need **CORS** — it's the permission slip that says: *"Actually, this specific website is allowed to open my mail."*