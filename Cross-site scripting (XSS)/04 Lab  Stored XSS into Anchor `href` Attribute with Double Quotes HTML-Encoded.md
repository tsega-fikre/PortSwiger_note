# Lab: Stored XSS into Anchor `href` Attribute with Double Quotes HTML-Encoded

**Difficulty:** Apprentice  
**Status:** Solved  
**Category:** Stored Cross-Site Scripting (XSS)

## Objective

Exploit the stored XSS vulnerability in the blog comment functionality so that clicking the comment author's name executes `alert(1)`.

## Lab Solving Steps

### Step 1: Open the Comment Section

I opened a blog post and went to the comment section.

### Step 2: Inject the Payload

I filled in the comment form and entered the following payload in the **Website** field:

```
javascript:alert(1)
```

I completed the required fields and submitted the comment.
![[Pasted image 20261015162933.png]]
### Step 3: Trigger the XSS

After submitting the comment, I clicked the comment author's name. The browser executed `alert(1)`, and the lab was solved.
![[Pasted image 20261015162950.png]]
**Boom — lab solved!**
![[Pasted image 20261015163002.png]]
## What I Learned

- Stored XSS payloads are saved by the application and can affect users who view the stored content.
    
- An anchor's `href` attribute can contain a dangerous URL scheme such as `javascript:`.
    
- HTML-encoding double quotes does not make a dangerous URL scheme safe.
    
- Clicking the comment author's name triggered the JavaScript.
    

## Remediation

- Allow only safe URL schemes, such as `https:` and `http:`, for website fields.
    
- Validate and sanitize user-supplied URLs before storing or rendering them.
    
- Use context-aware output encoding and safe templating.
    
- Avoid inserting untrusted values directly into HTML attributes.
    

## Conclusion

I exploited the stored XSS vulnerability by entering `javascript:alert(1)` in the Website field and triggering it by clicking the comment author's name.

**Key takeaway:** HTML attribute encoding alone cannot prevent XSS when an application accepts dangerous URL schemes.