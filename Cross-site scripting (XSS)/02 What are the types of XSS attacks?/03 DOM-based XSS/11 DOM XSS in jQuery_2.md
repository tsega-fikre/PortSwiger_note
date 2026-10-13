# jQuery `$()` Selector XSS — Explained Simply

## What `$()` does

In jQuery, `$(something)` means **"find this element on the page."** For example:

- `$('#backLink')` → find the element with ID `backLink`
- `$('.button')` → find all elements with class `button`
- `$('div')` → find all `<div>` elements

Usually you pass it a **selector** (like a CSS selector) and it finds matching elements.

---

## The problem

Here's the weird part: **`$()` can also create HTML.**

If you pass it something that looks like HTML, like:

```js
$('<img src=1 onerror=alert(1)>')
```

…jQuery **creates that element** and puts it into the DOM. And if that element has an `onerror`, it fires. 💥

So the same function that's supposed to "find" things can also be tricked into **creating** things — including malicious things.

That's the vulnerability.

---

## The classic vulnerable code

```js
$(window).on('hashchange', function() {
    var element = $(location.hash);
    element[0].scrollIntoView();
});
```

Let's translate:

- `$(window).on('hashchange', ...)` → "when the URL's `#` part changes, run this"
- `$(location.hash)` → take the `#` part of the URL and pass it to jQuery's `$()`
- `element[0].scrollIntoView()` → scroll to that element

So the page does: *"If the user changes the `#` in the URL, scroll to whatever element they named."*

Example: if the URL is `page.html#section2`, jQuery finds the element with ID `section2` and scrolls to it. Normal behavior.

But the input (`location.hash`) is **user-controlled**. If you put HTML in the hash, jQuery will **create that HTML** instead of just finding something.

---

## The attack idea

You set the hash to:

```
#<img src=1 onerror=alert(1)>
```

jQuery sees something that looks like HTML → creates an `<img>` element → the image fails to load → `onerror` fires → alert pops.

Simple in theory. **But there's a catch.**

---

## The catch — you need to trigger `hashchange`

The vulnerable code only runs when the `hashchange` event fires. That only happens when the hash **changes after the page has loaded**.

So if you just visit:

```
https://site.com/#<img src=1 onerror=alert(1)>
```

…nothing happens. Because the hash was already set when the page loaded. No **change** → no event → no XSS.

You need the hash to change **while the page is open**. And the lab usually wants this to happen **without the victim clicking anything**.

---

## The trick: use an iframe

An iframe is a page inside a page. The attacker's page loads the victim page inside an iframe, and can manipulate it.

```html
<iframe src="https://vulnerable-site.com#" 
        onload="this.src+='<img src=1 onerror=alert(1)>'">
```

Let's walk through it:

1. The iframe loads `https://vulnerable-site.com#` — an empty hash
2. When it finishes loading, `onload` fires
3. `this.src += '<img src=1 onerror=alert(1)>'` **appends** to the iframe's URL
4. The iframe's URL is now `https://vulnerable-site.com#<img src=1 onerror=alert(1)>`
5. The hash **changed** → `hashchange` fires inside the iframe
6. The vulnerable jQuery code runs → `$()` creates the img → `onerror` fires → alert

No user click needed. The whole thing triggers automatically.

---

## Why is `onload` important?

Without it, you'd have to change the hash **after** the iframe is already loaded. `onload` is the moment the iframe is ready — the perfect time to append the payload and trigger `hashchange`.

---

## Why did old jQuery allow this?

In older versions of jQuery, `$()` would happily parse HTML if the input **started with `<`** (like `<img>`). So passing `location.hash` directly into `$()` was dangerous.

Newer jQuery fixed the **`#` case specifically** — if the input starts with `#`, it's treated as a selector, not HTML.

**BUT** — the note at the end says something crucial:

> Even newer versions of jQuery can still be vulnerable via the `$()` selector sink, **provided you have full control over its input from a source that doesn't require a `#` prefix.**

Translation: if the code does `$(someVariable)` where `someVariable` comes from something **other than `location.hash`** (like a query parameter or a form field), and the value starts with `<`, newer jQuery will still parse it as HTML. The `#` fix only covers the hash case.

So the sink (`$()`) is still dangerous — it just depends on where the data comes from.

---

## Source → Sink recap

- **Source** = `location.hash` (the `#` part of the URL)
- **Sink** = jQuery's `$()` function
- **Trigger** = `hashchange` event

The attack = put HTML in the source, get jQuery to parse it in the sink, and trigger the event with an iframe.

---

## The mental model

| Thing | Purpose |
|---|---|
| `$('<img onerror=...>')` | jQuery **creates** the element (this is the bug) |
| `location.hash` | The attacker-controlled source |
| `hashchange` event | Only fires when the hash **changes** |
| `<iframe onload=...>` | Triggers the change automatically |

---

## Trigger styles to remember

| Technique | Fires when |
|---|---|
| `javascript:` in `href` | User clicks |
| `<img src=1 onerror=...>` | Page loads the image (fails) |
| `<svg onload=...>` | Element is inserted |
| `hashchange` + iframe | Automatically, no click |

---

## The takeaway

> **jQuery's `$()` doesn't just find elements — it can also create HTML. If you control its input, you control the page.**

And if the input comes from `location.hash`, you'll need a `hashchange` trigger — the iframe trick is the cleanest way to fire it automatically.

Even on modern jQuery, `$()` is dangerous when the input **doesn't start with `#`** — because jQuery falls back to HTML parsing.

---

## Three-sentence summary

1. Old (and sometimes new) jQuery code that passes user input into `$()` can be tricked into **creating HTML** — including malicious elements with `onerror`.
2. If the source is `location.hash`, you need to trigger a `hashchange` event — the iframe `onload` trick does this automatically.
3. Result: XSS with zero user interaction. 🎯