Exactly. This section is teaching you the **first phase of API penetration testing: API reconnaissance and attack-surface discovery**.

The most important idea is:

> **Before trying to exploit an API, understand what exists and how each endpoint is supposed to work.**

Let's break everything down carefully.

---

# 1. What does "attack surface" mean?

The **attack surface** is the collection of all the places where an attacker can interact with an application.

For an API, that includes things like:

```text
/api/login
/api/users
/api/users/101
/api/books
/api/orders
/api/payments
/api/admin/users
/api/upload
```

Each endpoint can potentially have:

* authentication requirements
* authorization requirements
* parameters
* different HTTP methods
* different responses
* different vulnerabilities

So your first job is:

```text
Find API
   ↓
Discover endpoints
   ↓
Understand requests
   ↓
Understand responses
   ↓
Map attack surface
   ↓
Begin security testing
```

---

# 2. What is an API endpoint?

The passage gives:

```http
GET /api/books HTTP/1.1
Host: example.com
```

The endpoint is:

```text
/api/books
```

Think of an endpoint as a **specific address where the API provides a particular function or resource**.

For example:

```text
/api/books
```

could mean:

> Give me the collection of books.

While:

```text
/api/books/123
```

could mean:

> Give me book number 123.

And:

```text
/api/books/mystery
```

could mean:

> Give me mystery books.

---

# 3. Endpoint vs URL vs HTTP request

This distinction is important.

Consider:

```http
GET /api/books/123 HTTP/1.1
Host: example.com
Authorization: Bearer TOKEN
```

### URL

The complete URL is:

```text
https://example.com/api/books/123
```

### Host

```text
example.com
```

### Path

```text
/api/books/123
```

### HTTP method

```text
GET
```

### Endpoint

In common API terminology, we'd refer to:

```text
/api/books/123
```

as the endpoint/path being accessed.

So don't confuse:

```text
GET
```

with:

```text
/api/books/123
```

The method tells you **what operation is being requested**, while the endpoint identifies **where the request is being made**.

---

# 4. Endpoint discovery

Now we reach one of the most important skills in API pentesting:

> **How do you find endpoints?**

You have several sources.

## A. Burp Suite

Suppose you open a website and log in.

Burp captures:

```http
GET /api/profile HTTP/1.1
Host: example.com
```

Then you click "My Orders":

```http
GET /api/orders HTTP/1.1
Host: example.com
```

Then you open an order:

```http
GET /api/orders/1234 HTTP/1.1
Host: example.com
```

You've already discovered:

```text
/api/profile
/api/orders
/api/orders/1234
```

This is why **Burp Proxy + HTTP history** is extremely useful.

---

# 5. JavaScript files

Sometimes the frontend JavaScript contains API paths.

For example:

```javascript
fetch("/api/users")
fetch("/api/orders")
fetch("/api/products")
```

You might discover:

```text
/api/users
/api/orders
/api/products
/api/admin
```

The interesting part is that an endpoint doesn't necessarily need to have a visible button in the UI.

For example, the frontend might never expose:

```text
/api/admin/users
```

but the endpoint could still exist.

That becomes particularly interesting when testing **authorization**.

---

# 6. API documentation

Look for documentation such as:

```text
/swagger
/swagger-ui
/api-docs
/openapi.json
/swagger.json
```

For example:

```text
https://example.com/swagger
```

A Swagger/OpenAPI document might reveal:

```text
GET    /api/users
POST   /api/users
GET    /api/users/{id}
PUT    /api/users/{id}
DELETE /api/users/{id}
```

Now your attack surface becomes much clearer.

---

# 7. Mobile applications

This is particularly important for API pentesting.

Suppose you have an Android application:

```text
Android App
     ↓
API
     ↓
Backend
```

The mobile application may reveal endpoints such as:

```text
/api/login
/api/profile
/api/messages
/api/files
/api/payment
```

You can analyze the application's network traffic in an authorized lab/test environment to discover how it communicates with the backend.

This is one reason **mobile pentesting and API pentesting overlap heavily**.

---

# 8. Endpoint parameters

Finding:

```text
/api/books
```

isn't enough.

You need to determine:

> **What input does this endpoint accept?**

Consider:

```http
GET /api/books?category=mystery
```

