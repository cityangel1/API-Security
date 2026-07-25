# Module 10 — Broken Object Level Authorization (BOLA)

> **Goal:** Learn how to identify, reason about, and systematically test authorization at the object level.
---

# Chapter 1 — What is an Object?

In APIs, an object is simply:

> **A piece of data the application manages.**

Examples:

```text
User
Order
Invoice
Payment
Transaction
File
Message
Project
Task
Photo
Comment
Team
Organization
```

Every one of these has an identifier.

For example:

```http
GET /users/42
```

Object:

```
User
```

Identifier:

```
42
```

---

# Chapter 2 — What is Object-Level Authorization?

Imagine a database.

```
Users

Alice
ID = 42

Bob
ID = 43

Carol
ID = 44
```

Alice logs in.

Authentication succeeds.

Now Alice requests:

```http
GET /users/42
```

The server asks:

```
Does Alice own User 42?
```

Yes.

Return the data.

---

Now Alice changes:

```http
GET /users/43
```

The server should ask:

```
Does Alice own User 43?
```

No.

Return:

```http
403 Forbidden
```

or sometimes

```http
404 Not Found
```

---

If instead the server returns Bob's profile...

That is BOLA.

---

# Chapter 3 — Why BOLA Happens

Consider this backend code:

```python
user = authenticate(token)

profile = database.get_user(request.user_id)

return profile
```

Looks innocent.

But notice what's missing.

Where is:

```python
if profile.owner == user.id:
```

Without that comparison...

Every authenticated user can retrieve every profile.

---

# The Correct Logic

A secure implementation should conceptually look like:

```python
user = authenticate(token)

profile = database.get_user(request.user_id)

if profile.owner != user.id:
    deny()

return profile
```

The comparison is everything.

---

# Chapter 4 — Authentication is NOT Enough

This is the biggest beginner mistake.

Developers think:

```
User logged in

↓

Safe
```

Wrong.

Correct model:

```
User logged in

↓

Who are they?

↓

What resource?

↓

Do they own it?

↓

Should they access it?
```

Authorization is a separate decision.

---

# Chapter 5 — The Simplest BOLA

Suppose Burp intercepts:

```http
GET /api/users/42
Authorization: Bearer AliceToken
```

Response:

```json
{
   "id":42,
   "name":"Alice"
}
```

Now change:

```http
GET /api/users/43
```

Possible outcomes.

Good:

```http
403 Forbidden
```

Good:

```http
404 Not Found
```

Bad:

```json
{
   "id":43,
   "name":"Bob"
}
```

---

# Chapter 6 — IDs Aren't Always Numbers

Beginners only test:

```
42

43

44
```

Professionals know identifiers come in many forms.

Examples:

UUIDs

```text
550e8400-e29b-41d4-a716-446655440000
```

MongoDB ObjectIDs

```text
507f1f77bcf86cd799439011
```

GUIDs

Hashes

Slugs

```text
alice-profile
```

Emails

```text
alice@example.com
```

Order numbers

```text
ORD-2026-100045
```

Never assume object IDs are integers.

---

# Chapter 7 — Objects Hide Everywhere

Consider:

```http
GET /orders/9001
```

Object:

```
Order
```

---

```http
GET /messages/500
```

Object:

```
Message
```

---

```http
GET /projects/18/tasks/7
```

Objects:

```
Project

Task
```

Notice something.

There are two authorization decisions.

Does the user own:

- Project 18?
    
- Task 7?
    

---

# Nested Object Problems

Suppose:

```http
GET /projects/10/tasks/4
```

Question:

Does Task 4 belong to Project 10?

Some vulnerable applications check only one relationship.

Example:

```
Task 4 exists

↓

Return it
```

Instead they should verify:

```
Task 4

↓

Belongs to Project 10

↓

Project belongs to Alice

↓

Allow
```

Every relationship matters.

---

# Chapter 8 — Multi-Tenant Applications

Imagine Slack.

Company A

```
Workspace A
```

Company B

```
Workspace B
```

User Alice belongs to Workspace A.

She requests:

```http
GET /api/workspaces/B/channels
```

Should that work?

Absolutely not.

Multi-tenant authorization is one of the most common sources of BOLA findings.

---

# Chapter 9 — Hidden Identifiers

Not every object ID is in the URL.

Example:

```json
{
   "userId":42,
   "amount":100
}
```

Question:

Who determines `userId`?

The client?

Or the server?

A secure backend derives identity from the authenticated token, not from client-supplied identifiers.

---

# Chapter 10 — HTTP Methods Matter

BOLA isn't limited to reading data.

Example:

Reading

```http
GET /users/42
```

Updating

```http
PATCH /users/42
```

Deleting

```http
DELETE /users/42
```

Creating related resources

```http
POST /users/42/addresses
```

Every operation involving an object requires authorization.

---

# Chapter 11 — Real-World Examples

## Banking

```http
GET /accounts/12345
```

Question:

Does the account belong to the authenticated customer?

---

## E-commerce

```http
GET /orders/900
```

Question:

Did this customer place order 900?

---

## Healthcare

```http
GET /patients/55
```

Question:

Is this clinician authorized to view patient 55?

---

## Cloud Storage

```http
GET /files/abc123
```

Question:

Does this user have permission to access this file?

The pattern is always the same.

---

# Chapter 12 — The BOLA Testing Methodology

When you find an endpoint, don't immediately think:

> "Can I change the ID?"

Think systematically.

---

## Step 1

Identify the object.

Example:

```http
GET /projects/22
```

Object:

```
Project
```

---

## Step 2

Identify the identifier.

```
22
```

---

## Step 3

Ask:

Who owns it?

---

## Step 4

Determine how ownership is established.

- User account?
    
- Organization?
    
- Team?
    
- Subscription?
    
- Workspace?
    

---

## Step 5

Test with different authorized identities (where your engagement provides them).

Examples include:

- Two normal users
    
- An administrator and a normal user
    
- Users in different organizations or workspaces
    

Comparing behavior across legitimate test accounts is one of the safest and most effective ways to validate authorization.

---

## Step 6

Observe the response.

Good:

```
403
```

Good:

```
404
```

Potential issue:

```
200
```

with another user's data.

---

# BOLA Checklist

Whenever you see an endpoint:

```http
GET /users/15
```

Ask:

✓ What is the object?

✓ Who owns it?

✓ How is ownership verified?

✓ Is the ID user-controlled?

✓ Is authorization checked on every request?

✓ Does changing the identifier expose another user's data?

✓ Does the application return the appropriate authorization response?

---

# Thinking Like the Backend

Imagine every request goes through an invisible security guard.

```
Incoming Request
        │
        ▼
Authenticate User
        │
        ▼
Identify Resource
        │
        ▼
Who Owns Resource?
        │
        ▼
Compare Ownership
        │
   Yes ─┴─ No
        │         │
        ▼         ▼
     Allow      Deny
```

If the application skips the **Compare Ownership** step, BOLA becomes possible.
