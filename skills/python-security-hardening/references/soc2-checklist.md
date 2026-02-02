# SOC 2 Compliance Checklist

Map SOC 2 Trust Service Criteria to Python code patterns and security controls.

## Relevant SOC 2 Controls

### CC6.1 - Logical Access Security

**Requirement:** The entity implements logical access security software, infrastructure, and architectures to protect information assets.

| Control | Implementation |
|---------|----------------|
| Authentication | Use established libraries (e.g., `passlib`, `argon2-cffi`) |
| Password storage | Argon2id or bcrypt, never plaintext or weak hashes |
| Session management | Cryptographically random tokens, secure expiration |
| MFA | Implement TOTP/WebAuthn where applicable |

**Code patterns to enforce:**

```python
# GOOD - Secure password hashing
from argon2 import PasswordHasher

ph = PasswordHasher()
hashed = ph.hash(password)
ph.verify(hashed, password)

# BAD - Forbidden patterns
hashlib.md5(password.encode())  # S303 - Insecure hash
hashlib.sha1(password.encode())  # S324 - Insecure hash
password == stored_password  # Plaintext comparison
```

---

### CC6.6 - System Boundaries

**Requirement:** The entity implements logical access security measures to protect against threats from sources outside its system boundaries.

| Control | Implementation |
|---------|----------------|
| Input validation | Validate all external input at system boundary |
| Output encoding | Encode output to prevent injection |
| API authentication | Require authentication for all API endpoints |
| Rate limiting | Implement rate limiting on public endpoints |

**Code patterns to enforce:**

```python
# GOOD - Input validation
from pydantic import BaseModel, validator, constr

class UserInput(BaseModel):
    email: constr(max_length=254)
    name: constr(max_length=100, regex=r'^[\w\s-]+$')

    @validator('email')
    def validate_email(cls, v: str) -> str:
        if '@' not in v or '..' in v:
            raise ValueError('Invalid email')
        return v.lower()

# BAD - No validation
user_input = request.json  # Unvalidated
db.execute(f"SELECT * FROM users WHERE id = {user_id}")  # SQL injection
```

---

### CC6.7 - Information Transmission

**Requirement:** The entity restricts the transmission, movement, and removal of information to authorized internal and external users.

| Control | Implementation |
|---------|----------------|
| TLS/SSL | Enforce HTTPS for all connections |
| Certificate validation | Never disable certificate verification |
| Data encryption | Encrypt sensitive data in transit and at rest |

**Code patterns to enforce:**

```python
# GOOD - Verify certificates
import httpx

response = httpx.get("https://api.example.com", verify=True)

# BAD - Disabled verification (S501)
requests.get("https://api.example.com", verify=False)
httpx.get("https://api.example.com", verify=False)
```

---

### CC7.2 - System Monitoring

**Requirement:** The entity monitors system components for anomalies that are indicative of malicious acts, natural disasters, and errors.

| Control | Implementation |
|---------|----------------|
| Structured logging | Use structured logging (JSON) |
| Audit trails | Log security-relevant events |
| No sensitive data in logs | Never log passwords, tokens, PII |
| Error handling | Don't expose stack traces to users |

**Code patterns to enforce:**

```python
# GOOD - Structured logging without sensitive data
import structlog

logger = structlog.get_logger()

def login(username: str, password: str) -> None:
    logger.info("login_attempt", username=username)  # No password logged
    # ... authentication logic
    logger.info("login_success", username=username, user_id=user.id)

# BAD - Logging sensitive data
logger.info(f"Login attempt with password: {password}")  # NEVER
logger.debug(f"Request headers: {request.headers}")  # May contain tokens

# GOOD - Error handling without information disclosure
try:
    result = dangerous_operation()
except Exception as e:
    logger.exception("operation_failed", error_type=type(e).__name__)
    raise HTTPException(status_code=500, detail="Internal error")

# BAD - Exposing stack trace
except Exception as e:
    return {"error": str(e), "traceback": traceback.format_exc()}
```

