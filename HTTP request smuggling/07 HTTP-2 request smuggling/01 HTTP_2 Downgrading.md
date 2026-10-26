# HTTP/2 Downgrading — Explained Simply

## The Setup: The Translator at the Front Desk

Imagine a fancy hotel (the website). 
*   **The Guests (Browsers):** They speak modern, fast HTTP/2.
*   **The Kitchen Staff (Back-End Server):** They are old-school and only understand HTTP/1.1.

To make things work, there is a **Receptionist (Front-End Server)**. The guest speaks HTTP/2, and the receptionist **translates** it into HTTP/1.1 so the kitchen can understand it.

**This translation process is called "HTTP/2 Downgrading."** 

## Why is this dangerous?

When the receptionist translates the message, she uses HTTP/1.1 rules (like `Content-Length` or `Transfer-Encoding`). 

If an attacker sends a specially crafted HTTP/2 request, they can trick the receptionist into writing a bad translation. 
*   She might write two `Content-Length` headers.
*   She might write a weird `Transfer-Encoding` header.

The receptionist thinks she did a great job, but when she hands the translated note to the kitchen, the kitchen gets confused. The kitchen stops reading early, leaves some text in its memory, and **that is how the smuggling attack happens.**

## 🎨 The "Artistic License" Note

The text mentions that HTTP/2 is a **binary protocol** (it looks like 1s and 0s on the wire). 

But looking at 1s and 0s is impossible for humans. So, tools like Burp Suite (and PortSwigger's tutorials) "translate" it back into readable text so you can see what's happening. 
*   They show it as plain text headers.
*   They add a colon `:` in front of special HTTP/2 headers (like `:method` or `:path`) so you know they aren't normal headers.
<img alt="HTTP/2 downgrading overview" src="https://portswigger.net/web-security/request-smuggling/advanced/images/http2-downgrading.jpg">
<img alt="Mapping an HTTP/2 request to an HTTP/1 request" src="https://portswigger.net/web-security/request-smuggling/advanced/images/http2-http1-mapping.jpg">

**The One-Sentence Summary:**
> **HTTP/2 Downgrading is when a modern server translates a fast HTTP/2 request into an old HTTP/1.1 request. Attackers abuse this translation step to sneak in bad formatting and smuggle requests.** 🎯