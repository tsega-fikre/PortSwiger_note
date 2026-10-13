# Lab: Stored XSS into HTML context with nothing encoded
![[Pasted image 20261003090810.png]]
## Step 1 : Open the lab
- I opened the lab and its got me the landing page to vloge website.

## Step 2 : Identify the Attack surface
- I navigate to one blog and i found text box to submit comment and i choose that as may attack surface
## Step 3 : Try An Attack
- I tired the attack pyload on the comment functionality and wait the response.
  ```HTML

 <script>alert("Hello XSS!")</script>
  ```
  
- And its worked its shows that it have XSS Vulnerability  
- ![[Pasted image 20261003091236.png]]