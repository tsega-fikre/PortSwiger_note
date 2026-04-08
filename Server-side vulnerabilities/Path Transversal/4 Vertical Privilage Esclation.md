

## How Vertical Privilege Escalation Occurs

### 1. **URL/Direct Access Attacks**
```javascript
// Normal user access
https://example.com/dashboard

// Admin functionality (poorly protected)
https://example.com/admin/delete-user?id=123
```
- Non-admin users directly accessing admin URLs
- No proper authorization check on the server

### 2. **Role/Privilege Manipulation**
```http
POST /api/update-profile HTTP/1.1
Cookie: session=abc123
{
    "email": "user@example.com",
    "role": "admin",  // User modifying their own role
    "isAdmin": true   // Tampering with privilege flags
}
```

### 3. **HTTP Method Manipulation**
```http
# Instead of GET (which might be blocked)
POST /admin/deleteUser HTTP/1.1
X-HTTP-Method-Override: GET  # Bypassing method restrictions
```

## Real-World Examples

### Example 1: Hidden Admin Panel
```html
<!-- Developer left this in production but "hidden" -->
<!-- <a href="/admin/console">Admin Access</a> -->
```
- User discovers `/admin/console` through directory enumeration
- No authentication check on the admin page

### Example 2: Parameter-Based Privileges
```http
GET /api/user/data HTTP/1.1
Cookie: session=user123
X-User-Role: admin  # Server trusts this client-side header
```

## Prevention Techniques

### 1. **Server-Side Authorization**
```python
# BAD - Only client-side check
def admin_page(request):
    if request.session.get('isAdmin'):  # Client-controlled data
        return render_admin_panel()

# GOOD - Server-side verification
def admin_page(request):
    user = get_authenticated_user(request.session['user_id'])
    if user.role == 'admin':  # Server-side database check
        return render_admin_panel()
    return access_denied()
```

### 2. **Principle of Least Privilege**
```javascript
// Role-based access control (RBAC)
const permissions = {
    'user': ['read:own_profile', 'update:own_profile'],
    'editor': ['read:all_posts', 'create:post', 'update:any_post'],
    'admin': ['*']  // All permissions
};

function checkPermission(user, action) {
    return permissions[user.role].includes(action) || 
           permissions[user.role].includes('*');
}
```

### 3. **Centralized Access Control**
```java
@PreAuthorize("hasRole('ADMIN')")  // Spring Security annotation
@GetMapping("/admin/users")
public String listUsers() {
    // Method only accessible to admins
    return userService.getAllUsers();
}
```

## Testing for Vertical Privilege Escalation
1. **Enumerate accessible endpoints** as low-privilege user
2. **Try accessing admin functions** directly
3. **Modify role/privilege indicators** in requests
4. **Check for hidden parameters** like `admin=true`
5. **Test HTTP method variations** to bypass restrictions

