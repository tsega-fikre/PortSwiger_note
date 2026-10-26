
# Lab: Reflected XSS into a JavaScript String with Angle Brackets HTML-Encoded

**Difficulty:** Apprentice  
**Status:** Solved  
**Category:** Reflected Cross-Site Scripting (XSS)

## Objective

Exploit the reflected XSS vulnerability by breaking out of a JavaScript string and executing `alert(1)`, even though angle brackets are HTML-encoded.

## Lab Solving Steps

### Step 1: Find the Injection Point

I opened the lab and used the search functionality. The search query was reflected inside a JavaScript string.

### Step 2: Inject the Payload

I used the following payload in the search input:

```
';alert(1)//
```
![[Pasted image 20261015163445.png]]
### Step 3: Execute the Payload

I submitted the payload. The single quote escaped the JavaScript string, `alert(1)` executed, and `//` commented out the remaining JavaScript on that line.

**Boom — lab solved!**
![[Pasted image 20261015163456.png]]
## What I Learned

- Reflected XSS can occur inside JavaScript strings, not just HTML elements.
    
- HTML-encoding angle brackets does not prevent attacks in JavaScript contexts.
    
- A single quote can break out of a single-quoted JavaScript string when it is not properly escaped.
    
- The `//` sequence comments out the remaining JavaScript on the same line.
    

## Remediation

- Use context-aware output encoding for JavaScript contexts.
    
- Avoid inserting untrusted input directly into executable JavaScript.
    
- Safely serialize user-controlled data instead of building JavaScript strings through concatenation.
    
- Apply a Content Security Policy (CSP) as an additional security layer.
    

## Conclusion

I successfully exploited the reflected XSS vulnerability using `';alert(1)//` to escape the JavaScript string and execute JavaScript.

**Key takeaway:** Always consider the context where user input is reflected. HTML encoding alone is not sufficient when the input enters JavaScript code.



