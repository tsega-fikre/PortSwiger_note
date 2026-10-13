# DOM XSS in AngularJS — Explained Simply

## First, what is AngularJS?

**AngularJS** is a JavaScript framework (made by Google, older than today's "Angular"). It lets developers write HTML with special **directives** — attributes that tell Angular to do stuff.

One key directive is **`ng-app`**.

---

## What `ng-app` does

When you put `ng-app` on an HTML element, you tell the browser:

> *"Hey Angular, take over this part of the page and process everything inside it."*

Example:

```html
<body ng-app>
    <p>Hello {{ 1 + 1 }}</p>
</body>
```

Angular sees the `{{ }}` and **evaluates it as JavaScript-ish expression**. The page shows:

```
Hello 2
```

That's the whole point of Angular — dynamic content inside `{{ }}`.

---

## Why this is a problem for XSS

Here's the dangerous part:

**Angular evaluates whatever is inside `{{ }}` as an expression.**

And you, the attacker, control some input that lands in the page.

So if you can get `{{ ... }}` into the page **inside an `ng-app` region**, Angular will run it. No `<script>` tag. No `onerror`. No angle brackets. Just curly braces.

That's why this XSS is sneaky — **most filters don't block `{{ }}`** because they're looking for `<script>` or `onerror`.

---

## The classic AngularJS XSS payload

```html
{{ constructor.constructor('alert(1)')() }}
```

Or in newer AngularJS versions:

```html
{{ $eval('alert(1)') }}
```

Or:

```html
{{ toString.constructor.prototype.toString=toString.constructor.prototype.call; ["a","alert(1)"].sort(toString.constructor); }}
```

The exact payload depends on the AngularJS version. But the idea is always the same: **abuse the `{{ }}` expression evaluator to run JavaScript**.

---

## The sandbox problem (why payloads are version-specific)

Older AngularJS had a **sandbox** — it tried to block dangerous things like accessing `window`, `constructor`, etc.

But researchers found escapes again and again. Each AngularJS version patched previous escapes but new ones were found. That's why:

- AngularJS **1.6+** removed the sandbox entirely (they gave up)
- Older versions had specific bypasses like `constructor.constructor`

So **the payload depends on which AngularJS version the site uses**. You have to check.

---

## How to figure out which payload to use

1. **View the page source.** Search for `ng-app`, `ng-version`, or `angular.js` script tags.
2. Look at the version number in the script URL, e.g., `angular.min.js?version=1.4.4`.
3. Match your payload to the version:
   - **1.0.x – 1.1.x** → simple payloads like `{{ alert(1) }}` might even work
   - **1.2.x – 1.5.x** → sandbox escapes needed
   - **1.6+** → sandbox removed, easiest to exploit

The **AngularJS XSS Cheat Sheet** (PortSwigger has one) lists which payloads work on which versions.

---

## Where does the payload go?

Two common places:

### 1. Directly in the HTML
If the app puts your input inside an `ng-app` region:

```html
<div ng-app>
    Your search: {{ userInput }}
</div>
```

You submit: `{{ constructor.constructor('alert(1)')() }}`
Angular runs it → alert ✅

### 2. Inside an attribute
Angular also processes `{{ }}` inside HTML attributes:

```html
<div ng-app>
    <p title="{{ userInput }}">Hover me</p>
</div>
```

Even in an attribute, Angular evaluates the `{{ }}`.

---

## Why this is called "XSS without angle brackets"

Compare:

| Payload type | Uses `<script>`? | Uses `onerror`? | Angular-blocked? |
|---|---|---|---|
| Classic innerHTML XSS | ✅ | ✅ | Filtered easily |
| AngularJS XSS | ❌ | ❌ | Not filtered — just `{{ }}` |

So filters that strip `<` `>` or `on*` attributes **won't catch it**. That's the whole point of this category.

---

## What to look for in a lab

1. **Does the page use AngularJS?** → Look for `ng-app` in the HTML, or AngularJS in script tags
2. **Where does my input land?** → Inside the `ng-app` region? Then `{{ }}` is executable
3. **Which AngularJS version?** → Check the script URL for the version number
4. **Pick the matching payload** from a cheat sheet (like PortSwigger's)

---

## Example lab scenario

The lab has a search box. When you search, your input is reflected into the page **and** the page has `ng-app`. You submit:

```
{{ constructor.constructor('alert(1)')() }}
```

Angular evaluates it → alert pops → lab solved.

---

## The mental model

| Framework | Executes what? | How to trigger XSS |
|---|---|---|
| Plain HTML | `<script>`, event handlers | `<img onerror>`, `<script>` |
| jQuery `.attr('href')` | `javascript:` URLs | `?returnUrl=javascript:...` |
| AngularJS `ng-app` | `{{ }}` expressions | `{{ payload }}` |

Each framework has its own **execution syntax**. Learn the syntax → know the payload.

---

## The takeaway

> **AngularJS + `ng-app` + user input inside `{{ }}` = XSS without needing any HTML tags.**

Just double curly braces. No angle brackets. No event handlers. The framework itself runs the code.

Payload depends on AngularJS version — check the version, then use the matching escape.

---

## Three-sentence summary

1. AngularJS processes `{{ }}` expressions inside any element marked `ng-app`.
2. If user input lands inside that region, you can inject `{{ payload }}` and Angular will execute it.
3. The exact payload depends on the AngularJS version — older versions had a sandbox that requires specific escapes, newer versions (1.6+) are trivial. 🎯