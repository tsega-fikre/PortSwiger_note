## 🔥 Stored Cross-Site Scripting (Stored XSS)

**Stored XSS** happens when an application **saves attacker-controlled input** and later displays that input to other users **without safely handling it**.

### 🧪 Simple Example

A blog allows users to submit comments:

```text
User → Comment → Server/Database → Blog Page
```

Normal comment:

```text
comment=Nice post!
```

Attacker submits:

```html
<script>alert(document.domain)</script>
```

The application stores it.

Later, another user opens the blog post:

```html
<p><script>alert(document.domain)</script></p>
```

💥 The browser executes the script.

### 🧠 Simple flow

`Attacker Input → Stored → Victim Opens Page → JavaScript Executes`

**Key difference:**  
**Reflected XSS** = reflected immediately from the request.  
**Stored XSS** = saved first, executed later.