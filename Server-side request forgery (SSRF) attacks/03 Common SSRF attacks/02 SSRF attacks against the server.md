# SSRF attacks against the server

---

## Breefly (30-second version)

**What happens:**  
You trick the server into calling *itself*.

**How:**  
You give it a special address like `http://localhost/admin` or `http://127.0.0.1/admin`.

**Why it's dangerous:**  
The server trusts itself. So if you make it ask itself for admin pages, it will say *"Oh, it's me asking — here's the secret admin panel!"* Then the server hands you the admin page even though you're not logged in.

**Result:**  
You become admin without a password.

---

## Detailed 

### Step 1: Understand the normal situation

Imagine you live in a **gated community** (a website).

- **You** live outside the gate (normal internet user).
- **The guard** sits at the gate (the server).
- **Inside the community** there's a **secret VIP club** (the `/admin` page).  
  Only people on a list (authenticated users) can enter.

Normally, you walk up to the guard and say: *"Let me into the VIP club."*  
The guard checks the list, doesn't see your name, and says: *"No."*

**You can't get in.**

---

### Step 2: Understand the trick

Now imagine the guard has a **radio** (the `stockApi` parameter).  
The guard uses the radio to call other buildings to get information.

You say: *"Guard, use your radio to call `http://localhost/admin`."*

**Important:** `localhost` means **"call yourself"**. It's like looking in a mirror — the guard is calling his own radio.

The guard picks up the radio and says: *"Hey, it's me, the guard. Let me into the VIP club."*

The VIP club hears the voice and thinks: *"Oh, it's the guard! He's trusted. Let him in."*

The guard walks into the VIP club, reads everything inside, and then **comes back and tells you** what he saw.

---

### Step 3: Why this breaks security

The VIP club normally asks: *"Are you on the authorized list?"*  
But for the **guard**, there's a special rule: *"If the request comes from my own radio, don't check the list — just let it in."*

Why? Because the system assumes: *"Only the guard would use the guard's radio. So he must be trustworthy."*

**You just tricked the guard into using his own radio to give you VIP access.**

---

### Step 4: Map it to your example

| Normal request | Attack request |
|----------------|----------------|
| `stockApi=http://stock.weliketoshop.net:8080/...` | `stockApi=http://localhost/admin` |
| Guard calls another store's computer | Guard calls himself |
| Gets stock info | Gets the admin page |
| Shows you stock status | Shows you admin panel (secrets, user list, delete buttons) |

---

### Step 5: Why you can't just visit `/admin` yourself

You try typing `http://shop.com/admin` in your browser.  
The server says: *"You're not logged in. Go away."*

But when the **server** requests `http://localhost/admin` from itself, the server says: *"Oh, it's me! Come on in."*

**The server trusts itself completely.** You're just riding along on that trust.

---

## Expert-level summary (still simple)

| Concept | Plain English |
|---------|----------------|
| **Loopback / localhost / 127.0.0.1** | The computer's address for talking to itself |
| **SSRF against the server** | Making the server request its own secret pages |
| **Why it works** | The server trusts internal requests more than external ones |
| **The bypass** | Admin checks are often skipped for "self" requests |
| **The result** | You see admin pages without a password |

---

## One picture in your head

> *The guard won't let YOU into the VIP room. But if you hand the guard a note saying "Ask YOURSELF to go into the VIP room," the guard will go in, look around, and come back to describe everything to you.*

**That's SSRF against the server.**