## 🔎 How to Find & Test Stored XSS

Stored XSS testing is mainly about finding the connection between **where attacker-controlled data enters the application** and **where that data later appears**.

### 🧪 Steps

**1. Find Entry Points**  
Test places where data can enter the application:

- URL parameters
    
- Request body
    
- URL paths
    
- Relevant HTTP headers
    
- Other features such as comments, messages, profiles, emails, etc.
    

**2. Find Exit Points**  
Look for places where the submitted data might appear later in responses.

**3. Track Your Input**  
Submit a unique value such as:

```text
x7Kp92Lm
```

Then navigate through relevant pages and check where that value appears.

**4. Confirm It Is Stored**  
Make sure the value appears in a **later request/response**, not just in the immediate response.

**5. Identify the Context**  
Determine whether your input appears in:

- HTML
    
- Attribute
    
- JavaScript
    
- Other contexts
    

**6. Test an XSS Payload**  
Choose a payload appropriate for that context and verify execution in a browser.

### 🧠 Simple Workflow

`Entry Point → Stored → Exit Point → Find Context → Test Payload → Confirm`

💡 **Key difference from Reflected XSS:**  
With Stored XSS, you must track the data **across multiple requests**.