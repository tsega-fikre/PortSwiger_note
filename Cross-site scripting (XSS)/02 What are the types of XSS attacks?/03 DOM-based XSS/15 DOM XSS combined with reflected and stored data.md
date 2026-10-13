# Reflected DOM XSS — Explained Simply

## Quick recap of what DOM XSS is

Normally in DOM XSS:

- A script reads data from the URL
- Writes it somewhere dangerous

All in the browser. The server isn't involved in the dangerous part.

---

## But here's the twist

The **source** doesn't have to come directly from the URL bar. It can also come from the **server's response**.

Sometimes the server takes something from your request and **reflects it back** into the page's HTML.

That means the page now contains **your input**. If a script on the page then reads that input and puts it into a dangerous sink → **DOM XSS**.

That's the "reflected + DOM" combo.

---

## Two halves of the attack

### Half 1 — The server reflects your input

You visit:

```
https://site.com/?name=hello
```

The server responds with HTML containing:

```html
<p>Welcome, hello!</p>
```

Your input `hello` is now **inside the page**. That's the reflection.

**Where can the reflection land?**

- Inside a JavaScript string literal: `var user = "hello";`
- Inside an HTML attribute: `<input value="hello">`
- Inside a data element in the DOM
- Anywhere in the HTML

---

### Half 2 — A client-side script reads it and puts it in a sink

Now the page has JavaScript that reads that reflected value and does something dangerous with it.

The example from the note:

```js
eval('var data = "reflected string"');
```

Let's break it down:

- The server puts **your input** in place of `reflected string`
- The JavaScript runs `eval(...)` on the whole string
- `eval` executes anything as code

So if you inject clever text, the server reflects it, and `eval` runs it → XSS.

---

## Why this is dangerous: `eval` is a sink

**`eval()`** takes a string and **runs it as JavaScript**. It's one of the most dangerous sinks in existence. Any user input reaching `eval` is basically game over.

In the example:

```js
eval('var data = "reflected string"');
```

If the server reflects your input into `reflected string`, and you break out of the quotes, you can inject code.

Example — you submit:

```
"; alert(1); //
```

The server responds with:

```js
eval('var data = ""; alert(1); //"');
```

Now `eval` runs:
1. `var data = "";` — declares a variable
2. `alert(1);` — runs your code 💥
3. `//"');` — commented out, ignored

XSS achieved.

---

## The full picture

```
You → Request with payload
     ↓
Server reflects payload into HTML/JS response
     ↓
Browser receives page with your payload embedded
     ↓
Page's JS reads reflected value (source)
     ↓
Passes it to a dangerous sink (eval, innerHTML, etc.)
     ↓
Code executes → XSS
```

The **server is involved** (it does the reflecting), but the **dangerous part** happens in the browser (the sink). That's why it's called **reflected DOM XSS**.

---

## Compare the three types

| Type | Where's the source? | Where's the sink? | Who triggers? |
|---|---|---|---|
| **Stored XSS** | Database (persisted) | Server-rendered HTML | Anyone loading the page |
| **Reflected XSS** | URL parameter | Server-rendered HTML | Victim clicks a link |
| **DOM XSS** | URL (`location.hash`, etc.) | Client-side JS | Victim loads the page |
| **Reflected DOM XSS** | URL, but reflected by server into the page | Client-side JS | Victim loads the page |

Reflected DOM XSS is a **mix** — the server hands the payload to the page, and the page's JavaScript mishandles it.

---

## Why this matters for testing

If you only test for pure DOM XSS, you'll miss this kind. If you only test for classic reflected XSS, you'll also miss this kind.

**You have to look at both sides:**

1. Does the server reflect my input into the response?
2. Is there a JavaScript sink that reads that reflected value?

If both are true → reflected DOM XSS.

---

## Common dangerous sinks (again, for memory)

| Sink | Danger |
|---|---|
| `eval(x)` | Runs x as JS |
| `Function(x)` | Same as eval |
| `setTimeout(x, ...)` | Runs x as JS if x is a string |
| `innerHTML = x` | HTML injection |
| `document.write(x)` | HTML injection |
| `$(x)` (jQuery) | HTML injection if x looks like HTML |

---

## How to spot it in a lab

1. **Submit a marker** like `test123` into any input field or URL param
2. **Look at the response** — is `test123` reflected somewhere?
3. **Look at the JS** — does any script read that reflected value?
4. **Trace the flow** — does it end up in a sink like `eval`, `innerHTML`, etc.?
5. **Craft a payload** that breaks out of the surrounding context (quotes, tags) and injects your code

---

## The key insight

> **A reflected XSS becomes a DOM XSS if the sink is client-side.**

The server is just the **messenger**. The browser's JavaScript is the one that **pulls the trigger**.

And the injection context matters — if you're inside a JS string like `"reflected string"`, you escape with `"` and inject code. If you're inside HTML, you escape with `>` and add tags.

---

## TL;DR

- **Source**: server-reflected user input (from a URL param, form field, etc.)
- **Sink**: a client-side function that executes or renders it dangerously (`eval`, `innerHTML`, etc.)
- **Attack**: inject a payload that breaks out of its context and executes code
- **Why it's tricky**: it's not pure server-side, and it's not pure client-side — it's both

---

## Three-sentence summary

1. The server reflects your input into the page, and a client-side script reads that reflected value.
2. If the script passes it to a dangerous sink like `eval`, your payload executes.
3. This is **reflected DOM XSS** — the server reflects, the browser executes. 🎯