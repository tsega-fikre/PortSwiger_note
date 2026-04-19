You've moved to the **default behavior of the modern web**—the setting that Chrome silently applies to every cookie unless told otherwise.

Let's break down the two conditions you listed. These are the **only two keys** that unlock the `Lax` bouncer's door from a cross-site request.

### The Two Golden Rules of Lax (Cross-Site)

The text gives us a strict **AND** gate. **Both** must be true.

| Condition | Requirement | What it Blocks |
| :--- | :--- | :--- |
| **1. Method** | `GET` | `POST`, `PUT`, `DELETE`, `PATCH` |
| **2. Navigation Type** | **Top-Level** (User Click) | IFrames, Images, Scripts, CSS, `fetch()` |

### Visualizing "Top-Level Navigation"

This is the most misunderstood part. Let's clarify what counts as "Top-Level."

**✅ Counts as Top-Level (Cookie Sent):**
- User clicks an `<a href="...">` link.
- User clicks a browser bookmark.
- User manually types URL in address bar.

**❌ Does NOT Count as Top-Level (Cookie Blocked):**
- `<img src="https://bank.com/image.jpg">` (This is a subresource).
- `<iframe src="https://bank.com/widget">` (This is a nested browsing context).
- `<script src="https://bank.com/api.js">` (This is a script load).
- `fetch('https://bank.com/data')` triggered by JavaScript. (This is a background API call).

### Why This Blocks 95% of CSRF Attacks

You noted the key phrase:
> *"...POST requests are generally used to perform actions that modify data... they are much more likely to be the target of CSRF attacks."*

**The Classic CSRF Attack (Blocked by Lax):**
```html
<!-- Hosted on evil.com -->
<form action="https://bank.com/transfer" method="POST">
    <input type="hidden" name="to" value="hacker">
    <input type="hidden" name="amount" value="1000">
</form>
<script>document.forms[0].submit();</script>
```
- **Method:** `POST` ❌ (Fails Condition 1)
- **Navigation:** Top-Level (Form Submit) ✅
- **Lax Verdict:** ❌ **Cookie Blocked.** Attack fails.

**The Sneaky Image Attack (Blocked by Lax):**
```html
<!-- Hosted on evil.com -->
<img src="https://bank.com/delete-account">
```
- **Method:** `GET` ✅
- **Navigation:** Subresource (Image Load) ❌ (Fails Condition 2)
- **Lax Verdict:** ❌ **Cookie Blocked.** Attack fails.

### The Lax Loophole (Where Your Testing Focuses)

The only CSRF attack that **survives** `Lax` is the one that satisfies **both** conditions.

**The Vulnerable Scenario:**
The developer made a mistake. They allowed a state-changing action via a **GET** request.

```html
<!-- Hosted on evil.com -->
<a href="https://bank.com/transfer?to=hacker&amount=1000">
    Click here to see cute puppies!
</a>
```
- **Method:** `GET` ✅
- **Navigation:** Top-Level (User Clicked Link) ✅
- **Lax Verdict:** ✅ **Cookie SENT.** Attack succeeds.

### The "Background Request" Clarification

You also highlighted:
> *"...the cookie is not included in background requests, such as those initiated by scripts, iframes, or references to images..."*

This is the **Site Isolation** feature of Lax. Even if you visit `evil.com`, the hacker cannot steal your data by simply doing:
```javascript
fetch('https://bank.com/account-details')
    .then(r => r.text())
    .then(data => sendToHacker(data));
```
Because this is a **background request** (initiated by `fetch`), the `Lax` cookie stays home. The hacker gets the **public, logged-out** version of the page.

### Summary Decision Tree for Lax

```
Is this a Cross-Site Request?
        |
        No ---> ✅ Send Cookie (Same-Site)
        |
       Yes
        |
Is the Request Method GET?
        |
        No ---> ❌ Block (CSRF Protected)
        |
       Yes
        |
Did the USER click a link (Top-Level Navigation)?
        |
        No ---> ❌ Block (Prevents data exfiltration via JS/Images)
        |
       Yes
        |
✅ SEND COOKIE (This is the bypass window for testers)
```