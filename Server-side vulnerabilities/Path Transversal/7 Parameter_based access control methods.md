

## The Fundamental Concept: Trust Boundaries

### What is a Trust Boundary?

```
[Client/Browser] <-- UNTRUSTED ZONE --> [Server] <-- TRUSTED ZONE -->
     ^                    ^                    ^
     |                    |                    |
  User input          Network traffic       Server-side
  can be              can be                data is
  manipulated         intercepted           controlled
```

**Key Principle**: Anything that comes from the client (browser) crosses a trust boundary and cannot be relied upon for security decisions.

## Why Parameter-Based Access Control Fails

### The Problem with Client-Side State

```javascript
// ❌ INSECURE: Trusting client-supplied parameters
app.get('/login/home.jsp', (req, res) => {
    // BAD: Using URL parameter for authorization
    const isAdmin = req.query.admin === 'true';
    const userRole = req.query.role;
    
    if (isAdmin || userRole === '1') {
        // Grant admin access based on client data!
        res.render('admin_dashboard');
    } else {
        res.render('user_dashboard');
    }
});

// What happens:
// 1. User visits: /login/home.jsp?admin=true
// 2. Server says: "Oh, admin=true? You must be admin!"
// 3. User gets admin access
```

## Common Insecure Patterns

### 1. **Hidden Form Fields**
```html
<!-- INSECURE: Hidden field storing role -->
<form action="/update-profile" method="POST">
    <input type="hidden" name="role" value="user">
    <input type="text" name="email" value="user@example.com">
    <input type="submit" value="Update">
</form>

<!-- Attacker modifies in browser tools -->
<input type="hidden" name="role" value="admin">
```

### 2. **Cookies for Authorization**
```http
// INSECURE: Cookie storing privileges
GET /dashboard HTTP/1.1
Cookie: session=abc123; isAdmin=false; role=user

// Attacker modifies:
Cookie: session=abc123; isAdmin=true; role=admin
```

### 3. **Query String Parameters**
```
https://example.com/api/users?view=own                    # Normal
https://example.com/api/users?view=all&admin=true         # Manipulated
https://example.com/dashboard?privilegeLevel=1            # 1=user, 2=admin
```

## The Root Cause: Confusing Data with Authority

```javascript
// The conceptual mistake
class InsecureAuthorization {
    constructor(clientData) {
        // Treating client-provided data as truth
        this.role = clientData.role;        // ❌ From URL/cookie
        this.permissions = clientData.perms; // ❌ From hidden field
        this.isAdmin = clientData.admin;     // ❌ From query param
    }
    
    hasAccess(resource) {
        return this.permissions.includes(resource);
    }
}

// How it should work
class SecureAuthorization {
    constructor(sessionId) {
        // Server-side truth source
        this.sessionData = database.getSession(sessionId); // ✅
        this.user = database.getUser(this.sessionData.userId); // ✅
    }
    
    hasAccess(resource) {
        // Check against server-side data only
        return this.user.role.permissions.includes(resource);
    }
}
```

## Why This Pattern Persists

### Developer Misconceptions
```javascript
// "But the user can't see hidden fields!"
<input type="hidden" name="role" value="user">

// Reality: Browser dev tools make all fields visible/modifiable
```

### Legacy Systems
```java
// Old JSP/ASP patterns from early web
<% 
    // Getting role from request parameter
    String role = request.getParameter("role"); 
    if(role.equals("admin")) {
        // Show admin panel
    }
%>
```

### Convenience Over Security
```python
# "It's easier to just pass the role in the URL"
@app.route('/dashboard')
def dashboard():
    role = request.args.get('role', 'user')
    # Quick and dirty access control
    return render_template(f'{role}_dashboard.html')
```

## Proper Server-Side Session Management

### 1. **Server-Side Session Store**
```javascript
// ✅ SECURE: Server maintains state
class SecureSessionManager {
    constructor() {
        this.sessions = new Map(); // In-memory store (use Redis/DB in production)
    }
    
    createSession(userId) {
        const sessionId = crypto.randomBytes(32).toString('hex');
        
        // Get user data from database
        const user = database.getUser(userId);
        
        // Store session data SERVER-SIDE
        this.sessions.set(sessionId, {
            userId: userId,
            role: user.role,
            permissions: user.permissions,
            createdAt: Date.now(),
            lastAccessed: Date.now()
        });
        
        // Only send session ID to client
        return sessionId;
    }
    
    getSession(sessionId) {
        const session = this.sessions.get(sessionId);
        if (session) {
            session.lastAccessed = Date.now();
        }
        return session;
    }
}

// Usage
app.post('/login', (req, res) => {
    const { username, password } = req.body;
    const user = authenticate(username, password);
    
    if (user) {
        const sessionId = sessionManager.createSession(user.id);
        
        // Send ONLY session ID to client
        res.cookie('sessionId', sessionId, {
            httpOnly: true,  // Prevents JavaScript access
            secure: true,    // HTTPS only
            sameSite: 'strict'
        });
        
        res.json({ success: true });
    }
});

app.get('/dashboard', (req, res) => {
    const sessionId = req.cookies.sessionId;
    const session = sessionManager.getSession(sessionId);
    
    if (!session) {
        return res.status(401).send('Not authenticated');
    }
    
    // Check permissions using SERVER-SIDE data
    if (session.role === 'admin') {
        res.render('admin_dashboard');
    } else {
        res.render('user_dashboard');
    }
});
```