Here:

```text
category=mystery
```

is a parameter.

Another example:

```http
GET /api/books?page=2&limit=20
```

Parameters:

```text
page=2
limit=20
```

You need to determine which parameters are:

### Required

For example:

```text
/api/books/{id}
```

might require:

```text
id
```

### Optional

For example:

```text
/api/books?page=2
```

might work without `page`.

Understanding this is important because parameters become potential **input points for security testing**.

---

# 9. JSON body parameters

Parameters aren't limited to the URL.

Consider:

```http
POST /api/users HTTP/1.1
Host: example.com
Content-Type: application/json

{
    "username": "alice",
    "email": "alice@example.com",
    "age": 20
}
```

Here the input parameters are:

```text
username
email
age
```

You should determine:

```text
Which fields are required?
Which fields are optional?
What data types are expected?
What values are accepted?
What happens if a field is missing?
What happens if an unexpected field is added?
```

---

# 10. Why optional parameters are interesting

Suppose the normal request is:

```json
{
    "username": "alice",
    "email": "alice@example.com"
}
```

But you discover the server also accepts:

```json
{
    "username": "alice",
    "email": "alice@example.com",
    "role": "admin"
}
```

Now you need to ask:

> Does the server allow the client to control `role`?

If an ordinary user can modify security-sensitive properties, that could indicate a **Broken Object Property Level Authorization** issue.

This is closely related to what is often called **mass assignment**.

So:

```text
Discover parameters
       ↓
Understand parameters
       ↓
Identify security-sensitive fields
       ↓
Test whether authorization is enforced
```

---

# 11. HTTP methods

The passage says you should discover:

> "the types of requests the API accepts"

This means determining the supported HTTP methods.

For example:

```text
GET
POST
PUT
PATCH
DELETE
```

Imagine you discover:

```http
GET /api/users/101
```

You shouldn't automatically assume that GET is the only operation.

You might discover that the resource also supports:

```http
PUT /api/users/101
```

or:

```http
DELETE /api/users/101
```

The security question becomes:

> **Is this method supposed to be available to this user?**

That's where **Broken Function Level Authorization** can become relevant.

---

# 12. Example: GET vs DELETE

Suppose a normal user can do:

```http
GET /api/users/101
```

But you test whether the same authenticated user can access:

```http
DELETE /api/users/101
```

If the server performs the deletion without proper authorization, that's potentially a serious authorization vulnerability.

The important principle is:

> **Changing the HTTP method can change the action performed by the server.**

---

# 13. Media formats

The API may accept different data formats.

Common ones include:

```text
application/json
application/x-www-form-urlencoded
multipart/form-data
application/xml
```

Example JSON:

```http
Content-Type: application/json

{
    "username": "alice"
}
```

A form request might look like:

```http
Content-Type: application/x-www-form-urlencoded

username=alice
```

A file upload might use:

```http
Content-Type: multipart/form-data
```

Understanding accepted formats is important because different parsers and processing paths can sometimes behave differently.

---

# 14. Authentication mechanisms

Next:

> "Rate limits and authentication mechanisms."

You need to determine **how the API identifies the user**.

Common mechanisms include:

### Session cookies

```http
Cookie: session=abc123
```

### Bearer tokens

```http
Authorization: Bearer eyJhbGciOi...
```

### API keys

```http
X-API-Key: abc123
```

### Basic authentication

```http
Authorization: Basic ...
```

### OAuth

The application may use OAuth access tokens.

### JWT

For example:

```text
eyJhbGciOiJIUzI1NiIs...
```

Understanding authentication is essential before testing authorization.

---

# 15. Authentication vs authorization

This distinction is **extremely important**.

### Authentication

> **Who are you?**

Example:

```text
Username + password
       ↓
Authenticated as Alice
```

### Authorization

> **What are you allowed to do?**

Example:

```text
Alice
 ↓
Can access Alice's profile
Can access Alice's orders
Cannot access Bob's profile
Cannot delete other users
Cannot access admin functions
```

Therefore:

```text
Authentication ≠ Authorization
```

You can have strong authentication but broken authorization.

For example:

```text
Alice successfully logs in
        ↓
Server knows Alice is Alice
        ↓
Alice requests /api/users/102
        ↓
Server fails to check ownership
        ↓
Bob's information returned
```

