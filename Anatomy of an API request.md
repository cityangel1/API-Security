
> **Goal:** Learn to read, understand, and manipulate every part of an HTTP request.
---
# The Big Picture

Let's start with a real request.

```http
POST /v1/payments HTTP/1.1
Host: api.bank.com
User-Agent: Mozilla/5.0
Authorization: Bearer eyJhbGc...
Content-Type: application/json
Accept: application/json
Origin: https://bank.com
Referer: https://bank.com/dashboard
Content-Length: 73

{
  "recipient":"alice",
  "amount":100,
  "currency":"USD"
}
```
---
# Layer 1 — The Request Line
```http
POST /v1/payments HTTP/1.1
```
Contains three things:

```text
Method

↓

Resource

↓

Protocol
```

---
## The Method

```http
POST
```

Question:

> What action is being requested?

---
A pentester immediately asks:

> **Can I change the method?**

Example:

Original:

```http
GET /profile
```

Try:

```http
POST /profile
```

or

```http
PUT /profile
```

---
## Layer 2 — The Path

Example:

```http
/v1/users/25/orders
```

Break it apart.

```text
v1

↓

users

↓

25

↓

orders
```

Each part tells a story.

---

## Version

```text
v1
```

Means API Version 1.

Sometimes you'll find:

```text
v2

v3

beta

internal
```

Different versions may have different security properties.

---

## Resource

```text
users
```

This is the object.

Examples

```text
orders

payments

accounts

messages

products

transactions
```

---

## Identifier

```text
25
```

Immediately ask:

> What if this becomes

```text
26

27

999

-1

0

00025

../../
```

This thought process is central to authorization and input validation testing.

---
# Layer 3 — Headers
Headers tell the server about the request.
Think of them as metadata.
---
## Host
```http
Host: api.bank.com
```
Question:
Which server should receive this request?
Sometimes virtual hosts rely on this header.

---
## User-Agent

```http
Mozilla/5.0
```
Who sent the request?
Browser
Burp
Python
curl
Many applications log or branch on this value, but it should never be trusted for security decisions.

---

## Authorization
One of the most important headers.

```http
Authorization: Bearer eyJ...
```

This header answers:
> Who are you?
Without it:

```http
401 Unauthorized
```

With it:

```http
200 OK
```

Most API pentests revolve around what happens when you:

- remove it,
    
- replace it,
    
- reuse another token,
    
- or use an expired or malformed token.
    

---

## Cookie
Example:

```http
Cookie:
session=abc123
```

This is another way the server identifies you.
Cookies and bearer tokens often serve similar purposes but are used differently depending on the application.

---

## Content-Type

```http
application/json
```

Meaning:

The body is JSON.

Other examples:

```text
application/xml

multipart/form-data

text/plain

application/x-www-form-urlencoded
```
If the server expects JSON and you send XML, you might get a `415 Unsupported Media Type`.

---
## Accept

```http
Accept: application/json
```

Meaning:

> I'd like the response in JSON.

Sometimes APIs support multiple formats.

---

## Origin

```http
Origin:
https://bank.com
```

This matters for browser security and CORS.
An important point:

> Browsers set the `Origin` header. Attackers can set it to anything when using tools like `curl` or Burp Suite.

If a server trusts `Origin` for authentication or authorization, that's a design flaw.

---

## Referer

```http
Referer:
https://bank.com/dashboard
```

This indicates where the request came from.

Like `Origin`, it is useful for context but should not be treated as proof that a request is legitimate.

---
## Content-Length

```http
73
```

How many bytes are in the body?

Usually handled automatically by clients.

---
# Layer 4 — Body

The body contains the data.

```json
{
    "recipient":"alice",
    "amount":100,
    "currency":"USD"
}
```

This is where many interesting tests happen.

A normal user submits exactly what the application generated.

A pentester asks:

- Can I change `amount`?
    
- Can I send `-100`?
    
- Can I send `"100"` instead of `100`?
    
- Can I omit `currency`?
    
- Can I add unexpected fields?
    

Example:

```json
{
    "recipient":"alice",
    "amount":100,
    "currency":"USD",
    "isAdmin":true
}
```

Even if the frontend never sends `isAdmin`, the backend must safely ignore or reject it unless explicitly supported.

---

# Responses Matter Too

Suppose you send:

```http
GET /users/25
```

Response:

```json
{
    "id":25,
    "email":"alice@example.com",
    "role":"admin"
}
```

Now ask:
Should the current user really see the `role` field?
Should they see the email?
Should they see internal IDs?
Sometimes the vulnerability isn't in what you can send—it's in what the server reveals.

---
# The Trust Boundary

This is arguably the single most important concept in API security.

Look at this diagram:

```text
Browser
    │
    │  ❌ Never trust
    ▼
------------------------------
Internet
------------------------------
    ▼
API Server
    │
    │  ✅ Validate everything
    ▼
Database
```

Everything that comes from the client is **untrusted input**.

That includes:

- Headers
    
- Cookies
    
- Query parameters
    
- JSON fields
    
- Hidden form fields
    
- File names
    
- HTTP methods
    

The frontend may restrict what users can do, but attackers can always craft their own requests.

---

# The Golden Rule of API Security

> **The frontend is a convenience. The backend is the authority.**

If a frontend disables a button:

```html
<button disabled>
```

that is **not** a security control.

An attacker can send the request manually.

If a JavaScript application hides an "Admin" menu, that is **not** a security control.

The backend must verify that the user is actually an administrator before performing privileged actions.

---
## Challenge

answer these questions mentally.

Given:

```http
PATCH /users/42 HTTP/1.1
Authorization: Bearer <token>
Content-Type: application/json

{
  "email":"new@example.com"
}
```

Think about:

1. What identifies the user making the request?
    
2. What identifies the account being modified?
    
3. Should the token owner always be allowed to modify user `42`?
    
4. What happens if you change `42` to `43`?
    
5. What if you add `"role":"admin"` to the JSON?
    
6. Should the backend trust the request simply because it came from the official frontend?
    

If you can reason through those questions, you're already starting to think like an API security tester
---
