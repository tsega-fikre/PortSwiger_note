# 🔑 **Brute-Forcing Passwords - Understanding Human Behavior**

## 🧠 **The Psychology of Passwords**

Think of how humans actually create passwords vs. how computers would:

```
COMPUTER'S IDEA OF A "STRONG" PASSWORD:
"k#8mP@9$qL2!xR5" - Random, secure, impossible to remember

HUMAN'S IDEA OF A "STRONG" PASSWORD:
"Password123!" - Follows rules, easy to remember, actually weak!
```

## 📊 **How Humans "Game" Password Policies**

### **Common User Transformations**

```
Base Word: "password"

Policy Requires:
┌─────────────────────────────────────┐
│ ✓ 8+ characters                     │
│ ✓ Uppercase letter                  │
│ ✓ Number                           │
│ ✓ Special character                 │
└─────────────────────────────────────┘

Users transform "password" into:
┌─────────────────────────────────────┐
│ Password1!      (Capitalize + number + !)│
│ P@ssw0rd       (Leet speak)              │
│ Password123!   (Add numbers + !)          │
│ Passw0rd!      (Mix leet + special)       │
│ Password1?     (Change special)           │
└─────────────────────────────────────┘
```

## 🎪 **Real-World Transformation Patterns**

### **1. The "Add a Number" Pattern**
```python
# Users always add numbers at the end
base_passwords = ["password", "welcome", "admin", "summer"]

common_transformations = []
for pwd in base_passwords:
    # Add years
    common_transformations.extend([
        pwd + "2023",
        pwd + "2024",
        pwd + "2025",
        pwd + "23",
        pwd + "24",
    ])
    
    # Add sequential numbers
    common_transformations.extend([
        pwd + "123",
        pwd + "1234",
        pwd + "12345",
    ])
    
    # Add common numbers
    common_transformations.extend([
        pwd + "1",
        pwd + "12",
        pwd + "123",
        pwd + "007",
        pwd + "69",
    ])
```

### **2. The "Capitalize First Letter" Pattern**
```python
# Users always capitalize the first letter
words = ["summer", "winter", "spring", "autumn", "monkey", "dragon"]

capitalized = [word.capitalize() + "!" for word in words]
# Results: Summer!, Winter!, Spring!, etc.

# Combine with numbers
variations = []
for word in words:
    variations.extend([
        word.capitalize() + "123",
        word.capitalize() + "2024",
        word.capitalize() + "!",
    ])
```

### **3. The "Leet Speak" Pattern**
```python
# Common letter substitutions
leet_map = {
    'a': ['@', '4'],
    'e': ['3'],
    'i': ['1', '!'],
    'o': ['0'],
    's': ['5', '$'],
    't': ['7'],
    'b': ['8'],
    'g': ['9'],
}

def generate_leet_variations(word):
    variations = [word]
    
    # Simple substitutions
    if 'a' in word:
        variations.append(word.replace('a', '@'))
        variations.append(word.replace('a', '4'))
    
    if 'e' in word:
        variations.append(word.replace('e', '3'))
    
    if 'i' in word:
        variations.append(word.replace('i', '1'))
        variations.append(word.replace('i', '!'))
    
    if 'o' in word:
        variations.append(word.replace('o', '0'))
    
    if 's' in word:
        variations.append(word.replace('s', '5'))
        variations.append(word.replace('s', '$'))
    
    # Multiple substitutions
    leet_word = word
    for char, subs in leet_map.items():
        if char in leet_word:
            leet_word = leet_word.replace(char, subs[0])
    variations.append(leet_word)
    
    return list(set(variations))

# Example
word = "password"
leet_variations = generate_leet_variations(word)
print(leet_variations)
# ['password', 'p@ssword', 'p4ssword', 'passw0rd', 'p@ssw0rd', 'p4ssw0rd']
```

