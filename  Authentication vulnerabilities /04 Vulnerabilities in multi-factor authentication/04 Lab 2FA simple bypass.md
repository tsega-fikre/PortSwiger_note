# Lab: 2FA Simple Bypass

**Difficulty:** Apprentice  
**Status:** Solved  
**Category:** Authentication  
**Credentials:** `carlos:montoya`
![[Pasted image 20261012204706.png]]
## Objective

Access Carlos's account page without having access to his two-factor authentication (2FA) verification code.

## Steps to Solve

1. **Log in as Carlos**  
    I opened the lab and logged in using the victim's credentials: `carlos:montoya`.
    
2. **Reach the 2FA page**  
    After submitting the credentials, the application redirected me to the `/login2` page to request the 2FA verification code.
    
3. **Modify the URL**  
    Instead of completing the 2FA verification, I removed `/login2` from the URL and navigated directly to the account page.
    
4. **Access Carlos's account**  
    The application allowed me to access Carlos's account without verifying the 2FA code, and the lab was marked as solved.
    

## Proof of Solution

### Screenshot 1 — 2FA Verification Page
![[Pasted image 20261012204603.png]]
Show the `/login2` page requesting the verification code.

### Screenshot 2 — URL Modification
![[Pasted image 20261012204630.png]]

Show the URL after removing `/login2` and navigating directly to the account page.

### Screenshot 3 — Successful Access
![[Pasted image 20261012204531.png]]
Show Carlos's account page and the lab's solved status.

## What I Learned

- Authentication must be enforced on every protected page.
    
- Hiding or bypassing the 2FA page must not grant access to authenticated resources.
    
- The server must verify that the user has completed both authentication factors before granting access.
    

## Remediation

- Enforce 2FA verification on the server for every protected route.
    
- Track authentication state using a server-side session.
    
- Do not grant full access until the second authentication factor has been successfully verified.
    
- Redirect unauthenticated or partially authenticated users to the appropriate verification page.