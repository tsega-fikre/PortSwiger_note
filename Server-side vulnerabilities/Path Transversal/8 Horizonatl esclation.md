## Understanding Horizontal Privilege Escalation

### The Core Concept

```
Normal User Access:
┌─────────────────┐     ┌─────────────────┐
│   User A (123)  │────▶│   User A's      │
│                 │     │   Resources     │
└─────────────────┘     └─────────────────┘

Horizontal Privilege Escalation:
┌─────────────────┐     ┌─────────────────┐
│   User A (123)  │────▶│   User B's      │
│                 │     │   Resources     │
└─────────────────┘     └─────────────────┘
         │                      ▲
         │                      │
         └──────────────────────┘
              Unauthorized Access
```

Unlike vertical escalation (going from user to admin), horizontal escalation involves accessing **same-level resources** belonging to **different users**.

## Insecure Direct Object References (IDOR)

### What is IDOR?

IDOR is a specific type of access control vulnerability where an application exposes direct references to internal objects (like database records, files, etc.) without proper authorization checks.

```javascript
// The IDOR pattern
Resource URL: /api/users/123
                   │   └─ Direct object reference (user ID)
                   └──── Resource type

// Attacker modifies: /api/users/456
// Gets another user's data
```

## Real-World Examples of Horizontal Privilege Escalation

### 1. **Sequential ID Manipulation**

```http
// Bank transaction records
GET /api/transactions?id=1001  HTTP/1.1  // User's own transaction
Cookie: session=abc123

// Attacker tries:
GET /api/transactions?id=1002  HTTP/1.1  // Someone else's transaction
GET /api/transactions?id=1003  HTTP/1.1  // Another user's transaction
GET /api/transactions?id=1004  HTTP/1.1  // And so on...
```

### 2. **UUID/GUID Exposure**

```javascript
// Even with complex IDs, exposure can happen
// Document sharing feature
https://docs.example.com/document/550e8400-e29b-41d4-a716-446655440000

// If these UUIDs are exposed elsewhere (shared links, emails, etc.)
// Users can access any document whose ID they discover
```

### 3. **Filename/Directory Traversal**

```http
// Profile picture access
GET /avatars/user_123_profile.jpg

// Attacker tries to access other users' pictures
GET /avatars/user_456_profile.jpg
GET /avatars/user_789_profile.jpg
```

### 4. **Multi-Step Horizontal Escalation**

```javascript
// Step 1: Access own invoice
GET / api / invoice / 2024 / 03 / 12345;

// Step 2: Notice pattern (year/month/invoice_number)
// Step 3: Access other invoices
GET / api / invoice / 2024 / 03 / 12346; // Another user's invoice
GET / api / invoice / 2024 / 02 / 12234; // Different month, different user
```

## The Anatomy of an IDOR Vulnerability

### How Applications Become Vulnerable

```javascript
// ❌ VULNERABLE: Direct object reference without ownership check
app.get("/api/order/:orderId", (req, res) => {
  const orderId = req.params.orderId;

  // BAD: No check if this order belongs to the logged-in user
  const order = database.getOrder(orderId);

  if (order) {
    res.json(order); // Returns ANY order!
  } else {
    res.status(404).json({ error: "Order not found" });
  }
});

// What's missing: Ownership verification
// The code checks if the order EXISTS, not if the user OWNS it
```

### Common IDOR Patterns

```javascript
// Pattern 1: Numeric IDs
/api/user/1
/api/user/2
/api/user/3

// Pattern 2: Base64 encoded references
/api/document/MTIzNA==  // Decodes to "1234"
/api/document/MTIzNQ==  // Decodes to "1235"

// Pattern 3: Hashed/obfuscated but predictable
/api/message/20240315_123_hash123
/api/message/20240315_124_hash456  // Predictable increment

// Pattern 4: Email/username references
/profile?user=john.doe@example.com
/profile?user=jane.smith@example.com
```

## Detection and Exploitation Techniques

### 1. **Parameter Fuzzing**

```python
# Automated testing for IDOR
import requests

def test_idor(base_url, session_cookie, valid_id, id_range):
    headers = {'Cookie': f'session={session_cookie}'}

    for test_id in id_range:
        if test_id == valid_id:
            continue

        url = base_url.replace('{id}', str(test_id))
        response = requests.get(url, headers=headers)

        if response.status_code == 200:
            print(f"Potential IDOR: {url} returned 200")
            # Check if data belongs to another user
            analyze_response(response.json())
```

### 2. **Mass Assignment/Parameter Pollution**

```http
# Original request
POST /api/update-profile
{
    "user_id": 123,
    "email": "user@example.com"
}

# Attacker modifies
POST /api/update-profile
{
    "user_id": 456,  # Changed to another user
    "email": "hacker@example.com"
}
```

## Prevention Strategies

### 1. **Indirect Reference Maps**

```javascript
// ✅ SECURE: Use indirect, user-specific references
class SecureReferenceMapper {
  constructor() {
    // Server-side mapping of indirect to direct references
    this.userReferenceMap = new Map();
  }

  createReference(userId, resourceId) {
    // Generate a random, unique reference for this user-resource pair
    const refId = crypto.randomBytes(16).toString("hex");

    // Store mapping
    this.userReferenceMap.set(`${userId}:${refId}`, {
      userId: userId,
      resourceId: resourceId,
    });

    return refId;
  }

  getResource(userId, refId) {
    // Look up the mapping
    const mapping = this.userReferenceMap.get(`${userId}:${refId}`);

    if (!mapping) {
      return null; // No access
    }

    // Return actual resource ID
    return mapping.resourceId;
  }
}

// Usage
app.get("/api/document/:refId", (req, res) => {
  const userId = req.session.userId;
  const refId = req.params.refId;

  // Convert indirect reference to direct
  const documentId = referenceMapper.getResource(userId, refId);

  if (!documentId) {
    return res.status(403).json({ error: "Access denied" });
  }

  // Now safely access the document
  const document = database.getDocument(documentId);
  res.json(document);
});
```

