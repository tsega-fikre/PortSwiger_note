### Reflected XSS in different contexts

The important idea is:

> **Where your input appears in the HTTP response determines how you need to exploit it.**

For example, imagine the application reflects your input:

```text
https://example.com/search?q=YOUR_INPUT
```

But the server could put `YOUR_INPUT` into different places.

#### 1. Inside HTML

```html
<p>Search results for: YOUR_INPUT</p>
```

Your input is in the **HTML context**.

You would need a payload that makes sense inside HTML.

---

#### 2. Inside an HTML attribute

```html
<input value="YOUR_INPUT">
```

Now your input is inside an **attribute context**.

A payload that works in normal HTML may not work here because you're already inside:

```html
value="..."
```

You may need to escape the attribute context first.

---

#### 3. Inside JavaScript

```html
<script>
    var search = 'YOUR_INPUT';
</script>
```

Now you're inside a **JavaScript context**.

The payload needs to break out of the JavaScript string/context rather than the HTML context.

---

### Why validation matters

The application might also modify your input before reflecting it.

For example:

```text
You submit:
<script>alert(1)</script>

Application reflects:
&lt;script&gt;alert(1)&lt;/script&gt;
```

The application has **HTML-encoded** the characters, so the original payload may no longer execute.

Or it might remove particular characters/strings.

Therefore, when testing reflected XSS, you need to determine:

```text
Where is my input reflected?
            ↓
What context is it in?
            ↓
Is my input modified/encoded?
            ↓
What payload fits that context?
```

### 🧠 Simple rule

**Context determines payload.**

Don't start by blindly trying random XSS payloads. First identify **where your input lands in the response and how the application processes it**.