This is a perfect example to dive deep into because it illustrates the core mechanism of an SSRF attack against the server itself. Let's break it down from the perspective of an ethical hacker, explaining both the concept and a detailed demo.

### The Core Concept: The Trusting Server

Think of the server as a high-security building. It has a very strict front gate (the public-facing website) that checks everyone's ID and only lets visitors into the lobby.

- **Normal User (The Attacker):** You are a visitor. You can only access the public lobby (`/product/stock`).
- **The Admin Panel:** Deep inside the building, there's a sensitive room called the "Admin Office" (`/admin`). To get in, you need a special keycard. The guards at *that* door are incredibly strict. They check keycards for everyone.
- **The Stock Checker (The Vulnerability):** The building has a service where you can fill out a form to request a document from the "Warehouse." A trusted internal mail clerk (the server's internal request mechanism) takes your form, goes to the Warehouse, gets the document, and brings it back to you in the lobby.

**The SSRF Attack:** As an attacker, you fill out the form, but instead of asking for a document from the Warehouse, you write: "Please go to the Admin Office and bring me back the first file you see."

The trusted internal mail clerk thinks, "I'm an internal staff member, I can go anywhere!" He walks to the Admin Office. The guard at the Admin Office sees *the mail clerk*, an internal employee, and thinks, "Oh, it's just you. You don't need a keycard. Go on in." The clerk grabs the file and, following your instructions, brings it right back to you in the lobby.

**You have successfully bypassed all security by exploiting the server's trust in itself and its own internal network.**

---

### Detailed Demo: The Ethical Hacker's Perspective

Let's imagine you are an ethical hacker tasked with testing the security of the `weliketoshop.net` application. You've just learned how their stock check feature works.

**Scenario:** An e-commerce site, `www.weliketoshop.net`, has a feature that lets users check product stock in different stores.

#### Step 1: Normal Application Flow (Reconnaissance)

First, you use the application as intended to understand how it works.

1.  **Action:** You browse to a product page and click "Check Stock."
2.  **Browser Request (What your browser sends):**
    ```
    POST /product/stock HTTP/1.1
    Host: www.weliketoshop.net
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 118

    stockApi=http://stock.weliketoshop.net:8080/product/stock/check%3FproductId%3D6%26storeId%3D1
    ```
    - **Explanation:** The `stockApi` parameter tells the server *where* to go to get the stock information. The value is a full URL to their internal stock service.
3.  **Server-Side Action:**
    - The `www.weliketoshop.net` server receives your request.
    - It reads the `stockApi` URL.
    - **Crucially, the server itself acts as a client.** It makes a brand new HTTP request to `http://stock.weliketoshop.net:8080/product/stock/check?productId=6&storeId=1`.
    - The internal stock server (`stock.weliketoshop.net`) processes the request and sends the stock number back to the `www` server.
    - The `www` server takes that number and sends it back to your browser in the HTTP response.
4.  **Result:** You see "Product 6, Store 1: In Stock (45 units)." Everything works as expected.

#### Step 2: The Ethical Hack - Probing for SSRF

As an ethical hacker, you know that user-controllable URLs are a massive red flag. You start by trying to point the server back at itself.

1.  **Action:** You intercept the previous request using a proxy tool like Burp Suite. You modify the `stockApi` parameter.
2.  **Modified Request (The Attack Payload):**
    ```
    POST /product/stock HTTP/1.1
    Host: www.weliketoshop.net
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 118

    stockApi=http://localhost/admin
    ```
    - **Explanation of the Payload:**
        - `http://localhost/`: `localhost` is a hostname that always points back to the computer you're on. By using it here, you are asking the `www.weliketoshop.net` server to make a request to... itself.
        - `admin`: This is a common path for administrative panels. You're guessing there might be an admin interface on the server itself.
3.  **Server-Side Action (The Vulnerability Exploited):**
    - The `www.weliketoshop.net` server receives your modified request.
    - It reads the `stockApi` URL: `http://localhost/admin`.
    - It thinks, "I need to fetch `/admin` from `localhost`." **It does not realize that `localhost` is itself.**
    - The server makes a new, internal HTTP request to `http://localhost/admin`.
    - This request originates *from the server's own IP address* and stays entirely within the server itself (the loopback interface).

#### Step 3: The Bypassed Access Control

This is the most critical part of the attack.

- **Normal User Access:** If you, as an external user, tried to visit `http://www.weliketoshop.net/admin` in your browser, the application would check your session cookie. You don't have admin privileges, so it would return an "Access Denied" or redirect you to the login page.
- **The Server's Internal Request:** When the server makes the request to `http://localhost/admin`, it's a completely new request. It's not carrying *your* browser's session cookie.
    - The code handling the `/admin` endpoint checks the *source* of the request. It sees that the request came from `127.0.0.1` (localhost), which is the server's own IP address.
    - The developer made a critical mistake: they assumed that any request coming from `localhost` is a trusted, internal request that has already been authenticated. Therefore, they bypass all normal authentication checks for requests originating from `localhost`.

#### Step 4: The Result (Proof of Concept)

The internal handler for `/admin` processes the request and sends the HTML content of the admin panel back to the component that made the request—which is the `www` server itself. The `www` server, following its original programming, then takes that content and sends it back to *you* in the HTTP response.

**The Ethical Hacker's Finding:** You now have a full HTML page of the admin panel in your Burp Suite response window. You have successfully proven that you can bypass authentication and access sensitive internal functionality.

### Why This Works: Key Takeaways for an Ethical Hacker

- **The Trust Relationship:** The server trusts itself and its network location (`localhost`). This trust is exploited.
- **Bypassing Network-Based Controls:** Firewalls often block external access to admin panels, but they don't block the server from talking to itself. SSRF bypasses this.
- **The Server as a Proxy:** The vulnerable application becomes an unwitting proxy, fetching content from internal locations and reflecting it back to the attacker.
- **From SSRF to Further Attacks:** Finding an `/admin` panel is just the beginning. The ethical hacker would now explore the admin panel's functionality. Does it have a feature to execute system commands? Upload files? If so, the hacker could chain the SSRF with these features to achieve **Remote Code Execution (RCE)** on the server.