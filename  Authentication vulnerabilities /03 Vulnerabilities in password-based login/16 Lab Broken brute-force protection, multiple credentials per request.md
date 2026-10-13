# Broken Brute-Force Protection, Multiple Credentials per Request

**Difficulty:** Expert  
**Status:** Solved  
**Category:** Authentication
![[Pasted image 20261012200016.png]]
## Objective

Brute-force Carlos's password and access his account page by exploiting a flaw in the login brute-force protection.

## Steps to Solve

1. **Test the login protection**  
    I opened the lab, went to the login page, and tried logging in as Carlos. After 2–3 failed attempts, the website temporarily locked me out for one minute.
    
2. **Intercept the login request**  
    I intercepted the login request using Burp Suite and discovered that I could submit multiple password candidates within a single request.
    ![[Pasted image 20261012200630.png]]
3. **Split the password list**  
    I prepared a list of 100 possible passwords and divided it into two groups of 50. I sent each group in a separate request to test which group contained the correct password.
    ![[Pasted image 20261012200553.png]]
4. **Narrow down the password**  
    After identifying the group containing the correct password, I used a divide-and-conquer approach, repeatedly narrowing down the candidates until I found Carlos's password.
    ![[Pasted image 20261012200700.png]]
5. **Log in successfully**  
    I logged in using the discovered password and accessed Carlos's account page, successfully completing the lab.
    ![[Pasted image 20261012200436.png]]

## What I Learned

- Brute-force protection can be bypassed when the application processes multiple credential candidates in a single request.
    
- Rate limiting individual requests is not enough if each request can contain many login attempts.
    
- Dividing a candidate list into smaller groups can help identify the correct password efficiently.
    

## Recommended Remediation

- Enforce rate limits based on the number of credential attempts, not only the number of HTTP requests.
    
- Limit the number of credential candidates processed per request.
    
- Implement progressive delays and temporary account protections.
    
- Monitor suspicious login activity and repeated authentication failures.