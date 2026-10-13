## 🔎 How to Test DOM-Based XSS

DOM XSS testing focuses on finding how **attacker-controlled data flows from a source to an unsafe sink** in client-side JavaScript.

### 🧪 Basic Steps

**1. Use Browser DevTools**  
Open the page and inspect its JavaScript.

**2. Identify Sources**  
Test attacker-controlled sources such as:

```text
URL → window.location
```

Also check query strings, fragments, and other browser-controlled data.

**3. Test Each Source Individually**  
Place a unique test value in the source and observe how the page processes it.

**4. Trace the Data Flow**  
Follow the value through the JavaScript and check whether it reaches a dangerous sink such as `innerHTML` or `eval()`.

**5. Confirm Execution**  
If the data reaches an unsafe sink, test whether JavaScript actually executes in the browser.

### 🧠 Simple Workflow

`Source → JavaScript → Sink → Execution`

💡 **Burp Scanner** can automate much of the detection, but manual testing with browser DevTools is useful for understanding the actual source-to-sink flow.