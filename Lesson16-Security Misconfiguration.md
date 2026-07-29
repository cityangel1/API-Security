# Module 16 — Security Misconfiguration

> **Goal:** Learn how API deployments become insecure due to poor configuration, how to systematically identify these weaknesses, and how to evaluate security posture during an authorized API assessment.

---

# Chapter 1 — What is Security Misconfiguration?

Imagine two identical APIs.

API A

```text
Excellent code

Poor configuration
```

API B

```text
Average code

Excellent configuration
```

Which one is more likely to be compromised?

Often...

API A.

Because security is more than writing good code.

It includes:

- Deployment
    
- Configuration
    
- Infrastructure
    
- Default settings
    
- Permissions
    
- Operational practices
    

---

# Chapter 2 — Layers of Configuration

Think of an API as multiple layers.

```text
Internet
      │
      ▼
CDN / WAF
      │
      ▼
Load Balancer
      │
      ▼
Reverse Proxy
      │
      ▼
Web Server
      │
      ▼
Application
      │
      ▼
Database
      │
      ▼
Operating System
```

A misconfiguration at **any** layer can weaken the overall security.

---

# Chapter 3 — Debug Mode

One of the most common mistakes.

Instead of:

```http
500 Internal Server Error
```

Production returns:

```text
NullReferenceException

File:
/var/www/app/controllers/user.py

Line:
214

Stack Trace:
...
```

The application just leaked:

- Framework
    
- File paths
    
- Internal code structure
    
- Function names
    
- Sometimes secrets
    

Debug information belongs in logs—not client responses.

---

# Chapter 4 — Verbose Error Messages

Example:

```json
{
   "error":"SQL syntax error near users WHERE id=42"
}
```

Or:

```json
{
   "exception":"MongoTimeoutException"
}
```

Or:

```json
{
   "framework":"Spring Boot"
}
```

Good error handling reveals only what the client needs to know.

Detailed diagnostics should remain server-side.

---

# Chapter 5 — Swagger & OpenAPI Exposure

Many APIs expose:

```text
/swagger

/swagger-ui

/api-docs

/openapi.json

/openapi.yaml
```

Documentation is useful.

But ask:

- Is it intended to be public?
    
- Does it expose internal endpoints?
    
- Does it list administrative functions?
    
- Does it include example API keys or secrets?
    

Exposed documentation often becomes a roadmap for attackers.

---

# Chapter 6 — Default Credentials

Imagine deploying:

```text
Username:
admin

Password:
admin
```

Or:

```text
root

password
```

This remains surprisingly common in:

- Appliances
    
- Containers
    
- Development systems
    
- Admin dashboards
    

Always verify that defaults have been changed.

---

# Chapter 7 — Default Secrets

Developers sometimes commit:

```text
JWT_SECRET=secret

API_KEY=123456

PASSWORD=password
```

Hardcoded or default secrets dramatically reduce security.

Production secrets should be unique and securely managed.

---

# Chapter 8 — Development Endpoints

Production unexpectedly exposes:

```text
/debug

/test

/dev

/staging

/internal

/phpinfo

/actuator
```

Questions:

- Should these exist?
    
- Are they authenticated?
    
- Are they intended for production?
    

Development features frequently leak sensitive information.

---

# Chapter 9 — Directory Listings

Suppose requesting:

```text
/uploads/
```

Returns:

```text
backup.zip

database.sql

config.yml

test.json

logs.txt
```

Directory indexing can unintentionally expose sensitive files.

---

# Chapter 10 — Backup Files

Common examples:

```text
config.bak

config.old

database.sql

backup.zip

site.tar.gz

app.rar
```

These files sometimes contain:

- Source code
    
- Credentials
    
- Configuration
    
- Database dumps
    

They should never be publicly accessible.

---

# Chapter 11 — CORS Misconfiguration

Browsers enforce the Same-Origin Policy.

APIs can relax it using CORS.

Example:

```http
Access-Control-Allow-Origin:
*
```

Questions:

- Is unrestricted cross-origin access appropriate?
    
- Are credentials allowed?
    
- Is the origin validated correctly?
    

Poor CORS configuration can expose authenticated APIs to unintended web origins.

---

# Chapter 12 — HTTP Methods

Suppose:

```http
OPTIONS /
```

Returns:

```text
GET

POST

PUT

PATCH

DELETE

TRACE
```

Ask:

Should every method be enabled?

Unused methods should generally be disabled where practical.

---

# Chapter 13 — Missing Security Headers

Common headers include:

```text
Strict-Transport-Security

Content-Security-Policy

X-Content-Type-Options

Permissions-Policy
```

Remember:

Headers are **defense-in-depth**.

Missing headers don't automatically create an exploitable vulnerability, but they may weaken browser-based protections.

---

# Chapter 14 — TLS Configuration

Things to review:

- Deprecated protocol versions
    
- Weak cipher suites
    
- Certificate validity
    
- Certificate chain
    
- HSTS support
    

---

# Chapter 15 — Cloud Storage Misconfiguration

Imagine:

```text
Storage Bucket

↓

Public

↓

Everyone Can Read
```

Or worse:

```text
Everyone Can Write
```

Misconfigured cloud storage has caused numerous real-world data exposures.

---

# Chapter 16 — Containers

Container deployments sometimes expose:

```text
Docker API

Kubernetes Dashboard

Metrics

Health Checks

Debug Ports
```

Questions:

- Are they authenticated?
    
- Should they be Internet-accessible?
    
- Do they reveal sensitive operational details?
    

---

# Chapter 17 — Logging

Logs should contain enough information for operations—but not expose sensitive data.

Avoid logging:

- Passwords
    
- Access tokens
    
- API keys
    
- Session cookies
    
- Personal data unless necessary and protected
    

Good logging balances visibility with confidentiality.

---

# Chapter 18 — Configuration Review Methodology

Whenever you begin assessing an API:

### Step 1

Identify the technologies.

Examples:

- Web server
    
- Framework
    
- CDN
    
- WAF
    

---

### Step 2

Look for exposed infrastructure.

Examples:

```text
Swagger

Actuator

Debug

Metrics

Health
```

---

### Step 3

Review responses.

Do errors reveal:

- Framework names?
    
- Versions?
    
- Stack traces?
    
- File paths?
    

---

### Step 4

Review transport security.

- HTTPS?
    
- TLS configuration?
    
- Security headers?
    

---

### Step 5

Look for forgotten artifacts.

Examples:

```text
.old

.bak

.zip

.sql

.tar.gz
```

---

### Step 6

Evaluate deployment posture.

Ask:

- Is this production?
    
- Are development features disabled?
    
- Are default settings removed?
    
- Is sensitive information unnecessarily exposed?
    

