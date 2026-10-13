## Reflected XSS — Impact

The main idea is:

> **If an attacker can get their JavaScript to execute in a victim's browser, the attacker can generally act with the victim's privileges inside the application.**

### What can the attacker do?

If the victim is logged in, the injected script can potentially:

- **Perform actions** the victim is allowed to perform.
    
- **Read information** that the victim can access.
    
- **Modify information** that the victim can modify.
    
- **Interact with other users** in ways that appear to come from the victim.
    

### How does reflected XSS reach the victim?

Unlike stored XSS, the malicious payload usually needs to be delivered to the victim somehow.

Common delivery methods mentioned in your material include:

```text
Attacker-controlled website
        ↓
Malicious link
        ↓
Victim clicks link
        ↓
Vulnerable application reflects input
        ↓
JavaScript executes in victim's browser
```

The attacker could also send the malicious link through:

- Email
    
- A tweet/social-media message
    
- Another website that allows user-generated content
    

### Reflected vs Stored XSS

The important distinction from your notes is **delivery**:

|Reflected XSS|Stored XSS|
|---|---|
|Payload is reflected in the immediate response|Payload is stored by the application|
|Usually requires an external delivery mechanism|Can be delivered from within the vulnerable application|
|Often requires the victim to visit/click a crafted URL|Victim can encounter the payload while using the application|
|Generally considered less severe than stored XSS in the provided material|Generally more severe because it can be self-contained|

### 🧠 Easy way to remember

**Reflected XSS = attacker sends the victim the payload.**

**Stored XSS = application stores the payload and later serves it to victims.**

The key impact is not simply _"JavaScript runs."_ The serious part is that the JavaScript runs **in the victim's browser with the victim's application context**, potentially allowing actions and access available to that victim.