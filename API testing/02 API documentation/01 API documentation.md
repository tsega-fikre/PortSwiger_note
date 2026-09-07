## API Documentation

API documentation is basically a **manual for using an API**. It tells developers what endpoints exist, what requests they should send, and what responses they can expect.

There are two main types:

### 1. Human-readable documentation

Designed for people to read.

It may explain:

```text
Endpoint: GET /api/books
Purpose: Get a list of books
Parameters: category, page
Authentication: Required
Example response: {...}
```

Common examples:

* Swagger UI
* API reference pages
* Developer guides

### 2. Machine-readable documentation

Designed for tools and software to process automatically.

Common formats:

```text
JSON
XML
OpenAPI / Swagger
```

For example, an OpenAPI file might define:

```json
{
  "paths": {
    "/api/books": {
      "get": {}
    }
  }
}
```

Tools can use this information to automatically understand and interact with the API.

### Why is documentation important for pentesting?

Documentation can reveal the API's **attack surface** before you start testing.

You may discover:

```text
/api/users
/api/books
/api/orders
/api/admin
/api/payments
```

along with:

* HTTP methods
* parameters
* authentication requirements
* request formats
* response formats

So if documentation is publicly available, **review it early during reconnaissance**.

> **Pentester mindset:** Don't assume the frontend shows you everything. API documentation may reveal endpoints that aren't visible in the website UI.
