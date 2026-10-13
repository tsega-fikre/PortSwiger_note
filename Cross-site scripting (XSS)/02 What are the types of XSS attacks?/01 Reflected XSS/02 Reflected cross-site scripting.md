### 🔥 Reflected XSS — Simple Explanation

**Reflected XSS** happens when an attacker puts a malicious payload into a **request**, and the website immediately sends that input back in its response **without safely encoding it**.

### Simple flow

```text
Attacker puts payload in URL
        ↓
Website receives it
        ↓
Website reflects it in the response
        ↓
Victim opens the URL
        ↓
Browser executes the JavaScript
```

### Example

Normal request:

```text
/status?message=Hello
```

Website responds:

```html
<p>Status: Hello</p>
```

But an attacker changes it to:

```text
/status?message=<script>alert(1)</script>
```

If the website reflects it unsafely:

```html
<p>Status: <script>alert(1)</script></p>
```

The victim's browser executes the script.

### 🧠 Key point

**Reflected XSS = payload comes in the request → gets reflected immediately in the response → executes in the victim's browser.**

Unlike **Stored XSS**, the payload usually isn't saved in the database; it is carried by the request itself.
