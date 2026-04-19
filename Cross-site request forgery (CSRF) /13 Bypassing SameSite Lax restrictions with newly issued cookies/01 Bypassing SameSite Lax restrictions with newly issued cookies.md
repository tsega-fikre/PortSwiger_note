# Bypassing SameSite Lax restrictions with newly issued cookies


Let’s imagine you have a **sticky note** that your web browser keeps for a specific website, like your online banking site. That sticky note is called a **cookie**, and it proves that you’re logged in.

Now, websites sometimes put a rule on that sticky note called **SameSite=Lax**. That rule says:  
> “I will *not* go with you if you try to visit me from a *different* website, unless you click a regular link (not a form or a hidden request).”

That’s a good security rule — it stops bad guys from tricking you into doing things on other websites without you knowing.

---

### But here’s the loophole:

If a website doesn’t write the rule “SameSite=Lax” explicitly, **Chrome automatically adds it for them** after a short time.  
But to avoid breaking things like **logging in through Google or Facebook** (single sign-on), Chrome gives a **2-minute grace period** after a new cookie is created.  
During those 2 minutes, the cookie *will* be sent on cross-site POST requests (like a fake form submission from a bad guy’s site).

---

### What can an attacker do with this?

Normally, 2 minutes is too short to launch an attack — you’d have to time it perfectly.

But if the attacker can **force the website to give the victim a fresh new cookie** right before the attack, then the 2-minute window restarts.  
For example:  
- The attacker tricks the victim into logging in again through a third-party service (like “Login with Google”).  
- The target website issues a brand new session cookie.  
- Now the attacker has another 2 minutes to launch their cross-site attack while the new cookie is still in the “grace period.”

---

### In plain English:

Think of it like a guard who usually doesn’t let a pass work if you come from a different door.  
But for the first 2 minutes after you get the pass, the guard *does* let it work from any door.  
If a bad guy can get you a new pass (by making you log in again), they get another 2 minutes to sneak you through the wrong door without you knowing.