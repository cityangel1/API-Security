**Goal:** Understand HTTP so well that you can read an API request like a developer and manipulate it like a pentester.
### What is HTTP?

- **HyperText Transfer Protocol** — a set of agreed rules for communication between clients and servers.
- **Stateless**: Each request is independent. The server does not remember previous requests.
    - This is why mechanisms like **Cookies, Sessions, JWTs, API keys, and OAuth tokens** are needed to maintain state (e.g., knowing a user is logged in).

### Anatomy of an HTTP Request
POST /api/v1/login HTTP/1.1
Host: api.example.com
User-Agent: Mozilla/5.0
Content-Type: application/json
Accept: application/json

{"email":"alice@example.com","password":"Password123!"}
**Components:**

- **Request Line**: METHOD PATH HTTP-VERSION
    - **Method**: The action (what to do)
    - **Path**: The resource/endpoint (where to go)
    - **Version**: Usually HTTP/1.1
- **Headers**: Metadata (Host, User-Agent, Content-Type, Accept, Authorization, etc.)
- **Body** (optional): The actual data/payload (often JSON)

### HTTP Response
HTTP/1.1 200 OK
Content-Type: application/json

{"token":"eyJhbGc..."}
**Components:**

- **Status Line**: Version + Status Code + Reason Phrase
- **Headers**
- **Body**

### HTTP Methods
|Method|Purpose|Key Notes|
|---|---|---|
|**GET**|Retrieve data|Should not change server state|
|**POST**|Create resource / submit data|Login, Register, Upload, etc.|
|**PUT**|Replace entire resource|Idempotent|
|**PATCH**|Partial update of a resource|Only changed fields|
|**DELETE**|Remove resource|—|
|**OPTIONS**|List supported methods|Useful for CORS & recon|
|**HEAD**|Like GET but no body|Check headers/metadata|
### Important Status Codes

- **2xx Success**: 200 OK, **201 Created**, 204 No Content
- **3xx Redirection**: 301 Moved Permanently, 302 Temporary Redirect
- **4xx Client Error**:
    - 400 Bad Request
    - **401 Unauthorized** (not authenticated)
    - **403 Forbidden** (authenticated but not allowed)
    - 404 Not Found
    - 405 Method Not Allowed
    - 409 Conflict
    - 415 Unsupported Media Type
    - 422 Validation Failed
    - 429 Too Many Requests (rate limiting)
- **5xx Server Error**: 500 Internal Server Error (often leaks info)