### **4. The "Seasonal Update" Pattern**
```python
# Users update passwords seasonally
seasons = {
    "spring": ["spring2023", "Spring2024", "spring!", "Spring!"],
    "summer": ["summer2023", "Summer2024", "summer!", "Summer!"],
    "fall": ["fall2023", "Fall2024", "fall!", "Fall!"],
    "winter": ["winter2023", "Winter2024", "winter!", "Winter!"],
    "autumn": ["autumn2023", "Autumn2024", "autumn!", "Autumn!"]
}

# Common yearly updates
yearly_patterns = []
base = "Password"
for year in range(2020, 2025):
    yearly_patterns.append(f"{base}{year}")
    yearly_patterns.append(f"{base}{year}!")
    yearly_patterns.append(f"{base}{str(year)[-2:]}")
    yearly_patterns.append(f"{base}{str(year)[-2:]}!")
```

## 🎮 **Complete Attack Scenario with Human Patterns**

### **Target: Corporate Password Policy**

**Step 1: Understand the Policy**
```python
# From company website:
password_policy = {
    "min_length": 8,
    "require_uppercase": True,
    "require_lowercase": True,
    "require_number": True,
    "require_special": True,
    "change_frequency": "90 days"
}
```

**Step 2: Generate Base Words from OSINT**
```python
# Company info gathered
company_info = {
    "name": "MegaCorp",
    "founded": 1995,
    "mascot": "eagle",
    "products": ["cloud", "ai", "security"]
}

# Employee info from LinkedIn
employee_info = {
    "john.doe@megacorp.com": {
        "name": "John Doe",
        "hometown": "Chicago",
        "sports": "Bulls",
        "kids": ["Emma", "Noah"]
    }
}

# Generate base words
base_words = [
    "megacorp",
    "eagle",
    "cloud",
    "security",
    "chicago",
    "bulls",
    "emma",
    "noah",
    "password",
    "welcome",
    "admin"
]
```

**Step 3: Apply Human Transformations**
```python
def generate_human_passwords(base_words, current_year=2024):
    passwords = set()
    
    for word in base_words:
        # Original
        passwords.add(word)
        
        # Capitalize
        passwords.add(word.capitalize())
        passwords.add(word.upper())
        
        # Add current and previous years
        for year in range(current_year-5, current_year+1):
            passwords.add(f"{word}{year}")
            passwords.add(f"{word.capitalize()}{year}")
            passwords.add(f"{word}{str(year)[-2:]}")
        
        # Add common suffixes
        for suffix in ["123", "1234", "!", "!!", "?", ".", "@"]:
            passwords.add(f"{word}{suffix}")
            passwords.add(f"{word.capitalize()}{suffix}")
        
        # Common combinations
        passwords.add(f"{word.capitalize()}{current_year}!")
        passwords.add(f"{word}{current_year}!")
        passwords.add(f"!{word}{current_year}")
        
        # Leet speak versions
        leet = word
        leet = leet.replace('a', '@')
        leet = leet.replace('e', '3')
        leet = leet.replace('i', '1')
        leet = leet.replace('o', '0')
        leet = leet.replace('s', '$')
        passwords.add(leet)
        
        # Common mutations
        if len(word) >= 6:
            passwords.add(word[:1].upper() + word[1:] + "!")
            passwords.add(word[:1].upper() + word[1:] + "123")
    
    return list(passwords)

# Generate targeted wordlist
targeted_passwords = generate_human_passwords(base_words)
print(f"Generated {len(targeted_passwords)} likely passwords")
```

**Step 4: The Password Change Pattern**
```python
# Users change: MyPassword1! → MyPassword2! → MyPassword3!

def generate_update_patterns(original_password):
    patterns = []
    
    # Increment numbers
    import re
    numbers = re.findall(r'\d+', original_password)
    if numbers:
        for num in numbers:
            # Try incrementing by 1
            new_num = str(int(num) + 1)
            patterns.append(original_password.replace(num, new_num))
            
            # Try incrementing by 1 with padding
            if len(num) > 1:
                new_num_padded = new_num.zfill(len(num))
                patterns.append(original_password.replace(num, new_num_padded))
    
    # Change special character
    special_chars = "!@#$%^&*?"
    for special in special_chars:
        if special in original_password:
            for new_special in special_chars.replace(special, ''):
                patterns.append(original_password.replace(special, new_special))
    
    # Toggle case of first letter
    if original_password[0].isupper():
        patterns.append(original_password[0].lower() + original_password[1:])
    else:
        patterns.append(original_password[0].upper() + original_password[1:])
    
    return list(set(patterns))

# Example
original = "Summer2024!"
updates = generate_update_patterns(original)
print(updates)
# ['Summer2025!', 'Summer2024@', 'Summer2024#', 'summer2024!']
```

