# How to develop public-facing REST APIs with enterprise production-grade standards?

If you are building a **public-facing REST API for enterprise use**, think of it less as "creating some HTTP endpoints" and more as **designing a long-lived product contract**.

A production-grade API needs to address **API design, security, reliability, scalability, observability, compatibility, documentation, and operations** from day one.

A useful mental model is:

> **API = Contract + Security + Reliability + Operability + Governance**

Microsoft's REST API guidance and OWASP's API Security Top 10 are good foundations for establishing such standards.

## Table of contents

- [1. Start with the API contract](#1-start-with-the-api-contract)
- [2. Use OpenAPI as the contract](#2-use-openapi-as-the-contract)
- [3. Authentication ≠ Authorization](#3-authentication--authorization)
- [4. Object-level authorization is critical](#4-object-level-authorization-is-critical)
- [5. Never trust client-supplied data](#5-never-trust-client-supplied-data)
- [6. Establish a consistent HTTP semantics](#6-establish-a-consistent-http-semantics)
- [7. Design status codes carefully](#7-design-status-codes-carefully)
- [8. Standardize errors](#8-standardize-errors)
- [9. Version your API](#9-version-your-api)
- [10. Pagination from day one](#10-pagination-from-day-one)
- [11. Filtering, sorting and searching need guardrails](#11-filtering-sorting-and-searching-need-guardrails)
- [12. Make operations retry-safe](#12-make-operations-retry-safe)
- [13. Optimistic concurrency](#13-optimistic-concurrency)
- [14. Rate limiting](#14-rate-limiting)
- [15. Protect sensitive business operations](#15-protect-sensitive-business-operations)
- [16. Secrets and credentials](#16-secrets-and-credentials)
- [17. TLS everywhere](#17-tls-everywhere)
- [18. Don't leak implementation details](#18-dont-leak-implementation-details)
- [19. Observability is part of the API](#19-observability-is-part-of-the-api)
- [20. Monitor business metrics as well](#20-monitor-business-metrics-as-well)
- [21. Define SLOs](#21-define-slos)
- [22. Design for dependency failures](#22-design-for-dependency-failures)
- [23. Separate synchronous and asynchronous operations](#23-separate-synchronous-and-asynchronous-operations)
- [24. API Gateway](#24-api-gateway)
- [25. Don't put business authorization exclusively in the gateway](#25-dont-put-business-authorization-exclusively-in-the-gateway)
- [26. API documentation](#26-api-documentation)
- [27. Testing strategy](#27-testing-strategy)
- [28. CI/CD quality gates](#28-cicd-quality-gates)
- [29. API lifecycle management](#29-api-lifecycle-management)
- [30. Enterprise API standards I would establish](#30-enterprise-api-standards-i-would-establish)
- [31. A production architecture](#31-a-production-architecture)
- [The most important shift in thinking](#the-most-important-shift-in-thinking)

---

# 1. Start with the API contract

Don't start by writing controllers.

First define the public contract:

```text
API
 ├── Resources
 ├── Operations
 ├── Request/Response schemas
 ├── Authentication
 ├── Authorization
 ├── Errors
 ├── Versioning
 ├── Pagination
 ├── Rate limits
 └── Compatibility rules
```

For example, suppose you're exposing a Customer API.

Good:

```http
GET    /api/v1/customers/{customerId}
POST   /api/v1/customers
PATCH  /api/v1/customers/{customerId}
DELETE /api/v1/customers/{customerId}
```

Avoid exposing your database structure:

```http
GET /api/v1/customer_table
GET /api/v1/customer_master
GET /api/v1/customer_addresses_join
```

The API should represent the **business/domain model**, not your database schema. This makes the API resilient when the underlying implementation changes.

---

# 2. Use OpenAPI as the contract

Define the API using **OpenAPI**.

For example:

```yaml
GET /api/v1/customers/{customerId}

Parameters:
  customerId: string

Response:
  200:
    Customer

  404:
    ProblemDetails

  401:
    ProblemDetails

  403:
    ProblemDetails
```

Then generate:

- Swagger documentation
- client SDKs
- API tests
- contract tests
- mock servers

This gives you a **contract-first** approach.

For a large enterprise, I would strongly recommend:

```text
OpenAPI
   ↓
API contract
   ↓
Server implementation
   ↓
Client SDKs
   ↓
Contract tests
```

rather than letting the implementation accidentally become the API contract.

---

# 3. Authentication ≠ Authorization

This is one of the most important distinctions.

### Authentication

> Who are you?

For a public API, commonly:

```text
OAuth 2.0 / OpenID Connect
        ↓
Access Token
        ↓
API
```

For example:

```http
Authorization: Bearer eyJhbGciOi...
```

### Authorization

> What are you allowed to do?

For example:

```text
User
 ├── Read Customer
 ├── Update Customer
 └── Cannot Delete Customer

Admin
 ├── Read Customer
 ├── Update Customer
 └── Delete Customer
```

Don't simply check:

```text
IsAuthenticated == true
```

You need authorization at multiple levels.

---

# 4. Object-level authorization is critical

Consider:

```http
GET /customers/123
```

The user has a valid token.

That does **not** mean they can access customer `123`.

You need:

```text
Authenticated?
      ↓
Has permission?
      ↓
Does this customer belong to this tenant/account?
      ↓
Allowed?
```

This is particularly important for public APIs.

OWASP identifies **Broken Object Level Authorization (BOLA)** as API Security Top 10 risk #1.

---

# 5. Never trust client-supplied data

Validate everything:

```text
Headers
Query parameters
Path parameters
Request body
File uploads
URLs
Pagination parameters
Sorting parameters
Filtering parameters
```

For example:

```http
GET /customers?pageSize=999999999
```

shouldn't result in:

```sql
SELECT * FROM Customers
```

Instead:

```text
requested page size = 999999999
maximum allowed = 100
actual page size = 100
```

OWASP specifically calls out unrestricted resource consumption as an API risk.

---

# 6. Establish a consistent HTTP semantics

Use HTTP methods consistently.

| Operation | HTTP |
|---|---|
| Get | GET |
| Create | POST |
| Replace | PUT |
| Partial update | PATCH |
| Delete | DELETE |

For example:

```http
GET /customers/123
```

rather than:

```http
POST /getCustomer
```

And:

```http
POST /customers
```

rather than:

```http
POST /createCustomer
```

Avoid RPC-style APIs unless the operation genuinely represents an action.

---

# 7. Design status codes carefully

Don't return `200 OK` for everything.

Typical standards:

```text
200 OK
201 Created
202 Accepted
204 No Content

400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
412 Precondition Failed
422 Unprocessable Content
429 Too Many Requests

500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
504 Gateway Timeout
```

For example:

```http
POST /customers

201 Created
Location: /api/v1/customers/123
```

---

# 8. Standardize errors

Don't return this:

```json
{
  "error": "something went wrong"
}
```

Use a standardized error structure.

RFC 9457 defines **Problem Details for HTTP APIs**, specifically for machine-readable HTTP API errors.

For example:

```json
{
  "type": "https://api.example.com/problems/customer-not-found",
  "title": "Customer not found",
  "status": 404,
  "detail": "Customer '12345' does not exist.",
  "instance": "/customers/12345",
  "traceId": "01J..."
}
```

This gives developers something they can program against.

---

# 9. Version your API

A public API can live for **many years**.

You cannot assume:

> "We'll just change the endpoint when we need to."

For example:

```text
/api/v1/customers
/api/v2/customers
```

The key principle is:

> **Don't break existing clients.**

Microsoft's API guidance explicitly emphasizes versioning so clients can depend on service stability over time.

But versioning isn't only about URLs.

You need a compatibility policy:

```text
Safe:
    Add optional response field
    Add new endpoint
    Add optional request field

Potentially breaking:
    Remove field
    Rename field
    Change datatype
    Change semantics
    Make optional field mandatory
```

---

# 10. Pagination from day one

Never design:

```http
GET /customers
```

to return potentially millions of records.

Use pagination:

```http
GET /customers?limit=50
```

Response:

```json
{
  "items": [
    ...
  ],
  "next": "eyJvZmZzZXQiOjUwfQ=="
}
```

Prefer **cursor-based pagination** for large/continuously changing datasets:

```text
GET /customers?limit=50&cursor=abc123
```

rather than relying exclusively on:

```text
?page=123456
```

Microsoft recommends server-side paging even where it may not initially appear necessary, because introducing it later can become a breaking change.

---

# 11. Filtering, sorting and searching need guardrails

This is particularly important for enterprise APIs.

Don't blindly expose:

```http
GET /customers?$filter=...
```

against your database.

Define what is supported:

```http
GET /customers?
    status=ACTIVE
    &country=IN
    &limit=50
```

If you support generic filtering:

```text
Allowed fields
Allowed operators
Maximum complexity
Maximum result size
Maximum execution time
```

Otherwise clients can accidentally—or deliberately—create extremely expensive queries.

---

# 12. Make operations retry-safe

This is a **major enterprise production concern**.

Imagine:

```http
POST /payments
```

The client sends the request.

The server processes the payment.

But the network connection dies before the response reaches the client.

The client thinks:

> "It failed."

It retries.

Now you might have:

```text
Payment #1 → ₹10,000
Payment #2 → ₹10,000
```

This is why **idempotency** matters.

For example:

```http
POST /payments

Idempotency-Key: 7f3b...
```

The server stores the result associated with that key.

Retry:

```text
same Idempotency-Key
        ↓
recognize previous operation
        ↓
return previous result
```

Microsoft's API guidance explicitly recommends idempotency and repeatability mechanisms for operations that need safe retries.

---

# 13. Optimistic concurrency

For updates, consider:

```http
ETag: "abc123"
```

Client retrieves:

```http
GET /customers/123

ETag: "abc123"
```

Client updates:

```http
PATCH /customers/123
If-Match: "abc123"
```

If another user changed the resource:

```text
ETag = "xyz789"
```

the API returns:

```http
412 Precondition Failed
```

This prevents:

```text
User A reads X
User B reads X

User A changes X → X1
User B changes X → X2

User B accidentally overwrites User A
```

---

# 14. Rate limiting

Public APIs **must assume abusive clients exist**.

For example:

```text
100 requests/minute/user
1,000 requests/minute/application
10,000 requests/hour/tenant
```

Response:

```http
429 Too Many Requests
Retry-After: 30
```

But rate limiting shouldn't be your only protection.

Consider:

```text
API Gateway
     ↓
Authentication
     ↓
Rate limiting
     ↓
WAF
     ↓
API
```

OWASP specifically identifies unrestricted resource consumption and abuse of sensitive business flows as API risks.

---

# 15. Protect sensitive business operations

Some operations are inherently more dangerous.

For example:

```http
POST /password-reset
POST /payments
POST /orders
POST /coupons/redeem
POST /accounts
```

You may need:

```text
Rate limiting
Bot detection
Fraud detection
CAPTCHA
Step-up authentication
Idempotency
Transaction limits
Business rules
```

This is different from simply protecting the endpoint with authentication.

---

# 16. Secrets and credentials

Never:

```text
API key in source code
Password in configuration
Client secret in Git
JWT signing key in application settings
```

Use:

```text
Azure Key Vault
AWS Secrets Manager
HashiCorp Vault
Kubernetes Secrets + external secret manager
```

And rotate credentials.

---

# 17. TLS everywhere

Public APIs should be:

```text
HTTPS only
```

Typically:

```text
Client
  ↓ TLS
API Gateway
  ↓ TLS
API
  ↓ TLS
Database / internal services
```

Don't expose sensitive traffic over plain HTTP.

---

# 18. Don't leak implementation details

Bad:

```json
{
  "error": "SqlException: Cannot insert NULL into dbo.Customer.Name"
}
```

Good:

```json
{
  "type": "https://api.example.com/problems/invalid-customer",
  "title": "Invalid customer",
  "status": 400,
  "detail": "Customer name is required."
}
```

Your logs can contain the technical exception.

The client shouldn't.

---

# 19. Observability is part of the API

Every production request should be traceable.

For example:

```text
Request
   │
   ├── traceId
   ├── spanId
   ├── tenantId
   ├── clientId
   └── operation
```

Response:

```http
X-Correlation-ID: 01JABC...
```

Logs:

```text
traceId=01JABC
client=customer-portal
operation=GET /customers/123
duration=84ms
status=200
```

Use distributed tracing, metrics and structured logging.

Typical stack:

```text
OpenTelemetry
      ↓
Metrics
Traces
Logs
      ↓
Application Insights / Grafana / Datadog / etc.
```

---

# 20. Monitor business metrics as well

Don't only monitor:

```text
CPU
Memory
HTTP 500
Latency
```

Also monitor:

```text
Customers created
Orders placed
Payments failed
Authentication failures
Rate-limit violations
Requests/client
Requests/tenant
```

An API can have:

```text
HTTP 200 = 99.99%
```

and still have a serious business problem.

---

# 21. Define SLOs

For an enterprise public API, define things such as:

```text
Availability: 99.9%

p50 latency: < 100 ms
p95 latency: < 300 ms
p99 latency: < 1 sec

Error rate: < 0.1%
```

And explicitly define:

```text
What counts as downtime?
Which endpoints are covered?
Are dependencies included?
What happens during planned maintenance?
```

---

# 22. Design for dependency failures

Suppose:

```text
Public API
    ↓
Customer Service
    ↓
Payment Service
    ↓
External Bank
```

The bank becomes unavailable.

Your API shouldn't simply hang until everything times out.

Use:

```text
Timeout
Retry
Circuit breaker
Bulkhead
Fallback
Queue
```

But be careful with retries.

For example:

```text
POST /payment
```

should **not** blindly retry unless the operation is safely idempotent.

---

# 23. Separate synchronous and asynchronous operations

If something takes 30 seconds, don't necessarily keep the HTTP request open.

Instead:

```http
POST /reports
```

Response:

```http
202 Accepted
Location: /reports/abc123
```

Then:

```http
GET /reports/abc123
```

Eventually:

```json
{
  "status": "Completed",
  "downloadUrl": "..."
}
```

This is much more resilient for long-running enterprise operations.

---

# 24. API Gateway

For a public API, I would generally put an API gateway/API management layer in front.

For example:

```text
                    Internet
                       │
                       ▼
                 ┌───────────┐
                 │    WAF    │
                 └─────┬─────┘
                       │
                       ▼
              ┌─────────────────┐
              │  API Gateway    │
              │                 │
              │ Auth            │
              │ Rate limiting   │
              │ Quotas          │
              │ Routing         │
              │ Logging         │
              │ Versioning      │
              └────────┬────────┘
                       │
              ┌────────┴────────┐
              ▼                 ▼
        Customer API       Order API
```

The gateway should handle **cross-cutting concerns**, while business rules remain inside the service.

---

# 25. Don't put business authorization exclusively in the gateway

For example:

```text
Gateway:
    "User has valid JWT"

Service:
    "User can access Customer 123"
```

The service must ultimately enforce domain authorization.

Otherwise someone may bypass the gateway through another internal route.

OWASP's API risks heavily emphasize authorization at object, property and function levels.

---

# 26. API documentation

A production API should have documentation covering:

```text
Authentication
Endpoints
Request schemas
Response schemas
Status codes
Errors
Pagination
Filtering
Rate limits
Idempotency
Versioning
Examples
SDKs
Changelog
Deprecation policy
```

OpenAPI is the machine-readable layer.

Your developer portal is the human-readable layer.

---

# 27. Testing strategy

Don't just write unit tests.

You want:

```text
                 API
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
    Unit       Integration  Contract
    Tests        Tests       Tests
       │          │          │
       └──────────┼──────────┘
                  ▼
              E2E Tests
                  │
                  ▼
          Performance Tests
                  │
                  ▼
           Security Tests
```

Particularly important:

### Contract testing

Ensure:

```text
Client expectation
        ↕
API implementation
```

never diverges accidentally.

### Security testing

Test:

```text
Unauthorized access
BOLA
Privilege escalation
Mass assignment
Injection
Rate limiting
Token manipulation
Sensitive data exposure
```

OWASP's API Top 10 is an excellent checklist for this.

---

# 28. CI/CD quality gates

A mature pipeline might look like:

```text
Developer
   ↓
Pull Request
   ↓
Build
   ↓
Unit Tests
   ↓
Static Analysis
   ↓
Dependency Scan
   ↓
OpenAPI Validation
   ↓
Contract Tests
   ↓
Security Scan
   ↓
Integration Tests
   ↓
Performance Tests
   ↓
Deploy
   ↓
Smoke Tests
   ↓
Production
```

And ideally:

```text
Canary deployment
       ↓
Monitor
       ↓
Progressive rollout
```

rather than:

```text
Deploy everything immediately
```

---

# 29. API lifecycle management

This is frequently forgotten.

You need:

```text
Design
  ↓
Development
  ↓
Preview
  ↓
GA
  ↓
Maintenance
  ↓
Deprecation
  ↓
Sunset
```

For example:

```text
v1 → GA
v2 → Preview
```

Later:

```text
v1 → Deprecated
```

and eventually:

```text
v1 → Retired
```

Clients should receive sufficient notice.

Also maintain an **API inventory**. OWASP explicitly lists improper inventory management as a major API security risk.

---

# 30. Enterprise API standards I would establish

If I were designing an organization's API platform, I'd create a mandatory standard like this:

### API Design

```text
✓ REST/HTTP semantics
✓ Resource-oriented URLs
✓ OpenAPI
✓ Consistent naming
✓ Consistent status codes
✓ Standard error model
✓ Versioning
✓ Pagination
✓ Filtering/sorting rules
✓ Idempotency
✓ Concurrency control
```

### Security

```text
✓ OAuth2/OIDC
✓ TLS
✓ Authentication
✓ Object-level authorization
✓ Function-level authorization
✓ Input validation
✓ Output filtering
✓ Rate limiting
✓ WAF
✓ Secret management
✓ Security headers
✓ Audit logging
```

### Reliability

```text
✓ Timeouts
✓ Retries
✓ Circuit breakers
✓ Idempotency
✓ Bulkheads
✓ Async processing
✓ Graceful degradation
```

### Operations

```text
✓ Structured logs
✓ Metrics
✓ Distributed tracing
✓ Correlation IDs
✓ Dashboards
✓ Alerts
✓ SLOs
✓ Health checks
```

### Developer Experience

```text
✓ Developer portal
✓ OpenAPI
✓ Swagger UI
✓ SDKs
✓ Code examples
✓ Changelog
✓ Migration guides
✓ Sandbox environment
```

### Delivery

```text
✓ Unit tests
✓ Integration tests
✓ Contract tests
✓ Security testing
✓ Performance testing
✓ CI/CD
✓ Canary deployment
✓ Rollback
```

---

# 31. A production architecture

Putting everything together:

```text
                    ┌────────────────────┐
                    │      Clients       │
                    │ Web / Mobile / B2B │
                    └─────────┬──────────┘
                              │
                            HTTPS
                              │
                              ▼
                    ┌────────────────────┐
                    │        CDN         │
                    └─────────┬──────────┘
                              │
                              ▼
                    ┌────────────────────┐
                    │       WAF          │
                    └─────────┬──────────┘
                              │
                              ▼
                    ┌────────────────────┐
                    │   API Gateway      │
                    │                    │
                    │ Authentication     │
                    │ Rate limiting      │
                    │ Quotas             │
                    │ Routing            │
                    │ Threat protection  │
                    └─────────┬──────────┘
                              │
                ┌─────────────┼─────────────┐
                ▼             ▼             ▼
          Customer API    Order API    Payment API
                │             │             │
                ▼             ▼             ▼
           Domain Layer   Domain Layer   Domain Layer
                │             │             │
                └─────────────┼─────────────┘
                              │
                       Message Broker
                              │
                ┌─────────────┼─────────────┐
                ▼             ▼             ▼
             DB/cache      Workers      External APIs

                              │
                              ▼
                    ┌────────────────────┐
                    │ Observability      │
                    │ Logs               │
                    │ Metrics            │
                    │ Traces             │
                    │ Alerts              │
                    └────────────────────┘
```

---

## The most important shift in thinking

Given your background with **.NET, microservices, Azure Service Bus, CQRS/event sourcing and RavenDB**, you already know how to build backend services.

The bigger challenge with a **public API** is thinking about the API as a **product with an external contract**.

Your internal service can change:

```text
RavenDB → PostgreSQL
CQRS → conventional architecture
Service Fabric → Kubernetes
.NET 8 → .NET 12
```

The customer shouldn't care.

The public contract should remain stable:

```text
             PUBLIC API
                 │
        ┌────────┴────────┐
        │                 │
   Stable contract    Stable semantics
        │                 │
        └────────┬────────┘
                 │
          Internal system
          can evolve freely
```

That is one of the biggest differences between an **internal REST API** and a **production-grade public API**.

For your situation, I'd learn this as a **full API engineering discipline**, not merely REST conventions. A particularly good reference set is Microsoft's REST API Guidelines plus OWASP API Security Top 10 and RFC 9457.
