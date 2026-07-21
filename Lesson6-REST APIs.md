> **Goal:** Learn how developers design APIs so you can predict hidden endpoints, understand application architecture, and discover attack surfaces without documentation.

---

# Chapter 1 — What is REST?

REST stands for:

> **Representational State Transfer**

Ignore the complicated name.

Instead, think of REST as a **design philosophy**.

It answers one question:

> **How should software expose data over HTTP?**

REST isn't a programming language.

It isn't software.

It isn't a protocol.

It's a **set of architectural principles**.

---

# Before REST

Imagine you're designing an API for a bookstore.

One developer creates:

```text
/getBook

/createBook

/deleteBook

/updateBook

/searchBooks
```

Another developer creates:

```text
/bookLookup

/newBook

/removeBook

/editBook
```

Another creates:

```text
/fetchBook

/bookDelete

/bookInsert
```

Everything works...

But nothing is consistent.

---

REST introduced consistency.

---

# The Core Idea

REST says:

**Resources are nouns.**

Actions are HTTP methods.

Instead of:

```text
/deleteUser
```

REST says:

```http
DELETE /users/15
```

Instead of:

```text
/createOrder
```

REST says:

```http
POST /orders
```

Instead of:

```text
/getInvoice
```

REST says:

```http
GET /invoices/88
```

The URL identifies **what**.

The HTTP method identifies **what to do**.

This is one of REST's biggest ideas.

---

# What is a Resource?

A resource is simply:

> **Anything the API manages.**

Examples

```text
Users

Orders

Payments

Products

Invoices

Accounts

Messages

Photos

Comments

Files

Transactions
```

Everything becomes a resource.

---

# Resources Have IDs

Imagine a database.

```text
Users

1

2

3

4
```

Each resource gets an identifier.

Examples:

```http
GET /users/1

GET /users/2

GET /users/3
```

These identifiers are exactly what you tested for IDOR in Module 5.

Now you understand _why_ they exist.

---

# Collections vs Individual Resources

This distinction appears everywhere.

## Collection

```http
GET /users
```

Meaning:

Return many users.

---

## Single Resource

```http
GET /users/15
```

Meaning:

Return one user.

Notice the pattern.

Plural noun.

Optional identifier.

---

# CRUD

REST maps HTTP methods to four fundamental database operations.

| Database | REST      |
| -------- | --------- |
| Create   | POST      |
| Read     | GET       |
| Update   | PUT/PATCH |
| Delete   | DELETE    |

This is called **CRUD**.

Let's see an example.

---

# Users

Create

```http
POST /users
```

---

Read

```http
GET /users/15
```

---

Update

```http
PATCH /users/15
```

---

Delete

```http
DELETE /users/15
```

Same resource.

Different methods.

---

# Real Example — E-commerce

Suppose Amazon has a Product resource.

View all products:

```http
GET /products
```

View one:

```http
GET /products/25
```

Create:

```http
POST /products
```

Update:

```http
PATCH /products/25
```

Delete:

```http
DELETE /products/25
```

Notice how predictable this becomes.

---

# Nested Resources

Real applications have relationships.

Example:

User

↓

Orders

Instead of:

```http
GET /orders?user=15
```

Developers often write:

```http
GET /users/15/orders
```

This says:

> Show the orders that belong to user 15.

---

# Query Parameters

Suppose:

```http
GET /products
```

returns 2 million products.

Not useful.

Instead:

```http
GET /products?page=2
```

or

```http
GET /products?category=laptops
```

or

```http
GET /products?price=1000
```

These are query parameters.

They modify the request without changing the resource.

---

Common Query Parameters

```text
?page=

?limit=

?offset=

?sort=

?order=

?search=

?filter=

?category=

?status=
```

During a pentest, always inspect them.

Ask:

- Can I exceed limits?
    
- Can I sort on unexpected fields?
    
- Can I filter by another user's ID?
    
- Can I inject special characters?
    

---

# Path Parameters vs Query Parameters

Compare these two requests.

```http
GET /users/15
```

vs

```http
GET /users?id=15
```

Both identify a user.

But:

The first uses a **path parameter**.

The second uses a **query parameter**.

Developers choose one style based on API design.

---

# Predicting Endpoints
Suppose you discover:

```http
GET /users
```

You can reasonably predict:

```http
POST /users

GET /users/15

PATCH /users/15

DELETE /users/15
```

You haven't discovered them.

You're inferring them.

That's an incredibly valuable reconnaissance skill.

---

# Versioning

You'll often see:

```text
/v1/

/v2/

/v3/
```

Example:

```http
GET /v1/users
```

Later:

```http
GET /v2/users
```

Why?

Because APIs evolve.

Versioning allows developers to introduce changes without breaking existing clients.

---

# Statelessness

Remember Lesson2?

REST embraces statelessness.

Each request contains everything needed.

Example:

```http
GET /users/15
Authorization: Bearer token
```

The server shouldn't need to remember previous requests to process this one.

This makes REST APIs easier to scale.

---

# Why REST Matters to Pentesters

Suppose you discover:

```http
GET /customers
```

An experienced tester immediately thinks:

Probably exists:

```text
POST /customers

PATCH /customers/{id}

DELETE /customers/{id}

GET /customers/{id}

GET /customers/{id}/orders

GET /customers/{id}/payments

GET /customers/{id}/subscriptions
```

Notice what's happening.

You're modeling the application's data.

---

# Thinking Like a Developer

Imagine you're building a food delivery app.

Resources might include:

```text
Users

Restaurants

Menus

Orders

Drivers

Payments

Reviews

Coupons

Addresses
```

Now imagine the API.

```http
GET /restaurants

GET /restaurants/10

GET /restaurants/10/menu

POST /orders

GET /orders/55

PATCH /orders/55

GET /drivers/22

GET /users/15/addresses
```

You didn't read documentation.

You designed it in your head.

That's exactly what skilled pentesters do.

---

# REST and Security

REST itself isn't secure or insecure.

But its design influences where vulnerabilities appear.

For example:

```http
GET /users/123
```

immediately raises questions:

- Is user 123 mine?
    
- What if I request 124?
    
- Does the server check ownership?
    

Nested resources raise similar questions:

```http
GET /users/15/orders/900
```

Ask:

- Does order 900 belong to user 15?
    
- What if I change only the order ID?
    
- What if I change only the user ID?
    
- Does the server verify both?
    

Understanding the resource model helps you identify where authorization checks should exist.

---
# A Real-World Example

Imagine you're testing a project management platform.

You find:

```http
GET /projects
```

The application's domain tells you there are probably:

```text
Projects
│
├── Tasks
├── Files
├── Members
├── Comments
├── Labels
├── Milestones
└── Activity
```

You can predict endpoints such as:

```http
GET /projects/5/tasks
POST /projects/5/tasks
GET /projects/5/members
GET /projects/5/files
POST /projects/5/comments
DELETE /projects/5/members/12
```

This is not guesswork—it's using the application's business model to guide your testing.
