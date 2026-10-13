## 🔎 Testing JavaScript Execution Sinks

JavaScript execution sinks are harder to test because your input **may not appear in the DOM**.

Instead, use the **JavaScript debugger** to trace how your input moves from a source to a sink.

### 🧪 Steps

**1. Find the source**

For example:

```javascript
location
```

Use DevTools search:

`Ctrl + Shift + F`

to find where the source is used in the JavaScript.

**2. Set a breakpoint**

Place a breakpoint where the source is read.

**3. Follow the data flow**

Track the value as it moves through variables:

`Source → Variable → Variable → Sink`

**4. Find the sink**

Identify where the attacker-controlled value reaches a JavaScript execution sink.

**5. Inspect the value**

Use the debugger to see exactly what value reaches the sink.

**6. Test the input**

Modify your input and check whether you can cause unintended JavaScript execution.

### 🧠 Key Point

`Source → Trace JavaScript → Find Sink → Inspect Value → Test`

💡 Unlike HTML sinks, **you can't always find your input by searching the DOM**, so debugging the JavaScript data flow is essential.