## 🔥 Stored XSS in Different Contexts

Stored XSS can appear in **different locations within a web response**, and the location determines what type of payload may work.

For example, stored input might appear inside:

- **HTML content**
    
- **HTML attributes**
    
- **JavaScript**
    
- Other parts of the page
    

The application may also **validate, filter, encode, or modify** the input before storing or displaying it.

### 🧠 Key Point

`Where Input Is Stored → Where It Appears in Response → Determines the Payload`

So when testing Stored XSS, don't just ask:

**“Is my input stored?”**

Also ask:

**“Where does my stored input appear, and how is it processed?”**