Authentication worked.

Authorization failed.

That's essentially the core idea behind **BOLA**.

---

# 16. Rate limits

You also need to determine whether the API has **rate limiting**.

Suppose an endpoint allows:

```http
POST /api/login
```

If the server allows unlimited login attempts, an attacker might repeatedly submit credentials.

A properly designed API may enforce something like:

```text
5 attempts
   ↓
temporary restriction
```

Rate limiting can also matter for:

```text
password reset
OTP verification
SMS requests
resource-intensive searches
file processing
account creation
API calls
```

The exact limits depend on the application.

As a pentester, your goal isn't simply:

> "Can I send lots of requests?"

Instead:

> **Does the application adequately limit sensitive or expensive operations?**

---

# 17. Build an API map

A very useful pentesting habit is to build an endpoint map.

For example:

```text
API
│
├── /api/auth
│   ├── POST /login
│   ├── POST /register
│   └── POST /logout
│
├── /api/users
│   ├── GET /users
│   ├── GET /users/{id}
│   ├── PUT /users/{id}
│   └── DELETE /users/{id}
│
├── /api/books
│   ├── GET /books
│   ├── GET /books/{id}
│   └── POST /books
│
└── /api/orders
    ├── GET /orders
    ├── GET /orders/{id}
    └── POST /orders
```

Then add information about each endpoint:

| Endpoint           | Method | Auth | Parameters         | Purpose      |
| ------------------ | ------ | ---- | ------------------ | ------------ |
| `/api/login`       | POST   | No   | username, password | Login        |
| `/api/users/{id}`  | GET    | Yes  | id                 | Get user     |
| `/api/users/{id}`  | PUT    | Yes  | id, email          | Update user  |
| `/api/orders/{id}` | GET    | Yes  | id                 | Get order    |
| `/api/orders`      | POST   | Yes  | product, quantity  | Create order |

Now you've transformed a vague application into a **structured attack surface**.

---

# 18. What you're really trying to discover

When performing API reconnaissance, your questions should be:

```text
1. What endpoints exist?
        ↓
2. What HTTP methods do they support?
        ↓
3. What parameters do they accept?
        ↓
4. Which parameters are required?
        ↓
5. Which parameters are optional?
        ↓
6. What data formats are accepted?
        ↓
7. Is authentication required?
        ↓
8. How does authentication work?
        ↓
9. What authorization rules exist?
        ↓
10. Are there rate limits?
```

Once you know these things, **then** you can start meaningful vulnerability testing.

---

# 19. A practical Burp Suite mindset

When you intercept an API request in Burp, don't immediately start changing random values.

First read the request systematically:

```http
POST /api/transfer HTTP/1.1
Host: example.com
Authorization: Bearer TOKEN
Content-Type: application/json

{
    "sender": 101,
    "receiver": 102,
    "amount": 50
}
```

Ask:

### Endpoint

```text
/api/transfer
```

### Method

```text
POST
```

### Authentication

```text
Bearer TOKEN
```

### Format

```text
application/json
```

### Parameters

```text
sender
receiver
amount
```

### Questions

```text
Is sender controlled by the client?
Is receiver controlled by the client?
Can I transfer from another account?
Can amount be negative?
Can amount be zero?
Can amount exceed balance?
Is there a race condition?
Are there rate limits?
```

Notice how **reconnaissance naturally leads into vulnerability testing**.

---

# 20. The mental model you should remember

Think of API testing as building a map before attacking:

```text
                API
                 │
        ┌────────┴────────┐
        ↓                 ↓
    Endpoints          Versions
        │
        ↓
   HTTP Methods
        │
        ↓
    Parameters
        │
   ┌────┴─────┐
   ↓          ↓
Required    Optional
   │          │
   └────┬─────┘
        ↓
 Authentication
        ↓
 Authorization
        ↓
 Rate Limits
        ↓
   Attack Surface
        ↓
 Security Testing
```

### The key lesson

**API reconnaissance = discovering what the API exposes and understanding how each exposed function works before testing whether it can be abused.**

And when you're using Burp Suite, every request you capture should become part of your **API inventory** rather than just a request you happen to intercept.
