## ❓ Reflected XSS — Common Questions

### 🔹 Reflected XSS vs Stored XSS

**Reflected XSS:**  
The payload comes from the **HTTP request** and is immediately reflected in the response.

`Request → Server → Immediate Response → XSS`

**Stored XSS:**  
The payload is **stored by the application** and executed later when someone views the affected page.

`Input → Database → Later Response → XSS`

### 🔹 Reflected XSS vs Self-XSS

**Reflected XSS** can normally be triggered through a crafted request or URL sent to the victim.

**Self-XSS** requires the victim to **manually submit/paste the payload themselves**, usually after being tricked by an attacker.

🧠 **Key difference:**  
`Reflected = Attacker can deliver the payload`  
`Self-XSS = Victim must execute/submit it themselves`