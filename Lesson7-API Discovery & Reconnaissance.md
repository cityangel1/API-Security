# Module 7 — API Discovery & Reconnaissance

> **Goal:** Learn how to discover APIs, hidden endpoints, versions, documentation, and attack surfaces before testing for vulnerabilities.

---

# Chapter 1 — The API Attack Surface

Imagine you're testing an online banking application.

A beginner sees:

```text
https://bank.com
```

An experienced tester imagines:

```text
bank.com
│
├── Website
├── Mobile API
├── Admin API
├── Internal API
├── GraphQL API
├── WebSocket API
├── Swagger Docs
├── OpenAPI Specs
├── Legacy API (v1)
├── Beta API (v2-beta)
├── Partner API
└── Authentication API
```

The first lesson:

> **The visible website is almost never the entire attack surface.**

---

# Chapter 2 — Sources of API Endpoints

There are many places where endpoints reveal themselves.

## 1. The Frontend

Every modern web application talks to APIs.

Open Developer Tools.

Network tab.

Refresh.

You'll see requests like:

```http
GET /api/v1/profile
GET /api/v1/orders
POST /api/v1/login
```

These requests immediately reveal:

- Endpoints
    
- Methods
    
- Parameters
    
- Headers
    
- Authentication
    

---

## 2. JavaScript Files

Modern frontends contain API references.

Example:

```javascript
fetch("/api/v1/users")
```

or

```javascript
axios.post("/payments")
```

or

```javascript
const API = "https://api.example.com/v2"
```

JavaScript often leaks:

- Hidden endpoints
    
- Internal API paths
    
- Debug routes
    
- Admin endpoints
    
- Version numbers
    

---

### Useful Tools

- `katana`
    
- `linkfinder`
    
- `xnLinkFinder`
    
- `hakrawler`
    

These help extract URLs and endpoints from web applications and JavaScript.

---

# Chapter 3 — Swagger / OpenAPI

This is every API pentester's dream.

Many developers publish API documentation.

Common locations:

```text
/swagger

/swagger-ui

/swagger/index.html

/api/docs

/docs

/openapi.json

/openapi.yaml

/swagger.json
```

Suppose you visit:

```text
https://api.example.com/swagger
```

You may discover:

```text
GET /users

GET /payments

DELETE /accounts

POST /admin/createUser
```

Sometimes the documentation exposes endpoints that the frontend doesn't use.

Sometimes it even documents internal APIs.

---

# Chapter 4 — Postman Collections

Developers often export Postman collections.

These may contain:

- Every endpoint
    
- Authentication methods
    
- Sample requests
    
- Environment variables
    

A publicly exposed Postman collection can be an invaluable source of reconnaissance.

---

# Chapter 5 — API Version Discovery

Suppose you discover:

```http
GET /v2/users
```

Ask yourself:

Does this exist?

```text
/v1/

/v3/

/beta/

/internal/

/dev/

/test/
```

Many organizations deploy multiple versions simultaneously.

Older versions are often less secure.

---

# Example

Current:

```http
GET /v3/profile
```

Test whether:

```http
GET /v2/profile
```

exists.

Sometimes you'll discover deprecated APIs that remain accessible.

---

# Chapter 6 — Content Discovery

Just like web directories, APIs have hidden paths.

Common ones include:

```text
/api

/api/v1

/api/v2

/internal

/admin

/private

/debug

/graphql

/openapi

/swagger

/metrics

/health

/status
```

Tools such as `ffuf` or `feroxbuster` can help enumerate directories and files where authorized.


---

# Chapter 7 — Wayback Machine & Archived URLs

Endpoints that no longer appear in the application may still be visible in archived data.

Useful tools include:

- `gau`
    
- `waybackurls`
    

These aggregate historical URLs from sources such as the Internet Archive and Common Crawl.

Example:

```bash
gau example.com
```

You might discover:

```text
/api/v1/users

/api/v1/login

/api/v1/export

/api/internal/report
```

