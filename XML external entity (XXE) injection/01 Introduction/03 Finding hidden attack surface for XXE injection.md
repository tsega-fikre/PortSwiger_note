# XInclude Attacks — Explained Simply

## The Problem This Solves

You've learned the classic XXE attack:

```xml
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<productId>&xxe;</productId>
```

**But that attack requires you to control the DOCTYPE.** What if you can't?

**Situation:** The application takes your input (say, a form field), and **inserts it into an XML document** on the server before parsing. You don't see the XML. You don't control the XML. You just provide **one small value** that ends up somewhere inside the XML.

**Example:**
- You submit a name: `John`
- The server builds: `<soap:Request><name>John</name></soap:Request>`
- The server sends it to a backend SOAP service
- The backend parses the XML

**You can't add a DOCTYPE.** You can't define `<!ENTITY>`.

But you **can** use **XInclude** instead.

---

## What Is XInclude?

**XInclude** is a **different part of the XML spec** that lets you build a document from **sub-documents**.

Think of it like HTML's `<iframe>` — but for XML:

```xml
<xi:include href="some-other-file.xml"/>
```

When the parser sees `<xi:include>`, it fetches the referenced document and **splices its contents** into the current XML, right where the tag was.

**Key difference from entities:**

| Feature | Where It's Used | Requires DOCTYPE? |
|---------|----------------|-------------------|
| **External entities (`<!ENTITY>`)** | Anywhere in XML | ✅ Yes — needs DOCTYPE |
| **XInclude (`<xi:include>`)** | Anywhere in XML | ❌ No — just needs the namespace |

**This is why XInclude is so useful:** it works **without a DOCTYPE**, which means it works even when you only control one small value inside the XML.

---

## 🎯 The Attack

### The Payload

```xml
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
<xi:include parse="text" href="file:///etc/passwd"/>
</foo>
```

**Breaking it down:**

| Part | Meaning |
|------|---------|
| `xmlns:xi="http://www.w3.org/2001/XInclude"` | Declares the XInclude namespace (required) |
| `<xi:include .../>` | The XInclude tag — tells the parser "fetch and include this" |
| `parse="text"` | Tells the parser to include the file as **plain text** (not as XML) |
| `href="file:///etc/passwd"` | The file to include |

**The `parse="text"` attribute is critical.** Without it, the parser would try to parse `/etc/passwd` as XML — which would fail. With `parse="text"`, the file contents are inserted as raw text.

---

## 🎬 How the Attack Works in Practice

### Scenario: A SOAP-Based Web App

You submit a value through a normal form:

```http
POST /api/userInfo HTTP/1.1
Host: victim.com
Content-Type: application/x-www-form-urlencoded

name=John
```

**What the server does behind the scenes:**

```xml
<?xml version="1.0"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <getUser>
      <name>John</name>       ← Your input goes here
    </getUser>
  </soap:Body>
</soap:Envelope>
```

The server then sends this to a backend service. The backend parses the XML.

**You can't see this XML. You can't add a DOCTYPE. You can only control `John`.**

### Your Attack

You submit this as your "name":

```xml
<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/passwd"/></foo>
```

**Now the server builds:**

```xml
<soap:Envelope ...>
  <soap:Body>
    <getUser>
      <name><foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/passwd"/></foo></name>
    </getUser>
  </soap:Body>
</soap:Envelope>
```

**When the parser hits `<xi:include>`:**
1. It fetches `/etc/passwd`
2. It inserts the file's contents as **plain text** where the tag was
3. The resulting XML now contains the file contents
4. If the response echoes this field back → **you see the file**

---

## 🧠 Why This Works

The key insight:

> **XInclude works anywhere in the XML, not just in the DOCTYPE. So you can inject it inside any data value.**

Compare:

| Attack | Where the payload goes | Requires DOCTYPE? |
|--------|-----------------------|-------------------|
| **XXE (entities)** | Inside the DOCTYPE block | ✅ Yes |
| **XInclude** | Inside any element or value | ❌ No |

Since you can only control a **single value** in the SOAP scenario, you can't add a DOCTYPE. But you **can** inject XInclude markup into the value itself.

**Analogy:**
- **XXE:** You need to edit the blueprint of the building
- **XInclude:** You only need to stick a post-it note inside one room — the room reads it and acts on it

