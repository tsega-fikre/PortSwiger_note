# SSRF attacks against other back-end systems

---

## Breefly (30-second version)

**What happens:**  
The server can talk to **secret computers** inside the company that you can't reach from the internet.

**How:**  
You give the server a private address like `http://192.168.0.68/admin` (a computer hidden inside the building).

**Why it's dangerous:**  
Those secret computers have **weak security** because they think *"No one from outside can reach us anyway."*  
You use the server as a **bridge** to break into them.

**Result:**  
You hack internal systems that were never meant to face the internet.

---

## Detailed 
### The Setup: Public vs. Private

Imagine a **mall** (the internet):

| You can see | You cannot see |
|-------------|----------------|
| The front entrance (public website) | The security office (private IP) |
| The gift shop (public pages) | The server room (private IP) |
| The food court (public data) | The manager's private computer (private IP) |

**Private IP addresses** (like `192.168.x.x`, `10.x.x.x`, `172.16.x.x`) are like **unlisted phone numbers** – they exist, but you can't call them from outside.

---

### The Bridge (The Vulnerable Server)

The **application server** (your assistant/guard) lives **inside** the building.  
It has a special phone that **can** call those private numbers.

You can't call `192.168.0.68` from your home.  
But the **server can**.

So you trick the server: *"Please call `192.168.0.68/admin` and tell me what they say."*

---

### Why Back-End Systems Have Weak Security

The people who built those internal systems thought:

> *"No one from the outside can ever reach this computer. So why bother with passwords? Why bother with login screens?"*

**They are wrong** – because SSRF turns the server into a **bridge** from the outside to the inside.

---

## Real Example (from your lab text)

### Normal request (what the app expects)
```http
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=6&storeId=1
```

### Attack request (SSRF against back-end system)
```http
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=https://192.168.0.68/admin
```

The server fetches `https://192.168.0.68/admin` – a private IP address you can't reach – and returns the admin page to you.

---

## Common Private IP Addresses to Try

| IP Range | What it's often used for |
|----------|--------------------------|
| `127.0.0.1` or `localhost` | The same server (loopback) |
| `10.0.0.0` to `10.255.255.255` | Large company internal networks |
| `172.16.0.0` to `172.31.255.255` | Medium company internal networks |
| `192.168.0.0` to `192.168.255.255` | Home/small office networks |

---

## What You Can Find on Internal Systems

| Internal system | What it might contain |
|-----------------|----------------------|
| `http://192.168.0.1/admin` | Router admin panel (change settings) |
| `http://10.0.0.10:8080` | Internal API (delete data, create users) |
| `http://172.31.0.5/backup` | Unprotected backups (passwords, database dumps) |
| `http://192.168.0.100:9200` | Elasticsearch database (all data exposed) |
| `http://169.254.169.254` | Cloud metadata server (AWS, GCP, Azure – **super dangerous**) |

---

## Why This Is Worse Than Localhost SSRF

| Localhost SSRF | Back-end SSRF |
|----------------|----------------|
| Attacks only the same computer | Attacks **any** computer inside the network |
| Limited to one machine | Can scan and attack entire internal network |
| Admin panel might have some security | Internal systems often have **zero** security |
| Damage is contained | Damage can spread to databases, internal APIs, cloud services |

---

## Simple Analogy

> **You're outside a bank.**  
> You can't go into the back office where the vault is.  
> But you see a security guard (the server) who can walk anywhere inside.  
> You tell the guard: *"Please go to the vault room (192.168.0.68) and open the log book. Then come back and tell me what it says."*  
> The guard does it because he's trained to follow instructions.  
> **You just robbed the vault without ever going inside.**

---

## One-line takeaway

SSRF against back-end systems turns the vulnerable server into a **Trojan horse** – you hide inside its trusted access to break into internal computers that were never supposed to see the internet.