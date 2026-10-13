# DOM XSS via jQuery `attr()` — Plain Notes

## The idea

Some sites use **jQuery** — a JavaScript library that simplifies DOM manipulation. jQuery has its own functions, and some of them are **sinks** for DOM XSS. If user-controlled data flows into one of these sinks, you can inject code.

The sink in this note: **`attr()`** — it changes an HTML element's attribute.

---

## The vulnerable pattern

```js
$(function() {
    $('#backLink').attr("href", (new URLSearchParams(window.location.search)).get('returnUrl'));
});
```

Line by line:

- `$(function() { ... })` → runs when the page loads
- `$('#backLink')` → finds the element with ID `backLink`
- `.attr("href", ...)` → sets its `href` attribute
- `URLSearchParams(...).get('returnUrl')` → pulls `returnUrl` from the URL

In plain English: *"Take `returnUrl` from the URL and set it as the back link's `href`."*

You control `returnUrl`. You control the `href`. That's the bug.

---

## Why `href` is dangerous

`href` accepts the **`javascript:` scheme** — just like `http:` or `mailto:`. When a browser sees `javascript:` in an `href`, it doesn't navigate — it **executes the code**.

Example:
```html
<a href="javascript:alert(1)">Click</a>
```
Click → alert fires.

So if `returnUrl` becomes `javascript:alert(document.domain)`, the back link becomes a JavaScript trigger.

---

## The attack

The URL has a parameter like:

```
?returnUrl=/home
```

Change it to:

```
?returnUrl=javascript:alert(document.domain)
```

Steps:
1. Page loads → jQuery sets back link's `href` to `javascript:alert(document.domain)`
2. **Click the back link**
3. Alert fires → XSS

Note: this one fires **on click**, not on page load.

---

## Why `document.domain`?

Either `alert(1)` or `alert(document.domain)` works. `document.domain` proves the code runs in the **site's own context** — which is what makes it real XSS, not just a random alert.

---

## Source → Sink (the DOM XSS formula)

- **Source** = where user input enters. Here: `location.search`.
- **Sink** = where input gets written dangerously. Here: jQuery's `.attr()`.

If user data flows from a source into a sink without sanitization → DOM XSS.

---

## Other jQuery sinks to watch for

| Function | Risk |
|---|---|
| `.attr("href", x)` | `javascript:` URL |
| `.html(x)` | Same as innerHTML — event handlers work |
| `$(x)` | HTML injection if `x` is raw |
| `.append(x)` / `.prepend(x)` | Injects elements |
| `.after(x)` / `.before(x)` | Same |

Rule: any of these with unsanitized user data = potential DOM XSS.

---

## Trigger styles to remember

- `javascript:` URL → fires **on click**
- `onerror` / `onload` → fires **on load**

Different payloads for different situations.

---

## Mental checklist for jQuery XSS

1. What jQuery sink is used? (`.attr`, `.html`, `$()`, etc.)
2. What user data flows into it? (check `location.search`, `location.hash`)
3. What context does it land in?
   - `href` / `src` → try `javascript:`
   - HTML content → try `<img onerror>` or `<svg onload>`
   - Attribute value → close the quote, add an event handler
4. Is the payload URL-safe? (no raw spaces, minimal quotes)

---

## The takeaway

> **jQuery's `.attr('href', userInput)` = `javascript:` URL injection.**

Whenever user data lands in an `href` or `src` (via jQuery or plain JS), reach for `javascript:alert(...)`.