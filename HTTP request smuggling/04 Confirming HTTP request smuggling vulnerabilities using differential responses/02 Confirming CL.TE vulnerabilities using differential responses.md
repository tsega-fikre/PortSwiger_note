# Confirming CL.TE — Explained Simply

## The Goal
We want 100% proof that the attack worked. To do this, we are going to "poison" the connection so that the **next normal request** gets glued to a fake URL (`/404`). If the server returns a `404 Not Found` to a request that *should* have returned `200 OK`, we win.

---

## Step 1: The Attack Request
You send this to the server:
```http
POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 49
Transfer-Encoding: chunked

e
q=smuggling&x=
0

GET /404 HTTP/1.1
Foo: x
```

### Why this works (The Math):
* **The Front-End (CL):** It uses `Content-Length: 49`. It counts exactly 49 bytes (everything down to `Foo: x`) and thinks, "This is all one big body. Looks safe!" It forwards the whole thing to the back-end.
* **The Back-End (TE):** It uses `Transfer-Encoding`. It sees `e` (hex for 14 bytes), reads `q=smuggling&x=`, and then sees `0` (end of request). It stops reading.
* **The Leftover:** The back-end leaves `GET /404 HTTP/1.1\r\nFoo: x` sitting in its memory.

---

## Step 2: The Normal Request
Immediately after, you send a completely innocent search request:
```http
POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

q=smuggling
```

---

## Step 3: The Collision
Because the back-end had leftover data in its memory, it takes that leftover data and **glues** your normal request to the end of it:

```http
GET /404 HTTP/1.1
Foo: xPOST /search HTTP/1.1   <-- Look here!
Host: vulnerable-website.com
...
```

The back-end server reads this and says, "Okay, I have a `GET` request for the URL `/404`." 

*(The `Foo: xPOST...` part just becomes a broken header, which the server ignores).*

### The Result
You asked for `/search`, but the server gives you a **`404 Not Found`** because it was tricked into looking for `/404`. 

This glaring difference in the response is the **ultimate proof** that your smuggling attack was successful. 🎯
