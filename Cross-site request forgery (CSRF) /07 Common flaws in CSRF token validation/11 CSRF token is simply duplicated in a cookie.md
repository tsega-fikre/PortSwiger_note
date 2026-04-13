Let’s imagine you have a clubhouse with a secret knock.  

Normally, to keep bad people out, you have a rule:  
- Everyone must show a **secret code** in two places:  
  1. In their **pocket** (like a cookie)  
  2. On a **piece of paper** they hand you (like a request parameter)  

If the code in the pocket matches the code on the paper, you let them in.  

---

## The problem

In this clubhouse, you don’t write down the secret codes in a book (server-side state) — you just check that the pocket and paper match.  

So, a troublemaker (attacker) thinks:  
> “If I can put *my own fake code* in your pocket, and then give you that same fake code on a paper, the guard will think everything is fine.”  

But how does the attacker put a code in your pocket?  
If the website has a way to **set a cookie** (like a “remember me” feature, or a search bar that sets a cookie), the attacker can trick your browser into storing *their* fake code as a cookie.  

Then, the attacker sends you a link that makes you submit a form with that same fake code on the paper.  
The server checks pocket vs paper → they match → the server thinks it’s a real request from you.  

---

## Example step by step

1. You’re logged into `vulnerable-website.com`.  
2. Attacker finds a page like `/set-cookie?name=csrf&value=FAKE123` — maybe a search page that sets a cookie.  
3. Attacker tricks you into visiting that page (via an image or link).  
4. Your browser now has `csrf=FAKE123` in its cookies.  
5. Attacker then sends you another link: `https://vulnerable-website.com/email/change?csrf=FAKE123&email=evil@hacker.com`.  
6. Your browser automatically includes the cookie `csrf=FAKE123` with the request.  
7. Server checks cookie value (`FAKE123`) vs request parameter (`FAKE123`) → match → changes your email to the attacker’s email.  

---

**In simple terms:**  
The website uses the “double submit” method — check that cookie token = form token.  
If the attacker can *write* any token into your cookie, they can *pretend* to be you.