## 🔍 **Common Password Patterns Database**

### **Top 100 Password Patterns**
```
CATEGORY          │ EXAMPLE               │ FREQUENCY
──────────────────┼───────────────────────┼──────────
Sports Teams      │ Cowboys2024!          │ 12%
Seasons           │ Summer2024!           │ 10%
Animals           │ Monkey123!            │ 8%
Names + Year      │ Emma2024!             │ 15%
Simple Words + !  │ Password!             │ 20%
Keyboard Patterns │ Qwerty123!            │ 7%
Double Words      │ Passpass123!          │ 5%
Company Name      │ MegaCorp2024!         │ 8%
Random + Number   │ Purple72!             │ 10%
Phrases           │ Iloveyou!             │ 5%
```

### **Password Mutation Rules**
```python
# Rules used by password crackers like Hashcat
mutation_rules = [
    "$1",           # Add 1 at end
    "$2",           # Add 2 at end  
    "$3",           # Add 3 at end
    "$!",           # Add ! at end
    "$123",         # Add 123 at end
    "^!",           # Add ! at beginning
    "c",            # Capitalize first letter
    "u",            # All uppercase
    "l",            # All lowercase
    "sa@",          # Replace a with @
    "se3",          # Replace e with 3
    "si1",          # Replace i with 1
    "so0",          # Replace o with 0
    "ss$",          # Replace s with $
    "$1$!",         # Add 1 and !
    "c$1$!",        # Capitalize + add 1 + !
    "$2024",        # Add current year
]
```

## 🎯 **Real Attack Examples**

### **Example 1: The Seasonal Update Attack**
```python
# Attacker knows password changes every 90 days

# First password found: Summer2024!
# Based on pattern, next will likely be:

predictions = [
    "Fall2024!",
    "Autumn2024!",
    "Winter2024!",
    "Spring2025!",
    "Summer2025!",
    "Summer2024@",
    "Summer2024#",
    "Summer2024?",
    "Summer2024!!",
    "Summer2025!"
]

# Try each prediction when password expires
```

### **Example 2: The Company Pattern Attack**
```python
# Company: TechCorp
# Employee: John Smith
# Known: Uses "TechCorp" variations

observed_passwords = [
    "TechCorp2023!",
    "TechCorp2024!",
    "John2023!",
    "Smith2023!"
]

# Predict next password
likely_next = [
    "TechCorp2025!",
    "John2024!",
    "Smith2024!",
    "TechCorp@2024",
    "TC2024!",
    "TechCorp2024!!"
]
```

## 🛡️ **Defense Strategies Against Human Patterns**

### **1. Better Password Policies**
```javascript
function validatePassword(password, userInfo) {
    const errors = [];
    
    // Length check
    if (password.length < 12) {
        errors.push("Password must be at least 12 characters");
    }
    
    // Check for common patterns
    const commonPatterns = [
        /^[A-Z][a-z]+\d+!$/,           // Password123!
        /^[A-Z][a-z]+\d{4}!$/,          // Summer2024!
        /\d{4}$/,                        // Ends with year
        /(.)\1{2,}/,                     // Repeated chars
        /^(password|welcome|admin)/i,    // Common words
    ];
    
    for (const pattern of commonPatterns) {
        if (pattern.test(password)) {
            errors.push("Password follows a common, guessable pattern");
            break;
        }
    }
    
    // Check against user info
    const userTerms = [
        userInfo.firstName,
        userInfo.lastName,
        userInfo.company,
        userInfo.birthYear,
        ...userInfo.interests || []
    ];
    
    for (const term of userTerms) {
        if (term && password.toLowerCase().includes(term.toLowerCase())) {
            errors.push("Password contains personal information");
            break;
        }
    }
    
    // Check against common passwords database
    if (isCommonPassword(password)) {
        errors.push("Password is too common");
    }
    
    return {
        valid: errors.length === 0,
        errors: errors
    };
}
```

