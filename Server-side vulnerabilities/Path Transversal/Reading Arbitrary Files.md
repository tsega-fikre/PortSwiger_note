

## How This Attack Works

### Path Resolution Process
1. **Base directory**: `/var/www/images/`
2. **User input**: `../../../etc/passwd`
3. **Combined path**: `/var/www/images/../../../etc/passwd`
4. **Normalized path**: `/etc/passwd`

The `../` sequences effectively "cancel out" portions of the base path:
- First `../` goes up from `images/` to `/var/www/`
- Second `../` goes up from `/var/www/` to `/var/`
- Third `../` goes up from `/var/` to `/`
- Then appends `etc/passwd`

## Why This is Dangerous

The `/etc/passwd` file example is classic, but attackers can target many other sensitive files:

**Linux/Unix targets:**
- `/etc/shadow` - password hashes
- `/etc/ssh/sshd_config` - SSH configuration
- `/root/.bash_history` - command history
- `/var/log/auth.log` - authentication logs
- Application config files (`.env`, `config.php`, `settings.py`)

**Windows targets:**
- `C:\Windows\System32\drivers\etc\hosts`
- `C:\Windows\System32\config\SAM` - password hashes
- `C:\inetpub\wwwroot\web.config` - IIS configuration
- `C:\Program Files\application\config.ini`

## Variations of the Attack

### URL Encoding Bypass
If basic `../` is blocked, attackers might try:
- `%2e%2e%2f` - URL encoded `../`
- `%252e%252e%252f` - Double encoded
- `..%252f` - Mixed encoding
- `..%c0%af` - Unicode overlong representation

### Absolute Paths
Some applications might also accept absolute paths:
```
https://insecure-website.com/loadImage?filename=/etc/passwd
```

### Nested Traversal
If the application adds extensions automatically:
```
https://insecure-website.com/loadImage?filename=../../../etc/passwd%00.png
```
(The null byte might terminate the string before `.png` is appended)

## Real-World Impact

In this shopping application scenario, an attacker could:
1. **Extract database credentials** from config files
2. **Read source code** to find more vulnerabilities
3. **Access customer data** from database dumps
4. **Retrieve SSH keys** for further system access
