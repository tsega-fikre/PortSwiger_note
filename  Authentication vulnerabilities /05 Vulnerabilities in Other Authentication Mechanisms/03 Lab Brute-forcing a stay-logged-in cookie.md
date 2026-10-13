# Lab: Brute-Forcing a Stay-Logged-In Cookie

**Difficulty:** Practitioner  
**Status:** Solved  
**Category:** Authentication

## Objective

Brute-force Carlos's `stay-logged-in` cookie to access his **My account** page.
![[Pasted image 20261013175937.png]]
## Steps to Solve

1. **Analyze the cookies**  
    I logged in as `wiener:peter` and intercepted the request to the My account page. I found two cookies: `session` and `stay-logged-in`.
    
2. **Decode the persistent cookie**  
    I selected the `stay-logged-in` cookie and decoded its Base64 value using CyberChef. I discovered that it contained the username and an MD5 password hash. I cracked the hash and recovered the password `peter`.
    
3. **Identify the cookie format**  
    I determined that the cookie followed this format:
    
    `Base64(username:MD5(password))`
    
4. **Prepare the brute-force attack**  
    I used Burp Suite Intruder with the provided candidate password list. I generated the corresponding MD5 hashes, combined each hash with the username `carlos:`, and Base64-encoded the results to create candidate cookies.
    
5. **Identify Carlos's password**  
    I launched the attack and compared the responses. One response had a significantly different length from the others, indicating a potentially valid cookie. I examined the result and identified the correct candidate.
    
6. **Access Carlos's account**  
    I used the successful `stay-logged-in` cookie and accessed Carlos's My account page, solving the lab.
    

## Proof of Solution

### Screenshot 1 — Cookie Analysis
![[Pasted image 20261013180018.png]]
Show the `stay-logged-in` cookie and its decoded structure in CyberChef.

### Screenshot 2 — Burp Intruder Results
![[Pasted image 20261013180106.png]]
Show the generated cookie candidates and the response whose length differs significantly from the others.

### Screenshot 3 — Successful Access
![[Pasted image 20261013180130.png]]
Show Carlos's My account page and the lab's solved status.

## What I Learned

- Persistent login cookies can introduce authentication vulnerabilities.
    
- Base64 is an encoding scheme, not encryption.
    
- MD5 is unsuitable for securely storing passwords because it is fast to brute-force.
    
- Burp Intruder can automate testing of candidate credentials and identify unusual responses.
    
- Cookie security depends on both secure token construction and proper server-side validation.
    

## Remediation

- Avoid storing password hashes directly in persistent login cookies.

- Use cryptographically random, securely generated persistent authentication tokens.
    
- Store token identifiers server-side and support token expiration and revocation.
    
- Use a modern password-hashing algorithm, such as Argon2id, for password storage.
    
- Apply rate limits and monitor suspicious authentication attempts.