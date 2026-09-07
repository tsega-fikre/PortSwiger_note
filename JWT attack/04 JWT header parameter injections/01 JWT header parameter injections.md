# JWT Header Parameters (`jwk`, `jku`, and `kid`)

So far, we've learned that the most important field in a JWT header is:

```json
{
    "alg": "HS256"
}
```

The `alg` parameter tells the server **which algorithm should be used to verify the JWT signature**.

However, JWT headers can contain **other parameters** that help the server decide **which key** to use during verification.

These parameters are:

* `jwk`
* `jku`
* `kid`

Although they are useful for legitimate applications, they can become dangerous if the server trusts them without proper validation.

---

# Why Do These Parameters Exist?

Imagine a large company with several servers.

Instead of using one secret key, the company might have:

```text
Key A
Key B
Key C
```

When a JWT arrives, the server must know **which key** to use to verify its signature.

The JWT header helps the server identify the correct key.

---

# 1. `jwk` (JSON Web Key)

The `jwk` parameter contains **the public key itself** inside the JWT.

Example:

```json
{
    "alg": "RS256",
    "jwk": {
        "kty": "RSA",
        "n": "...",
        "e": "AQAB"
    }
}
```

Normally:

1. The server reads the public key from the `jwk`.
2. It verifies the JWT signature using that key.

### Why Is This Dangerous?

If the server blindly trusts any `jwk` supplied by the client, an attacker can:

1. Generate their **own RSA key pair**.
2. Put **their own public key** in the `jwk` header.
3. Modify the JWT payload (for example, change `"role":"user"` to `"role":"admin"`).
4. Sign the JWT using **their own private key**.

When the server uses the attacker-supplied public key from the `jwk` header, the signature appears valid.

The attacker has effectively convinced the server to trust **their own key** instead of the server's legitimate key.

---

# 2. `jku` (JSON Web Key Set URL)

Instead of embedding a key directly, the JWT can contain a URL pointing to a set of public keys.

Example:

```json
{
    "alg": "RS256",
    "jku": "https://example.com/keys.json"
}
```

Normally:

1. The server downloads the keys from the URL.
2. It finds the correct key.
3. It verifies the JWT.

### Why Is This Dangerous?

If the server allows **any URL**, an attacker can create:

```json
{
    "jku":"https://attacker.com/keys.json"
}
```

The attacker hosts their own key on that website.

The server downloads the attacker's public key and uses it to verify the JWT.

Since the attacker also owns the corresponding private key, they can create any JWT they want.

This attack is called **JKU Header Injection**.

---

# 3. `kid` (Key ID)

Sometimes a server stores many keys.

Instead of sending the whole key, the JWT simply identifies which one to use.

Example:

```json
{
    "alg":"RS256",
    "kid":"key-1"
}
```

The server reads:

```text
kid = key-1
```

Then loads:

```text
Key-1
```

and verifies the signature.

### Why Is This Dangerous?

If the application doesn't validate the `kid` value properly, attackers may manipulate it.

Depending on how the server looks up keys, this can sometimes lead to vulnerabilities such as:

* Loading the wrong key.
* Reading unexpected files.
* Bypassing signature verification.

The exact attack depends on how the application is implemented.

---

# What Do These Three Parameters Have in Common?

All three influence **which key** the server uses.

| Header | Purpose                                                  |
| ------ | -------------------------------------------------------- |
| `jwk`  | Provides the verification key directly inside the JWT.   |
| `jku`  | Tells the server where to download the verification key. |
| `kid`  | Tells the server which stored key to use.                |

The problem is that **these values come from the client**, not the server.

If the server trusts them without proper validation, an attacker can trick it into using a key controlled by the attacker.

---

# Real-World Analogy

Imagine a security guard who checks employee ID cards.

Normally, the guard uses the company's official employee list.

Now imagine the visitor says:

* **`jwk`**: "Here's my own employee list."
* **`jku`**: "Download the employee list from my website."
* **`kid`**: "Use employee list number 5."

If the guard blindly follows these instructions without checking whether they are trustworthy, an attacker can convince the guard that a fake ID is genuine.

The same thing happens with vulnerable JWT implementations.

---

# Key Takeaways

* Besides `alg`, JWT headers may contain parameters such as **`jwk`**, **`jku`**, and **`kid`**.
* These parameters tell the server **which key** should be used to verify the JWT signature.
* Because the JWT header is **controlled by the client**, these values should never be trusted without strict validation.
* If a server accepts attacker-controlled keys or key locations, an attacker can sign JWTs with **their own key** and have the server accept them as valid.
* These vulnerabilities are commonly known as **JWT header injection attacks** and can lead to authentication bypass or privilege escalation if implemented insecurely.
