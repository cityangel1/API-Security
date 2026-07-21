> **Goal:** Understand how servers identify users, why authentication exists, how modern authentication works, and where it fails.

This is the foundation for understanding:
- Broken Authentication
- JWT attacks 
- Session hijacking
- API Key abuse
- OAuth vulnerabilities
- Account takeover
- Authentication bypasses
---
## Authentication vs Authorization

This is the single most confused concept in cybersecurity.
Let's clear it up permanently.
## Authentication
Authentication answers:
> **Who are you?**

Example:
You enter:

```
Username: Alice
Password: ********
```

Server:

```
✓ You are Alice.
```

Authentication is about **identity**.

---

## Authorization

Authorization answers:

> **What are you allowed to do?**

Example:
Alice logs in.
Can Alice:

- View her account?
    
- Delete another user's account?
    
- Transfer money?
    
- Become an admin?
    

Those are authorization decisions.

---
## Simple Analogy

Imagine an airport.
WELL IF YOU HAVE NEVER BEEN TO ONE.THEN IMAGINE.
### Step 1

Security checks your passport.

```
Who are you?
```

Authentication.

---
### Step 2

Now they check your boarding pass.

```
Which gate can you enter?
```
Authorization.
Different problem.

---
## Why Authentication Exists

Imagine an API.

```
GET /balance
```

Without authentication...
How does the server know which balance?
It doesn't.
Authentication creates identity.
Example:

```
Request

↓

Server identifies user

↓

Server loads user's data

↓

Response
```
No identity.
No personalized data.

---

# The Evolution of Authentication
Let's see how authentication evolved.

---
## Generation 1

Username + Password

```
Alice

Password123
```
Server checks database.
If correct:

```
Access granted.
```

Simple.
But here's a problem.
HTTP is stateless.
Remember Lesson 2?
The server forgets every request.
So after login...

```
GET /profile
```

How does the server know you're still Alice?
It doesn't.
We need memory.

---

# Sessions
The earliest solution.
Imagine a hotel.
You check in.
Reception verifies your ID.
Then gives you a room key.
After that...
They don't ask for your passport every time.
You simply present the room key.
Sessions work exactly like this.

---
## Login

```
POST /login

username

password
```

Server verifies credentials.
Then creates:

```
Session #A92F7B
```

Stores it:

```
Session Database

↓

A92F7B

↓

Alice
```

Then sends:

```
Set-Cookie:

session=A92F7B
```

Browser stores it.

---

# Next Request

```
GET /profile
```

Browser automatically sends:

```
Cookie:

session=A92F7B
```

Server looks it up.

```
A92F7B

↓

Alice
```

Now it knows who you are.

---

## Visual Flow

```
Browser
   │
   │ Username + Password
   ▼
Server
   │
   │ Creates Session
   ▼
Session Database
   │
   │ Session ID
   ▼
Browser Cookie
```

---

# Session IDs

Notice something important.
The cookie does **not** contain:

```
Alice

Password

Permissions
```

Instead it contains only:

```
A92F7B
```

The real information stays on the server.
That's why session-based authentication is called **stateful**.
The server stores the state.

---

# Hacker Thinking

Suppose you steal:

```
Cookie:

session=A92F7B
```

Question:
Can you now impersonate Alice?
Yes.
This is called:
**Session Hijacking.**
The server trusts whoever presents that session ID.
That's why secure cookies (`HttpOnly`, `Secure`, `SameSite`) and TLS are so important.

---

# API Keys
Many APIs don't have users.
Instead they authenticate applications.

Example:

```
Weather App

↓

Weather API
```

The Weather API needs to know:

```
Which application is calling me?
```

It issues:

```
API Key

↓

9ab3f83bdf...
```

Every request includes:

```http
GET /forecast
X-API-Key: 9ab3f83bdf...
```

or sometimes:

```http
Authorization: ApiKey 9ab3f83bdf...
```

The server checks:

```
Is this key valid?

Yes.

Continue.
```

---

## Characteristics of API Keys

API keys are:

- Usually identify an application or client.
    
- Often long random strings.
    
- Typically grant predefined permissions.
    
- Easier to manage than user passwords.
    

But they are **not** user identities.

If an API key leaks, anyone who has it can often use the API until the key is revoked.

---
# Bearer Tokens
Now we move to modern APIs.
Instead of cookies:

```
Cookie:
session=abc123
```

You see:

```http
Authorization:

Bearer eyJhbGc...
```
What does "Bearer" mean?
It literally means:

> **Whoever bears (possesses) this token is treated as the authenticated party.**

Imagine cash.
If you hold a valid $100 bill, the cashier doesn't ask how you obtained it.
Possession is enough.
Bearer tokens work the same way.

---

## Flow

```
Login

↓

Server creates token

↓

Client stores token

↓

Client sends token

↓

Server verifies token

↓

Request accepted
```

---

# JWT (JSON Web Token)
One of the most common authentication mechanisms today.
A JWT looks like this:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
.
eyJzdWIiOiIxMjMiLCJuYW1lIjoiQWxpY2UiLCJyb2xlIjoidXNlciJ9
.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

Three parts.

```
Header

↓

Payload

↓

Signature
```

The payload often contains claims such as:

```json
{
  "sub": "123",
  "name": "Alice",
  "role": "user"
}
```

The signature allows the server to detect tampering.

---

# Stateful vs Stateless Authentication

This distinction is critical.

## Sessions (Stateful)

```
Browser

↓

Session ID

↓

Server

↓

Session Database
```

The server must remember every active session.

---

## JWT (Stateless)

```
Browser

↓

JWT

↓

Server
```

The server verifies the JWT's signature and claims without looking up a session record for every request.
This can improve scalability, but it introduces different security considerations.

---

# Where Authentication Can Fail

Here are some common classes of authentication weaknesses:

- Weak or predictable passwords.
    
- Credential stuffing (reusing leaked credentials).
    
- Missing or weak multi-factor authentication.
    
- Session IDs that are predictable or not invalidated on logout.
    
- Tokens that never expire.
    
- JWT signature validation mistakes.
    
- Exposed API keys.
    
- Login rate limits that are too permissive.
    

The root cause is often the same: the server incorrectly establishes or maintains a user's identity.

---

# Authentication Flow in a Modern API

Let's put it all together.

```
User
   │
   ▼
Login Request
   │
   ▼
API Server
   │
   ▼
Verify Credentials
   │
   ▼
Generate Token
   │
   ▼
Client Stores Token
   │
   ▼
Every Future Request
   │
Authorization: Bearer <token>
   ▼
API Server Verifies Token
   │
   ▼
Request Continues
```

Once authentication succeeds, the server can move on to authorization checks for each request.

---

# Hacker Mindset

Whenever you encounter an API, ask these questions:

1. **How does it authenticate clients?**
    
    - Session cookie?
        
    - JWT?
        
    - API key?
        
    - OAuth?
        
    - Something custom?
        
2. **Where is the credential stored?**
    
    - Cookie?
        
    - Local storage?
        
    - Authorization header?
        
    - Query parameter (a bad practice)?
        
3. **What happens if I remove it?**
    
    - `401 Unauthorized`?
        
    - Still works unexpectedly?
        
4. **Can I reuse someone else's credential?**
    
5. **Does it expire?**
    
6. **Can I modify it?**
    
7. **How does the server verify it?**
    

Those questions guide your authentication testing.
