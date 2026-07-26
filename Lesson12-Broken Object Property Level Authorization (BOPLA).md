# Module 12 — Broken Object Property Level Authorization (BOPLA)

> **Goal:** Learn how APIs should protect individual fields (properties) within an object, how these failures happen, and how to systematically identify them during an authorized API security assessment.

---

# Chapter 1 — Objects vs Properties

Let's start with something simple.

Suppose an API returns:

```json
{
    "id":42,
    "username":"alice",
    "email":"alice@example.com",
    "role":"admin",
    "salary":150000,
    "socialSecurityNumber":"123-45-6789"
}
```

What is the object?

```text
User
```

What are the properties?

```text
id

username

email

role

salary

socialSecurityNumber
```

One object.

Many properties.

---

# Chapter 2 — BOLA vs BOPLA
## BOLA

Question:

> **Should I be allowed to access this object?**

Example:

```http
GET /users/42
```

Should Alice access Bob's profile?

If no...

Return:

```http
403 Forbidden
```

---

## BOPLA

Question:

> **I'm allowed to access this object... but should I see every field?**

Example:

Alice requests:

```http
GET /users/me
```

She should absolutely access her own profile.

But should she receive:

```json
{
   "salary":150000
}
```

Maybe not.

Or:

```json
{
   "internalNotes":"User under fraud investigation"
}
```

Definitely not.

This is BOPLA.

---

# Visual Comparison

```text
BOLA
────
Can Alice open Bob's profile?

↓

No

↓

403
```

---

```text
BOPLA
─────
Alice opens her own profile.

↓

Should Alice see salary?

↓

Should Alice see internal notes?

↓

Should Alice see audit history?

↓

Maybe not.
```

---

# Chapter 3 — Why Does BOPLA Happen?

Consider backend code:

```python
user = db.get_user(id)

return user
```

Looks harmless.

But the `user` object contains:

```python
{
    "id":42,
    "email":"alice@example.com",
    "passwordHash":"...",
    "salary":120000,
    "isAdmin":True,
    "apiSecret":"xxxxx"
}
```

Returning the whole object leaks information that wasn't intended for that client.

---

# The Better Approach

Instead of returning everything:

```python
return user
```

Return only what's needed.

Conceptually:

```python
return {
    "id": user.id,
    "username": user.username,
    "email": user.email
}
```

This idea is often called **allow-listing** fields.

---

# Chapter 4 — Read Exposure

Suppose:

```http
GET /profile
```

Returns:

```json
{
    "id":42,
    "email":"alice@example.com",
    "passwordHash":"$2b$12...",
    "lastLoginIP":"192.168.1.50",
    "internalRiskScore":92
}
```

Questions to ask:

Should users see:

- Password hashes?
    
- Internal risk scores?
    
- Internal notes?
    
- Infrastructure details?
    

Often the answer is no.

---

# Chapter 5 — Write Exposure

BOPLA isn't only about reading.

Suppose:

```http
PATCH /users/me
```

Request:

```json
{
   "displayName":"Alice"
}
```

Perfectly normal.

Now imagine the client sends:

```json
{
   "displayName":"Alice",
   "role":"admin"
}
```

Should that field be accepted?

Of course not.

---

# Chapter 6 — Mass Assignment

This is one of the most common causes of BOPLA.

Imagine this backend:

```python
user.update(request.json)
```

Convenient.

Dangerous.

Why?

Because every field supplied by the client is accepted unless explicitly restricted.

A safer design validates and selectively maps only expected fields.

---

# Example

Suppose the database contains:

```json
{
   "username":"alice",
   "role":"user",
   "credit":100,
   "isAdmin":false
}
```

The intended request:

```json
{
   "username":"Alice Cooper"
}
```

Unexpected request:

```json
{
   "username":"Alice",
   "isAdmin":true
}
```

If the server updates every property without checking, that's a serious authorization flaw.

---

# Chapter 7 — Hidden Fields

Developers often assume clients won't know about certain properties.

Examples:

```text
isAdmin

isPremium

accountBalance

isVerified

isEmployee

isInternal

tier

discountRate

approvalStatus

creditLimit
```

Just because a field isn't displayed in the UI doesn't mean it can't be submitted or exposed by the API.

---

# Chapter 8 — Where Do Hidden Fields Come From?

You can learn about properties from many legitimate sources:

- API documentation
    
- OpenAPI specifications
    
- Swagger UI
    
- JSON responses
    
- Error messages
    
- Different user roles
    
- Mobile applications
    
- Version differences
    

As a tester, compare what different authorized users can observe.

---

# Chapter 9 — Comparing Roles

Suppose you have two test accounts.

Admin response:

```json
{
    "username":"Alice",
    "salary":150000,
    "bonus":25000,
    "department":"Finance"
}
```

Regular user response:

```json
{
    "username":"Alice",
    "department":"Finance"
}
```

This comparison helps you understand which properties are intended to be role-restricted.

---

# Chapter 10 — Nested Objects

Example:

```json
{
    "project":{
        "id":22,
        "owner":{
            "id":4,
            "salary":200000,
            "phone":"555-1111"
        }
    }
}
```

The primary object may be the project.

But nested objects also require careful property filtering.

Every nested field deserves consideration.

---

# Chapter 11 — Sensitive Information

Examples of fields that frequently require restricted access:

```text
Password Hash

API Keys

Private Tokens

Secrets

Salary

Medical Records

Internal Notes

Audit Logs

Risk Scores

Fraud Scores

Bank Account Numbers

Encryption Metadata

Security Questions

Recovery Codes
```

Their presence in an API response isn't automatically a vulnerability—but it should prompt careful review of who is authorized to receive them.

---

# Chapter 12 — Partial Updates

Suppose:

```http
PATCH /users/me
```

Body:

```json
{
   "phone":"555-1000"
}
```

Ask:

Can the client also update:

```json
{
   "role":"admin"
}
```

or

```json
{
   "accountType":"enterprise"
}
```

Every writable field should have explicit authorization rules.

---

# Chapter 13 — GraphQL

GraphQL introduces another dimension.

Query:

```graphql
query {
    user {
        username
        email
        salary
        apiSecret
    }
}
```

The API should evaluate authorization **for every requested field**, not just for the `user` object itself.

---

# Chapter 14 — Building a Property Map

Instead of documenting only endpoints, document fields.

Example:

```text
User
│
├── id
├── username
├── email
├── phone
├── address
├── role
├── salary
├── apiKey
├── passwordHash
└── notes
```

Then classify them.

```text
Public

Authenticated User

Administrator

Internal Only
```

This creates a field-level authorization matrix.

---

# Property-Level Authorization Matrix

A useful way to organize your findings is:

| Property     | User         | Manager | Admin      | Internal |
| ------------ | ------------ | ------- | ---------- | -------- |
| username     | ✅ Read/Write | ✅       | ✅          | ✅        |
| email        | ✅ Read/Write | ✅       | ✅          | ✅        |
| role         | ❌            | ❌       | ✅          | ✅        |
| salary       | ❌            | Read    | Read/Write | ✅        |
| apiKey       | ❌            | ❌       | Read       | ✅        |
| passwordHash | ❌            | ❌       | ❌          | ✅        |
