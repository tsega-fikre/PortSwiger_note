## 💥 Impact of Stored XSS

If an attacker can make their JavaScript execute in a victim's browser, they can potentially **perform actions and access data available to that user**.

### 🔥 Why Stored XSS is dangerous

The biggest difference from Reflected XSS is that **Stored XSS is self-contained**.

The attacker doesn't need to send a malicious URL to every victim.

Instead:

`Attacker → Injects Payload → Application Stores It → Victim Visits Page → XSS Executes`

### 🧠 Key Point

With **Stored XSS**, the malicious payload stays inside the application and can affect users whenever they encounter the affected content.

This is especially important for applications where users must be **logged in**, because the payload can execute within their authenticated session.

**Reflected:** Victim must be induced to make the malicious request.  
**Stored:** Payload is already inside the application.