# Module 11 — Broken Authentication

> **Goal:** Learn how authentication implementations fail, how to recognize those failures, and how to systematically evaluate authentication during an authorized API security assessment.

---

# Chapter 1 — What is Broken Authentication?

Authentication answers one question:

> **"Who are you?"**

Broken authentication occurs when the application incorrectly answers that question.

Examples include:

- Weak password policies
    
- Poor session management
    
- Weak token handling
    
- Password reset weaknesses
    
- Missing MFA where required
    
- Login logic flaws
    
- Inadequate brute-force protection
    

Notice something:

It's about **implementation**.

---

# Chapter 2 — The Authentication Lifecycle

Think of authentication as a complete journey.

```text
Register

↓

Login

↓

Issue Token

↓

Use Token

↓

Refresh Token

↓

Logout

↓

Invalidate Session
```

Any step can contain weaknesses.

Professional testers evaluate **every stage**, not just the login endpoint.

---

# Chapter 3 — Registration

Suppose an API exposes:

```http
POST /register
```

Questions to ask:

- Are duplicate accounts allowed?
    
- Is email verification required?
    
- Can disposable email addresses be used (if policy intends to restrict them)?
    
- Is input validated?
    
- Are passwords handled securely?
    
- Does registration leak information about existing users?
    

A secure registration process lays the foundation for secure authentication.

---

# Chapter 4 — Login

Suppose:

```http
POST /login
```

Request:

```json
{
  "email":"alice@example.com",
  "password":"Password123!"
}
```

Response:

```json
{
   "access_token":"..."
}
```

Your first goal is to understand the flow:

- What credentials are submitted?
    
- What credential is returned?
    
- What HTTP status codes are used?
    
- How are errors reported?
    

---

# Chapter 5 — Error Messages

Compare these responses.

Example A:

```json
{
   "error":"Invalid username"
}
```

Example B:

```json
{
   "error":"Wrong password"
}
```

Problem:

An attacker can distinguish:

- Existing users
    
- Non-existent users
    

A more balanced approach is to avoid revealing which part of the credential pair was incorrect while still giving users useful feedback.

---

# Chapter 6 — Brute Force Protection

Imagine sending many failed login attempts.

Questions to evaluate:

- Is rate limiting present?
    
- Are delays introduced?
    
- Is CAPTCHA used after repeated failures?
    
- Is account lockout implemented appropriately?
    
- Are repeated failures monitored?
    

Strong brute-force defenses reduce the risk of password guessing.

---

# Chapter 7 — Password Policies

A secure API should support strong passwords.

Things to review include:

- Minimum length
    
- Support for long passphrases
    
- Password change process
    
- Password reset process
    
- Password history (if applicable)
    

Modern guidance generally favors allowing long passphrases rather than imposing overly complex composition rules.

---

# Chapter 8 — Session Creation

After login:

The server issues something.

Maybe:

```http
Set-Cookie:
session=abc123
```

Or:

```json
{
   "access_token":"..."
}
```

Ask:

- Is it random?
    
- Is it sufficiently long?
    
- Is it transmitted only over HTTPS?
    
- Does it expire?
    
- Is it unique per login?
    

---

# Chapter 9 — Session Management

Authentication doesn't stop after login.

Imagine:

```text
Login

↓

Session Created

↓

1 Hour Later

↓

Still Active
```

Ask:

- Does inactivity cause expiration?
    
- Is there an absolute timeout?
    
- Is the session rotated after login?
    
- Can users terminate active sessions?
    

Good session management limits exposure if credentials are compromised.

---

# Chapter 10 — Logout

Many applications implement:

```http
POST /logout
```

Questions:

- Does logout invalidate the session?
    
- Does it revoke access tokens?
    
- Are refresh tokens revoked?
    
- What happens on another device?
    

Logging out should meaningfully end the authenticated session.

---

# Chapter 11 — Password Reset

Password reset is often overlooked.

Typical flow:

```text
Forgot Password

↓

Email

↓

Reset Link

↓

New Password
```

Evaluate:

- Is the reset token unpredictable?
    
- Does it expire?
    
- Can it only be used once?
    
- Is it invalidated after use?
    
- Is the user notified of the reset?
    

Weak password reset flows can undermine an otherwise strong login system.

---

# Chapter 12 — Multi-Factor Authentication

Suppose the flow is:

```text
Username

↓

Password

↓

One-Time Code

↓

Authenticated
```

Questions:

- Is MFA required for sensitive accounts?
    
- Is MFA consistently enforced?
    
- Are recovery options secure?
    
- Are new devices verified?
    

MFA significantly raises the difficulty of account compromise when implemented correctly.

---

# Chapter 13 — Refresh Tokens

Many APIs use:

```text
Access Token

↓

Expires

↓

Refresh Token

↓

New Access Token
```

Things to assess:

- Does the refresh token expire?
    
- Is it rotated after use?
    
- Can it be revoked?
    
- Is it stored securely?
    

Refresh tokens usually deserve stronger protection because they can obtain new access tokens.

---

# Chapter 14 — Remember Me

Some applications allow:

```text
Keep me signed in
```

Questions:

- How long does the session last?
    
- Is the device remembered securely?
    
- Can remembered devices be removed?
    
- Is re-authentication required for sensitive actions?
    

Convenience should not come at the expense of security.

---

# Chapter 15 — Device Management

Many modern APIs expose endpoints such as:

```http
GET /sessions

DELETE /sessions/{id}
```

This allows users to:

- View active sessions
    
- Remove old devices
    
- Revoke compromised sessions
    

These features improve account security and visibility.

---

# Chapter 16 — Secure Transport

Credentials should never travel over plaintext HTTP.

Verify:

- HTTPS is enforced.
    
- Strong TLS versions are supported.
    
- Cookies use the `Secure` attribute.
    
- HSTS is configured where appropriate.
    

Transport security protects credentials from interception in transit.

---

# Chapter 17 — Authentication Logging

Healthy authentication systems record important events such as:

- Successful logins
    
- Failed logins
    
- Password resets
    
- MFA enrollment
    
- Session revocation
    

These logs help detect suspicious activity and support incident response.

---

# Chapter 18 — A Complete Authentication Review

When you encounter a new API, build a map.

```text
Authentication
│
├── Registration
├── Login
├── Password Policy
├── Password Reset
├── MFA
├── Session Creation
├── Session Storage
├── Session Expiration
├── Refresh Token
├── Logout
├── Active Sessions
└── Logging
```

This helps ensure you don't overlook part of the authentication lifecycle.

# Practical Exercise

```http
POST /register
POST /login
POST /forgot-password
POST /reset-password
POST /refresh
POST /logout
GET /sessions
DELETE /sessions/{id}
```


1. What stage of the authentication lifecycle does it belong to?
    
2. What credentials or tokens are involved?
    
3. What security properties should the implementation provide?
    
4. What evidence would you look for to confirm those properties during an authorized assessment?