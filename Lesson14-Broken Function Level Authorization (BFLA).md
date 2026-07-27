
- **BOLA:** Can you enter **Room 203**?
    
- **BOPLA:** Once inside Room 203, can you open the **safe**?
    
- **BFLA:** Are you even allowed to enter the **CEO's office**?
    
# Module 14 — Broken Function Level Authorization (BFLA)

> **Goal:** Learn how APIs expose privileged functionality, how authorization should protect business functions, and how to systematically identify function-level authorization weaknesses during an authorized API security assessment.

---

# Chapter 1 — What is a Function?

A function is an **action** the API performs.

Examples:

```text
Create User

Delete User

Approve Payment

Reset Password

Refund Order

Generate Invoice

Create API Key

Delete Project

Promote User

Disable MFA

Export Data
```

Notice something.

These aren't **objects**.

They're **operations**.

---

# Chapter 2 — Object vs Function

Suppose:

```http
GET /users/42
```

Object:

```text
User
```

---

Now:

```http
DELETE /users/42
```

Object?

Still:

```text
User
```

Function?

```text
Delete User
```

Different vulnerability.

---

# Chapter 3 — BOLA vs BFLA

This distinction is critical.

## BOLA

Question:

> **Should Alice access User 42?**

---

## BFLA

Question:

> **Should Alice be allowed to delete users at all?**

Different questions.

One checks ownership.

The other checks permissions to perform an operation.

---

# Visual Comparison

```text
BOLA

Can Alice read Bob's profile?

↓

No

↓

403
```

---

```text
BFLA

Can Alice call DELETE /users?

↓

No

↓

403
```

Notice:

The object may be valid.

The operation is not.

---

# Chapter 4 — Typical Roles

Most APIs define roles.

Example:

```text
Guest

↓

User

↓

Moderator

↓

Administrator

↓

Super Administrator
```

Each role gains additional functions.

Example:

| Function         | User | Admin |
| ---------------- | ---- | ----- |
| View Profile     | ✅    | ✅     |
| Edit Own Profile | ✅    | ✅     |
| Delete User      | ❌    | ✅     |
| Promote User     | ❌    | ✅     |
| Export Database  | ❌    | ✅     |

---

# Chapter 5 — Hidden Endpoints

Imagine the web application has no Delete button.

Developers assume users cannot delete accounts.

But the API exposes:

```http
DELETE /api/users/42
```

Hidden from the UI.

Visible to anyone who inspects traffic.

Security must be enforced on the server—not by hiding buttons.

---

# Chapter 6 — Admin APIs

Many applications separate endpoints.

Normal:

```http
GET /profile
```

Administrator:

```http
GET /admin/users
```

Questions:

Can a normal user call it?

Should they?

If yes...

Broken Function Level Authorization.

---

# Chapter 7 — HTTP Methods Matter

Sometimes only the HTTP method changes.

```http
GET /users/42
```

Allowed.

---

```http
PATCH /users/42
```

Maybe allowed.

---

```http
DELETE /users/42
```

Maybe not.

Authorization often depends on both:

- Endpoint
    
- Method
    

---

# Chapter 8 — Function Discovery

Where do privileged functions come from?

Examples:

- API documentation
    
- OpenAPI specifications
    
- Swagger UI
    
- Mobile applications
    
- JavaScript files
    
- Version differences
    
- Error messages
    
- Traffic captured while using administrator accounts (during authorized testing)
    

Always build an inventory of available functions.

---

# Chapter 9 — Administrative Actions

Examples:

```http
POST /admin/create-user

DELETE /admin/users/42

POST /admin/promote

POST /admin/shutdown

POST /admin/export
```

Ask:

Who should perform this action?

If the answer is "only administrators," then every request must verify that role.

---

# Chapter 10 — Business Functions

BFLA isn't limited to admin panels.

Examples:

```http
POST /orders/123/refund
```

Who may refund?

---

```http
POST /payments/approve
```

Who may approve?

---

```http
POST /loan/approve
```

Who should approve?

Manager?

Supervisor?

Finance?

The API must enforce the business workflow.

---

# Chapter 11 — State Transitions

Suppose:

```text
Draft

↓

Submitted

↓

Approved

↓

Completed
```

An employee should not be able to jump directly from:

```text
Draft

↓

Approved
```

Unless their role permits it.

Authorization also protects business process transitions.

---

# Chapter 12 — Role Changes

Example:

```http
POST /users/promote
```

Questions:

Can users promote themselves?

Can managers promote administrators?

Can anyone assign arbitrary roles?

Role-management functions deserve special scrutiny.

---

# Chapter 13 — Feature Flags

Imagine:

```text
Premium Feature
```

Backend:

```http
POST /premium/report
```

The UI hides it.

But the endpoint still exists.

The server must verify:

```text
Does this account have access?
```

Not:

```text
Did the UI display the button?
```

---

# Chapter 14 — Internal APIs

Some applications expose:

```http
/internal/

```

or

```http
/admin/
```

Never assume:

```text
Internal

↓

Safe
```

Internal endpoints still require proper authorization if they are reachable.

---

# Chapter 15 — Building a Function Map

When testing an API, don't only list endpoints.

List functions.

```text
User

├── Login
├── Logout
├── Change Password
├── Upload Avatar
├── Delete Account
├── Reset Password
├── Export Data
└── Generate API Key
```

Then assign expected roles.

```text
Administrator

↓

Delete User

Export All Users

Reset MFA

View Audit Logs
```

This creates a **Function Authorization Matrix**.

---

# Chapter 16 — Authorization Matrix

Example:

| Function        | Guest | User | Manager | Admin |
| --------------- | ----- | ---- | ------- | ----- |
| Login           | ✅     | ✅    | ✅       | ✅     |
| Register        | ✅     | ✅    | ✅       | ✅     |
| Change Password | ❌     | ✅    | ✅       | ✅     |
| View Reports    | ❌     | ✅    | ✅       | ✅     |
| Approve Reports | ❌     | ❌    | ✅       | ✅     |
| Delete Reports  | ❌     | ❌    | ❌       | ✅     |
| Manage Users    | ❌     | ❌    | ❌       | ✅     |

Creating this matrix helps identify inconsistencies.

---

# Chapter 17 — Testing Methodology

When you discover:

```http
POST /admin/create-user
```

Ask:

### Step 1

What function is this?

```text
Create User
```

---

### Step 2

Who should perform it?

Administrator?

Manager?

Support?

---

### Step 3

What role is the authenticated account using?

---

### Step 4

Does the server actually enforce that authorization?

A secure implementation should deny requests from accounts without the required privileges.
