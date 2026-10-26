# Timing Attack for TE.CL — Explained Simply

## The Trick: Starving the Back-End

Just like the CL.TE timing attack, the goal here is to make the **back-end server hang and wait** for data that will never arrive. 

Here is the exact payload:
```http
POST / HTTP/1.1
Host: vulnerable-website.com
Transfer-Encoding: chunked
Content-Length: 6

0

X
```

Here is how the two servers read it differently:

### 1. The Front-End (Uses `Transfer-Encoding`)
It ignores `Content-Length` and reads it as chunks:
* It sees `0` (chunk size = 0). 
* In chunked encoding, a `0` means **"End of request."**
* It says, **"Request done!"** and forwards only the headers and the `0\r\n\r\n` to the back-end. 
*(Notice that the `X` gets left behind!)*

### 2. The Back-End (Uses `Content-Length`)
It ignores `Transfer-Encoding` and looks at the header `Content-Length: 6`. 
It counts the bytes it received: `0`, `\r\n`, `\r\n` ... that's only **5 bytes**. 
It says, "Wait, I need 6 bytes! Where is the last byte?" It **sits there and waits** for more data to arrive.

### 3. The Result (The Time Delay)
Because the back-end is frozen waiting for that missing 6th byte, the connection is blocked. Any subsequent request will hang. You will observe a long delay (or a timeout) in Burp.

**This time delay is your proof that the TE.CL vulnerability exists.** 🎯

---

### 💡 The Quick Mental Model
* **Front-end:** "I hit a `0`, I'm done! Forwarding now."
* **Back-end:** "I'm supposed to get 6 bytes, but I only got 5. Where's the rest? *...waits forever...*" 
* **You:** "The page is taking forever to load. Smuggling confirmed!"