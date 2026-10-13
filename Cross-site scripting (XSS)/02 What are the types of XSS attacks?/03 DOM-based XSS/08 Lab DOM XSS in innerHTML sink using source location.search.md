![[Pasted image 20261006074231.png]]

# Steps
## Step 1
I open the lab and go to search functionality and i try to work payload:
```HTML
<script>aler('xxs')</script>
```
When the browser parses your input via `innerHTML`, it deliberately **ignores `<script>` tags**. The tag gets added to the DOM but **never executes**. This is a security feature baked into the HTML spec.
## Step 2
HTML elements have **event attributes**  things like `onerror`, `onload`, `onmouseover`, `onclick`. These are **attributes**, not `<script>` tags. And `innerHTML` **does** execute them when the triggering event fires. so i used payload:
```HTML
<img src=1 onerror=alert(1)>
```
![[Pasted image 20261006074630.png]]
![[Pasted image 20261006074650.png]]

-----------------------------------------------------------------------
# If You Need more knowledge 
----
## What the lab is doing

There's a blog with a **search box**. When you search, the page takes whatever you typed — which lives in the URL — and writes it into a `<div>` on the page using JavaScript.

The vulnerable code looks roughly like this:

```js
var searchTerm = new URLSearchParams(location.search).get('search');
document.getElementById('searchResults').innerHTML = searchTerm;
```

Translation in plain English: *"Grab the `search` parameter from the URL and stuff it into the page's HTML."*

That's the whole vulnerability. Your input becomes **real HTML** on the page.

## Why this is dangerous

`innerHTML` doesn't just set **text** — it sets **HTML markup**. So if you type:

```
<b>hello</b>
```

The browser actually renders **bold** hello. It treats your input as markup, not plain text.

That means if you type HTML tags, the browser builds them into the page. You control the page's structure. That's the door.

## The trap you need to remember

Your first instinct might be:

```html
<script>alert(1)</script>
```

**This will NOT work.** Here's why:

When the browser parses your input via `innerHTML`, it deliberately **ignores `<script>` tags**. The tag gets added to the DOM but **never executes**. This is a security feature baked into the HTML spec.

This is the #1 thing people get wrong on this lab. They inject a script tag, see nothing happen, and assume the injection failed. It didn't fail — the browser just refuses to run it.

So you need a **different execution vector**.

## The vector that works: event handlers

HTML elements have **event attributes** — things like `onerror`, `onload`, `onmouseover`, `onclick`. These are **attributes**, not `<script>` tags. And `innerHTML` **does** execute them when the triggering event fires.

The classic payload:

```html
<img src=1 onerror=alert(1)>
```

Step by step:
1. The browser sees an `<img>` tag with `src=1`
2. It tries to load an image from a file called "1"
3. That file doesn't exist → the load **fails**
4. The failure triggers the `onerror` event
5. `alert(1)` runs 💥

No `<script>` tag needed. The event handler does the job.

## Do you need to break out of anything?

**No.** In this lab, your input goes **directly into the `<div>`**. There's nothing surrounding it that traps you.

Compare this to the *select* lab (the stock checker), where your input was trapped inside `<select><option>...</option></select>`. There you needed `</select>` first to escape. Here, no escape needed — you just drop your payload in.

This is an important distinction to remember: **always check what context your input lands in.**

