# Lab: Reflected XSS into Attribute with Angle Brackets HTML-Encoded

**Difficulty:** Apprentice  
**Status:** Solved  
**Category:** Reflected Cross-Site Scripting (XSS)

## Objective

Exploit the reflected XSS vulnerability in the blog search functionality by injecting an event handler that executes `alert(1)`, even though angle brackets are HTML-encoded.

## Lab Solving Steps

### Step 1: Intercept the Search Request

I entered a random alphanumeric string into the blog search box, intercepted the request using Burp Suite, and sent it to **Burp Repeater**.

### Step 2: Identify the Injection Point

I checked the response and noticed that my input was reflected inside a quoted HTML attribute. The application HTML-encoded angle brackets, but my input was still placed inside the attribute.

### Step 3: Inject the Payload

I replaced my search input with this payload:

```
"onmouseover="alert(1)
```

This payload escapes the original quoted attribute and injects an `onmouseover` event handler.

### Step 4: Trigger the XSS

I copied the generated URL and opened it in the browser. When I moved my mouse over the injected element, the JavaScript `alert(1)` executed.

**Boom — lab solved!**

## What I Learned

- Reflected XSS can occur inside HTML attributes, even when angle brackets are encoded.
    
- HTML encoding alone does not protect every injection context.
    
- Breaking out of a quoted attribute can allow an attacker to inject event handlers.
    
- The `onmouseover` event executes JavaScript when the mouse moves over an element.
    

## Remediation

- Use context-aware output encoding for HTML attributes.
    
- Avoid placing untrusted input directly into HTML markup.
    
- Use safe templating practices and appropriate output sanitization.
    
- Apply a Content Security Policy (CSP) as an additional layer of defense.
    

## Conclusion

I exploited the reflected XSS vulnerability by escaping a quoted HTML attribute and injecting an `onmouseover` event handler that executed `alert(1)`.

**Key takeaway:** Always encode untrusted data for its specific output context. Encoding angle brackets alone is not enough to prevent attribute-based XSS.