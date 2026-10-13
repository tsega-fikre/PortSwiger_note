# Lab: Password Reset Poisoning via Middleware

**Difficulty:** Practitioner  
**Status:** Solved  
**Category:** Authentication / Password Reset

## Objective

Exploit password reset poisoning to obtain Carlos's password reset token, reset his password, and log in to his account.

## Steps to Solve

1. **Request a password reset**  
    I opened the lab and submitted a password reset request for the victim, `carlos`.
    
2. **Intercept the reset request**  
    I intercepted the password reset request using Burp Suite before forwarding it to the server.
    
3. **Inject the forwarded-host header**  
    I added the following header, replacing the placeholder with my Burp Collaborator URL:
    
    ```
    X-Forwarded-Host: My-Collaborator-URL
    ```
    
    I then forwarded the modified request.
    
4. **Capture the reset token**  
    I monitored Burp Collaborator and found the incoming request containing Carlos's password reset token.
    
5. **Reset Carlos's password**  
    I used the captured token in the password reset URL in my browser. The application displayed the password reset form, allowing me to set a new password.
    
6. **Log in successfully**  
    I logged in to Carlos's account using the new password and successfully solved the lab.
    

## Proof of Solution

### Screenshot 1 — Modified Request
![[Pasted image 20261013190856.png]]
Show the intercepted password reset request containing the injected `X-Forwarded-Host` header.

### Screenshot 2 — Captured Reset Token
![[Pasted image 20261013190953.png]]
Show the relevant Burp Collaborator interaction. Redact the token before publishing.

### Screenshot 3 — Password Reset
![[Pasted image 20261013191039.png]]
Show the password reset form or confirmation after setting the new password.

### Screenshot 4 — Successful Login
![[Pasted image 20261013191108.png]]
Show Carlos's account page and the lab's solved status.

## What I Learned

- Password reset poisoning can occur when an application trusts an attacker-controlled forwarded-host header.
    
- Password reset links may become dangerous if their host is constructed from untrusted request headers.
    
- A leaked password reset token can allow an attacker to take over an account.
    
- Middleware and reverse-proxy configurations must handle forwarded headers securely.
    

## Remediation

- Generate password reset links using a trusted, configured application domain.
    
- Do not blindly trust `X-Forwarded-Host` or other forwarded headers.
    
- Configure reverse proxies to remove untrusted incoming forwarding headers and set trusted values.
    
- Use short-lived, single-use password reset tokens.
    
- Monitor suspicious password reset activity.
    

## Conclusion

The lab demonstrated how trusting an attacker-controlled `X-Forwarded-Host` header can poison a password reset link and expose a victim's reset token. I used the captured token to reset Carlos's password and access his account.