You've provided a great definition of SQL injection. To build on that, here’s a breakdown of how it works, the main types, and a simple example.

### How SQL Injection Works

The vulnerability occurs when an application incorporates user-supplied input into a SQL query without properly validating or sanitizing it. The attacker provides maliciously crafted input that changes the structure of the SQL query, causing the database to execute unintended commands.

### A Simple Example

Consider a web application with a login form that asks for a username and password. The back-end code might construct a SQL query like this:

```sql
SELECT * FROM users WHERE username = 'john' AND password = 'password123'
```

If the application directly inserts user input into the query, an attacker could enter the following into the username field:

```
admin' --
```

If the application does not sanitize this, the query becomes:

```sql
SELECT * FROM users WHERE username = 'admin' -- ' AND password = 'anything'
```

In SQL, `--` denotes a comment. The database ignores the rest of the query. If an `admin` user exists, the attacker is logged in without ever knowing the password.

### Main Types of SQL Injection

1.  **In-band SQLi (Classic)**  
    The attacker uses the same channel to both launch the attack and retrieve results.  
    - *Error-based*: The attacker triggers database error messages that reveal structure or data.  
    - *Union-based*: The attacker uses the `UNION` SQL operator to combine results from multiple queries into a single HTTP response.

2.  **Blind SQLi**  
    The application does not return actual data or error messages, but the attacker can still infer information.  
    - *Boolean-based*: The attacker sends payloads that cause the page to behave differently (e.g., “true” vs. “false” responses) and deduces data bit by bit.  
    - *Time-based*: The attacker forces the database to wait for a specified number of seconds and observes whether the response is delayed, using that as a signal.

3.  **Out-of-band SQLi**  
    The attacker uses a different channel to retrieve results, often relying on database features like DNS or HTTP requests. This is typically used when direct responses are too slow or unreliable.

### Potential Impact

- **Data exfiltration**: Stealing sensitive information like passwords, credit card numbers, or personal data.
- **Authentication bypass**: Logging in as any user without credentials.
- **Data modification or deletion**: Altering or destroying critical records.
- **Privilege escalation**: Gaining administrative access to the database.
- **Remote code execution**: In some configurations (e.g., Microsoft SQL Server with `xp_cmdshell`), an attacker can execute arbitrary operating system commands on the server.
- **Denial of service**: Running resource-intensive queries or deleting database objects.

### Prevention

- **Use parameterized queries (prepared statements)**: This is the most effective defense. It ensures that user input is treated as data, not executable code.
- **Use stored procedures** (with care, ensuring they do not concatenate input).
- **Apply input validation**: Whitelist allowed values where possible (e.g., dropdown selections).
- **Escape user input** only as a last resort, and ensure it’s done correctly for the specific database system.
- **Use the principle of least privilege**: Database accounts used by the application should have only the permissions they absolutely need.
- **Regularly update and patch database software**.
- **Use web application firewalls (WAFs)** as a defense-in-depth measure, but not as a primary solution.

Would you like to go deeper into any specific type, such as how to exploit or defend against blind SQL injection in practice?