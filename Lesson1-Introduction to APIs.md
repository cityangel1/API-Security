# What is an API?
API stands for
> **Application Programming Interface**
Although the name sounds complicated, the concept is actually simple.
An API is simply a **messenger** between two software applications.
Instead of programs communicating directly with databases or internal systems, they communicate through APIs.
---
# Restaurant Analogy
Imagine you're at a restaurant.
There are three parties involved:
```
Customer
     │
     ▼
 Waiter (API)
     │
     ▼
 Kitchen
```
You don't walk into the kitchen to prepare your own food.
Instead:
1. You tell the waiter what you want.
2. The waiter tells the kitchen.
3. The kitchen prepares the food.
4. The waiter brings it back.
The waiter is the API.
The API carries requests and responses between two systems.
---
# API in Computing
Suppose your weather app needs today's weather.
Instead of directly accessing the weather company's database, it sends a request to an API.
```
Your Phone
     │
     │ Request
     ▼
Weather API
     │
     ▼
Database
     │
     ▼
Weather API
     │ Response
     ▼
Your Phone
```
Notice something important:
Your phone **never talks directly to the database.**
The API does.
---
# Definition
An API is:
> **A controlled interface that allows two software applications to communicate with each other.**
The important word here is **controlled**.
The API decides:
- What data can be accessed
- Who can access it
- How it can be modified
- Which actions are allowed
---
# Real-World Examples
## Instagram
```
Instagram App
       │
       ▼
Instagram API
       │
       ▼
Instagram Servers
```
Every time you refresh your feed, the app makes several API requests.
---
## Uber
```
Uber App
     │
     ▼
Google Maps API
     │
     ▼
Google Maps Servers
```
Uber doesn't own Google Maps.
It requests map information through Google's API.
---
## Banking Apps
```
Bank App
    │
    ▼
Bank API
    │
    ▼
Bank Database
```
Checking your balance?
That's an API request.
Sending money?
Another API request.
Viewing transactions?
Another API request.
---
# Where Are APIs Used?
The answer is:
> **Almost everywhere.**
## Social Media
- Instagram
- Facebook
- TikTok
- X (Twitter)
- LinkedIn
Every refresh uses APIs.
---
## Banking
Every banking app relies heavily on APIs.
Examples:
- Login
- Account balance
- Transactions
- Transfers
- Card management
---
## Shopping
Amazon uses APIs for almost everything.
Examples:
- Product API
- Cart API
- Order API
- Payment API
- Shipping API
- Recommendation API
---
## Streaming
Netflix
Disney+
YouTube
Spotify
Every movie recommendation comes through APIs.
---
## Games
Modern games use APIs for:
- Login
- Matchmaking
- Friends
- Inventory
- Purchases
- Statistics
Examples include:
- Efootball
- Steam
- Minecraft
---
## Smart Devices
- Alexa
- Google Home
- Smart TVs
- Smart Lights
- Smart Watches
- Smart Cars
Nearly every IoT device communicates using APIs.
---
# Websites Use APIs Too
Many beginners believe APIs are only for mobile apps.
This is incorrect.
Modern websites are mostly API clients.
When you visit Amazon:
```
amazon.com
```
The browser loads the page.
Then JavaScript silently requests:
```
/api/products
/api/cart
/api/orders
/api/recommendations
/api/profile
```
# How APIs Work
Let's use a login example.
Suppose you enter:
```
Username
Password
```
The application sends:
```
POST /login
```
The server receives the request.
It then:
```
Check username
↓

Check password

↓

Check database

↓

Generate token

↓

Return response
```

If successful:

```
{
    "token": "abc123..."
}
```

If unsuccessful:

```
{
    "error": "Invalid credentials"
}
```

---

# The Request–Response Cycle
Every API interaction follows the same process.
```
Client

↓

Send Request

↓

API

↓

Business Logic

↓

Database

↓

API

↓

Response

↓

Client
```
This process happens thousands of times every second in large applications.
---
# Example: Checking Your Bank Balance
Suppose your banking app requests:
```
GET /balance
```
The API performs several checks:

```
Who are you?

↓

Are you authenticated?

↓

Which account belongs to you?

↓

Retrieve balance

↓

Return response
```
Example response:

```json
{
    "balance": 1520.25,
    "currency": "USD"
}
```
The app simply displays the information.

---

# Common API Types
## 1. REST APIs (Most Common)
Example endpoints:
```
GET /users

POST /login

PUT /profile

DELETE /users/15
```
Most web and mobile applications use REST.

---
## 2. GraphQL
Instead of making many requests:
```
Get Name

Get Email

Get Address
```
You send one query requesting exactly the fields you need.
Popular companies include:
- Facebook
- GitHub
- Shopify

---
## 3. SOAP
An older enterprise standard.
Still common in:
- Banks
- Insurance
- Healthcare
- Government systems

---
## 4. gRPC
Designed for:
- High performance
- Microservices
- Cloud applications
Common inside large companies.

---
# Why Companies Use APIs
Imagine if every application connected directly to the database.
Problems:
- No security
- No access control
- Hard to maintain
- Easy to break
Instead:
```
Application

↓

API

↓

Database
```
The API acts as a **security guard**.
It decides:
- Who enters
- What data they see
- What actions they can perform

---

# Why Attackers Target APIs
APIs are attractive because they expose the application's most valuable assets.
---
## 1. Sensitive Data
APIs often expose:
- User profiles
- Emails
- Phone numbers
- Addresses
- Payment information
- Medical records
- Orders
- Messages
---
## 2. Business Logic
APIs contain important business operations like:
- Login
- Payments
- Password reset
- File upload
- Money transfer
- Purchases
Finding a flaw here can have serious consequences.
---
## 3. Public Exposure
Unlike internal databases,
APIs are usually accessible from the Internet.
Anyone can send HTTP requests.
Attackers don't need the official mobile app.
They can create their own requests.
---
## 4. Trusting User Input
Every request contains user-controlled input.
Examples:
- Username
- Password
- Search queries
- File uploads
- JSON data
- IDs
Improper validation can lead to vulnerabilities.
---
## 5. Authorization Mistakes
A common mistake is failing to verify ownership.
Example:
```
GET /users/123
```
If the API doesn't verify ownership,
another user might access someone else's data.
This type of vulnerability is known as **Broken Object Level Authorization (BOLA)** or **IDOR (Insecure Direct Object Reference)**.
---
# Common Goals of Attackers
Attackers may attempt to:
- Read sensitive data
- Access other users' accounts
- Bypass authentication
- Abuse business logic
- Steal API tokens
- Enumerate users
- Exploit authorization flaws
- Perform denial-of-service attacks
---
# Why API Security Matters
Modern applications are built around APIs.
If an API is compromised:
- Mobile apps become vulnerable.
- Websites become vulnerable.
- Third-party integrations become vulnerable.
- Customer data may be exposed.
- Business operations may be disrupted.
Protecting APIs is therefore critical to the security of the entire application.
---
