![[Pasted image 20261006084500.png]]

# DOM-Based XSS via WebSocket URL Fragment

## Objective

The objective of this lab was to identify and exploit a DOM-based Cross-Site Scripting (XSS) vulnerability caused by unsafe handling of data from the URL fragment.

## What I Learned

- How to identify vulnerable JavaScript code using Burp Suite or browser DevTools.
    
- How URL fragments can be manipulated to inject malicious HTML.
    
- How an exploit server can be used to host and deliver an XSS payload.
    
- How an `iframe` and `onload` event can be used to modify the victim's URL and trigger JavaScript execution.
    

## Goal

The goal was to create an exploit that executes the `print()` function in the victim's browser and deliver it through the exploit server.

## Lab Working Steps

### 1. Identify the Vulnerable Code

I first inspected the application's home page using **Burp Suite** and the browser's **Developer Tools**.

While reviewing the JavaScript code, I identified functionality that processes data from the URL fragment (`#`) in an unsafe manner.

This indicated that the URL fragment could potentially be manipulated to inject malicious HTML and execute JavaScript.

### 2. Open the Exploit Server

From the lab banner, I opened the **Exploit Server**.

The exploit server provides a location where a malicious page can be hosted and delivered to the simulated victim.

### 3. Create the Malicious Payload

In the **Body** section of the exploit server, I inserted the following payload:

```html
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>
```

The payload creates an iframe pointing to the vulnerable application.

When the iframe loads, the `onload` handler appends an HTML `<img>` element to the URL fragment.

The injected image uses:

```html
onerror=print()
```

When the image fails to load, the `onerror` event executes `print()`.

### 4. Test the Exploit

I stored the exploit and selected **View exploit**.

The exploit successfully executed the `print()` function, confirming that the XSS payload worked.

**Screenshot suggestion:** Capture the browser showing the print dialog/function execution after clicking **View exploit**.

### 5. Deliver the Exploit to the Victim

After confirming that the exploit worked, I returned to the exploit server and clicked:

**Deliver to victim**

The victim accessed the malicious page, causing the payload to execute against the vulnerable application.

The lab was successfully solved.

## Vulnerability

The application was vulnerable to **DOM-based Cross-Site Scripting (XSS)** because client-side JavaScript processed attacker-controlled URL fragment data without properly sanitizing or encoding it before inserting it into the DOM.

## Impact

An attacker could potentially execute arbitrary JavaScript in the security context of the vulnerable application.

Depending on the application's functionality and browser protections, this could allow an attacker to:

- Perform actions as the victim.
    
- Access sensitive information available to JavaScript.
    
- Modify page content.
    
- Perform phishing attacks within the trusted application origin.
    

## Remediation

The application should avoid inserting untrusted URL data directly into dangerous DOM sinks.

Recommended protections include:

- Properly validate and sanitize user-controlled input.
    
- Use safe DOM APIs such as `textContent` instead of unsafe HTML insertion where possible.
    
- Avoid dangerous sinks such as `innerHTML` when processing untrusted data.
    
- Apply an appropriate Content Security Policy (CSP) as an additional defense-in-depth measure.
    

## Conclusion

I successfully identified and exploited a **DOM-based XSS vulnerability** by manipulating the URL fragment and using an iframe-based payload. The exploit was hosted on the PortSwigger exploit server and successfully delivered to the simulated victim, solving the lab.

![[Pasted image 20261006090058.png]]