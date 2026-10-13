# Lab: Broken brute-force protection, IP block


<h2 align=center>Exploitation Steps</h2>

### Step 1:

I first opened the login page and entered the valid credentials:

```text
wiener:peter
```

I intercepted the login request using Burp Suite and sent it to **Intruder** for testing.
![alt text](%20Authentication%20vulnerabilities%20/03%20Vulnerabilities%20in%20password-based%20login/image.png)
### Step 2:

I created a username wordlist called `username.txt`. The list contained the usernames `carlos` and `wiener` repeated in sequence, with a total of **147 username entries**.

I also created a password list containing **147 entries**, where the password `peter` was repeated throughout the list.
![alt text](%20Authentication%20vulnerabilities%20/03%20Vulnerabilities%20in%20password-based%20login/image-2.png)
### Step 3:

In Burp Intruder, I selected the **username** and **password** parameters as the attack positions and configured the attack type as **Pitchfork**.

I used Pitchfork because I wanted each username to be tested against the corresponding password entry at the same position in the two lists.
![alt text](image-3.png)
### Step 4:

I also changed the **Resource Pool** settings. Since the default pool allows up to 100 concurrent requests, I reduced the maximum concurrent requests to **1** so that the login attempts would be sent sequentially, one at a time.
![alt text](image-4.png)
### Step 5:

I started the attack and reviewed the responses. Most requests returned different responses from the successful login attempt, but one request using the `carlos` username returned a **302 status code**, matching the behavior of the valid `wiener` login request.

I opened and checked that request manually, and the credentials were accepted.
![alt text](image-5.png)
**Result:** I successfully logged in as `carlos`, which solved the lab.

![alt text](%20Authentication%20vulnerabilities%20/03%20Vulnerabilities%20in%20password-based%20login/image-1.png)