### 2. **Ownership Verification Middleware**

```javascript
// ✅ SECURE: Verify resource ownership
const verifyOwnership = (resourceType) => {
  return async (req, res, next) => {
    const userId = req.session.userId;
    const resourceId = req.params.id || req.body.id;

    try {
      // Check if user owns this resource
      const ownsResource = await database.checkOwnership(
        userId,
        resourceType,
        resourceId,
      );

      if (!ownsResource) {
        // Log potential attack
        securityLog.warn(
          `User ${userId} attempted to access ${resourceType} ${resourceId} without ownership`,
        );

        return res.status(403).json({
          error: "Access denied",
        });
      }

      // Add resource to request for later use
      req.resource = { type: resourceType, id: resourceId };
      next();
    } catch (error) {
      res.status(500).json({ error: "Server error" });
    }
  };
};

// Apply to routes
app.get("/api/order/:id", verifyOwnership("order"), (req, res) => {
  // Safe to access order - ownership verified
  const order = database.getOrder(req.resource.id);
  res.json(order);
});
```

### 3. **Hierarchical Access Control**

```javascript
// ✅ SECURE: Check access based on relationships
class AccessControl {
  async canAccess(userId, resourceType, resourceId, accessType = "read") {
    // Get the resource's owner
    const resource = await database.getResource(resourceType, resourceId);

    if (!resource) {
      return false;
    }

    // Direct ownership
    if (resource.ownerId === userId) {
      return true;
    }

    // Shared with user
    if (await this.isSharedWithUser(userId, resourceType, resourceId)) {
      return true;
    }

    // User is in a group that has access
    if (await this.groupHasAccess(userId, resourceType, resourceId)) {
      return true;
    }

    // Admin override (vertical check)
    if ((await this.isAdmin(userId)) && accessType === "read") {
      return true; // Admins can read all
    }

    return false;
  }

  async isSharedWithUser(userId, resourceType, resourceId) {
    // Check sharing table
    return database.checkShare({
      userId: userId,
      resourceType: resourceType,
      resourceId: resourceId,
    });
  }
}
```

### 4. **Context-Aware Authorization**

```javascript
// ✅ SECURE: Consider context in access decisions
class ContextAwareAuth {
  async authorize(request) {
    const { user, resource, action, context } = request;

    // Base access rules
    if (resource.ownerId === user.id) {
      return true; // Own resources always accessible
    }

    // Check relationship context
    if (
      context.department &&
      resource.department === user.department &&
      action === "read"
    ) {
      // Read access to same department
      return true;
    }

    // Check temporal context
    if (
      context.project &&
      (await this.isProjectMember(user.id, context.project))
    ) {
      // Project members can access project resources
      return true;
    }

    // Default deny
    return false;
  }
}
```

## Testing for Horizontal Privilege Escalation

### Security Test Checklist

```javascript
// 1. Create two test users
const userA = await createTestUser();
const userB = await createTestUser();

// 2. Create resources for each user
const resourceA = await createResource(userA.id, "A's private data");
const resourceB = await createResource(userB.id, "B's private data");

// 3. Test User A accessing User B's resources
const userASession = await login(userA);
const result = await request(userASession).get(`/api/resource/${resourceB.id}`);

// 4. Verify access is denied
expect(result.status).toBe(403);

// 5. Try various ID formats and encoding
const attempts = [
  resourceB.id,
  resourceB.id + 1,
  encodeURIComponent(resourceB.id),
  Buffer.from(resourceB.id).toString("base64"),
  `../../users/${resourceB.id}/data`,
];

for (const attempt of attempts) {
  const response = await request(userASession).get(`/api/resource/${attempt}`);

  if (response.status === 200) {
    console.log(`IDOR found with: ${attempt}`);
  }
}
```

## Real-World Impact Scenarios

### Healthcare Application

```javascript
// Vulnerability: Medical records accessible by ID
GET / api / patient / records / 12345; // User A's records
GET / api / patient / records / 12346; // User B's records

// Impact:
// - HIPAA violation
// - Exposure of sensitive medical data
// - Identity theft
// - Legal penalties ($50k+ per record)
```

### Financial Services

```javascript
// Vulnerability: Bank statements by account number
GET /api/statements?account=12345678

// Impact:
// - Financial data exposure
// - Transaction history leakage
// - Potential for fraud
// - Regulatory fines
```

## Key Takeaways

1. **Horizontal escalation is about accessing same-level resources belonging to others**
2. **IDOR is the most common vector for horizontal privilege escalation**
3. **Never trust user-supplied references to access resources**
4. **Always verify ownership before granting access**
5. **Use indirect references or access control lists**
6. **Implement defense in depth with multiple verification layers**
7. **Log and monitor unusual access patterns**

The fundamental principle is that **access to resources must be based on the authenticated user's relationship to those resources, not on the ability to supply a valid resource identifier**.
