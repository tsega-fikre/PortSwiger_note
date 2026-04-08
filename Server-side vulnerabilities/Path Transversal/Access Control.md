

## Common Types of Access Control Vulnerabilities

### 1. **Vertical Privilege Escalation**
- A standard user gains admin-level privileges
- Example: Modifying a URL from `/user/profile` to `/admin/delete-user`

### 2. **Horizontal Privilege Escalation**
- Accessing another user's data with same privilege level
- Example: Changing `account_id=123` to `account_id=456` in a request

### 3. **Insecure Direct Object References (IDOR)**
- Direct access to objects without proper authorization checks
- Example: Accessing `/download?file=../../etc/passwd`

### 4. **Missing Function-Level Access Control**
- Unprotected administrative functions
- Example: `/admin/deleteUser` accessible without admin privileges

## Real-World Impact
- Data breaches
- Unauthorized transactions
- Account takeover
- Privacy violations

## Prevention Strategies
- Implement principle of least privilege
- Use centralized access control mechanisms
- Deny by default
- Test access controls thoroughly
- Avoid relying on obscurity or client-side controls
