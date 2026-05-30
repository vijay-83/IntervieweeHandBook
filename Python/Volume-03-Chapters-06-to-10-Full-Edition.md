# Volume 03 – FastAPI & Production Architecture
# Chapters 6–10 Full Edition

## Chapter 6 – Middleware Architecture

### Learning Objectives
- Understand middleware pipelines
- Implement cross-cutting concerns

### What is Middleware?

Middleware executes before and after request processing.

### Common Uses

- Logging
- Authentication
- Correlation IDs
- Rate Limiting
- Security Headers

### Example

```python
@app.middleware("http")
async def log_requests(request, call_next):
    response = await call_next(request)
    return response
```

### Interview Questions

1. What is middleware?
2. Execution order of middleware?
3. Middleware vs dependency?

---

## Chapter 7 – Exception Handling

### Goals

- Consistent API responses
- Better observability
- Improved client experience

### Example

```python
from fastapi import HTTPException

raise HTTPException(404, "User not found")
```

### Global Exception Handler

```python
@app.exception_handler(Exception)
async def handler(request, exc):
    pass
```

### Architecture Discussion

Use centralized exception handling for consistency.

---

## Chapter 8 – Authentication & Authorization

### Authentication

Verifies identity.

Example:

- Username/Password
- SSO
- OAuth2

### Authorization

Determines permissions.

Example:

- Admin
- User
- Auditor

### Interview Question

Authentication vs Authorization?

### Expected Answer

Authentication verifies who you are.

Authorization verifies what you can do.

---

## Chapter 9 – JWT, OAuth2 & OpenID Connect

### JWT Structure

```text
Header
Payload
Signature
```

### Benefits

- Stateless
- Scalable
- Portable

### OAuth2

Delegated authorization framework.

### Common Flows

- Authorization Code
- Client Credentials
- Device Code

### OpenID Connect

Adds identity on top of OAuth2.

### Enterprise Identity Providers

- Azure AD
- Okta
- Auth0
- Keycloak

### Interview Questions

1. What is JWT?
2. OAuth2 vs OIDC?
3. Why avoid storing sensitive data in JWT payloads?

---

## Chapter 10 – RBAC, ABAC & API Security

### RBAC

Role-Based Access Control.

Example:

```text
Admin
Manager
User
```

### ABAC

Attribute-Based Access Control.

Example:

```text
Department
Region
Ownership
```

### Security Best Practices

- HTTPS everywhere
- Token expiration
- Refresh tokens
- Secret rotation
- Input validation

### OWASP Risks

- Injection
- Broken Authentication
- Security Misconfiguration
- SSRF

### Architect Discussion

Security should be built into architecture from day one.

---

# Staff Engineer Questions

1. How would you standardize authentication across services?
2. How would you implement centralized authorization?
3. How would you secure internal APIs?

---

# Solution Architect Questions

1. OAuth2 vs JWT-only solutions?
2. How would you design enterprise SSO?
3. How would you secure multi-tenant APIs?

---

# Summary

Key Takeaways

- Middleware handles cross-cutting concerns.
- Centralized exception handling improves consistency.
- Authentication and authorization solve different problems.
- OAuth2 and OIDC are enterprise standards.
- RBAC and ABAC support scalable authorization models.
- Security must be considered at every layer.

# End of Volume 03 Chapters 6–10
