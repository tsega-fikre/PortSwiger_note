# How to Actually Perform a Request Smuggling Attack — Simple Version

## The Core Move (All Types)

Every classic attack does the **same three things**:

1. Put **both** `Content-Length` and `Transfer-Encoding` in **one** request.
2. **Craft the body** so it "ends" at two different points.
3. **Exploit the gap** — the leftover bytes become a hidden request.

The only thing that changes between attack types is **which server trusts which header**.

## The Three Flavors

Think of it as **who believes what**:

| Type | Front-end trusts | Back-end trusts |
|------|-----------------|-----------------|
| **CL.TE** | `Content-Length` | `Transfer-Encoding` |
| **TE.CL** | `Transfer-Encoding` | `Content-Length` |
| **TE.TE** | Both *can* use TE | But one is tricked into **ignoring** it |

### CL.TE — "Front reads length, back reads chunks"
The front-end counts bytes and stops early. The back-end keeps reading chunks and finds a **second request hidden inside the first one's body**. The front-end never sees that second request as a request.

### TE.CL — "Front reads chunks, back reads length"
Reversed. The front-end reads chunks and stops early; the back-end counts bytes and **keeps going**, swallowing the start of the next request into the body — or leaving a hidden one queued.

### TE.TE — "Both speak chunked, but one can be fooled"
Both support `Transfer-Encoding`, so you **obfuscate the header** — write it in a way one server ignores but the other accepts. Now they disagree even though both "support" it. (This is the sneaky one — it's about **making one server skip the header**, not about the header being unsupported.)

## The Practical Rule of Thumb

- **CL.TE** → front-end counts **bytes**, back-end counts **chunks**
- **TE.CL** → front-end counts **chunks**, back-end counts **bytes**
- **TE.TE** → both *could* count chunks, but you **hide the header** from one of them

The hidden request always ends up **prepended to the next request** on the shared connection — that's the payload you exploit.

## ⚠️ The Big Practical Warning

> **These attacks only work over HTTP/1.**

Modern browsers — and **Burp Suite itself** — **default to HTTP/2** when the server advertises support during the TLS handshake. So if you just fire a request in Burp Repeater, it may go out as HTTP/2 and the smuggling trick **won't work**.

**The fix:** manually **downgrade to HTTP/1** in Burp:

> **Repeater → Inspector panel → Request attributes → switch protocol to HTTP/1**

Do this **before** testing, or you'll be chasing a bug that isn't there.

## One-Sentence Summary

> **Smuggling = send one request with both length headers, shaped so the front-end and back-end disagree on where it ends — and the type (CL.TE / TE.CL / TE.TE) just tells you which server trusts which header.** Then force HTTP/1 in Burp or the attack never fires.

## Quick Mental Model

```
CL.TE  →  front: "I counted 6 bytes."   back: "I'm still reading chunks… oh, a new request!"
TE.CL  →  front: "Chunks done."         back: "I counted 6 bytes… there's more here!"
TE.TE  →  front: "No TE header? Use CL." back: "TE header! Use chunks."
```

Same crack, three ways to open it. 🔓