### **2. Password Blacklisting**
```python
# Maintain list of forbidden patterns
blacklist = {
    "patterns": [
        r"\d{4}$",  # Ends with year
        r"[A-Z][a-z]+\d{2,}[!@#$]",  # Word + numbers + special
        r"^(Password|P@ssw0rd)",  # Common bases
    ],
    "words": [
        "password", "welcome", "admin", "user",
        "login", "qwerty", "abc123", "letmein",
        "monkey", "dragon", "master", "hello",
    ],
    "sequences": [
        "123456", "12345", "123456789", "qwerty",
        "asdfgh", "zxcvbn", "111111", "abc123",
    ]
}

def is_password_blacklisted(password):
    # Check patterns
    for pattern in blacklist["patterns"]:
        if re.search(pattern, password):
            return True, f"Matches forbidden pattern: {pattern}"
    
    # Check words (case insensitive)
    password_lower = password.lower()
    for word in blacklist["words"]:
        if word in password_lower:
            return True, f"Contains common word: {word}"
    
    # Check sequences
    for seq in blacklist["sequences"]:
        if seq in password_lower:
            return True, f"Contains common sequence: {seq}"
    
    return False, "Password accepted"
```

### **3. Progressive Password Strength Meter**
```javascript
function getPasswordStrength(password) {
    let score = 0;
    let issues = [];
    
    // Length score
    if (password.length >= 16) score += 3;
    else if (password.length >= 12) score += 2;
    else if (password.length >= 8) score += 1;
    else issues.push("Too short");
    
    // Character variety
    if (/[a-z]/.test(password)) score += 1;
    if (/[A-Z]/.test(password)) score += 1;
    if (/[0-9]/.test(password)) score += 1;
    if (/[^a-zA-Z0-9]/.test(password)) score += 2;
    
    // Pattern penalties
    if (/^[A-Z][a-z]+\d+!$/.test(password)) {
        score -= 2;
        issues.push("Follows common pattern (Word+number+!)");
    }
    
    if (/(\d{4})$/.test(password)) {
        score -= 1;
        issues.push("Ends with year - easily guessed");
    }
    
    if (/(.)\1{3,}/.test(password)) {
        score -= 1;
        issues.push("Contains repeated characters");
    }
    
    // Check against common passwords
    if (isCommonPassword(password)) {
        score = 1;
        issues.push("This is a very common password!");
    }
    
    return {
        score: Math.max(1, Math.min(10, score)),
        issues: issues,
        strength: score >= 8 ? "Strong" : score >= 5 ? "Medium" : "Weak",
        suggestions: generateSuggestions(issues)
    };
}
```

### **4. Password Expiration with Intelligence**
```python
class SmartPasswordExpiry:
    def __init__(self):
        self.password_history = {}
    
    def check_password_change(self, username, old_password, new_password):
        # Don't allow simple mutations
        if self.is_simple_mutation(old_password, new_password):
            return False, "New password too similar to old password"
        
        # Don't allow seasonal patterns
        if self.is_seasonal_pattern(new_password):
            return False, "Avoid seasonal patterns (Summer2024, etc.)"
        
        # Don't allow incrementing numbers
        if self.is_incremented_number(old_password, new_password):
            return False, "Don't just increment numbers"
        
        # Check history (last 5 passwords)
        if self.is_password_reused(username, new_password):
            return False, "Cannot reuse recent passwords"
        
        return True, "Password accepted"
    
    def is_simple_mutation(self, old, new):
        # Check for common mutations
        mutations = [
            old + "!",
            old + "?",
            old + "1",
            old.capitalize(),
            old.upper(),
            old.lower(),
            old.replace('a', '@'),
            old.replace('e', '3'),
        ]
        return new in mutations
```

