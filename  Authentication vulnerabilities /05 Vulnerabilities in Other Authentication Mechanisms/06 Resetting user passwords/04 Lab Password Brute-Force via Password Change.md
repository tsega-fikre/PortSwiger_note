# Lab: Password Brute-Force via Password Change

**Difficulty:** Practitioner  
**Status:** Solved  
**Category:** Authentication

## Objective

Exploit a flaw in the password change functionality to brute-force Carlos's current password and access his **My account** page.

## Steps to Solve

1. **Log in and test password changing**  
    I logged in using my credentials, `wiener:peter`, and opened the password change functionality. I tested what happened when the new password and confirmation fields did not match.
    
2. **Analyze the password change request**  
    I intercepted the password change request using Burp Suite. I noticed that the request contained the username, current password, new password, and password confirmation fields.
    
3. **Modify the request**  
    I changed the username from `wiener` to `carlos`. I also made the new password and confirmation fields different so that the password change would fail, allowing me to test candidate current passwords without successfully changing the password.
    
4. **Brute-force the current password**  
    I sent the modified request to Burp Intruder and used the provided candidate password list to test Carlos's current password. I compared the response lengths and identified one response that differed significantly from the others.
    
5. **Identify the correct password**  
    I examined the unusual response and identified `taylor` as the likely correct password for Carlos.
    
6. **Log in successfully**  
    I returned to the login page and logged in using Carlos's username and the discovered password. I accessed his **My account** page and successfully solved the lab.
    

## Proof of Solution

### Screenshot 1 — Password Change Request
![[Pasted image 20261013200505.png]]
Show the intercepted request in Burp Suite with the username and password fields.

### Screenshot 2 — Intruder Results
![[Pasted image 20261013200523.png]]
Show the candidate password attack results and the response with the different length.

### Screenshot 3 — Successful Login
![[Pasted image 20261013200542.png]]
Show Carlos's **My account** page and the lab's solved status.

## What I Learned

- Password change functionality can expose authentication weaknesses when it handles invalid requests incorrectly.
    
- Different server responses can reveal whether a password candidate is correct.
    
- Burp Suite Intruder can automate password testing using a candidate list.
    
- Password change endpoints need proper authentication checks and brute-force protection.
    

## Remediation

- Enforce rate limits on password change and authentication attempts.
    
- Return generic error messages that do not reveal whether the current password is correct.
    
- Validate all password change fields consistently on the server.
    
- Monitor repeated failed password change attempts.
    
- Require appropriate authentication and authorization checks for every password change request.
    

## Conclusion

The lab demonstrated how a flaw in password change functionality can allow an attacker to test candidate passwords against another user's account. By modifying the request, automating the tests with Burp Intruder, and analyzing response differences, I discovered Carlos's password and accessed his account.