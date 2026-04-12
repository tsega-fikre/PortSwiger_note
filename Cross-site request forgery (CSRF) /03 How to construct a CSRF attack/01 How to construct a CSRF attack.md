Let me explain that in plain, step-by-step language — no prior Burp Suite knowledge assumed.

---

**The problem:**  
Manually writing the HTML for a CSRF attack is annoying if the request has lots of details (like 10 hidden fields, special headers, or weird formatting).

**The solution:**  
Use a tool like Burp Suite Professional’s **CSRF PoC Generator** — it builds the attack HTML for you automatically.

---

### Step-by-step (like you’re doing it yourself):

1. **Capture a normal request**  
   In Burp Suite, find the request that does the sensitive action (e.g., `POST /email/change` with `email=something`).  
   This is the request you want to trick the victim into sending.

2. **Open the CSRF PoC generator**  
   Right-click that request → `Engagement tools` → `Generate CSRF PoC`.

3. **Burp builds the HTML for you**  
   It creates a complete HTML page with:
   - A hidden form containing all the necessary parameters
   - An auto-submit script
   - Any special headers or quirks handled automatically

4. **Tweak if needed**  
   Sometimes a request is unusual (e.g., needs a specific `Content-Type` or uses JSON). The generator lets you adjust options to handle those rare cases.

5. **Copy and test**  
   - Copy the generated HTML into a file (e.g., `attack.html`)
   - Open it in a browser where you’re logged into the vulnerable website
   - See if the request fires successfully and the action (like email change) actually happens

---

### Why this is useful:

- **Saves time** – No manual typing of form fields.
- **Reduces errors** – The tool handles escaping, encoding, and formatting.
- **Handles complex requests** – Works even if the request has many parameters or unusual structure.

---

### In simple terms:

> The CSRF PoC Generator is like a “cheat code” that turns any normal request into a ready-to-use attack page. You just copy, paste, and test.

Once you confirm it works, you host that HTML somewhere (or email it to the victim), and wait for them to visit it while logged in.