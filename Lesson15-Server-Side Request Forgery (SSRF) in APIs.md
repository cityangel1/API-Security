# Module 15 — Server-Side Request Forgery (SSRF) in APIs

> **Goal:** Understand how SSRF works, why APIs are especially vulnerable, how modern cloud environments increase its impact, and how to systematically evaluate SSRF during an authorized API security assessment.

---

# Chapter 1 — What is SSRF?

Normally, communication looks like this:

```text
You
 │
 ▼
API Server
 │
 ▼
Database
```

With SSRF:

```text
You
 │
 ▼
API Server
 │
 ▼
ANYTHING the server can reach
```

The important realization is:

> **The server is making the request—not you.**

---

# Chapter 2 — Why APIs Are Vulnerable

APIs frequently fetch resources for users.

Examples:

```text
Import image

Import PDF

Webhook verification

RSS reader

Open Graph preview

Avatar downloader

URL preview

Document converter

Import from GitHub

Import from Google Drive

Import from Dropbox
```

Notice something.

Every one of those features accepts a URL.

That URL becomes a potential SSRF target.

---

# Chapter 3 — A Simple Example

Suppose the API offers:

```http
POST /preview
```

Request:

```json
{
   "url":"https://example.com"
}
```

The backend does:

```python
download(url)
```

Looks harmless.

But now imagine:

```json
{
   "url":"http://127.0.0.1"
}
```

Who connects?

The server.

Not your browser.

---

# Chapter 4 — The Internal Network

Imagine this infrastructure:

```text
Internet
      │
      ▼
API Server
      │
      ├───────────────┐
      ▼               ▼
Database          Redis
      │
      ▼
Admin Service
```

You cannot reach:

- Redis
    
- Internal admin APIs
    
- Internal dashboards
    

But the API server can.

SSRF abuses that trust.

---

# Chapter 5 — Localhost

Developers often trust:

```text
127.0.0.1

localhost

::1
```

Suppose:

```json
{
   "url":"http://127.0.0.1:8080/admin"
}
```

If the backend fetches it, you've reached a service that was never intended to be Internet-accessible.

---

# Chapter 6 — Private IP Addresses

Common internal ranges include:

```text
10.0.0.0/8

172.16.0.0/12

192.168.0.0/16
```

Example:

```json
{
   "url":"http://10.0.0.5"
}
```

The request originates from inside the environment.

---

# Chapter 7 — Cloud Metadata Services

This is why SSRF is especially serious in cloud environments.

Cloud providers expose instance metadata services that applications use to retrieve configuration and credentials.

Examples include:

```text
AWS metadata service

Azure Instance Metadata Service (IMDS)

Google Cloud metadata server
```

Historically, SSRF vulnerabilities have been abused to access cloud metadata when applications lacked proper protections. Modern cloud platforms provide additional defenses (such as IMDSv2 on AWS), but applications should still prevent arbitrary server-side requests.

---

# Chapter 8 — Why Metadata Matters

Metadata may include:

```text
Instance identity

Temporary credentials

Region

Hostname

Instance ID

Tags

Configuration
```

Depending on the cloud provider and configuration, this information can be highly sensitive.

---

# Chapter 9 — SSRF Targets

Think beyond web pages.

Possible server-side targets include:

```text
HTTP

HTTPS

Internal REST APIs

Monitoring systems

Configuration services

Search engines

Object storage APIs

Service discovery endpoints

Health endpoints
```

The objective isn't "fetch a website."

It's "reach something the server can reach."

---

# Chapter 10 — Blind SSRF

Sometimes the application never returns the fetched response.

Example:

```text
You

↓

API

↓

Internal Request

↓

200 OK

↓

No Response Body
```

You cannot see what happened.

This is called **Blind SSRF**.

Evidence may instead come from:

- Server behavior
    
- Timing differences
    
- Out-of-band interaction systems (such as those used by authorized security testing tools)
    

---

# Chapter 11 — Redirects

Imagine:

```text
https://example.com

↓

302 Redirect

↓

http://internal-service
```

Question:

Does the application validate only the original URL?

Or every redirect destination?

Secure implementations validate the final destination before connecting.

---

# Chapter 12 — URL Parsing Pitfalls

Developers often validate URLs incorrectly.

Examples of tricky inputs include:

```text
Different host representations

Unexpected ports

Embedded credentials

Alternate encodings
```

Security decisions should rely on robust URL parsing rather than simple string comparisons.

---

# Chapter 13 — DNS Rebinding

A hostname may initially resolve to:

```text
example.com

↓

Public IP
```

Later it resolves to:

```text
example.com

↓

Internal IP
```

If the application validates only the first lookup, DNS rebinding can sometimes bypass network restrictions.

---

# Chapter 14 — Allowlisting

A stronger approach is:

```text
Allowed:

images.example.com

cdn.example.com
```

Everything else:

```text
Reject
```

Allowlisting trusted destinations is generally safer than attempting to block every dangerous address.

---

# Chapter 15 — Timeouts

Suppose:

```text
http://10.0.0.8
```

The request hangs.

Without timeouts:

```text
Request

↓

Waiting

↓

Waiting

↓

Thread Blocked
```

Good SSRF defenses also include sensible network timeouts.

---

# Chapter 16 — Real-World API Features

Common API functionality that may involve outbound requests:

```text
Avatar import

Webhook delivery

Payment callbacks

RSS readers

Image resizing

Link previews

Video previews

Document import

Website screenshots

URL validation
```

Whenever you see one of these, ask:

> **Does the server fetch a user-controlled URL?**

---

# Chapter 17 — SSRF Testing Methodology

When you discover:

```http
POST /preview
```

Ask yourself:

### Step 1

Does the API accept a URL?

---

### Step 2

Does the server fetch that URL?

---

### Step 3

Can the destination be controlled by the user?

---

### Step 4

Are there restrictions?

- Scheme
    
- Host
    
- Port
    
- Redirects
    
- Timeouts
    
- Allowlists
    

---

### Step 5

Can you safely demonstrate the behavior within the scope of your engagement?

Document observable evidence rather than attempting to access systems outside the authorized scope.

---
