<div align="center">

```
┌──(tsega㉿cyberlord)-[~/portswigger]
└─$ cat mission.txt
```

# 👻 PortSwigger Web Security Academy — Lab Journal

### Documenting every exploit, every bypass, every broken authentication — one lab at a time.

<br>

[![Labs Solved](https://img.shields.io/badge/Labs%20Solved-65-22d3ee?style=for-the-badge&logo=hackthebox&logoColor=white)](#-solved-labs)
[![Categories](https://img.shields.io/badge/Categories%20Active-10-3fb950?style=for-the-badge&logo=target&logoColor=white)](#-categories-covered)
[![Last Updated](https://img.shields.io/badge/Last%20Updated-Oct%202026-0891b2?style=for-the-badge&logo=clockify&logoColor=white)](#)

<br>

**[🌐 Portfolio](https://nilephantom.netlify.app/) · [💻 GitHub](https://github.com/tsega-fikre) · [💼 LinkedIn](https://www.linkedin.com/in/tsegazeab-fikre-g-yesus-640452395) · [📡 Telegram](https://t.me/tsega0001)**

</div>

---

## `>_` whoami

```
┌──(tsega㉿cyberlord)-[~]
└─$ whoami

  Name     : Tsegazeab Fikre
  Role     : Cybersecurity Enthusiast & Web Penetration Tester
  Focus    : Ethical Hacking · Web Exploitation · AppSec
  Platform : PortSwigger Web Security Academy
  Tools    : Burp Suite · DevTools · HTTP Proxy · Kali Linux
```

This repo is my **live lab notebook** through [PortSwigger Web Security Academy](https://portswigger.net/web-security) — the same curriculum trusted by real AppSec engineers and pentesters worldwide.

Every lab was broken manually: no walkthroughs copy-pasted, no auto-solvers, no shortcuts. Just reading the app, forming a hypothesis, and exploiting it. The methodology mirrors exactly what professional penetration testers do in the field — and it feeds directly into my [portfolio](https://nilephantom.netlify.app/) and CTF work.

---

## `>_` categories covered

> Only categories with actual notes or lab files in this repo are listed here.

| Category | Folder | Labs | Status |
|---|---|:---:|:---:|
| SQL injection | `SQL injection/` | 15 | 🟡 In progress |
| Server-side request forgery (SSRF) | `Server-side request forgery (SSRF) attacks/` | 5 | ✅ Done |
| File upload vulnerabilities | `File upload vulnerabilities/` | 6 | ✅ Done |
| Path traversal | `Path traversal/` | 6 | ✅ Done |
| Authentication vulnerabilities | `Authentication vulnerabilities/` | 8 | ✅ Done |
| Cross-site request forgery (CSRF) | `Cross-site request forgery (CSRF)/` | 7 | 🟡 In progress |
| Cross-origin resource sharing (CORS) | `Cross-origin resource sharing (CORS)/` | 3 | ✅ Done |
| JWT attacks | `JWT attack/` | 4 | 🟡 In progress |
| Cross-site scripting (XSS) | `Cross-site scripting (XSS)/` | 11 | 🟡 In progress |
| API testing | `API testing/` | — | 📖 Notes only |
| HTTP request smuggling | `HTTP request smuggling/` | — | 📖 Notes only |
| XXE injection | `XML external entity (XXE) injection/` | — | 📖 Notes only |

---

## `>_` topic notes index

Beyond labs, this repo contains structured notes on every topic. Here's what's documented:

### 🔷 SQL Injection
```
SQL injection/
├── 01 labs/                          → 16 lab walkthroughs (SQLi basics → UNION attacks)
├── 02 Examining the database/        → DB type/version, listing contents
├── 03 Blind SQL injection/           → Conditional responses technique
├── 04 Error-based SQL injection/     → Conditional errors, visible error-based
├── 05 Time delays/                   → Blind SQLi via time delays
├── 06 Out-of-band (OAST) techniques/ → Async exfiltration
├── 07 SQLi in different contexts/    → Non-standard injection points
├── 08 Second Order SQL injection/    → Stored/second-order injection
├── 09 How to prevent SQL injection/
└── sql Cheat sheet.md                → Quick reference
```

### 🔷 SSRF
```
Server-side request forgery (SSRF) attacks/
├── 01 What is SSRF?/
├── 02 Impact of SSRF attacks/
├── 03 Common SSRF attacks/           → Lab_1 (local server), Lab_2 (back-end system)
├── 04 Circumventing SSRF defenses/   → Lab_3 (blacklist filter), Lab_4 (open redirect)
├── 05 Blind SSRF/                    → Lab_5 (out-of-band detection)
└── Finding hidden attack surface/    → Partial URLs, data formats, Referer header
```

### 🔷 File Upload Vulnerabilities
```
File upload vulnerabilities/
├── 01 Impact/
├── 02 How vuln arises/
├── 03 How web servers handle static files/
├── 04 Unrestricted web shell upload/ → Lab_1
├── 05 Flawed validation/             → Lab_2 (Content-Type bypass)
├── 06 Preventing execution/
├── 07 Insufficient blacklisting/     → Lab_3 (path traversal), Lab_4 (blacklist bypass),
│                                       Lab_5 (obfuscated extension)
├── 08 Flawed content validation/     → Lab_6 (polyglot web shell)
├── 09 Race conditions/
├── 10 Without remote code execution/
├── 11 Uploading via PUT/
└── 12 How to prevent/
```

### 🔷 Path Traversal
```
Path traversal/
├── 01 Reading arbitrary files/       → Lab_1 (simple case)
├── 02 Common obstacles/              → Lab_2–6 (absolute path, stripped sequences,
│                                       URL-decode, start of path, null byte)
└── 03 How to prevent/
```

### 🔷 Authentication Vulnerabilities
```
Authentication vulnerabilities/
├── 01 Intro/                         → What is auth, auth vs authorization
├── 02 Intro to its vuln/             → How vulns arise, impact
├── 03 Vulnerabilities in password-based login/
│       → Lab: Broken brute-force protection, IP block
│       → Lab: Broken brute-force protection, multiple credentials per request
├── 04 Vulnerabilities in multi-factor authentication/
│       → Lab: 2FA simple bypass
│       → Lab: 2FA bypass using a brute-force attack
└── 05 Vulnerabilities in Other Authentication Mechanisms/
        → Lab: Brute-forcing a stay-logged-in cookie
        → Lab: Offline password cracking
        → Lab: Password reset poisoning via middleware
        → Lab: Password brute-force via password change
```

### 🔷 CSRF
```
Cross-site request forgery (CSRF)/
├── 01–04  What is CSRF / How it works / Constructing / Delivering
│           → Lab_1: CSRF vulnerability with no defenses
├── 05     Common defences
├── 06     CSRF tokens
├── 07     Flaws in token validation
│           → Lab_2: Token depends on request method
│           → Lab_3: Token depends on token being present
│           → Lab_4: Token not tied to user session
│           → Lab_5: Token tied to non-session cookie
│           → Lab_6: Token duplicated in cookie
├── 08–10  SameSite cookie restrictions (Strict / Lax / None)
├── 11     Bypass SameSite Lax via GET request → Lab_7
├── 12     Bypass via on-site gadgets           → Lab_8 (client-side redirect)
└── 13     Bypass with newly issued cookies     → notes only
```

### 🔷 CORS
```
Cross-origin resource sharing (CORS)/
├── 01 What is CORS?
├── 02 Same-origin policy
├── 03 Relaxation of same-origin policy
├── 04 Vulnerabilities from CORS config issues
│       → Lab_1: Basic origin reflection
│       → Lab_2: Trusted null origin
│       → Lab_3: Trusted insecure protocols
├── 05 How to prevent CORS attacks
└── TIP.md
```

### 🔷 JWT Attacks
```
JWT attack/
├── 01 What is JSON Web Tokens (JWTs)?  → Format, signature, JWS vs JWE
├── 02 Exploiting flawed signature verification
│       → Lab_1: Unverified signature
│       → Lab_2: alg:none — no signature
├── 03 Brute-forcing secret keys        → Lab_3: Weak key via hashcat
└── 04 JWT header parameter injections  → Lab_4: jwk embedded key injection
```

### 🔷 Cross-site Scripting (XSS)
```
Cross-site scripting (XSS)/
├── 01 What is Cross-site scripting/
├── 02 What are the types of XSS attacks?/
│   ├── 01 Reflected XSS/
│   │       → Lab: Reflected XSS into HTML context with nothing encoded
│   ├── 02 Stored XSS/
│   │       → Lab: Stored XSS into HTML context with nothing encoded
│   ├── 03 DOM-based XSS/
│   │       → Lab: DOM XSS in document.write sink using source location.search
│   │       → Lab: DOM XSS in document.write sink inside a select element
│   │       → Lab: DOM XSS in innerHTML sink using source location.search
│   │       → Lab: DOM XSS in jQuery anchor href attribute sink
│   │       → Lab: DOM XSS in jQuery selector sink using hashchange event
│   │       → Lab: DOM XSS in AngularJS expression (angle brackets + HTML-encoded)
│   │       → Lab: Reflected DOM XSS
│   ├── 04 Exploiting XSS/
│   │       → Lab: Exploiting XSS to steal cookies
│   └── 05 Dangling markup injection/
│           → Lab: Reflected XSS with strict CSP + dangling markup attack
```

### 🔷 API Testing *(notes only)*
```
API testing/
├── 01 API testing/   → Intro, API recon
└── 02 API documentation/ → Reading docs, discovering undocumented endpoints
```

### 🔷 HTTP Request Smuggling *(notes only)*
```
HTTP request smuggling/
└── 01 Intro/         → Intro notes
```

### 🔷 XXE Injection *(notes only)*
```
XML external entity (XXE) injection/   → Folder exists, notes coming soon
```

---

## `>_` solved labs

| No | Date | Category | Lab Title | Difficulty |
|:---:|---|---|---|:---:|
| 1 | 2026-06-10 | SSRF | Basic SSRF against the local server | Apprentice |
| 2 | 2026-06-10 | SSRF | Basic SSRF against another back-end system | Apprentice |
| 3 | 2026-06-11 | SSRF | SSRF with blacklist-based input filter | Practitioner |
| 4 | 2026-06-11 | SSRF | SSRF with filter bypass via open redirection vulnerability | Practitioner |
| 5 | 2026-06-11 | SSRF | Blind SSRF with out-of-band detection | Practitioner |
| 6 | 2026-06-16 | SQL injection | SQLi in WHERE clause — retrieval of hidden data | Apprentice |
| 7 | 2026-06-16 | SQL injection | SQLi vulnerability allowing login bypass | Apprentice |
| 8 | 2026-06-16 | SQL injection | UNION attack — determining the number of columns | Practitioner |
| 9 | 2026-06-16 | SQL injection | UNION attack — finding a column containing text | Practitioner |
| 10 | 2026-06-16 | SQL injection | UNION attack — retrieving data from other tables | Practitioner |
| 11 | 2026-06-16 | SQL injection | UNION attack — retrieving multiple values in a single column | Practitioner |
| 12 | 2026-06-17 | SQL injection | Querying the database type and version (MySQL/MS-SQL) | Practitioner |
| 13 | 2026-06-17 | SQL injection | Listing the database contents on non-Oracle databases | Practitioner |
| 14 | 2026-06-19 | Path traversal | File path traversal, simple case | Apprentice |
| 15 | 2026-06-19 | Path traversal | Traversal sequences blocked with absolute path bypass | Practitioner |
| 16 | 2026-06-19 | Path traversal | Traversal sequences stripped non-recursively | Practitioner |
| 17 | 2026-06-19 | Path traversal | Traversal sequences stripped with superfluous URL-decode | Practitioner |
| 18 | 2026-06-19 | Path traversal | Validation of start of path | Practitioner |
| 19 | 2026-06-19 | Path traversal | Validation of file extension with null byte bypass | Practitioner |
| 20 | 2026-06-20 | SQL injection | Blind SQLi with conditional responses | Practitioner |
| 21 | 2026-06-20 | SQL injection | Blind SQLi with conditional errors | Practitioner |
| 22 | 2026-06-20 | SQL injection | Visible error-based SQL injection | Practitioner |
| 23 | 2026-06-21 | SQL injection | Blind SQLi with time delays and information retrieval | Practitioner |
| 24 | 2026-06-22 | SQL injection | SQLi with filter bypass via XML encoding | Practitioner |
| 25 | 2026-06-23 | File upload | Remote code execution via web shell upload | Apprentice |
| 26 | 2026-06-23 | File upload | Web shell upload via Content-Type restriction bypass | Apprentice |
| 27 | 2026-06-23 | File upload | Web shell upload via path traversal | Practitioner |
| 28 | 2026-06-23 | File upload | Web shell upload via extension blacklist bypass | Practitioner |
| 29 | 2026-06-24 | File upload | Web shell upload via obfuscated file extension | Practitioner |
| 30 | 2026-06-24 | File upload | Remote code execution via polyglot web shell upload | Practitioner |
| 31 | 2026-08-03 | SQL injection | SQL injection — querying DB type and version on Oracle | Practitioner |
| 32 | 2026-08-04 | CSRF | CSRF vulnerability with no defenses | Practitioner |
| 33 | 2026-08-04 | CSRF | CSRF where token validation depends on request method | Practitioner |
| 34 | 2026-08-04 | CSRF | CSRF where token validation depends on token being present | Practitioner |
| 35 | 2026-08-05 | CSRF | CSRF where token is not tied to user session | Practitioner |
| 36 | 2026-08-05 | CSRF | CSRF where token is tied to a non-session cookie | Practitioner |
| 37 | 2026-08-05 | CSRF | CSRF where token is simply duplicated in a cookie | Practitioner |
| 38 | 2026-08-06 | CSRF | SameSite Lax bypass via GET request | Practitioner |
| 39 | 2026-08-06 | CSRF | SameSite Strict bypass via client-side redirect | Practitioner |
| 40 | 2026-08-07 | CORS | CORS vulnerability with basic origin reflection | Apprentice |
| 41 | 2026-08-07 | CORS | CORS vulnerability with trusted null origin | Practitioner |
| 42 | 2026-08-07 | CORS | CORS vulnerability with trusted insecure protocols | Practitioner |
| 43 | 2026-08-08 | JWT | JWT authentication bypass via unverified signature | Apprentice |
| 44 | 2026-08-08 | JWT | JWT authentication bypass via flawed signature verification (alg:none) | Apprentice |
| 45 | 2026-08-08 | JWT | JWT authentication bypass via weak signing key | Practitioner |
| 46 | 2026-08-09 | JWT | JWT authentication bypass via jwk header injection | Practitioner |
| 47 | 2026-10-03 | Authentication | Broken brute-force protection, IP block | Practitioner |
| 48 | 2026-10-03 | Authentication | 2FA simple bypass | Apprentice |
| 49 | 2026-10-06 | Authentication | Brute-forcing a stay-logged-in cookie | Practitioner |
| 50 | 2026-10-06 | Authentication | Offline password cracking | Practitioner |
| 51 | 2026-10-06 | Authentication | Password reset poisoning via middleware | Practitioner |
| 52 | 2026-10-06 | Authentication | Password brute-force via password change | Practitioner |
| 53 | 2026-10-07 | Authentication | Broken brute-force protection, multiple credentials per request | Expert |
| 54 | 2026-10-08 | Authentication | 2FA bypass using a brute-force attack | Expert |
| 55 | 2026-10-12 | XSS | Reflected XSS into HTML context with nothing encoded | Apprentice |
| 56 | 2026-10-12 | XSS | Stored XSS into HTML context with nothing encoded | Apprentice |
| 57 | 2026-10-12 | XSS | DOM XSS in document.write sink using source location.search | Apprentice |
| 58 | 2026-10-12 | XSS | DOM XSS in document.write sink inside a select element | Practitioner |
| 59 | 2026-10-12 | XSS | DOM XSS in innerHTML sink using source location.search | Apprentice |
| 60 | 2026-10-12 | XSS | DOM XSS in jQuery anchor href attribute sink | Apprentice |
| 61 | 2026-10-13 | XSS | DOM XSS in jQuery selector sink using a hashchange event | Apprentice |
| 62 | 2026-10-13 | XSS | DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded | Apprentice |
| 63 | 2026-10-13 | XSS | Reflected DOM XSS | Practitioner |
| 64 | 2026-10-13 | XSS | Exploiting XSS to steal cookies | Practitioner |
| 65 | 2026-10-13 | XSS | Reflected XSS with strict CSP + dangling markup attack | Expert |

---

## `>_` tools of the trade

<div align="center">

![Burp Suite](https://img.shields.io/badge/Burp%20Suite-FF6633?style=for-the-badge&logo=burpsuite&logoColor=white)
![Kali Linux](https://img.shields.io/badge/Kali%20Linux-268BEE?style=for-the-badge&logo=kalilinux&logoColor=white)
![Browser DevTools](https://img.shields.io/badge/DevTools-4285F4?style=for-the-badge&logo=googlechrome&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Hashcat](https://img.shields.io/badge/Hashcat-black?style=for-the-badge&logo=kalilinux&logoColor=white)

</div>

---

## `>_` next targets

```
┌──(tsega㉿cyberlord)-[~/portswigger]
└─$ cat objectives.txt

  [ ] Finish XSS track                 (password capture, CSRF bypass via XSS)
  [ ] Finish CSRF SameSite bypass labs (newly issued cookies)
  [ ] Complete JWT header injections   (jku, kid parameter attacks)
  [ ] Start API testing labs           (mass assignment, SSPP)
  [ ] Start XXE injection track
  [ ] Start HTTP request smuggling track
  [ ] Push toward first Expert lab
```

---

## `>_` connect

<div align="center">

[![Portfolio](https://img.shields.io/badge/Portfolio-nilephantom-22d3ee?style=for-the-badge&logo=firefox&logoColor=white)](https://nilephantom.netlify.app/)
[![GitHub](https://img.shields.io/badge/GitHub-tsega--fikre-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/tsega-fikre)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Tsegazeab%20Fikre-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/tsegazeab-fikre-g-yesus-640452395)
[![Telegram](https://img.shields.io/badge/Telegram-tsega0001-26A5E4?style=for-the-badge&logo=telegram&logoColor=white)](https://t.me/tsega0001)

</div>

---

<div align="center">
<sub>

```
┌──(tsega㉿cyberlord)-[~/portswigger]
└─$ echo "Built lab by lab. No walkthroughs. No shortcuts. Just pure exploitation."
```

</sub>
</div>
