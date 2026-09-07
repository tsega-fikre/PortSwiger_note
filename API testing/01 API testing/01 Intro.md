This passage is introducing **API testing** from a penetration-testing perspective. Let’s break it down piece by piece and connect it to what you’ll actually do during a pentest.

## 1. What is an API?

**API = Application Programming Interface.**

An API is the communication layer that allows one piece of software to request data or actions from another piece of software.

For example, imagine a banking application.

When you log in, the frontend might send:

```http
POST /api/login
Content-Type: application/json

{
  "username": "alice",
  "password": "Password123"
}
```

The server processes this request and responds:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "user_id": 101,
  "username": "alice"
}
```

The **frontend** is what you see.

The **API** is what the frontend communicates with.

So conceptually:

```text
User
  ↓
Web Browser / Mobile App
  ↓
API
  ↓
Backend Application
  ↓
Database
```

As a penetration tester, you're often testing the communication between the client and backend.

---

# 2. Why is API testing important?

The passage says API vulnerabilities can affect:

### Confidentiality

Confidentiality means:

> Can an attacker access information they shouldn't be able to see?

Example:

```http
GET /api/users/101
```

returns:

```json
{
  "id": 101,
  "name": "Alice",
  "email": "alice@example.com"
}
```

If you change:

```http
GET /api/users/101
```

to:

```http
GET /api/users/102
```

and receive another user's private information, you may have an **authorization vulnerability**, commonly associated with **BOLA/IDOR**.

---

### Integrity

Integrity means:

> Can an attacker modify data they shouldn't be able to modify?

For example:

```http
PUT /api/users/101
```

```json
{
  "email": "attacker@example.com"
}
```

If user 101 belongs to another account and the server doesn't properly verify authorization, an attacker could modify their information.

---

### Availability

Availability means:

> Can an attacker make the service unavailable or consume excessive resources?

Examples include:

* resource exhaustion
* excessive requests
* expensive API operations
* unrestricted file uploads
* denial-of-service conditions

---

# 3. "All dynamic websites are composed of APIs"

This statement needs a little clarification.

It doesn't literally mean **every dynamic website exposes a REST API**.

Rather, modern dynamic applications generally have some kind of programmatic communication between the client and server.

For example:

```text
Browser
   │
   │ HTTP request
   ↓
Backend
   │
   ↓
Database
```

The browser might communicate using:

```text
REST
GraphQL
JSON endpoints
WebSockets
RPC
traditional form requests
```

From a pentesting perspective, you should therefore pay attention to **the requests made by the application**, not only what is visible in the UI.

---

# 4. Why can SQL Injection be considered API testing?

Suppose a website has:

```http
GET /api/products?id=10
```

The backend might execute something conceptually similar to:

```sql
SELECT * FROM products WHERE id = 10;
```

If the API improperly incorporates user input into SQL queries, an SQL injection vulnerability may exist.

So:

```text
API
 ↓
Backend code
 ↓
Database query
 ↓
SQL Injection
```

That's why API testing overlaps heavily with traditional web application security testing.

API testing isn't a completely separate universe.

It includes many vulnerabilities you've probably already encountered:

* SQL injection
* XSS
* SSRF
* authentication flaws
* authorization flaws
* file upload
* command injection
* business logic vulnerabilities
* information disclosure

---

# 5. The important part: APIs not fully used by the frontend

This is one of the most important ideas in the passage.

Imagine a website has these visible features:

```text
Homepage
Login
Profile
Products
Logout
```

But while testing the application, you discover:

```text
GET /api/users
GET /api/admin/users
POST /api/admin/create-user
DELETE /api/admin/user/102
GET /api/debug/config
```

The frontend may not provide buttons for these endpoints.

But that **doesn't necessarily mean the endpoints don't exist or can't be accessed**.

This creates an important pentesting opportunity.

You might discover an endpoint through:

* JavaScript files
* API documentation
* OpenAPI/Swagger files
* mobile applications
* Burp Suite Proxy
* Burp Site Map
* source code
* error messages
* predictable endpoint names
* historical API versions

For example:

```text
Frontend
   │
   ├── /api/profile
   ├── /api/products
   └── /api/orders

But backend also exposes:

   ├── /api/admin/users
   ├── /api/internal/reports
   └── /api/v2/users
```

Those "hidden" endpoints are extremely interesting during API security testing.

---

# 6. RESTful APIs

The passage specifically mentions **RESTful APIs**.

REST commonly uses HTTP methods to represent operations.

| HTTP method | Typical purpose       |
| ----------- | --------------------- |
| GET         | Retrieve data         |
| POST        | Create data/action    |
| PUT         | Replace/update data   |
| PATCH       | Partially update data |
| DELETE      | Delete data           |

Example:

```http
GET /api/users/101
```

means:

> Give me user 101.

While:

```http
DELETE /api/users/101
```

means:

> Delete user 101.

A pentester should therefore test whether the server correctly controls **who is allowed to perform each operation**.

---

# 7. JSON APIs

JSON is one of the most common formats used by modern APIs.

Example:

```json
{
  "username": "alice",
  "email": "alice@example.com",
  "role": "user"
}
```

A request could be:

```http
POST /api/users
Content-Type: application/json
```

```json
{
  "username": "alice",
  "email": "alice@example.com"
}
```

The server might return:

```json
{
  "id": 101,
  "username": "alice",
  "email": "alice@example.com"
}
```

As a pentester, don't automatically assume that every JSON parameter is trustworthy.

You should investigate things such as:

```text
Can I modify parameters?
Can I add parameters?
Can I remove parameters?
Can I change IDs?
Can I change roles?
Can I change ownership?
Can I manipulate quantities?
Can I change prices?
Can I access another user's object?
```

This is where API security becomes very interesting.

---

# 8. Example: BOLA

Consider:

```http
GET /api/accounts/101
Authorization: Bearer USER_A_TOKEN
```

Response:

```json
{
  "id": 101,
  "balance": 5000
}
```

You change:

```text
101 → 102
```

and send:

```http
GET /api/accounts/102
Authorization: Bearer USER_A_TOKEN
```

If the server responds with:

```json
{
  "id": 102,
  "balance": 9000
}
```

you have potentially discovered **Broken Object Level Authorization (BOLA)**.

The key question is:

> Does the server verify that the authenticated user is authorized to access object 102?

This is one of the most important API security concepts.

---

# 9. Server-Side Parameter Pollution

This is another important concept from the passage.

Suppose the application receives:

```http
GET /api/search?query=phone
```

The frontend sends:

```text
query=phone
```

But internally the server might construct another request:

```text
/internal/search?query=phone
```

If your input is inserted into that internal request without proper validation, you may be able to manipulate the parameters of the **internal API request**.

Conceptually:

```text
Attacker
   ↓
