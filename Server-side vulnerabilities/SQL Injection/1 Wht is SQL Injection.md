This is an excellent, textbook-perfect definition of SQL injection. You've accurately captured its essence as a data-driven attack that exploits poor coding practices.

To build on that strong foundation, let's break down the "how" and "why" with a simple example, as it helps to visualize the vulnerability.

### How it Works: A Simple Analogy

Imagine a legitimate user goes to a website and logs in by entering a username and password. The application takes that input and builds a command (a SQL query) to send to the database. A simplified version of that query might look like this:

**Intended Query:**
```sql
SELECT * FROM users WHERE username = 'john' AND password = 'mypassword';
```
If "john" and "mypassword" match a record in the database, the user is granted access.

Now, imagine an attacker types something **malicious** into the username field, like:

`admin' --`

And leaves the password field blank.

The application, without any security checks, builds the query exactly as the attacker has provided it:

**Malicious Query:**
```sql
SELECT * FROM users WHERE username = 'admin' -- ' AND password = 'anything';
```

**Here's the breakdown of the attack:**
1.  The single quote (`'`) after `admin` closes the first string in the query.
2.  The double dash (`--`) is a comment symbol in SQL. It tells the database to ignore everything that follows it.

The database effectively sees:
```sql
SELECT * FROM users WHERE username = 'admin';
```

The database checks for the user "admin" and, if found, **completely ignores the password check**. The attacker has successfully logged in as the admin user without knowing the password.

### Key Impacts You Mentioned (Expanded)

- **Data Breach (Confidentiality):** This is the most common goal. Attackers can extract lists of usernames, passwords, credit card numbers, personal data, and proprietary business information. Your definition correctly points out that they can access data belonging to *other users* or any other data in the database.
- **Data Loss/Corruption (Integrity):** An attacker isn't limited to just *reading* data. They can also use other SQL commands to modify or delete it. For example, they could change product prices, alter user privileges, or wipe entire tables.
- **Bypassing Authentication:** As shown in the example above, SQLi can allow an attacker to log in as any user without needing valid credentials.
- **Compromising the Server (Availability/Further Attacks):** As you mentioned, this is a severe escalation. On some database systems, an attacker can use SQLi to execute operating system commands. They could potentially install backdoors, create new user accounts on the server itself, or launch denial-of-service (DoS) attacks against other systems from the compromised server.

In short, SQL injection is a powerful and dangerous attack because it targets the very heart of an application's data layer, turning the application's own database against it. Your initial definition perfectly sets the stage for understanding this critical vulnerability.