This section isolates the **maximum security, minimum convenience** setting.

You already know `Strict` is the "Hermit" mode. Now let's examine the exact browser behavior described in that paragraph and the real-world **UX nightmare** it creates.

### The Strict Logic (The Address Bar Check)

The text defines the rule perfectly:
> *"...if the target site for the request does not match the site currently shown in the browser's address bar, it will not include the cookie."*

Let's visualize that with two scenarios.

**Scenario 1: Same-Site (Cookie Sent)**
- **Address Bar:** `https://bank.com/dashboard`
- **Action:** You click a button to view `/settings`.
- **Target:** `https://bank.com/settings`
- **Bouncer Check:** Address Bar (`bank.com`) **matches** Target (`bank.com`).
- **Verdict:** ✅ **Cookie Sent.**

**Scenario 2: Cross-Site (Cookie Blocked)**
- **Address Bar:** `https://google.com/search?q=bank`
- **Action:** You click a search result link to `https://bank.com/login`.
- **Target:** `https://bank.com/login`
- **Bouncer Check:** Address Bar (`google.com`) does **NOT match** Target (`bank.com`).
- **Verdict:** ❌ **Cookie Blocked.** You arrive at the bank **logged out**.

### The "Negative Impact on User Experience" (The Email Problem)

The text warns about this. Here is the classic pain point that makes developers hate `Strict`:

**The Email Verification Flow:**
1.  You sign up for a SaaS app (`app.saas.com`).
2.  You log in. The app sets `SameSite=Strict`.
3.  You check your **Gmail**. You click the **"Verify Email"** link (`app.saas.com/verify?token=123`).
4.  **Expected:** Link opens, you are logged in, email is verified.
5.  **Reality with Strict:** Link opens in a new tab. The **Address Bar is Gmail**. The **Target is app.saas.com**. Mismatch. **You are logged out.**
6.  **User Experience:** The user sees a "Login Required" page. They log in again, and the token is lost. They click "Resend Email." They get angry.

### Why Use Strict Despite the Pain?

The text says it's recommended for:
> *"...cookies that enable the bearer to modify data or perform other sensitive actions..."*

**The Strategy: Separate Your Cookies.**
Smart developers don't put `Strict` on the **main session cookie**. They use two cookies:

| Cookie Name | Purpose | SameSite Setting | Reason |
| :--- | :--- | :--- | :--- |
| `session` | **Read-Only** (Browsing) | `Lax` | So links from Google work. |
| `session-rw` | **Read-Write** (Admin/Payment) | `Strict` | Critical actions require a fresh cross-site check. |

**The Attack Prevention:**
Even if a hacker tricks you into clicking a link from an email (`Lax` allows this for GET requests), the `session-rw` cookie required for the **POST** action to `/delete-account` will **never** be sent because it's `Strict`.

### The Edge Case: The "Open in New Tab" Reflex

Here's a subtle bypass awareness note for your testing. `Strict` blocks links from Google, but what if the user **Right-Clicks > Open in New Tab**?

- **Result:** Same block. The **referrer** is still Google. The new tab's **initial** navigation is cross-site. Strict blocks it.

### Summary Comparison (Strict vs. Lax)

| Event | **Strict** | **Lax** |
| :--- | :--- | :--- |
| Click link from Email to Site | ❌ Logged Out | ✅ Logged In |
| Click link from Twitter to Site | ❌ Logged Out | ✅ Logged In |
| Hidden POST form on evil.com | ❌ Blocked | ❌ Blocked |
| Image tag on evil.com | ❌ Blocked | ❌ Blocked |