# How does CSRF work?

**Step-by-step example:**

1. **You log into your bank**  
   You type your username and password. The bank gives your browser a “cookie” (like a temporary ID badge) so it remembers it’s you.

2. **You visit a bad website** (without knowing it’s bad)  
   This site has a hidden form or link. For example, a button that says “Click for cute cat pictures” — but behind the scenes, it’s really sending a request to your bank to transfer $1000.

3. **The bad site tricks your browser**  
   Your browser sees the hidden request and automatically attaches your bank’s ID badge (the cookie), because you’re still logged in.

4. **The bank sees the badge and thinks it’s you**  
   The bank processes the request — money is sent to the hacker.

Now, the **three conditions** from your text, in plain English:

- **Relevant action** – There has to be something worth doing, like transferring money or changing your password.
- **Cookie-based session** – The bank only checks your ID badge (cookie), not anything else like a secret code you type each time.
- **No unpredictable request parameters** – The hacker doesn’t need to guess anything secret, like your current password, to make the request work. If they did, CSRF wouldn’t be possible.

So:  
If all three are true → hacker can silently make your browser do things you never intended.