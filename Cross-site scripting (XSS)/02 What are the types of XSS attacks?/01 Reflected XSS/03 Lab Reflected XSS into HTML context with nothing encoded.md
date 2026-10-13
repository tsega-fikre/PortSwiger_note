# Lab: Reflected XSS into HTML context with nothing encoded
![alt text](0%20Proof%20images/image.png)

## Step 1 : Open the lab
- I opened the lab and its got me the landing page to vloge website.

## Step 2 : Identify the Attack surface
- I saw search funtionality and when I in serte some input its send `GET` requst to the server so  I think wht if i tried this one.
![alt text](0%20Proof%20images/image-1.png)
## Step 3 : Try An Attack
- I tired the attack pyload on the search functionality and wait the response.
  ```HTML

 <script>alert("Hello XSS!")</script>
  ```
  
- And its workes its shows that it have XSS Vulnirablitlity  

![alt text](image-2.png)
![[Pasted image 20261003073252.png]]