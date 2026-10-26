# Timing Attack for CL.TE — Explained Simply

## The Trick: Leaving the Server "On Read"

To find a CL.TE vulnerability, you need to prove that the **back-end server is waiting for something that will never arrive**. 

Here is the exact payload:
```http
POST / HTTP/1.1
Host: vulnerable-website.com
Transfer-Encoding: chunked
Content-Length: 4

1
A
X
```

Here is how the two servers read it differently:

### 1. The Front-End (Uses `Content-Length`)
It sees `Content-Length: 4`. It counts exactly 4 bytes:
`1`, `\r\n`, `A`, `\r\n` = 4 bytes. 
It says, **"Request done!"** and forwards those 4 bytes to the back-end. 
*(Notice that the `X` gets left behind!)*

### 2. The Back-End (Uses `Transfer-Encoding`)
It ignores `Content-Length` and reads it as chunks:
* It sees `1` (chunk size = 1 byte).
* It reads the data: `A`.
* Then it looks for the next chunk size... and **it sees nothing**. 

Because the front-end dropped the `X`, the back-end is left hanging. It thinks the request isn't finished, so it **sits there and waits** for the next chunk to arrive. 

### 3. The Result (The Time Delay)
Because the back-end is frozen waiting for data, the connection is blocked. If you send another request, it will hang. If you look at the response time in Burp, you will see it takes several seconds (or times out completely). 

**This time delay is your proof that the CL.TE vulnerability exists.** 🎯

---

### 💡 The Quick Mental Model
* **Front-end:** "I counted 4 bytes, I'm done. Here's the rest."
* **Back-end:** "I read 1 chunk. Where's the next one? *...waits forever...*" 
* **You:** "The page is taking 10 seconds to load. Smuggling confirmed!"