---

### CC8.1 - Change Management

**Requirement:** The entity authorizes, designs, develops or acquires, configures, documents, tests, approves, and implements changes to meet its objectives.

| Control | Implementation |
|---------|----------------|
| Code review | Require PR reviews via CODEOWNERS |
| Automated testing | 100% coverage, mutation testing |
| Static analysis | Ruff, mypy, bandit in CI |
| Dependency management | Dependabot for automated updates |

---

## SECURITY.md Template

Create `SECURITY.md` in repository root:

```markdown
# Security Policy

## Supported Versions

| Version | Supported          |
| ------- | ------------------ |
| 1.x.x   | :white_check_mark: |
| < 1.0   | :x:                |

## Reporting a Vulnerability

We take security vulnerabilities seriously. If you discover a security issue, please report it responsibly.

### How to Report

1. **DO NOT** create a public GitHub issue for security vulnerabilities
2. Email security@yourcompany.com with:
   - Description of the vulnerability
   - Steps to reproduce
   - Potential impact
   - Any suggested fixes (optional)

### What to Expect

- **Acknowledgment:** Within 48 hours
- **Initial Assessment:** Within 5 business days
- **Resolution Timeline:** Depends on severity
  - Critical: 24-72 hours
  - High: 1-2 weeks
  - Medium: 2-4 weeks
  - Low: Next release cycle

### Disclosure Policy

- We follow coordinated disclosure
- We will credit researchers who report valid vulnerabilities (unless they prefer anonymity)
- We will not take legal action against researchers who follow this policy

## Security Measures

This project implements:

- Static analysis with ruff, mypy, and bandit
- 100% test coverage requirement
- Mutation testing for test quality
- Dependency vulnerability scanning
- Secret scanning with gitleaks
- Required code review for all changes
```

---

## Cryptographic Standards

### Allowed Algorithms

| Purpose | Allowed | Forbidden |
|---------|---------|-----------|
| Password hashing | Argon2id, bcrypt, scrypt | MD5, SHA1, SHA256 (alone) |
| Symmetric encryption | AES-256-GCM, ChaCha20-Poly1305 | DES, 3DES, AES-ECB |
| Asymmetric encryption | RSA-4096, Ed25519 | RSA-1024, DSA |
| Hashing (integrity) | SHA-256, SHA-3, BLAKE2 | MD5, SHA1 |
| Random generation | `secrets` module | `random` module |

### Code patterns:

```python
# GOOD - Secure random
import secrets

token = secrets.token_urlsafe(32)
otp = secrets.randbelow(1000000)

# BAD - Insecure random (S311)
import random
token = ''.join(random.choices(string.ascii_letters, k=32))

# GOOD - Secure hashing
import hashlib

digest = hashlib.sha256(data).hexdigest()

# BAD - Weak hashing for security (S303, S324)
digest = hashlib.md5(data).hexdigest()
digest = hashlib.sha1(data).hexdigest()
```

---

## Compliance Checklist

```
Access Control (CC6.1):
- [ ] Password hashing uses Argon2id or bcrypt
- [ ] Sessions use cryptographically random tokens
- [ ] Authentication required for sensitive operations
- [ ] Role-based access control implemented

System Boundaries (CC6.6):
- [ ] All input validated with Pydantic or similar
- [ ] SQL queries use parameterization
- [ ] API endpoints require authentication
- [ ] Rate limiting implemented

Information Transmission (CC6.7):
- [ ] TLS enforced for all connections
- [ ] Certificate verification enabled
- [ ] Sensitive data encrypted at rest

Monitoring (CC7.2):
- [ ] Structured logging implemented
- [ ] No sensitive data in logs
- [ ] Security events logged (login, permission changes)
- [ ] Error messages don't expose internals

Change Management (CC8.1):
- [ ] CODEOWNERS configured
- [ ] Branch protection enabled
- [ ] CI/CD pipeline enforces all checks
- [ ] Dependabot enabled
```