## 📊 **Statistics on Human Password Behavior**

```
PASSWORD CREATION STATISTICS:
─────────────────────────────────────────
Users who use personal info:           65%
Users who reuse passwords:              52%
Users who use seasonal patterns:        28%
Users who increment numbers:            35%
Users who capitalize first letter:       85%
Users who add ! at end:                  45%
Users who use 4-digit year:              40%

TIME TO CRACK DIFFERENT PASSWORDS:
─────────────────────────────────────────
"password"                    → Instant
"Password123!"                 → 2 hours
"P@ssw0rd2024!"               → 3 days
"PurpleMonkeyDishwasher2024"  → 500 years
```

## 🛠️ **Testing Your Password Against Human Patterns**

### **Password Pattern Checker**
```python
def analyze_password_pattern(password):
    analysis = {
        "length": len(password),
        "patterns_found": [],
        "risk_score": 0,
        "recommendations": []
    }
    
    # Check for dictionary words
    common_words = load_dictionary()
    password_lower = password.lower()
    for word in common_words[:1000]:  # Top 1000 words
        if word in password_lower and len(word) > 3:
            analysis["patterns_found"].append(f"Contains dictionary word: {word}")
            analysis["risk_score"] += 2
    
    # Check for keyboard patterns
    keyboard_patterns = ["qwerty", "asdfgh", "zxcvbn", "123456"]
    for pattern in keyboard_patterns:
        if pattern in password_lower:
            analysis["patterns_found"].append(f"Keyboard pattern: {pattern}")
            analysis["risk_score"] += 3
    
    # Check for years
    import re
    years = re.findall(r'19\d\d|20\d\d', password)
    if years:
        analysis["patterns_found"].append(f"Contains year: {years[0]}")
        analysis["risk_score"] += 2
    
    # Check for sequential numbers
    if re.search(r'123|234|345|456|567|678|789', password):
        analysis["patterns_found"].append("Sequential numbers")
        analysis["risk_score"] += 2
    
    # Check for repeated characters
    if re.search(r'(.)\1{2,}', password):
        analysis["patterns_found"].append("Repeated characters")
        analysis["risk_score"] += 1
    
    # Recommendations
    if analysis["risk_score"] > 5:
        analysis["recommendations"].append("Use a random password generator")
    if len(set(password)) < len(password) * 0.7:
        analysis["recommendations"].append("Use more unique characters")
    if re.search(r'\d{4}$', password):
        analysis["recommendations"].append("Don't end with a year")
    
    return analysis

# Test
result = analyze_password_pattern("Summer2024!")
print(json.dumps(result, indent=2))
```

## 🔐 **Best Practices for Users**

### **What NOT to Do:**
```
❌ Password123!        (Follows common pattern)
❌ Summer2024!         (Seasonal pattern)
❌ John2024!           (Name + year)
❌ P@ssw0rd            (Common leet version)
❌ Monday1!            (Day of week)
❌ !QAZ2wsx            (Keyboard pattern)
```

### **What TO Do:**
```
✓ "correct horse battery staple"  (Random words)
✓ "purple-monkey-dishwasher-784"   (Random words + numbers)
✓ "G#7kL9@pQ2mX"                   (Completely random)
✓ "My dog eats purple pancakes!"    (Sentence with punctuation)
```

## 🎓 **Key Takeaways**

1. **Humans are predictable** - We follow patterns even when trying to be secure
2. **Password policies create patterns** - Users will always find the path of least resistance
3. **Common transformations are guessable** - Leet speak, adding years, capitalizing first letter
4. **Seasonal patterns are dangerous** - Summer2024, Winter2024 are easily predicted
5. **Personal info is weak** - Names, birthdates, interests are public knowledge
6. **Password managers are essential** - They break all human patterns

Remember: **When you force humans to create passwords, they'll create passwords that humans can guess. Use a password manager!** 🔐