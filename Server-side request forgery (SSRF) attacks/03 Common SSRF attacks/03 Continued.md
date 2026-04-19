
# SSRF attacks against the server - Continued

---

## Breefly (30-second version)

Servers trust themselves because:
- **Firewalls** block outsiders but trust inside traffic
- **Emergency backdoors** exist for admins who lose passwords
- **Secret doors** (different ports) are hidden from you but open for the server

The server thinks: *"If the request comes from me, it must be safe."*  
You trick the server into being the messenger — and it hands you the keys.

---

## Detailed (explained like you're 5 years old)

Remember the **gated community** and the **guard** (server)?

The guard has a rule: *"I trust anyone who calls me on my own radio."*

Here are **three reasons** why that rule exists:

---

### Reason 1: The front gate has a security check, but the back door doesn't

**What happens:**  
Imagine the community has:
- A **front gate** with a guard who checks IDs (the normal login page)
- A **back door** that has no guard (the admin interface)

You can only reach the front gate from outside.  
But the guard (server) can walk to the back door easily because he lives inside.

**When you make the guard call himself (`localhost/admin`):**  
He walks to the back door, opens it, and reads everything. No ID check because there's no guard there.

**In real computers:**  
The access control is in a **different component** (like a firewall or load balancer) that only checks external traffic. Internal traffic (from the server to itself) never gets checked.

---

### Reason 2: Emergency key for when the admin forgets their password

**What happens:**  
The community has a **master key** hidden under a rock. The rule says: *"If someone is standing at the back door AND they look like they live here, give them the master key."*

Normally, only the guard can stand at the back door. So the guard gets the master key.

**When you trick the guard:**  
He goes to the back door, gets the master key, and brings it back to you.

**In real computers:**  
For **disaster recovery**, the application allows admin access without a password if the request comes from `localhost`. This helps an admin who lost their password. But an attacker can abuse it.

---

### Reason 3: The admin panel is on a secret channel you can't hear

**What happens:**  
The guard has a **special radio channel** (different port number) that only he uses. You don't even know that channel exists.

You say: *"Guard, go to `http://localhost:8080/admin`"* — which is a channel you didn't know about.  
The guard goes there, finds the admin panel, and brings back everything.

**In real computers:**  
The admin interface listens on a **different port** (like port 8080 or 8443) that is blocked from the internet. But the server can reach it because it's on the same computer.

---

## Why this becomes a **critical vulnerability**

| Reason | Why it's dangerous |
|--------|---------------------|
| **Different components** | The security check is at the front door, not the back door |
| **Emergency access** | The server hands out master keys without asking for a password |
| **Hidden ports** | You can discover secret admin panels you never knew existed |
| **Trust assumption** | The server assumes: *"If it's me asking, it's safe"* — but you're controlling the question |

---

## Simple analogy summary

> **Your house has a smart lock.**  
> Normally, you need a code to enter.  
> But there's a secret rule: *"If the doorbell rings itself (from inside the house), just open — it must be the robot vacuum."*  
> You tape the doorbell button to a stick and press it from outside. The door opens.  
> **That's SSRF against the server.**

---

## One-line takeaway

Servers trust local requests because of **lazy security** — they assume anything coming from inside is safe. SSRF breaks that assumption by making the server ask *itself* for secrets, and the server happily hands them over.