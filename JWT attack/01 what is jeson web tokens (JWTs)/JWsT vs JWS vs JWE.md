# JWT vs JWS vs JWE (Simple but Deep Explanation)

Many beginners get confused because people use **JWT**, **JWS**, and **JWE** as if they are the same thing. They are closely related, but they have different purposes.

Think of it like this:

* **JWT** = The **format** (the container)
* **JWS** = A JWT that is **digitally signed**
* **JWE** = A JWT that is **encrypted**

---

# What is JWT?

**JWT (JSON Web Token)** is simply a **standard format** for sending information between two parties.

The information inside a JWT is called **claims**.

Example claims:

```json
{
  "username": "alice",
  "role": "admin",
  "exp": 1750000000
}
```

These claims tell the server:

* Who the user is
* Their permissions
* When the token expires

---

## Important Concept

A JWT **does not tell you how to protect the data.**

It only says:

> "This is how the information should be organized."

Imagine a cardboard box.

The JWT specification only tells you:

* the size of the box
* where the label goes
* how the contents are arranged

It **does not** tell you whether the box should be:

* locked
* signed
* encrypted

That is where **JWS** and **JWE** come in.

---

# What is JWS?

**JWS (JSON Web Signature)** is the **most common type of JWT**.

It adds a **digital signature**.

A signature proves:

* nobody changed the data
* the token really came from the server

A JWS has three parts:

```
Header.Payload.Signature
```

Example:

```
eyJhbGc...
.
eyJ1c2Vy...
.
Kx9f2...
```

The payload is only **Base64URL encoded**.

That means anyone can decode it.

For example:

```
{
  "username":"alice",
  "role":"admin"
}
```

Even though you can read it, you **cannot change it** because changing anything makes the signature invalid.

### JWS provides

✔ Integrity (data wasn't modified)

✔ Authentication (came from the server)

❌ Confidentiality (the data is NOT secret)

---

# What is JWE?

Sometimes the information is sensitive.

For example:

* customer data
* financial information
* private identifiers

In this case, just signing isn't enough.

Instead, we use **JWE (JSON Web Encryption).**

The payload is **encrypted**.

Instead of seeing

```json
{
 "username":"alice"
}
```

an attacker only sees random-looking encrypted text.

Without the encryption key, the contents cannot be read.

### JWE provides

✔ Confidentiality (hidden)

✔ Integrity

✔ Authentication

---

# JWT vs JWS vs JWE

| Feature           | JWT             | JWS             | JWE             |
| ----------------- | --------------- | --------------- | --------------- |
| Is it a format?   | ✅ Yes           | Uses JWT format | Uses JWT format |
| Signed?           | ❌ Not by itself | ✅ Yes           | Usually yes     |
| Encrypted?        | ❌ No            | ❌ No            | ✅ Yes           |
| Payload readable? | Depends         | ✅ Yes           | ❌ No            |

---

# Why do people call JWS a JWT?

This is where many people get confused.

Technically:

```
JWT
├── JWS
└── JWE
```

But in the real world, most websites use **JWS**.

So developers simply say:

> "Here's your JWT."

Even though what they actually mean is:

> "Here's your JWS token."

This is why the note says:

> **When people use the term "JWT", they almost always mean a JWS token.**

---

# Real-World Example

Imagine you log in to a website.

The server creates this payload:

```json
{
  "user":"Alice",
  "role":"admin"
}
```

### If it uses JWS

The payload is signed.

Anyone can decode it and read:

```
Alice
admin
```

But nobody can change `"admin"` to `"superadmin"` because the signature would fail.

---

### If it uses JWE

The payload is encrypted.

An attacker only sees something like:

```
A9Gh82KslP...
```

They cannot read the username or role without the encryption key.

---

# Why does this matter in security?

As a penetration tester, this distinction is important:

* **JWS** protects against **tampering** but **does not hide the data**. If you decode a JWS, you can inspect its claims, but modifying them without the signing key should invalidate the token.
* **JWE** protects against **eavesdropping** by encrypting the claims. Even if you capture the token, you cannot read its contents without the decryption key.

---

# Key Takeaways

* **JWT** is only a **standard format** for carrying claims.
* **JWS** is a JWT that is **digitally signed** to prevent tampering.
* **JWE** is a JWT that is **encrypted** to keep its contents secret.
* In practice, when people say **"JWT"**, they almost always mean a **JWS token**, because it is the most commonly used implementation for authentication.