Public API
   ↓
Backend constructs request
   ↓
Internal API
   ↓
Database/service
```

The interesting part is the second API:

```text
Public API → Internal API
```

You're testing whether your input can interfere with that internal communication.

This is called **Server-Side Parameter Pollution (SSPP)**.

---

# 10. Why internal APIs matter

A company might have:

```text
Internet
   ↓
Public API
   ↓
Backend
   ├── User API
   ├── Payment API
   ├── Admin API
   └── Internal service
```

The internal APIs may assume:

> "Only our trusted backend can call me."

But if an attacker can manipulate requests sent by the public API to the internal API, that assumption may break.

That's why SSPP can sometimes become a path toward:

```text
Public endpoint
      ↓
Parameter manipulation
      ↓
Internal API manipulation
      ↓
Unauthorized behavior
```

---

# 11. OWASP API Security Top 10

The passage then connects API testing to the **OWASP API Security Top 10 (2023)**.

A useful way to think about the list is:

| #     | Vulnerability                                   | Simple meaning                                             |
| ----- | ----------------------------------------------- | ---------------------------------------------------------- |
| API1  | Broken Object Level Authorization               | Accessing another user's object                            |
| API2  | Broken Authentication                           | Authentication can be bypassed/abused                      |
| API3  | Broken Object Property Level Authorization      | Accessing/modifying properties you shouldn't               |
| API4  | Unrestricted Resource Consumption               | Making the API consume excessive resources                 |
| API5  | Broken Function Level Authorization             | Accessing functions you're not authorized to use           |
| API6  | Unrestricted Access to Sensitive Business Flows | Abusing important business operations                      |
| API7  | Server Side Request Forgery                     | Making the server request attacker-controlled destinations |
| API8  | Security Misconfiguration                       | Unsafe configuration                                       |
| API9  | Improper Inventory Management                   | Unknown/old/shadow APIs                                    |
| API10 | Unsafe Consumption of APIs                      | Trusting external APIs unsafely                            |

The key thing is that **OWASP API Top 10 is not separate from web security**.

It organizes API-specific risks around the way modern applications communicate.

---

# 12. The most important mindset

When testing an API, don't only ask:

> "What does this endpoint do?"

Ask:

### Authentication

```text
Who am I?
```

### Authorization

```text
What am I allowed to access?
```

### Object authorization

```text
Can I access someone else's object?
```

### Function authorization

```text
Can I perform an administrative operation?
```

### Input validation

```text
What happens if I manipulate this parameter?
```

### Business logic

```text
Can I make the application perform something that shouldn't be possible?
```

### Resource consumption

```text
Can I make the server perform excessive work?
```

### API inventory

```text
What endpoints, versions, and APIs exist?
```

### Internal communication

```text
Does this API communicate with another internal API?
```

---

# 13. How this fits into your pentesting workflow

For an authorized lab or assessment, think of API testing as:

```text
1. Discover
      ↓
2. Understand
      ↓
3. Map endpoints
      ↓
4. Identify authentication
      ↓
5. Identify authorization
      ↓
6. Manipulate parameters
      ↓
7. Test business logic
      ↓
8. Test API-specific vulnerabilities
      ↓
9. Validate impact
      ↓
10. Document findings
```

For example:

```text
Burp Suite
    ↓
Capture API request
    ↓
POST /api/transfer
    ↓
Understand JSON parameters
    ↓
sender = 101
receiver = 102
amount = 50
    ↓
Test authorization
    ↓
Test parameter manipulation
    ↓
Test business logic
    ↓
Test race conditions
    ↓
Document result
```

This connects directly to the API labs you've been studying.

---

## The big picture

The passage is essentially telling you:

> **Don't limit penetration testing to what the website's interface allows you to see. Find the APIs behind the application, understand how they communicate, and test whether the server properly enforces authentication, authorization, input validation, business rules, and resource limits.**

And the **OWASP API Security Top 10** gives you a structured checklist for doing that.

[PortSwigger API testing and OWASP API Security Top 10](https://portswigger.net/web-security/api-testing/top-10-api-vulnerabilities?utm_source=chatgpt.com)

If you're aiming to become strong at API pentesting, the next concepts I'd learn deeply are **BOLA → Broken Function Level Authorization → Mass Assignment/Property-Level Authorization → API discovery → JWT/authentication → SSRF → SSPP → business-logic testing**.
