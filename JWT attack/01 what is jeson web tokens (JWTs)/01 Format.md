<h1 aline=center>JWT Formate</h1>

---

## The Token Breakdown (Your Actual Example)

```
eyJraWQiOiI5MTM2ZGRiMy1jYjBhLTRhMTktYTA3ZS1lYWRmNWE0NGM4YjUiLCJhbGciOiJSUzI1NiJ9
.
eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTY0ODAzNzE2NCwibmFtZSI6IkNhcmxvcyBNb250b3lhIiwic3ViIjoiY2FybG9zIiwicm9sZSI6ImJsb2dfYXV0aG9yIiwiZW1haWwiOiJjYXJsb3NAY2FybG9zLW1vbnRveWEubmV0IiwiaWF0IjoxNTE2MjM5MDIyfQ
.
SYZBPIBg2CRjXAJ8vCER0LA_ENjII1JakvNQoP-Hw6GG1zfl4JyngsZReIfqRvIAEi5L4HV0q7_9qGhQZvy9ZdxEJbwTxRs_6Lb-fZTDpW6lKYNdMyjw45_alSCZ1fypsMWz_2mTpQzil0lOtps5Ei_z7mM7M8gCwe_AGpI53JxduQOaB5HkT5gVrv9cKu9CsW5MS6ZbqYXpGyOG5ehoxqm8DL5tFYaW3lB50ELxi0KsuTKEbD0t5BCl0aCR2MBJWAbN-xeLwEenaqBiwPVvKixYleeDQiBEIylFdNNIMviKRgXiYuAvMziVPbwSgkZVHeEdF5MQP1Oe2Spac-6IfA
```

---

### Part 1: The Header (Decoded)

The first chunk (`eyJraWQiOi...`) decodes from Base64URL to:

```json
{
  "kid": "9136ddb3-cb0a-4a19-a07e-eadf5a44c8b5",
  "alg": "RS256"
}
```

**What this tells us:**

- **`alg`: "RS256"** → The server uses **RSA SHA-256** to sign this token. This is **asymmetric encryption**.
  - *Private key* on the server → used to *create* the signature.
  - *Public key* on the server → used to *verify* the signature.
  
- **`kid`: "9136ddb3..."** → This is the **Key ID**. 
  - The server might have *multiple* RSA keys (rotated for security). 
  - The `kid` tells the server *which* public key to use for verification.
  - **Attack vector:** If the server doesn't validate the `kid` properly, an attacker could point it to a key they control (path traversal, SQL injection, etc.) and forge tokens.

---

### Part 2: The Payload (Decoded)

The second chunk (`eyJpc3MiOi...`) decodes to:

```json
{
  "iss": "portswigger",
  "exp": 1648037164,
  "name": "Carlos Montoya",
  "sub": "carlos",
  "role": "blog_author",
  "email": "carlos@carlos-montoya.net",
  "iat": 1516239022
}
```

**Breaking down the claims:**

| Claim | Value | Meaning |
|-------|-------|---------|
| `iss` | `"portswigger"` | **Issuer** → who created the token. The server checks this to reject tokens from untrusted sources. |
| `exp` | `1648037164` | **Expiration** → Unix timestamp. Converts to **March 23, 2022**. Token is dead after that. |
| `name` | `"Carlos Montoya"` | Human-readable name (informational). |
| `sub` | `"carlos"` | **Subject** → usually the user ID or username. The server uses this to identify *who* is making the request. |
| `role` | `"blog_author"` | Custom claim → **Authorization**. This tells the server what permissions Carlos has. |
| `email` | `"carlos@carlos-montoya.net"` | Another custom claim → user's email. |
| `iat` | `1516239022` | **Issued At** → when the token was created. Converts to **January 17, 2018**. |

**Critical observation:** The `iat` is **January 2018**, but the `exp` is **March 2022**. That's a **4-year validity window**. 
- In production, this is **terrible practice**. JWTs should expire in minutes/hours.
- If this token is stolen, the attacker has *4 years* of access.

---

### Part 3: The Signature

The third chunk (`SYZBPIBg2CRjXAJ8vCER0LA_ENjII1JakvNQoP-Hw6GG1zfl4JyngsZReIfqRvIAEi5L4HV0q7_9qGhQZvy9ZdxEJbwTxRs_6Lb-fZTDpW6lKYNdMyjw45_alSCZ1fypsMWz_2mTpQzil0lOtps5Ei_z7mM7M8gCwe_AGpI53JxduQOaB5HkT5gVrv9cKu9CsW5MS6ZbqYXpGyOG5ehoxqm8DL5tFYaW3lB50ELxi0KsuTKEbD0t5BCl0aCR2MBJWAbN-xeLwEenaqBiwPVvKixYleeDQiBEIylFdNNIMviKRgXiYuAvMziVPbwSgkZVHeEdF5MQP1Oe2Spac-6IfA`) is the cryptographic signature.

**How it's generated (the math):**

```
signature = RSA_256_Sign(
    private_key,
    base64url(header) + "." + base64url(payload)
)
```

The server **hashes** the header + payload, then encrypts that hash with its **private RSA key**.

**Verification process (when the server receives this token):**

1. Server extracts the header + payload.
2. Server looks up the public key matching the `kid`.
3. Server runs the same hash on the header + payload.
4. Server uses the public key to *decrypt* the signature and compares it to the hash.
5. If they match → **token is authentic**. If not → **rejected**.

---

### The Critical Security Reality

You said it yourself: *"In most cases, this data can be easily read or modified by anyone with access to the token."*

Let's prove it:

**Can you read it?** 
Yes. Just copy the payload chunk into any Base64 decoder. You'll see Carlos's name, email, and role in plaintext. **No encryption here.**

**Can you modify it?**
Yes. Change `"role": "blog_author"` to `"role": "admin"`. Re-encode to Base64URL. Swap it into the token.

**Will the server accept it?**
**NO.** Because you don't have the server's private RSA key to re-sign the token. When the server verifies the signature against your tampered payload, it will fail. The token gets rejected.

**The ONLY way to break this:**
- Steal the server's private key.
- Find an implementation flaw (like `alg: none` or `kid` injection).
- Use a weak algorithm (like HS256 with a guessable password).

---

### The Takeaway

This token is a **self-contained authorization package**. It says:

> *"I am Carlos. I am a blog_author. I was issued by Portswigger. I expire in 2022. Trust me because I'm signed with RSA."*

The server doesn't need to query a database to know who Carlos is or what his role is. It just checks the signature, trusts the claims, and grants access.

**The danger:** If that signature verification is flawed in *any* way, an attacker can change `role` to `admin` and own the system. That's why the cryptographic signature is the **only** thing standing between a user and full system compromise.