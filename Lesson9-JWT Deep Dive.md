# Module 9 — JWT (JSON Web Token) Deep Dive

> **Goal:** Understand JWT internals, how servers validate them, what every claim means, and how to analyze JWT implementations during an API security assessment.

By the end of this module, you'll be able to:

- Read a JWT without tools
    
- Decode every field
    
- Understand the signing process
    
- Explain symmetric vs asymmetric signatures
    
- Understand how validation should work
    
- Know what questions to ask when reviewing a JWT-based authentication system
    

---

# Chapter 1 — What Is a JWT?

A JWT is simply a **portable identity document**.

Imagine an airport boarding pass.

It contains:

- Your name
    
- Flight
    
- Seat
    
- Departure time
    

It doesn't prove anything by itself.

What makes it trustworthy is that the airline issued it.

JWTs work similarly.

---

# Example JWT

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
.
eyJzdWIiOiIxMjMiLCJuYW1lIjoiQWxpY2UiLCJyb2xlIjoidXNlciJ9
.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

Notice the dots.

There are exactly **three parts**.

```text
Header
    .
Payload
    .
Signature
```

---

# Chapter 2 — The Three Parts

Let's separate them.

## Header

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

---

## Payload

```text
eyJzdWIiOiIxMjMiLCJuYW1lIjoiQWxpY2UiLCJyb2xlIjoidXNlciJ9
```

---

## Signature

```text
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

The first two parts are Base64URL-encoded JSON.

The third part is the cryptographic signature.

---

# Chapter 3 — Header

Decode the header.

```json
{
  "alg":"HS256",
  "typ":"JWT"
}
```

Only two fields.

---

## typ

Usually:

```json
"JWT"
```

Simply identifies the token type.

---

## alg

This is very important.

```json
"HS256"
```

It tells the server which signature algorithm was used.

Examples:

```text
HS256

HS384

HS512

RS256

ES256
```

We'll return to these shortly.

---

# Chapter 4 — Payload

The payload contains **claims**.

Claims are statements about the authenticated subject.

Example:

```json
{
  "sub":"123",
  "name":"Alice",
  "role":"user"
}
```

This is the identity information carried by the token.

Remember:

> **Anyone who has the JWT can usually decode the payload.**

So it should not contain secrets like passwords or private keys.

---

# Chapter 5 — Registered Claims

These are standardized claim names.

---

## sub (Subject)

```json
"sub":"123"
```

Who is this token about?

Usually:

- User ID
    
- Account ID
    
- Service account ID
    

---

## iss (Issuer)

```json
"iss":"auth.example.com"
```

Who issued this token?

The server should verify that it comes from a trusted issuer.

---

## aud (Audience)

```json
"aud":"payments-api"
```

Who is allowed to accept this token?

Imagine a company has:

```text
Payments API

Orders API

Analytics API
```

A token intended for the Payments API shouldn't automatically be accepted by the Analytics API.

---

## exp (Expiration)

```json
"exp":1788000000
```

The expiration timestamp.

After this point:

The token should no longer be accepted.

---

## iat (Issued At)

```json
"iat":1787000000
```

When was the token created?

Useful for tracking age and implementing additional validation logic.

---

## nbf (Not Before)

```json
"nbf":1787001000
```

Do not accept this token before this time.

Useful for scheduled activation.

---

## jti (JWT ID)

```json
"jti":"d81f..."
```

A unique identifier for the token.

It can support replay detection or token revocation strategies.

---

# Chapter 6 — Custom Claims

Applications often define their own claims.

Example:

```json
{
   "role":"admin",
   "department":"finance",
   "subscription":"premium"
}
```

These claims help applications make authorization decisions, but the server must trust only tokens that have been properly validated.

---

# Chapter 7 — Base64URL

JWTs use **Base64URL encoding**.

Important distinction:

Encoding ≠ Encryption.

Encoding:

```text
Hello

↓

SGVsbG8=
```

Anyone can reverse it.

Encryption:

Requires a secret key to recover the original data.

JWT headers and payloads are encoded, not encrypted.

---

# Chapter 8 — Signing

Here's the crucial question.

If anyone can decode and modify the payload...

What stops someone from changing:

```json
"role":"user"
```

to

```json
"role":"admin"
```

The answer:

The **signature**.

---

# Signing Process

The server creates:

```text
Header

+

Payload
```

Then computes a cryptographic signature using a secret (or private key, depending on the algorithm).

The result is:

```text
Header

Payload

Signature
```

If the payload changes later...

The signature no longer matches.

The server should reject the token.

---

# Visual Flow

```text
Header
      │
Payload
      │
      ▼
Signing Algorithm
      │
Secret / Private Key
      │
      ▼
Signature
```

---

# Chapter 9 — HS256 vs RS256

These are two common signing approaches.

## HS256 (Symmetric)

Both signing and verification use the **same secret key**.

```text
Server

Secret Key

↓

Signs Token

↓

Verifies Token
```

Simple.

Fast.

The secret must remain confidential.

---

## RS256 (Asymmetric)

Uses a key pair.

```text
Private Key

↓

Signs
```

```text
Public Key

↓

Verifies
```

The private key never leaves the issuer.

The public key can be distributed to services that need to verify tokens.

This model is common in larger distributed systems.

---

# Chapter 10 — Validation

Every request arrives with:

```http
Authorization: Bearer eyJ...
```

A robust implementation should verify things like:

1. Is the signature valid?
    
2. Has the token expired?
    
3. Is the issuer trusted?
    
4. Is the audience correct?
    
5. Is the token active yet (`nbf`)?
    
6. Is the algorithm one that the server expects?
    

Only after successful validation should the application trust the claims.

---
# Chapter 11— Reading JWTs

Suppose Burp intercepts:

```http
Authorization:
Bearer eyJ...
```

Your first instinct should be:

Decode it.

Questions to answer:

- Which algorithm?
    
- Who issued it?
    
- Who is the subject?
    
- When does it expire?
    
- What audience is it intended for?
    
- What custom claims are present?
    

You are building understanding—not assuming there's a vulnerability.

---

# Chapter 12 — Common JWT Implementation Mistakes

JWTs themselves are not insecure.

Problems usually arise from incorrect implementation.

Examples include:

- Accepting expired tokens.
    
- Failing to verify the signature.
    
- Not validating the issuer (`iss`).
    
- Not validating the audience (`aud`).
    
- Accepting unexpected signing algorithms.
    
- Trusting claims before completing validation.
    

Notice the pattern:

The weaknesses are generally in **validation logic**, not in the JWT format.

---

# A JWT Analysis Workflow

Whenever you encounter a JWT:

### Step 1

Identify the algorithm.

```json
"alg":"HS256"
```

---

### Step 2

Inspect the claims.

```json
{
  "sub":"25",
  "role":"user",
  "exp":1788000000
}
```

---

### Step 3

Review time-related claims.

- `exp`
    
- `iat`
    
- `nbf`
    

---

### Step 4

Review trust-related claims.

- `iss`
    
- `aud`
    

---

### Step 5

Understand how the application uses the claims.

For example:

- Does `role` influence authorization?
    
- Does `sub` determine which resources are accessible?
    