---

## 🔍 When to Use XInclude vs. XXE

| Situation | Use This |
|-----------|----------|
| You control the **entire XML body** | XXE (with DOCTYPE) |
| You control **only one value**, and it's inserted into XML server-side | **XInclude** |
| DOCTYPE is **blocked** by a WAF | XInclude (or parameter entities) |
| The app parses XML with external entities **disabled** | Neither works — try SSRF or other attacks |

---

## 🛠️ The Attack in Burp — Step by Step

### Step 1: Identify the Attack Surface

Look for endpoints where:

- Your input is submitted as a **plain value** (form data, JSON, URL param)
- The **response echoes your input back**
- The app **might build XML on the backend** (look for SOAP, XML APIs, or responses containing `<soap:Envelope>` or `<xml>` tags)
- The app mentions "SOAP", "XML-RPC", or uses XML-like error messages

### Step 2: Test With a Simple Payload

Send your input as:

```xml
<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/passwd"/></foo>
```

### Step 3: Look for File Contents in the Response

If successful, you'll see:

```
Invalid name: root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
```

**Or you'll see an error that includes the file contents.**

### Step 4: Try Different Files

Once confirmed:

```xml
<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/hostname"/></foo>
<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/hosts"/></foo>
<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///proc/self/environ"/></foo>
```

### Step 5: Escalate

Try reading application config files, SSH keys, or environment files:

```xml
<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///var/www/html/config.php"/></foo>
```

---

## ⚠️ Common Gotchas

| Problem | Cause | Fix |
|---------|-------|-----|
| **Entity doesn't expand** | Parser doesn't support XInclude | Try classic XXE instead |
| **Nothing appears in response** | The response doesn't echo the input | This is blind XInclude — needs a different technique |
| **"Invalid XML" error** | Your payload broke the XML structure | Check for closing tags / quotes |
| **Empty response** | The parser tried to include as XML, not text | Make sure `parse="text"` is present |
| **The whole request fails** | Namespace URL is wrong | Use exactly `http://www.w3.org/2001/XInclude` |

---

## 🛡️ How to Defend Against XInclude

| Defense | Why It Works |
|---------|--------------|
| **Disable XInclude in the parser** | Most parsers have an option like `setXIncludeAware(false)` |
| **Disable external entity resolution** | `setFeature("http://apache.org/xml/features/disallow-doctype-decl", true)` |
| **Reject XML markup in user input** | Strip `<`, `>`, `&` from values that go into XML |
| **Prefer JSON / parameterized APIs** | Avoid XML entirely if possible |
| **Validate and sanitize user input** | Even if you build XML, escape special characters |

**The best fix:** Disable XInclude in the parser configuration. Most modern XML parsers have this option.

---

## Summary Table

| Concept | Plain English |
|---------|--------------|
| **XInclude** | An XML feature that includes other documents inline |
| **Why it's useful** | Works without DOCTYPE — injectable inside any value |
| **The payload** | `<xi:include parse="text" href="file://..."/>` |
| **Required namespace** | `xmlns:xi="http://www.w3.org/2001/XInclude"` |
| **The `parse="text"` trick** | Include the file as plain text, not XML |
| **When to use it** | When you control only a value inside server-built XML |
| **The goal** | Same as XXE — read local files, escalate to SSRF/RCE |
| **The fix** | Disable XInclude in the XML parser |

---

## The Big Takeaway

> **XInclude is XXE's stealthy cousin. It works when DOCTYPE doesn't, and it fits into spaces where entities can't.**

The pattern:

- You can't add a DOCTYPE
- You can only control one small value
- But that value **can contain XInclude markup**
- The server builds XML with your value inside
- The parser processes the XInclude → reads the file
- The response shows the file contents

**This is why attack surface analysis matters:** XXE isn't just about requests that "look like XML." It's about any input that ends up inside XML on the backend.

---

## 🎯 What's Coming Next

You've now covered:

1. ✅ Basic XXE file retrieval
2. ✅ XInclude attacks
3. ⏳ **Blind XXE** — when the response doesn't echo back
4. ⏳ **SSRF via XXE**
5. ⏳ **XXE via file uploads**

Ready for **blind XXE** — the trickiest but most rewarding version — or want to practice XInclude on a lab first?