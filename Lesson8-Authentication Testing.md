# Module 8 — Authentication Testing

> **Goal:** Learn a structured methodology for assessing API authentication mechanisms.
---

# Chapter 1 — The Authentication Mindset

When you first encounter an API, don't immediately think:

> "How do I hack this?"

Instead, ask:

> **"How does this API know who I am?"**

Every protected endpoint depends on one or more credentials.

Examples:

```text
Username + Password

↓

Session Cookie

↓

JWT

↓

OAuth Access Token

↓

API Key

↓

Client Certificate
```

Your first task is to identify **which mechanism is being used**.

---

# Chapter 2 — Identify the Authentication Mechanism

Suppose Burp intercepts:

```http
GET /api/v1/profile HTTP/1.1

Authorization: Bearer eyJhbGc...
```

Immediately recognize:

```text
Bearer Token
```

---

Suppose you see:

```http
Cookie:

session=ab1234...
```

That's likely:

```text
Session Authentication
```

---

Suppose you see:

```http
X-API-Key:
A82F...
```

That's an:

```text
API Key
```

---

If you see:

```http
Authorization: Basic YWxpY2U6cGFzcw==
```

That's:

```text
HTTP Basic Authentication
```

Knowing the mechanism determines the kinds of tests that make sense.

---

# Chapter 3 — Authentication Flow

Map the authentication lifecycle.

A typical sequence looks like:

```text
Login
    │
    ▼
Receive Credential
    │
    ▼
Store Credential
    │
    ▼
Send Credential
    │
    ▼
Protected Endpoint
```

For each step, ask:

- What credential is issued?
    
- Where is it stored?
    
- How is it transmitted?
    
- How long is it valid?
    
- How is it revoked?
    

---

# Chapter 4 — Session Authentication

Imagine the server returns:

```http
Set-Cookie:

session=A92F7B
```

On later requests:

```http
Cookie:

session=A92F7B
```

Things to evaluate include:

- Is the cookie marked `Secure`?
    
- Is it marked `HttpOnly`?
    
- Does it use an appropriate `SameSite` setting?
    
- Does it expire?
    
- Is it invalidated after logout?
    
- Does the server rotate the session ID after login?
    

These are all indicators of a mature session implementation.

---

# Chapter 5 — JWT Authentication

Suppose you intercept:

```http
Authorization: Bearer eyJ...
```

Before thinking about attacks, inspect the token.

JWTs have three parts:

```text
Header

↓

Payload

↓

Signature
```

Decode the first two parts (they are Base64URL-encoded, not encrypted) and examine claims such as:

```json
{
  "sub":"25",
  "role":"user",
  "exp":1788000000
}
```

Ask:

- Who is this token for (`sub`)?
    
- When does it expire (`exp`)?
    
- Does it contain roles or permissions?
    
- Is it intended for this audience (`aud`)?
    
- Who issued it (`iss`)?
    

# Chapter 6 — Token Lifetime

Every token should have a reasonable lifetime.

Questions to answer:

- Does it expire?
    
- What happens after expiration?
    
- Is there a refresh mechanism?
    
- Does logout invalidate it?
    
- Can an old token still be used unexpectedly?
    

Understanding the token lifecycle is just as important as understanding its format.

---

# Chapter 7 — Logout

Many APIs implement logout.

Questions to evaluate:

- Does logout actually invalidate the session or token?
    
- If you retry the same request after logging out, what happens?
    
- Are all active sessions terminated, or only the current one?
    

A logout endpoint that doesn't invalidate credentials can leave users exposed if credentials are stolen.

---

# Chapter 8 — API Keys

Suppose an API expects:

```http
X-API-Key:
abc123...
```

Things to assess include:

- Is the key required?
    
- Are keys scoped to specific capabilities?
    
- Can keys be revoked?
    
- Are they tied to a specific account or application?
    
- Are they transmitted over HTTPS?
    

Good API key management includes rotation, revocation, and least privilege.

---

# Chapter 9 — Multi-Factor Authentication (MFA)

Some APIs support MFA.

A typical flow is:

```text
Username

↓

Password

↓

One-Time Code

↓

Authenticated
```

When testing MFA, evaluate whether:

- MFA is consistently enforced where expected.
    
- Recovery workflows are appropriately protected.
    
- Session establishment only occurs after successful completion of all required factors.
    

---

# Chapter 10 — Error Handling

Authentication errors should be informative enough for legitimate users, but not so detailed that they reveal unnecessary information.

Compare:

```http
401 Unauthorized
```

with:

```text
Unknown username
```

or

```text
Password incorrect
```

Excessively specific messages can help attackers enumerate valid accounts, while overly vague messages can frustrate users. Finding the right balance is part of secure design.

---

# Chapter 11 — Rate Limiting

Authentication endpoints are common targets for password guessing.

Questions to assess:

- Is there rate limiting?
    
- Is there account lockout or progressive delays?
    
- Are repeated failed attempts monitored?
    
- Is there CAPTCHA or another mitigation after repeated failures?
    

A login endpoint with no protections against repeated authentication attempts increases risk.

---

# Chapter 12 — Password Policies

If the application manages passwords, evaluate:

- Minimum length.
    
- Support for long passphrases.
    
- Secure password reset process.
    
- Password change process.
    
- Whether old passwords can be reused (if policy requires preventing that).
    

The focus is on whether the implementation supports secure user behavior.

---

# Chapter 13 — Transport Security

Authentication credentials should always be protected in transit.

Verify:

- HTTPS is enforced.
    
- HTTP requests are redirected appropriately or rejected.
    
- Credentials are never sent over plaintext connections.
    
- TLS configuration follows current best practices.
    

Without transport security, even strong authentication can be undermined.

---

# Chapter 14 — Building an Authentication Map

For every API you assess, document something like this:

```text
Authentication
├── Login Endpoint
├── Credential Type
├── Storage Method
├── Expiration
├── Refresh Flow
├── Logout
├── MFA
├── Rate Limiting
└── Password Reset
```

This gives you a complete view of how authentication is implemented.

---

# Authentication Testing Checklist

When assessing an API, work through questions like these:

### Login

- How do users authenticate?
    
- What credential is returned?
    
- Are responses appropriate?
    

### Credential

- Session cookie?
    
- JWT?
    
- API key?
    
- OAuth token?
    

### Storage

- Cookie?
    
- Authorization header?
    
- Local storage (client-side)?
    
- Secure cookie attributes?
    

### Lifetime

- Does it expire?
    
- Is refresh implemented?
    
- Is logout effective?
    

### Protection

- HTTPS?
    
- MFA?
    
- Rate limiting?
    
- Password policy?
    

### Monitoring

- Are repeated failures handled appropriately?
    
- Are authentication events logged by the application (where observable)?
    