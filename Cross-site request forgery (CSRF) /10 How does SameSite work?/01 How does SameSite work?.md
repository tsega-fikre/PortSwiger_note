# How does SameSite work?



### Level 1: `SameSite=Strict` (The Hermit)
**The Rule:** *"Do not show this wristband to ANYONE unless the user typed the address into the bar themselves or used a bookmark."*

- **Scenario:** You are logged into your Bank (`bank.com`). Your session cookie is set to `Strict`.
- **User Action:** You click a link in an email that goes to `bank.com/account`.
- **What Happens?**
    1. Browser navigates to `bank.com`.
    2. Bouncer checks the request source: **Email Client** (Cross-Site).
    3. **Result:** **Wristband is hidden.** You arrive at `bank.com` **logged out**. You have to type your password again.
- **Security Verdict:** **Maximum Protection.** Even if you click a malicious link, the action fails because the site doesn't recognize you.
- **User Experience Verdict:** **Terrible.** Every single link from Twitter, Google, or Slack to your bank makes you log in again.

### Level 2: `SameSite=Lax` (The "Plus One" Default)
**The Rule:** *"Hide the wristband for sneaky background requests (Forms/Images), but SHOW the wristband if the user is **actively navigating** (Clicking a Link)."*

This is the **Default** behavior in Chrome since 2021. It's the sweet spot between security and usability.

- **Scenario:** You are logged into your Bank (`bank.com`). Cookie is `Lax`.
- **User Action A:** You click a link in an email to `bank.com/statement`.
    - **Method:** `GET` (Top-Level Navigation).
    - **Bouncer Check:** "Is this a top-level navigation? **Yes.** Is it a safe method (GET)? **Yes.**"
    - **Result:** **Wristband is SHOWN.** You see your statement immediately. (Good UX).

- **User Action B:** You visit `evil.com`. There is a hidden form that auto-submits to `bank.com/transfer-money`.
    - **Method:** `POST` (Top-Level Navigation).
    - **Bouncer Check:** "Is this a top-level navigation? Yes. Is it a safe method? **No (POST is unsafe).**"
    - **Result:** **Wristband is HIDDEN.** The transfer fails. (Good Security).

- **User Action C:** You visit `evil.com`. There is an image tag: `<img src="bank.com/delete-account">`.
    - **Method:** `GET` (Subresource Request).
    - **Bouncer Check:** "Is this a top-level navigation? **No.** (It's just loading an image)."
    - **Result:** **Wristband is HIDDEN.**

**Summary of Lax Logic:**
> **Cookie Sent?** = **Yes** ONLY IF (**Method is GET** AND **User Clicked a Link**).
> **Cookie Sent?** = **No** for IFrames, Images, Scripts, CSS, or POST Forms.

### Level 3: `SameSite=None; Secure` (The Old Way / The Backdoor)
**The Rule:** *"Show the wristband everywhere, to everyone, no questions asked. BUT, the building must have a Steel Reinforced Door (`Secure` flag)."*

- **Requirement:** This setting **MUST** be paired with the `Secure` attribute (meaning the cookie only travels over **HTTPS**). If you try to set `SameSite=None` on an `http://` site, the browser will **reject the cookie entirely**.
- **Use Case:** Embedded widgets, payment gateways, or third-party chat apps.
- **Security Implication:** You have **zero CSRF protection** from the browser. You must implement your own anti-CSRF tokens. This is the state that hackers love because it means the bouncer is on a coffee break.

### The Table of Bouncer Decisions (Cross-Site Request)

Here is the decision matrix for the browser bouncer based on the cookie's setting:

| Request Type | Example Scenario | **Strict** | **Lax** (Default) | **None** |
| :--- | :--- | :--- | :--- | :--- |
| **Link Click** | User clicks `<a href="...">` | ❌ Blocked | ✅ **Allowed** | ✅ Allowed |
| **Form POST** | Hidden form on `evil.com` | ❌ Blocked | ❌ Blocked | ✅ Allowed |
| **Image/Iframe** | `<img src="...">` | ❌ Blocked | ❌ Blocked | ✅ Allowed |
| **JavaScript Fetch** | `fetch('...')` | ❌ Blocked | ❌ Blocked | ✅ Allowed |

### The Crucial Bypass Window (2-Minute Lax-Allowing-Unsafe)
Since you're building a deep understanding, you should know about the **Lax + POST** grace period.

There is an exception to the Lax rule above. To prevent breaking single sign-on flows (where you log in and get immediately redirected via POST), Chrome has a **2-minute timer**.
- **The Trick:** If the website **sets a new cookie** (or updates an existing one), for the **first 2 minutes** after that cookie is set, `SameSite=Lax` behaves **exactly like** `SameSite=None`.
- **The Bypass:** If a hacker can force the website to issue you a **new** session cookie (perhaps via a cookie fixation vulnerability or an OAuth redirect loop), they have a **2-minute window** to launch a **POST**-based CSRF attack, even against `Lax` cookies.

This is why the text says *"it's essential to have a solid grasp of how these restrictions work, as well as how they can potentially be bypassed"*—because even "Default Lax" isn't a perfect silver bullet.