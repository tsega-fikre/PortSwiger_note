Let's tell this as a simple story with **zero technical words**.

---

### The Story of the Two Delivery Men

You live in a **Secure Mansion** called **MyBank**. You have a **Golden Key** (Cookie) that opens all doors inside.

The mansion has a **Super Strict Guard** named `Strict`. His rule is:

> *"If you come from **outside the gate**, I take your Golden Key away. You can walk around the garden, but you cannot open any doors inside."*

A thief wants to open the **Vault Door** inside your mansion using your Golden Key. But the Guard at the gate is too good. The thief cannot just walk in from the street.

---

### The Two Ways to Deliver a Message

The thief writes a note that says: **"GO TO THE VAULT."**

He needs to get this note into the mansion. He has two delivery men. One is smart. One is forgetful.

---

### Delivery Man #1: The Server-Side Guy (The One Who Fails)

**How he works:**
The Server-Side Guy stands **at the gate**. When a visitor arrives, he hands them a note and says: *"Take this inside."*

**What happens:**
1.  You are outside. You click a link in an email.
2.  You arrive at the **Gate**.
3.  **Guard checks:** "You came from outside. **Give me your Golden Key.** " (Key is taken).
4.  The Server-Side Guy hands you the note: *"GO TO THE VAULT."*
5.  You walk to the Vault door.
6.  You try to open it.
7.  **Vault Door:** "Where is your Golden Key?"
8.  You: "The Guard took it."
9.  **Vault Door:** "No Key. No Entry." ❌

**Why this fails:**
The Guard **remembers** you came from outside. He never gave your Key back. The note traveled with you, but you were **powerless** the whole time.

---

### Delivery Man #2: The Client-Side Guy (The One Who Works)

**How he works:**
The Client-Side Guy is **already inside the mansion**. He stands in the **Garden Shed** (a harmless public area inside the gate). He waits for you to arrive.

**What happens:**
1.  You are outside. You click a link in an email.
2.  You arrive at the **Gate**.
3.  **Guard checks:** "You came from outside. **Give me your Golden Key.** " (Key is taken).
4.  You walk to the **Garden Shed**. You have **no Key**. That's fine. The Shed is public.
5.  You meet the Client-Side Guy inside the Shed.
6.  He whispers: *"Hey. You just arrived. As far as I'm concerned, you're **inside the mansion now**. Here is your Golden Key back. Now, **GO TO THE VAULT**."*
7.  You take the Key. You walk from the **Garden Shed** to the **Vault**.
8.  **Guard (inside):** "Where did you come from?"
9.  You: "The Garden Shed."
10. **Guard (inside):** "The Shed is **inside** the mansion. You're good. Keep your Key."
11. You arrive at the Vault door.
12. **Vault Door:** "You have the Golden Key! Come in!" ✅

**Why this works:**
The Client-Side Guy **reset the journey**. He made it look like your trip **started from the Garden Shed**, not from outside the gate. The Guard inside the mansion doesn't know or care that you were outside five minutes ago. He only sees: *"Shed $\to$ Vault. Same building. Key allowed."*

---

### The One Key Difference

| Delivery Man | Where He Works | Does He Reset Your Status? | Result |
| :--- | :--- | :---: | :---: |
| **Server-Side Guy** | At the Gate (outside) | ❌ No | Key stays taken. Vault locked. |
| **Client-Side Guy** | Garden Shed (inside) | ✅ Yes | Key returned. Vault opened. |

---

### The Moral of the Story

The **Server-Side Guy** is like a signpost outside the building. He can point you somewhere, but you're still outside.

The **Client-Side Guy** is like a person **already inside** the building. Once you step inside (even into a harmless public area like a Garden Shed), he can give you back your privileges and send you anywhere.

**The Thief's Trick:**
Find a Client-Side Guy (a JavaScript redirect on the website) and give him instructions through the window (by putting commands in the URL). He will do the dirty work **from the inside**, where the Strict Guard can't see.