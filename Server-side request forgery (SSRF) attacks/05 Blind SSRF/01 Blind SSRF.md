# **Blind SSRF** 
---

## First, let's remember what normal SSRF is

Imagine you have a website that lets you check the price of a product from another server.  
You give it a web address (URL), and it fetches data from that address and **shows you the result** on the screen.

That's like asking a friend: *"Go look inside that room and tell me what you see."*  
And your friend comes back and describes everything.

**Normal SSRF** = you see the response.

---

## Now, Blind SSRF

Blind SSRF is when you give the website a URL, it **goes and fetches it**, but **never tells you what it found**.

It's like asking a friend: *"Go knock on that door and come back."*  
Your friend goes, knocks, returns, but **says nothing** about who opened the door or what the room looked like.

You don't get any information back.

---

## So how do you even know it worked?

Since you can't **see** the response, you have to look for **indirect clues** (side effects).

Think of it like throwing a rock into a dark forest. You can't see where it lands, but you can listen for:

- A **thud** (hit something solid)
- A **splash** (hit water)
- A **scream** (hit someone)

Similarly, with blind SSRF, you watch for things like:

| Clue | What it means |
|------|----------------|
| The website loads slower than usual | Maybe your request reached a slow server |
| You get an error message | Maybe the server tried to connect and failed |
| The website crashes or behaves weirdly | Maybe your URL caused a problem |
| You see a log entry somewhere (if you have access) | Confirms the request happened |
| A file gets created or deleted | The server executed your request |

---

## Why is blind SSRF dangerous?

Even though you don't see the response, you can still **make the server do things**:

1. **Scan internal networks** – You can't see results, but you can measure **time differences** to guess if an internal server exists.
2. **Attack internal services** – You can send malicious requests to databases, caches, or admin panels that are hidden inside the company network.
3. **Trigger actions** – For example, make the server delete a user, restart a service, or send an email.
4. **Remote code execution** – If the internal service is vulnerable, your blind request might trigger a hidden command that runs on the server.

---

## Real-world simple example

Let's say a website has a "Contact Support" form.  
When you submit it, the website sends an internal request to a logging server to save your message.

You can't see the logging server's response. But you can change the URL to:

```
http://secret-admin-panel.local/delete?user=admin
```

The website's server will blindly send that request. If the admin panel is vulnerable, **admin gets deleted** – even though you never saw any response.

---

## How do hackers exploit blind SSRF?

They use **out-of-band techniques** (OOB):

- Make the server send a request to a **server they control** (e.g., `http://hacker.com/xxx`)
- Then watch their own server logs to see if a request arrived
- That confirms the blind SSRF works

They also use **timing attacks**:
- Try to connect to a closed port (fast response)
- Try to connect to an open port (slow response)
- The difference tells them if a service exists

---

## Summary table

| | Normal SSRF | Blind SSRF |
|--|-------------|-------------|
| You see the response? | Yes | No |
| Easy to exploit? | Yes | Harder |
| How to confirm it works? | Look at the response | Look for side effects (time, errors, crashes, external callbacks) |
| Can it be dangerous? | Yes | Yes – sometimes worse because people ignore it |

---

## Simple analogy to remember

**Normal SSRF** = You call a restaurant and ask what soups they have. They tell you.

**Blind SSRF** = You call a restaurant and ask them to scream into the kitchen. You can't hear the scream, but you see customers looking confused. That's your clue something happened.