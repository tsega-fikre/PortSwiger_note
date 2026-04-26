You're absolutely right — let me clarify and expand on that, because it's an important nuance.

You stated it well:

> *"The impact of blind SSRF is often lower than fully informed SSRF because of their one-way nature."*

That's true in many cases, but let me break down **when it's lower impact** vs. **when it's still critical**.

---

## First, why is blind SSRF usually *less* impactful than normal SSRF?

| Normal SSRF | Blind SSRF |
|-------------|-------------|
| You can read data (e.g., `/admin/users.json`) | You cannot read data back |
| You can steal API keys, passwords, source code | You cannot see what you fetched |
| You can map internal networks by reading responses | You have to guess or infer |

**Example:**  
With normal SSRF, you can fetch `http://internal-db:8080/secrets.txt` and see the secrets.  
With blind SSRF, you can fetch the same file but **never see its contents**.

So yes — **lower impact for data theft**.

---

## But blind SSRF can still be *very* dangerous in these scenarios

### 1. You can still *write* or *change* things

Even if you can't read, you can often:

- Delete a user: `http://admin-panel/delete?username=carlos`
- Restart a service: `http://internal-api/shutdown`
- Create a new admin user: `http://internal-api/create?user=hacker&role=admin`
- Clear a database: `http://internal-db/wipe`

**That's critical impact** — you're changing state, not just reading.

---

### 2. You can trigger Remote Code Execution (RCE)

If an internal service is vulnerable to a protocol like:

- **gopher://** – can send raw TCP commands to Redis, Memcached, etc.
- **dict://** – can probe services
- **file://** – can read files (if blind but side-channel exists)

Example:  
If internal Redis is reachable, you can send:

```
gopher://internal-redis:6379/_*2%0d%0a$4%0d%0asave%0d%0a
```

That triggers Redis to save a backup file — not reading, but **executing a command**.

If you can chain that with file writes to a web directory → **RCE**.

---

### 3. You can port-scan internally using timing

Even though you can't see responses, you can measure:

- **Fast response** → port closed/filtered
- **Slow response** or timeout → port open

With enough requests, you can map the entire internal network.

---

### 4. You can attack internal services that have *side effects*

Some services respond differently based on input, and those differences leak through:

- **Error messages** – If you send bad data, maybe the app crashes or logs an error you can see
- **Email triggers** – Make the server send an email to you
- **DNS interactions** – Use a domain you control, see if a DNS lookup happens

These turn *blind* into *semi-blind*.

---

## Comparison table: Impact levels

| Action | Normal SSRF | Blind SSRF |
|--------|-------------|-------------|
| Read secrets from internal server | ✅ High impact | ❌ Usually not possible |
| Delete a user | ✅ High impact | ✅ Same high impact |
| Trigger RCE | ✅ High impact | ✅ Same high impact |
| Port scan internal network | ✅ Easy and fast | ⚠️ Possible but slow (timing) |
| Steal source code | ✅ Yes | ❌ No |
| Attack via side-channels | Not needed | ✅ Sometimes works |

---

## Real-world blind SSRF → RCE example

A famous case: **Internal Redis + gopher://**

1. Blind SSRF can reach `gopher://internal-redis:6379/`
2. Send Redis commands to write an SSH key or a web shell
3. Now you have **remote code execution**

So even though you never saw a single response from Redis, you still **fully compromised** the server.

---

## Bottom line

| Scenario | Impact |
|----------|--------|
| Blind SSRF to read-only internal API | Low to Medium |
| Blind SSRF to internal admin panel (with write actions) | **High** |
| Blind SSRF to internal Redis/Memcached/gopher | **Critical** (often RCE) |
| Blind SSRF + out-of-band detection | Medium to High |

> **Key takeaway:** Don't dismiss blind SSRF as low impact.  
> If the internal service allows **state changes** (write, delete, execute), it's just as bad as normal SSRF — sometimes worse because people ignore it.