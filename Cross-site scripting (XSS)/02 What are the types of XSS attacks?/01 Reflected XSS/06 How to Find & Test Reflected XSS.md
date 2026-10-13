## 🔎 How to Find & Test Reflected XSS

The basic idea is to find **where your input is reflected** in the server's response, understand **where it appears**, and then test whether you can make the browser execute JavaScript.

### 🧪 Steps

**1. Test every entry point**  
Check URL parameters, request body, path, and relevant headers.

**2. Send a random value**  
Use something like:  
`x7Kp92Lm`

Then check whether this value appears in the response.

**3. Find the reflection context**  
See where your input appears:

- HTML text
    
- HTML attribute
    
- JavaScript
    
- Other contexts
    

**4. Test an XSS payload**  
Based on the context, send a suitable payload and check whether the application modifies or blocks it.

**5. Confirm in a browser**  
If it appears to work in Burp Repeater, test it in a real browser.

A simple confirmation payload is:

```html
<script>alert(document.domain)</script>
```

### 🧠 Remember

`Find Input → Find Reflection → Identify Context → Test Payload → Confirm Execution`

**Burp Repeater** is especially useful for manually testing and inspecting reflected input.