Some archived endpoints are still live.

---

# Chapter 8 — Subdomain Enumeration

Large organizations rarely use a single API host.

Examples:

```text
api.example.com

mobile.example.com

admin-api.example.com

partner.example.com

internal.example.com
```

Subdomain enumeration helps reveal additional attack surfaces.

Popular tools include:

- `subfinder`
    
- `assetfinder`
    
- `amass`
    

Once discovered, probe them with tools like `httpx` to identify live services.

---

# Chapter 9 — Fingerprinting APIs

Suppose you find:

```text
https://api.example.com
```

Questions to ask:

- Is it behind Cloudflare?
    
- Is it using NGINX?
    
- Does it expose HTTP/2 or HTTP/3?
    
- Does it return JSON?
    
- Is it GraphQL?
    
- Is it REST?
    
- Is it FastAPI?
    
- Is it Express?
    
- Is it Spring Boot?
    

Fingerprinting helps you understand the technology stack and potential areas to investigate.

---

# Chapter 10 — Burp Suite as a Recon Tool

Burp isn't just for modifying requests.

It builds an API map.

As you browse:

```text
Target
│
├── /login
├── /profile
├── /orders
├── /payments
└── /admin
```

Proxy history reveals:

- Methods
    
- Parameters
    
- Cookies
    
- Tokens
    
- Status codes
    
- Response sizes
    

By the end of your exploration, you should have a mental model of the application.

---

# Chapter 11 — GraphQL Discovery

GraphQL often lives at:

```text
/graphql
```

or

```text
/api/graphql
```

If introspection is enabled, a single query can reveal the entire schema, including:

- Types
    
- Queries
    
- Mutations
    
- Relationships
    

Even when introspection is disabled, GraphQL endpoints can often be identified by their request patterns and error messages.

---

# Chapter 12 — Reading the API Like a Story

Suppose you discover:

```text
GET /projects

GET /projects/15

GET /projects/15/tasks

POST /projects/15/tasks

PATCH /projects/15/tasks/3
```

Don't just list them.

Ask:

- Can a user edit another user's task?
    
- Are task IDs globally unique?
    
- Can I enumerate project IDs?
    
- Does task 3 actually belong to project 15?
    
- Are there hidden resources like `/comments` or `/attachments`?
    

Recon isn't collecting URLs.

It's understanding the application's business logic.

---

# Building an Attack Surface Map

As you enumerate, create a map like this:

```text
Authentication
├── POST /login
├── POST /logout
├── POST /refresh
└── GET /me

Users
├── GET /users
├── GET /users/{id}
├── PATCH /users/{id}
└── DELETE /users/{id}

Orders
├── GET /orders
├── GET /orders/{id}
├── POST /orders
└── PATCH /orders/{id}

Payments
├── GET /payments
├── POST /payments
└── GET /payments/{id}
```

Once you have this map, you can systematically test:

- Authentication
    
- Authorization
    
- Input validation
    
- Business logic
    
- Rate limiting
    

Instead of randomly clicking around.

---

# A Recon Workflow 

When beginning an authorized API assessment, a structured workflow might look like this:

1. **Identify live hosts**
    
    - Find API subdomains.
        
    - Verify which hosts respond.
        
2. **Fingerprint the technology**
    
    - Frameworks
        
    - WAF/CDN
        
    - HTTP versions
        
    - Response formats
        
3. **Collect endpoints**
    
    - Browser traffic
        
    - Burp Proxy
        
    - JavaScript
        
    - Public documentation
        
    - Historical URLs
        
4. **Organize resources**
    
    - Authentication
        
    - Users
        
    - Orders
        
    - Payments
        
    - Admin
        
    - Reporting
        
5. **Predict undocumented endpoints**
    
    - Apply the REST patterns from Module 6.
        
6. **Prioritize testing**
    
    - Authentication
        
    - Authorization
        
    - Sensitive operations
        
    - Administrative functions
        

This approach scales much better than ad hoc exploration.

---
