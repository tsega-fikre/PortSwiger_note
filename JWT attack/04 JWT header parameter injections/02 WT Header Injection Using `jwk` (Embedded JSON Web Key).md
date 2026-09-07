# JWT Header Injection Using `jwk` (Embedded JSON Web Key)

This vulnerability occurs when a server **trusts the public key embedded inside the JWT header** instead of using its own trusted public key.

Normally, the server should decide **which public key to use**. A vulnerable server lets the **attacker choose the key**, which completely breaks the security of JWT verification.

---

# What Is a JWK?

A **JWK (JSON Web Key)** is simply a **JSON representation of a cryptographic key**.

For example:

```json id="1m2z6g"
{
    "kty": "RSA",
    "e": "AQAB",
    "kid": "ed2Nf8sb-sD6ng0-scs5390g-fFD8sfxG",
    "n": "yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9m"
}
```

This object represents an **RSA public key**.

The important fields are:

| Parameter | Meaning                                                   |
| --------- | --------------------------------------------------------- |
| `kty`     | Key type (RSA).                                           |
| `n`       | RSA modulus (part of the public key).                     |
| `e`       | Public exponent (usually `AQAB`, which represents 65537). |
| `kid`     | Key ID used to identify the key.                          |

You don't need to memorize `n` and `e`; just remember that together they define the RSA public key.

---

# Normal JWT Verification (Secure)

Suppose a server uses **RS256**.

The server has:

```text id="l98g58"
Private Key  → Used to sign JWTs

Public Key   → Used to verify JWTs
```

### Login

The server creates:

```json id="7hl01g"
{
    "username":"carlos",
    "role":"user"
}
```

It signs the JWT with its **private key**.

Later, when the JWT comes back, the server verifies the signature using **its own trusted public key**.

The client never chooses which public key is used.

This is secure.

---

# What Does the `jwk` Header Do?

A JWT may contain a `jwk` field like this:

```json id="1g4g6o"
{
    "alg":"RS256",
    "jwk":{
        ...
    }
}
```

The `jwk` field contains a **public key**.

A secure server ignores attacker-supplied keys unless they come from a trusted source.

A vulnerable server does something dangerous:

> "The JWT already contains a public key. I'll use that key."

This means the attacker gets to decide which key the server trusts.

---

# The Attack

## Step 1: Attacker Generates Their Own RSA Keys

The attacker creates:

```text id="4vcjlwm"
Attacker Private Key

Attacker Public Key
```

Just like the server has its own key pair, the attacker creates their own.

---

## Step 2: Modify the JWT

The attacker changes:

```json id="gkz6xk"
{
    "username":"carlos",
    "role":"user"
}
```

to:

```json id="2q9n2g"
{
    "username":"administrator",
    "role":"admin"
}
```

---

## Step 3: Sign the JWT

Instead of using the server's private key (which the attacker doesn't have), the attacker signs the JWT with **their own private key**.

Normally this shouldn't work.

---

## Step 4: Embed the Attacker's Public Key

The attacker adds their public key to the JWT header:

```json id="80szy7"
{
    "alg":"RS256",
    "jwk":{
        "kty":"RSA",
        "e":"AQAB",
        "n":"AttackerPublicKey"
    }
}
```

---

## Step 5: Vulnerable Server Verifies the JWT

Instead of using its own trusted public key, the vulnerable server reads:

```text id="rmog3j"
jwk = Attacker's Public Key
```

It verifies the signature using the attacker's public key.

Since the attacker also signed the JWT with the matching private key, the signature is valid.

The server now believes:

```json id="krwm1o"
{
    "username":"administrator",
    "role":"admin"
}
```

The attacker gains administrator access.

---

# Why Does This Work?

Remember how RSA works:

```text id="86q49g"
Private Key
      │
      ▼
Signs JWT

Public Key
      │
      ▼
Verifies JWT
```

Normally:

```text id="91by8t"
Server Private Key
        ↓
Signs JWT

Server Public Key
        ↓
Verifies JWT
```

In the attack:

```text id="01u50z"
Attacker Private Key
        ↓
Signs JWT

Attacker Public Key
        ↓
Embedded in JWT

Server uses attacker's key
        ↓
Signature appears valid
```

The problem isn't RSA itself.

The problem is that the server trusts a public key supplied by the attacker.

---

# What About the `kid` Field?

You may also notice:

```json id="85p20f"
{
    "kid":"ed2Nf8sb-sD6ng0-scs5390g-fFD8sfxG"
}
```

`kid` stands for **Key ID**.

It identifies which key is being used.

When embedding your own `jwk`, the JWT's `kid` often needs to match the `kid` inside the embedded key.

Tools like **Burp Suite's JWT Editor** update this automatically, which is why the documentation mentions that the extension handles this step for you.

---

# Why Does Burp's JWT Editor Help?

Instead of manually:

* Generating RSA keys.
* Embedding the public key.
* Updating the `kid`.
* Re-signing the JWT.

The **Embedded JWK** attack in Burp's JWT Editor automates these steps.

It:

1. Generates an RSA key pair.
2. Inserts the public key into the `jwk` header.
3. Updates the `kid` if necessary.
4. Re-signs the JWT using the matching private key.

This lets you quickly test whether the server incorrectly trusts embedded keys.

---

# Real-World Analogy

Imagine a bank verifies signatures using its **official list of customer signatures**.

A normal customer signs a check, and the bank compares it against the signature it already has on file.

Now imagine someone writes on the check:

> **"Please use this new signature sample attached below."**

The bank compares the signature against the attacker's own sample instead of its official records.

Of course the signatures match—because the attacker provided both.

That is exactly what happens in a **JWK header injection attack**.

---

# Key Takeaways

* A **JWK** is a JSON representation of a cryptographic key.
* With **RS256**, JWTs are signed using a **private key** and verified using a **public key**.
* A secure server always uses **its own trusted public key** to verify JWT signatures.
* A vulnerable server trusts the **`jwk` value supplied in the JWT header**.
* An attacker can generate their own RSA key pair, sign a forged JWT with their private key, embed the matching public key in the `jwk` header, and trick the server into accepting the forged token.
* This vulnerability is known as **JWT Header Injection via Embedded JWK** and can lead to authentication bypass and privilege escalation.
