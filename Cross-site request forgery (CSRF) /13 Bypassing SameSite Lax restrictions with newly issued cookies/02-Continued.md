Let’s continue with the same simple explanation.

---

## The problem: how to refresh the cookie *without* the user realizing

We know the attack only works in the first 2 minutes after a **new cookie** is issued.  
So the attacker needs a way to make the website issue a new cookie **right before** the attack.

One way: trick the user into logging in again through something like “Login with Google.”  
But the attacker can’t force the user to type their password again — that’s too obvious.

Instead, the attacker can send the user to a special link that **reuses their existing login session** with Google or Facebook to get a fresh cookie from the target site.  
This happens automatically in the background if done right — no password needed.

But there’s a catch:  
To make that work, the user’s browser has to **leave the attacker’s page** and go to the target site. Once they leave, the attacker loses control and can’t launch the final attack anymore.

---

## The clever trick: open a new tab

Instead of leaving the current page, the attacker can open the cookie-refresh link in a **new browser tab**.  
That way, the original tab stays open and ready to launch the final attack after the new cookie is issued.

But there’s another catch:  
Browsers **block popups** by default unless they’re opened by a real user click (not automatically by code).

So this won’t work:

```javascript
window.open('https://vulnerable-website.com/login/sso'); // blocked
```

---

## The bypass: wait for a user click

The attacker can set up their page so that the popup only opens **when the user clicks anywhere on the page** — for example, on a fake button or even blank space.

Like this:

```javascript
window.onclick = () => {
    window.open('https://vulnerable-website.com/login/sso');
}
```

Now:

1. The user lands on the attacker’s page.
2. They click somewhere (maybe out of curiosity or by accident).
3. A new tab opens with the login page of the target site, which silently refreshes their session cookie.
4. The original tab stays open and now launches the CSRF attack within the 2-minute window.

---

## Plain English summary

The attacker needs to **refresh your cookie** without you leaving their page.  
They try to open a hidden new tab to do it, but browsers block popups.  
So they wait for you to **click anywhere** on their page — then they open the popup, refresh your cookie, and immediately attack you from the original tab.

It’s like a pickpocket who needs you to turn around for 2 seconds.  
Instead of forcing you to turn, they wait for you to naturally look away — then strike.