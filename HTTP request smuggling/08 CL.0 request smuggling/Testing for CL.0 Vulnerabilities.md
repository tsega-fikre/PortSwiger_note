# Testing for CL.0 Vulnerabilities — Explained Simply

## The Goal
You suspect a server might have the CL.0 bug (the back-end ignores `Content-Length` and assumes the request ends immediately after the headers). How do you test for it without guessing blindly?

You use a **Two-Tab Trick** in Burp Repeater to see if the server gets confused.

---

## The "Two-Tab" Setup in Burp

To test for CL.0, you need to send two requests back-to-back over the same connection. Burp makes this easy:

1. **Tab 1 (The Attack):** You put a fake request in the body of a `POST`.
   ```http
   POST /vulnerable-endpoint HTTP/1.1
   Host: vulnerable-website.com
   Connection: keep-alive
   Content-Type: application/x-www-form-urlencoded
   Content-Length: 34

   GET /hopefully404 HTTP/1.1
   Foo: x
   ```
   *(Notice: `Content-Length: 34` correctly counts the bytes of `GET /hopefully404`)*

2. **Tab 2 (The Victim):** You put a normal request for the homepage.
   ```http
   GET / HTTP/1.1
   Host: vulnerable-website.com
   ```

3. **Group them:** Select both tabs, right-click, and choose **"Add to group"**.
4. **Send in sequence:** At the top of Burp, change the send mode from "Send" to **"Send group in sequence (single connection)"**. 
5. **Send the group.**

---

## Why this works (The Logic)

When you send them over a **single connection**:

### If the server is NOT vulnerable:
The front-end forwards the `POST` with the body. The back-end reads the `Content-Length: 34` and treats the body as normal data. It responds `200 OK`. Then it processes the `GET /` request and responds `200 OK`.
**Result:** Two `200 OK` responses.

### If the server IS vulnerable (CL.0):
The front-end forwards the `POST` and its body. The back-end ignores `Content-Length` and assumes the `POST` ended at the headers. It leaves the body (`GET /hopefully404...`) in its memory.

When Tab 2 arrives (`GET /`), the back-end glues it to the leftover trap:
`GET /hopefully404 HTTP/1.1\r\nFoo: xGET / HTTP/1.1`

The back-end processes the first line: `GET /hopefully404`. It looks for that page, fails, and returns a **`404 Not Found`**.
**Result:** A `200 OK` (for the POST) and a `404 Not Found` (for your normal request).

---

## 🚨 Where to look for CL.0 (Prime Candidates)

The text notes that CL.0 usually happens on endpoints that **aren't expecting a body at all**. Because they don't expect a body, their code just says, "Read the headers, process the request, ignore the rest."

Look for:
*   **Redirects:** Endpoints that just return a `301 Moved Permanently` (e.g., `/login`).
*   **Static files:** Endpoints serving images or CSS. They don't process form data.
*   **Error pages:** Endpoints that just return a 404.

If you send a `POST` to a static file or a redirect endpoint with a hidden request in the body, the back-end might just ignore the length and leave the trap for the next user.

### The One-Sentence Summary:
> **To test for CL.0, send a POST with a hidden request in the body, immediately followed by a normal GET on the same connection. If the normal GET returns a 404, the back-end ignored the length and the vulnerability is confirmed.** 🎯