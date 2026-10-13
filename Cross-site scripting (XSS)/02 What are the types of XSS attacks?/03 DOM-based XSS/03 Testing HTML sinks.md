## 🔎 Testing HTML Sinks for DOM XSS

When testing a DOM XSS **HTML sink**, the goal is to find where attacker-controlled input appears in the **live DOM**.

### 🧪 Steps

**1. Put a random value in the source**

For example:

```text
?test=x7Kp92Lm
```

**2. Inspect the live DOM**

Open **Developer Tools** and search for your value with:

`Ctrl + F`

⚠️ Don't rely on **View Source** — JavaScript may have modified the DOM after the page loaded.

**3. Identify the context**

Check where your input appears:

- HTML text
    
- Attribute
    
- Other HTML context
    

**4. Test how the input is processed**

For example, if your value appears inside:

```html
<input value="x7Kp92Lm">
```

you can test whether special characters are handled safely by introducing appropriate delimiter characters and observing whether the context changes.

### 🧠 Key Point

`Source → Live DOM → Find Context → Test Processing → Check for XSS`

💡 Also remember that browsers may URL-encode values from sources such as `location.search` and `location.hash`, which can affect whether a payload reaches the sink as intended.