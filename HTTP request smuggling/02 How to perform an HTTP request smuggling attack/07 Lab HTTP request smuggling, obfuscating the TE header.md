# Lab: HTTP Request Smuggling — Obfuscating the TE Header

- **Platform:** PortSwigger Web Security Academy
    
- **Difficulty:** Practitioner
    
- **Vulnerability:** HTTP Request Smuggling (TE.TE)
    
- **Status:** Solved
    

## Objective

Smuggle a request to the back-end server so that the next request it processes appears to use the HTTP method `GPOST`. The vulnerability occurs because the front-end and back-end handle duplicate HTTP request headers differently.

## Step-by-Step Solution

### Step 1: Capture the Homepage Request

1. Open the lab in your browser.
    
2. Open Burp Suite and make sure the Proxy is configured.
    
3. Refresh the homepage to capture its HTTP request.
    
4. Right-click the captured request and select **Send to Repeater**.
    
5. Open the Repeater tab to modify and send the request manually.
    

### Step 2: Modify the HTTP Request

In Repeater:

1. Change the request method from `GET` to `POST`.
    
2. Change the HTTP version to `HTTP/1.1`.
    
3. Make sure the request contains the correct `Host` header for your lab.
    
4. Disable automatic Content-Length updates if necessary, because the payload requires precise request framing.
    

### Step 3: Add Duplicate Transfer-Encoding Headers

Add the following headers to the request:

```
Transfer-Encoding: chunked
Transfer-Encoding: hacker
```

The purpose is to exploit the different ways the front-end and back-end process duplicate headers. One server may use the valid `chunked` value, while the other may interpret the headers differently.

### Step 4: Add the Smuggling Payload

Place the following payload in the request body:

```
5c
GPOST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0
```

**Important:** Preserve the blank lines and line breaks. HTTP request smuggling depends on precise message boundaries, so avoid allowing Burp to automatically modify the payload.

### Step 5: Send the Request

1. Click **Send** in Repeater.
    
2. Examine the response.
    
3. Send the request a second time.
    
4. Observe whether the back-end processes the next request with the method `GPOST`.
    

The second request is important because the first request can leave bytes queued for the back-end to interpret when processing a subsequent request.

### Step 6: Verify the Result

If the request desynchronization works as intended, the back-end sees the next request as using the method `GPOST`, and the lab displays the **Solved** status.

## What I Learned

- **TE.TE** means both servers support `Transfer-Encoding`, but they handle an obfuscated or duplicated header differently.
    
- Duplicate HTTP headers can cause inconsistencies between front-end and back-end request parsing.
    
- Request smuggling exploits disagreements about where an HTTP request ends.
    
- Sending a follow-up request can reveal the effect of the desynchronization.
    
- Precise control of HTTP headers, body content, and line breaks is essential when testing request-smuggling vulnerabilities.
    

## Key Takeaway

The main idea is that two servers may interpret the same HTTP request differently. When their parsing behavior becomes inconsistent, an attacker may be able to smuggle a request to the back-end.

**Lab result: Solved successfully.**

![[Pasted image 20261021202018.png]]

![[Pasted image 20261021202033.png]]