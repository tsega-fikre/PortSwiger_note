# Brute-Forcing JWT Secret Keys Using Hashcat

This attack is possible when a JWT uses **HS256** and the server's **secret key is weak or predictable**.

The goal is **not** to attack the server directly. Instead, the attacker tries to discover the secret key that was used to sign the JWT. Once the secret is known, the attacker can create **new JWTs with valid signatures**, allowing them to impersonate other users or escalate privileges.

---

# Why Can Hashcat Crack the Secret?

Remember how an HS256 JWT signature is created:

```text id="wqf04r"
Signature =
HMAC_SHA256(
    Header.Payload,
    Secret Key
)
```

If an attacker already has:

* The JWT header
* The JWT payload
* The JWT signature

then the **only unknown value is the secret key**.

Hashcat repeatedly guesses possible secret keys until it finds one that produces the **same signature** as the original JWT.

---

# Step 1: Capture a Valid JWT

Suppose you log in and receive this JWT:

```text id="rfx1ts"
Header.Payload.Signature
```

This JWT contains everything needed for the attack except the secret key.

---

# Step 2: Use a Wordlist

Hashcat uses a wordlist containing common secrets, for example:

```text id="1cm2om"
secret
password
admin
jwtsecret
mysecret
123456
```

These are common values that developers sometimes use accidentally.

---

# Step 3: Run Hashcat

The command is:

```bash id="4s5whk"
hashcat -a 0 -m 16500 <jwt> <wordlist>
```

### What do the options mean?

* `-a 0` → Straight (dictionary) attack.
* `-m 16500` → Hash mode for JWT (HS256/HS384/HS512).
* `<jwt>` → The captured JWT.
* `<wordlist>` → File containing possible secret keys.

---

# Step 4: How Hashcat Works Internally

Suppose the JWT is:

```text id="u6t4ep"
Header.Payload.Signature
```

Hashcat tries each candidate secret.

### First guess

```text id="l4fkhn"
Secret = password
```

It computes:

```text id="r7vif7"
HMAC_SHA256(
Header.Payload,
password
)
```

The signature doesn't match.

---

### Second guess

```text id="7mnazp"
Secret = admin
```

Again, the signatures don't match.

---

### Third guess

```text id="5o1n6w"
Secret = mySecretKey
```

This time the computed signature matches the JWT's signature.

Hashcat knows it has found the correct secret.

---

# Step 5: Hashcat Displays the Result

Hashcat outputs something like:

```text id="kkaxw5"
<jwt>:mySecretKey
```

This means the JWT was signed using:

```text id="d1uw8u"
mySecretKey
```

If you run Hashcat again later, use:

```bash id="yjw2t7"
hashcat --show -m 16500 <jwt>
```

to display previously cracked results.

---

# Step 6: What Happens Next?

Now the attacker knows the server's signing key.

They can create a new payload:

```json id="j4jwxq"
{
    "username": "administrator",
    "role": "admin"
}
```

Then sign it using the discovered secret key.

Because the signature is valid, the server accepts the forged JWT as authentic.

---

# Why Is This Attack So Fast?

Hashcat runs **entirely on the attacker's computer**.

It does **not** send thousands of requests to the server.

Instead, it repeatedly performs the HMAC calculation locally:

```text id="jmh9tn"
Guess Secret
        ↓
Create Signature
        ↓
Compare with JWT Signature
        ↓
Match?
```

This process can test thousands or even millions of candidate secrets very quickly, depending on the hardware and wordlist.

---

# Why Does This Only Work Against Weak Secrets?

If the server uses a long, random secret such as:

```text id="ehx68m"
N7$kL!9Qv2#Xa8Pz@41LmR0w
```

it is practically impossible to guess using a wordlist.

However, if the secret is something like:

```text id="d8ig2d"
secret
password
jwtsecret
admin123
```

Hashcat can often discover it within seconds because these values appear in common password dictionaries.

---

# Real-World Analogy

Imagine a company seals every official document using a rubber stamp.

You have several genuine documents, but you don't have the stamp.

Instead of asking the company for the stamp, you try every stamp in a large collection until one produces an identical seal.

Once you find the matching stamp, you can create your own official-looking documents.

Hashcat works the same way: it repeatedly tries possible **secret keys** until one generates the exact same JWT signature.

---

# Key Takeaways

* Hashcat can brute-force JWT secret keys when **HS256** (or other HMAC-based JWT algorithms) is used with a **weak secret**.
* The attacker only needs:

  * A valid signed JWT.
  * A wordlist of possible secret keys.
* Hashcat repeatedly computes **HMAC(Header.Payload, Candidate Secret)** and compares the result with the JWT's signature.
* When a matching signature is found, the corresponding secret key has been identified.
* Once the secret key is known, an attacker can generate **new JWTs with any claims they choose**, and the server will accept them because the signatures are valid.
