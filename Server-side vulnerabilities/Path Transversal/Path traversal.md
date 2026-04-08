

## How Path Traversal Works

Path traversal attacks exploit insufficient security validation when an application accesses files based on user input. Attackers use special character sequences to navigate outside the intended directory.

### Common Traversal Sequences
- `../` (Unix/Linux) - moves up one directory
- `..\` (Windows) - moves up one directory
- URL encoded versions: `%2e%2e%2f` (represents `../`)
- Double encoded: `%252e%252e%252f`

## Example Scenarios

**Vulnerable URL:**
```
https://example.com/view?file=report.pdf
```

**Attack attempt:**
```
https://example.com/view?file=../../../etc/passwd
```

**Windows example:**
```
https://example.com/view?file=..\..\..\windows\win.ini
```

## Real Impact Examples

1. **Reading sensitive files:**
   - `/etc/passwd` (user information)
   - `/etc/shadow` (password hashes)
   - Application configuration files with database credentials
   - Source code files

2. **Writing files (more severe):**
   - Adding malicious scripts to web directories
   - Modifying configuration files
   - Overwriting critical system files

## Prevention Measures

1. **Avoid user input in file paths** when possible
2. **Use a whitelist** of allowed files
3. **Validate file paths** by:
   - Canonicalizing the path (resolving all `..` and symlinks)
   - Checking if the resolved path is within the intended directory
4. **Use chroot jails** or containerization
5. **Run applications with minimal privileges**

