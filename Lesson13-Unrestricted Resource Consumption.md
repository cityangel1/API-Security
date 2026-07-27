# Module 13 — Unrestricted Resource Consumption

> **Goal:** Learn how APIs can be abused by consuming excessive resources, how to recognize weak resource controls, and how to systematically evaluate them during an authorized assessment.

---

# Chapter 1 — What is a Resource?

When you send one API request, the server doesn't simply "return JSON."

It consumes resources.

```text
Your Request
      │
      ▼
Authentication
      │
      ▼
Application Logic
      │
      ▼
Database Queries
      │
      ▼
Memory Allocation
      │
      ▼
CPU Processing
      │
      ▼
Response Generation
```

Every request costs something.

That cost may be:

- CPU
    
- Memory
    
- Disk I/O
    
- Database time
    
- Network bandwidth
    
- Cloud compute
    
- Third-party API usage
    
- Money
    

One request is usually inexpensive.

A million requests may not be.

---

# Chapter 2 — Resource Exhaustion

Imagine a coffee shop.

One customer orders one coffee.

No problem.

Now imagine 5,000 customers all ordering at once.

The coffee machines become the bottleneck.

The same principle applies to APIs.

```text
1 Request

↓

1 Database Query

↓

Fast Response
```

versus

```text
100,000 Requests

↓

100,000 Database Queries

↓

Slow System

↓

Outage
```

---

# Chapter 3 — It's Not Always About Volume

Many beginners think denial of service means:

> "Send millions of requests."

Not necessarily.

One request can be extremely expensive.

Example:

```http
GET /reports?from=2010&to=2026
```

Generating a report covering 16 years might:

- Read millions of database rows
    
- Aggregate statistics
    
- Generate charts
    
- Compress files
    
- Produce PDFs
    

One request.

Huge cost.

---

# Chapter 4 — Pagination

Consider:

```http
GET /users
```

How many users should be returned?

10?

50?

100?

Now imagine:

```http
GET /users?limit=1000000
```

Without limits, the server might attempt to:

- Load one million records
    
- Serialize one million objects
    
- Allocate large amounts of memory
    
- Send a massive response
    

Good APIs enforce reasonable maximum page sizes.

---

# Chapter 5 — File Uploads

Suppose an API accepts uploads:

```http
POST /upload
```

Questions to ask:

- Is there a maximum file size?
    
- Is the number of uploads limited?
    
- Are file types restricted?
    
- Are uploads scanned if required by the application's security model?
    
- Is storage usage monitored?
    

Without controls:

```text
Upload

↓

500 GB

↓

Disk Full

↓

Application Failure
```

---

# Chapter 6 — Image Processing

Imagine:

```http
POST /profile/photo
```

The server:

- Resizes images
    
- Generates thumbnails
    
- Converts formats
    
- Compresses files
    

A single oversized or highly complex image can consume substantial CPU and memory during processing.

Applications should define and enforce upload constraints appropriate for their use case.

---

# Chapter 7 — Expensive Searches

Example:

```http
GET /search?q=*
```

Simple.

Now imagine:

```http
GET /search?q=very_complex_expression
```

Some search operations are significantly more expensive than others.

Questions:

- Does the API limit search complexity?
    
- Are expensive searches cached?
    
- Are execution time limits enforced?
    

---

# Chapter 8 — Export Endpoints

Many APIs include:

```http
GET /export
```

or

```http
GET /reports/export
```

These endpoints often:

- Query large datasets
    
- Generate spreadsheets
    
- Produce PDFs
    
- Compress archives
    

Exports are common candidates for resource exhaustion if they can be requested repeatedly or without limits.

---

# Chapter 9 — Third-Party APIs

Some APIs call external services.

Example:

```text
Client

↓

Application

↓

Payment Provider

↓

Fraud Detection

↓

SMS Gateway

↓

Email Service
```

Each request may incur:

- Latency
    
- Rate limits
    
- Usage quotas
    
- Direct financial cost
    

A poorly protected endpoint could become expensive even without overwhelming your own infrastructure.

---

# Chapter 10 — Infinite Loops

Some endpoints allow recursion.

Example:

```http
GET /comments?depth=999999
```

Without sensible limits:

```text
Comment

↓

Replies

↓

Replies

↓

Replies

↓

Memory Growth
```

Recursive operations should have maximum depth limits.

---

# Chapter 11 — GraphQL Complexity

GraphQL introduces unique challenges.

A query might request:

```graphql
User

↓

Orders

↓

Products

↓

Reviews

↓

Authors

↓

Comments

↓

Likes
```

Even one deeply nested query can generate a large amount of backend work.

Common mitigations include:

- Query depth limits
    
- Complexity scoring
    
- Timeouts
# Chapter 12 — Rate Limiting

One of the most important defenses.

Example policy:

```text
100 requests

↓

Per Minute

↓

Per User
```

Rate limits can also be applied:

- Per API key
    
- Per IP
    
- Per organization
    
- Per endpoint
    

Different endpoints often deserve different limits.

For example:

```text
GET /profile

↓

High Limit
```

versus

```text
POST /login

↓

Much Lower Limit
```

---

# Chapter 13 — Quotas

Rate limits control **speed**.

Quotas control **total usage**.

Example:

```text
Free Plan

↓

10,000 API Calls / Month
```

Premium:

```text
Unlimited
```

or

```text
1,000,000 Calls
```

Quotas prevent unlimited long-term consumption.

---

# Chapter 14 — Timeouts

Suppose a database query runs forever.

Without a timeout:

```text
Request

↓

Database

↓

Waiting...

↓

Waiting...

↓

Waiting...
```

Threads remain occupied.

Eventually:

No resources remain for legitimate users.

Good systems enforce execution time limits.

---

# Chapter 15 — Asynchronous Processing

Instead of:

```text
Client

↓

Wait 5 Minutes

↓

Download Report
```

Modern APIs often do:

```text
Client

↓

Create Job

↓

Job Queue

↓

Background Worker

↓

Report Ready
```

The API returns immediately while expensive work happens asynchronously.

---

# Chapter 16 — Caching

Suppose 10,000 users request:

```http
GET /exchange-rates
```

Without caching:

```text
10,000 Database Queries
```

With caching:

```text
1 Database Query

↓

9,999 Cache Responses
```

Caching greatly reduces resource consumption for frequently requested data.

---

# Chapter 17 — Monitoring

Healthy APIs monitor metrics such as:

- Requests per second
    
- CPU usage
    
- Memory usage
    
- Database query time
    
- Queue length
    
- Error rates
    
- Average response time
    

Security isn't only prevention.

It's also detection.

---

# Chapter 18 — Resource Consumption Testing Workflow

When you discover an endpoint:

```http
GET /reports
```

Think like this:

### Step 1

What resources does this endpoint use?

- CPU?
    
- Memory?
    
- Database?
    
- External APIs?
    
- Storage?
    

---

### Step 2

Can the user influence the cost?

Examples:

- Large date ranges
    
- Large page sizes
    
- Complex filters
    
- Large uploads
    
- Deep recursion
    

---

### Step 3

What limits exist?

- Rate limits
    
- Page size limits
    
- File size limits
    
- Timeouts
    
- Query complexity limits
    

---

### Step 4

Observe behavior.

Does the API:

- Reject excessive requests?
    
- Return a clear error?
    
- Remain responsive?
    

A resilient API should degrade gracefully rather than failing unpredictably.