### 2. **JWT with Server-Side Validation**
```javascript
// ✅ BETTER: Signed tokens with minimal client data
const jwt = require('jsonwebtoken');

class TokenManager {
    generateToken(user) {
        // Only include non-sensitive identifier
        const payload = {
            sub: user.id,           // Subject (user ID)
            jti: crypto.randomUUID(), // Unique token ID
            iat: Date.now()          // Issued at
        };
        
        // Sign with server secret
        return jwt.sign(payload, process.env.JWT_SECRET, {
            expiresIn: '1h'
        });
    }
    
    async validateToken(token) {
        try {
            // Verify signature
            const payload = jwt.verify(token, process.env.JWT_SECRET);
            
            // Check with database (token not revoked)
            const isValid = await database.checkToken(payload.jti);
            
            if (!isValid) {
                return null;
            }
            
            // Get fresh user data from database
            const user = await database.getUser(payload.sub);
            return user;
            
        } catch (error) {
            return null;
        }
    }
}

// Usage
app.use(async (req, res, next) => {
    const token = req.headers.authorization?.split(' ')[1];
    
    if (token) {
        // Validate and get fresh user data
        req.user = await tokenManager.validateToken(token);
    }
    
    next();
});
```

### 3. **Authorization Middleware Pattern**
```javascript
// ✅ SECURE: Centralized authorization
const authorize = (requiredRole) => {
    return (req, res, next) => {
        // Session contains user ID, not role directly
        const userId = req.session.userId;
        
        if (!userId) {
            return res.status(401).json({ error: 'Not authenticated' });
        }
        
        // Get fresh user data from database
        const user = database.getUser(userId);
        
        if (!user) {
            return res.status(401).json({ error: 'User not found' });
        }
        
        // Check role against database
        if (user.role !== requiredRole) {
            // Log potential attack
            securityLog.warn(`User ${userId} attempted to access ${requiredRole} resource`);
            
            // Don't reveal why access was denied
            return res.status(403).json({ error: 'Forbidden' });
        }
        
        // Attach user to request for later use
        req.user = user;
        next();
    };
};

// Apply to routes
app.get('/admin', authorize('admin'), (req, res) => {
    // Only reaches here if user is actually admin in database
    res.render('admin_panel');
});

app.get('/profile', authorize('user'), (req, res) => {
    res.render('profile', { user: req.user });
});
```

## Security Concepts to Remember

### 1. **Stateless vs Stateful Sessions**
```javascript
// Stateless (risky if not careful)
JWT with role in payload → Can't be changed without secret

// Stateful (more controllable)
Session ID only → All data server-side → Can be revoked/changed anytime
```

### 2. **Defense in Depth**
```javascript
// Multiple layers of protection
Layer 1: HTTPS (encryption)
Layer 2: HTTP-only cookies (prevents JS access)
Layer 3: Server-side session validation
Layer 4: Database permission checks
Layer 5: Audit logging
```

### 3. **Principle of Least Privilege**
```javascript
// Don't store roles/permissions client-side
// Even encrypted, they represent authority decisions

// Instead, store identifiers and look up privileges:
Client: "I am user 123"
Server: "User 123 has these permissions today"
```

## Common Attack Patterns and Prevention

### Attack: Parameter Tampering
```http
Original: GET /api/account?user_id=123
Modified: GET /api/account?user_id=456&admin=true
```

**Prevention:**
```javascript
// Don't trust user_id from client
app.get('/api/account', (req, res) => {
    // Use session to get user ID
    const userId = req.session.userId; // From secure session
    const account = database.getAccount(userId);
    res.json(account);
});
```

### Attack: Cookie Manipulation
```http
Original: Cookie: role=user; permissions=read
Modified: Cookie: role=admin; permissions=read,write,delete
```

**Prevention:**
```javascript
// Use httpOnly cookies with session IDs only
res.cookie('sessionId', sessionId, {
    httpOnly: true,  // Prevents JavaScript access
    secure: true     // HTTPS only
});
```

## Key Takeaways

1. **Never trust client-supplied security data** - Not in URLs, cookies, or hidden fields
2. **Session IDs should be the only client-stored identifier**
3. **Always verify against server-side data** - Database, session store, etc.
4. **Use httpOnly cookies** - Prevent JavaScript access to session tokens
5. **Implement defense in depth** - Multiple layers of security
6. **Log and monitor** - Track unusual access patterns

The fundamental concept is that **authorization decisions must be made based on server-side truth, not client-supplied assertions**. The client should only prove their identity (via session token), and the server should determine what that identity is allowed to do.