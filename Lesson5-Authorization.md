> **Goal:** Understand how servers decide what an authenticated user is allowed to do, and learn how Broken Authorization vulnerabilities arise.

# The Security Decision

Let's revisit what we learned.

Authentication asks:

```text
Who are you?
```

Authorization asks:

```text
Now that I know who you are...

What are you allowed to do?
```

This decision happens **on nearly every request**.

---

# Example

Suppose Alice logs in.

Authentication succeeds.

The server knows:

```text
User ID: 25
Name: Alice
Role: User
```

Now Alice requests:

```http
GET /users/25
```

Should the server allow it?

Probably yes.

Now Alice requests:

```http
GET /users/26
```

Should that be allowed?

Maybe.

Maybe not.

**Authorization decides.**

---

Now she changes:

```http
GET /users/27
```

Question:

Should Bob's account be returned?

Absolutely not.

The server should perform this check:

```text
Requested Account

↓

Find Owner

↓

Compare Owner

↓

Authenticated User

↓

Allow or Deny
```

If that comparison never happens…

You have a vulnerability.

---

# The Biggest Mistake Developers Make

Many beginners assume:

```text
The user is logged in.

Therefore...

Everything is safe.
```

Wrong.

Logging in proves identity.

It does **not** prove permission.

Imagine this code:

```python
user = authenticate(token)

order = database.get_order(order_id)

return order
```

Looks fine.

But where is:

```python
if order.owner == user:
```

Without that check…

Anyone can retrieve any order.

---

# IDOR

One of the most famous web vulnerabilities.

IDOR stands for:

> **Insecure Direct Object Reference**

Let's understand each word.

---

## Object

Anything the API manages.

Examples:

```text
User

Order

Invoice

Payment

Message

Profile

Image

Document
```

Everything has an identifier.

---

## Direct Reference

Suppose the URL is:

```http
GET /orders/500
```

The client directly references:

```text
500
```

That identifier is called a **Direct Object Reference**.

---

## Insecure

Now imagine:

Alice owns:

```text
Order 500
```

She changes:

```text
500

↓

501
```

Server replies:

```json
{
   "owner":"Bob",
   "amount":900
}
```

Congratulations.

You just found an IDOR.

---

# Visual Example

```text
Alice

↓

GET /users/25

↓

Server

↓

Alice's Profile
```

Now:

```text
Alice

↓

GET /users/26

↓

Server

↓

Bob's Profile
```

That's a Broken Authorization vulnerability.

---

# BOLA

OWASP renamed IDOR for APIs.

The new name is:

> **Broken Object Level Authorization**

Why?

Because APIs rarely use pages.

They expose objects.

Example:

```text
User

Invoice

Transaction

Order

Account
```

Every object needs an authorization decision.

If the decision is missing:

BOLA.

---

# Horizontal Privilege Escalation

Suppose:

Alice

and

Bob

both have:

```text
Role = User
```

Alice accesses Bob's data.

```text
User

↓

User
```

Same privilege level.

Different account.

This is called:

**Horizontal Privilege Escalation.**

---

# Vertical Privilege Escalation

Now imagine:

Alice:

```text
Role = User
```

Admin:

```text
Role = Administrator
```

Alice accesses:

```http
DELETE /admin/users
```

If successful:

That's:

**Vertical Privilege Escalation.**

User

↓

Admin

---

# Role-Based Access Control (RBAC)

Many applications use roles.

Example:

```text
Guest

↓

User

↓

Moderator

↓

Administrator
```

The server checks:

```text
Role

↓

Permission

↓

Action
```

Example:

```http
DELETE /users/15
```

Allowed only if:

```text
Role == Admin
```

---

# Ownership vs Role

These are different.

Suppose:

Admin requests:

```http
GET /users/25
```

Allowed.

Role permits it.

---

Alice requests:

```http
GET /users/25
```

Allowed.

Ownership permits it.

---

Bob requests:

```http
GET /users/25
```

Denied.

Neither ownership nor role permits it.

---

# Real API Example

Suppose you intercept:

```http
GET /api/v1/orders/9001
Authorization: Bearer AliceToken
```

Response:

```json
{
   "owner":"Alice",
   "total":250
}
```

Now change:

```http
GET /api/v1/orders/9002
```

Possible outcomes:

### Good

```http
403 Forbidden
```

Authorization works.

---

### Also acceptable

```http
404 Not Found
```

Some APIs intentionally hide whether the object exists.

---

### Bad

```json
{
   "owner":"Bob",
   "total":900
}
```

That's a Broken Authorization issue.

---
# The Golden Rule

Never trust:

```text
Object IDs

User IDs

Account Numbers

Invoice Numbers

Order Numbers
```

Just because the client sends:

```json
"userId":25
```

doesn't mean:

User 25

should be modified.

The server must verify:

```text
Authenticated User

↓

Owns User 25?
```

---

# How Hackers Think

Suppose you see:

```http
GET /users/125
```

Immediately ask:

Can I try:

```text
124

126

127

1

999

10000
```

Now suppose:

```http
GET /payments/4001
```

Try:

```text
4002

4003

4004
```

This isn't random guessing.

It's testing whether the server performs authorization checks.

---

# Common Objects to Test

Always look for identifiers like:

```text
id

userId

accountId

invoiceId

paymentId

orderId

customerId

projectId

teamId

messageId

fileId

organizationId
```

If changing one identifier returns another user's data, you've likely found a serious issue.

---

# The Backend Must Never Trust the Client

Imagine this request:

```json
{
   "userId":25,
   "amount":100
}
```

A vulnerable backend might do:

```python
transfer_money(
    from_user=request.userId,
    amount=request.amount
)
```

A secure backend ignores the client-supplied `userId` for authorization and instead uses the authenticated identity:

```python
authenticated_user = token.subject

transfer_money(
    from_user=authenticated_user,
    amount=request.amount
)
```

This is a core security principle:

> **Identity comes from authentication, not from user-controlled input.**

---

# Authorization Decision Flow

```text
Request
    │
    ▼
Authenticate User
    │
    ▼
Who is making the request?
    │
    ▼
What resource is requested?
    │
    ▼
Does this user own it?
        │
   Yes ─┴─ No
    │         │
    ▼         ▼
Allow     Deny (403/404)
```

Every protected API endpoint should follow this pattern.

---
