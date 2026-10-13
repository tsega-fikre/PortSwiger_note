## 🕵️ DOM Invader

Finding DOM XSS manually can be **time-consuming**, especially when the application has large or minified JavaScript files.

**Burp's browser** includes **DOM Invader**, which helps automate much of the DOM XSS investigation.

### 🧠 Simple idea

`Complex JavaScript → DOM Invader → Identify Sources/Sinks → Find DOM XSS`

💡 **DOM Invader** helps you trace attacker-controlled data through client-side JavaScript, making DOM XSS testing much easier.