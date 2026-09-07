# Brute-Forcing Weak JWT Secret Keys

This vulnerability occurs when the server uses a **weak or predictable secret key** to sign JWTs.

For JWTs that use **HS256 (HMAC + SHA-256)**, the **same secret key** is used to:

* Create the JWT signature.
* Verify the JWT signature.

Because of this, **anyone who discovers the secret key can create valid JWTs** that the server will trust.

---

# How HS256 Normally Works

## Step 1: User Logs In

The user enters their username and password.

The server verifies the credentials using its database.

For example:

```text id="xwqqzj"
Username: carlos
Role: user
```

---

## Step 2: Server Creates the JWT

The server creates the payload:

```json id="4rr1p2"
{
    "username": "carlos",
    "role": "user"
}
```

Then it signs the JWT using its secret key.

Example:

```text id="s4lxls"
Secret Key:

mySuperSecretKey
```

The signature is created like this:

```text id="qiv28q"
Signature =
HMAC_SHA256(
    Header.Payload,
    Secret Key
)
```

The completed JWT is sent to the user.

---

# Why Is the Secret Key So Important?

The server trusts **every JWT that has a valid signature**.

A valid signature can only be created using the correct secret key.

So the server assumes:

> "If the signature is valid, then I must have created this JWT."

This assumption is only true if **the secret key remains secret**.

---

# What Happens If the Secret Key Is Weak?

Suppose the developer uses:

```text id="jlwm3n"
secret
```

or

```text id="nkgmzi"
password123
```

or

```text id="4smwjr"
jwtsecret
```

These are common, predictable secrets.

An attacker can try thousands of common passwords from a **wordlist** until one successfully verifies the JWT.

This is called **brute-forcing** the secret key.

---

# The Attack

Suppose the attacker captures this JWT:

```json id="3wx80l"
{
    "username": "carlos",
    "role": "user"
}
```

The attacker uses a tool to test many possible secret keys.

For example:

```text id="7tr4zv"
password
admin
secret
jwtsecret
123456
```

Eventually, the tool discovers that the correct secret is:

```text id="cr6u54"
secret
```

Now the attacker knows the server's signing key.

---

# What Can the Attacker Do Next?

The attacker changes the payload:

```json id="8uxv84"
{
    "username": "administrator",
    "role": "admin"
}
```

Then they create a **new signature** using the stolen secret key:

```text id="0owvhx"
HMAC_SHA256(
    Header.Payload,
    secret
)
```

The signature is now completely valid.

When the attacker sends the forged JWT, the server verifies the signature using the same secret key.

The signatures match.

The server believes:

> "I must have created this JWT."

The attacker is authenticated as an administrator.

---

# Why Does This Happen?

The security of HS256 depends entirely on the secrecy of the signing key.

If the secret key becomes known, the attacker can:

* Create JWTs for any user.
* Change usernames.
* Change roles.
* Set `isAdmin` to `true`.
* Generate valid signatures.

At that point, the server cannot distinguish between a legitimate JWT and a forged one.

---

# How Do Developers Accidentally Create This Vulnerability?

Common mistakes include:

### 1. Leaving the Default Secret

During development, a developer may use:

```text id="jl7yl8"
secret
```

and forget to replace it before deploying the application.

---

### 2. Copying Example Code

Many tutorials use example secrets like:

```text id="q6jmk8"
your-secret-key
```

or

```text id="pohzj5"
mySecret
```

If the developer copies the code without changing the secret, anyone familiar with the tutorial may guess it.

---

### 3. Using Simple Passwords

Examples include:

```text id="ws6d4d"
admin
password
test
123456
qwerty
```

These can often be guessed or found using publicly available wordlists.

---

# Real-World Analogy

Imagine a company stamps every employee ID card with a special stamp.

The stamp is supposed to be locked in a secure office.

Instead, the company leaves it on a desk labeled:

```text id="0b12iq"
Official Company Stamp
```

Anyone who finds the stamp can make fake employee IDs that look completely genuine.

The **secret key** in HS256 works the same way.

If an attacker obtains it, they can produce valid JWT signatures, and the server has no way to tell the difference.

---

# Key Takeaways

* **HS256** uses a shared **secret key** for both signing and verifying JWTs.
* The security of the JWT depends entirely on keeping this secret key confidential.
* If the secret key is weak, predictable, leaked, or left as a default value, attackers can discover it through guessing or brute-force attacks.
* Once the attacker knows the secret key, they can create **valid JWTs with any claims they choose**, such as `"role": "admin"` or `"isAdmin": true`.
* Because the forged JWT has a valid signature, the server trusts it as if it had created it itself, resulting in complete authentication bypass and privilege escalation.
