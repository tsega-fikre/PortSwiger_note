## 🔥 DOM-Based XSS

**DOM-based XSS** happens when **client-side JavaScript** takes attacker-controlled data from a **source** and sends it to an unsafe **sink**, causing JavaScript execution.

### 🧠 Simple Flow

`Attacker Input → Source → JavaScript → Sink → XSS`

### 🔹 Source

A place where attacker-controlled data comes from, commonly the URL:

```text
window.location
```

For example, data from the URL query string or fragment.

### 🔹 Sink

A JavaScript function/property that can process the data unsafely, such as:

```javascript
eval()
innerHTML
```

### 🎯 Example idea

```text
URL → window.location → JavaScript → innerHTML → JavaScript Executes
```

**Key point:**  
Unlike Reflected/Stored XSS, DOM XSS can occur **entirely in the browser**, without the server needing to reflect or store the payload.