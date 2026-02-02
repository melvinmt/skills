# Security Testing Methodology

Write tests like a white hat hacker. Test the security premise first, then verify implementation.

## Core Principles

### 1. Security Premise First

```python
# WRONG - Testing implementation details
def test_password_is_hashed():
    """Tests that we call the hash function."""
    user = create_user("test@example.com", "password123")
    assert user.password != "password123"  # Weak test

# RIGHT - Testing security premise
def test_password_cannot_be_recovered_from_storage():
    """Security premise: stored passwords cannot be reversed."""
    user = create_user("test@example.com", "password123")

    # Attacker has access to database
    stored_hash = user.password

    # Verify: cannot recover original password
    assert stored_hash != "password123"
    assert "password" not in stored_hash.lower()

    # Verify: hash is computationally expensive to brute-force
    # (Argon2id parameters should make this infeasible)
    assert len(stored_hash) >= 64  # Minimum hash length
    assert stored_hash.startswith("$argon2")  # Correct algorithm
```

### 2. Implementation Adapts to Tests

Never change tests to match broken implementation:

```python
# WRONG - Weakening test for implementation
def test_rate_limiting():
    """Rate limit is 100 requests per minute."""
    # Changed from 10 to 100 because implementation was "too restrictive"
    for _ in range(100):
        response = client.post("/api/login", json={"user": "test"})
    assert response.status_code == 429

# RIGHT - Keep security requirement, fix implementation
def test_rate_limiting():
    """Security premise: Brute-force attacks are prevented."""
    # 10 attempts is security requirement - implementation must comply
    for _ in range(10):
        response = client.post("/api/login", json={"user": "test"})
        assert response.status_code in [200, 401]

    # 11th attempt must be blocked
    response = client.post("/api/login", json={"user": "test"})
    assert response.status_code == 429
```

### 3. No Exceptions Ever

```python
# FORBIDDEN - Never use these
@pytest.mark.skip(reason="TODO: fix later")  # NO
@pytest.mark.xfail  # NO
# TODO: add security test  # NO - do it now
```

---

## OWASP Top 10 Test Patterns

### A01: Broken Access Control

```python
class TestAccessControl:
    """Verify users cannot access unauthorized resources."""

    def test_user_cannot_access_other_users_data(
        self, client: TestClient, user_a: User, user_b: User
    ) -> None:
        """IDOR prevention: User A cannot access User B's data."""
        client.login(user_a)

        # Attempt to access User B's private resource
        response = client.get(f"/api/users/{user_b.id}/profile")
        assert response.status_code == 403

        response = client.get(f"/api/users/{user_b.id}/documents")
        assert response.status_code == 403

    def test_regular_user_cannot_access_admin_endpoints(
        self, client: TestClient, regular_user: User
    ) -> None:
        """Privilege escalation prevention."""
        client.login(regular_user)

        admin_endpoints = [
            "/api/admin/users",
            "/api/admin/config",
            "/api/admin/logs",
        ]

        for endpoint in admin_endpoints:
            response = client.get(endpoint)
            assert response.status_code == 403, f"Unauthorized access to {endpoint}"

    def test_direct_object_reference_with_uuid_guessing(
        self, client: TestClient, user: User
    ) -> None:
        """Test that sequential/guessable IDs don't leak data."""
        client.login(user)

        # Generate random UUIDs to probe
        import uuid
        for _ in range(100):
            random_id = str(uuid.uuid4())
            response = client.get(f"/api/documents/{random_id}")
            # Should be 404 (not found) not 200 (data leak)
            assert response.status_code in [403, 404]
```

### A02: Cryptographic Failures

```python
class TestCryptography:
    """Verify proper cryptographic practices."""

    def test_passwords_use_secure_hashing(self) -> None:
        """Passwords must use Argon2id."""
        from argon2 import PasswordHasher, Type

        user = create_user("test@example.com", "SecureP@ss123!")
        hash_value = user.password_hash

        # Verify Argon2id is used
        assert hash_value.startswith("$argon2id$")

        # Verify we can verify the password
        ph = PasswordHasher()
        assert ph.verify(hash_value, "SecureP@ss123!")

    def test_sensitive_data_encrypted_at_rest(self) -> None:
        """PII must be encrypted in database."""
        user = create_user_with_ssn("test@example.com", "123-45-6789")

        # Directly query database
        raw_record = db.execute(
            "SELECT ssn_encrypted FROM users WHERE email = ?",
            ("test@example.com",)
        ).fetchone()

        # SSN should not appear in plaintext
        assert "123-45-6789" not in str(raw_record)
        assert raw_record["ssn_encrypted"] is not None

    def test_tokens_are_cryptographically_random(self) -> None:
        """Session tokens must be unpredictable."""
        tokens = [create_session_token() for _ in range(1000)]

        # All tokens unique
        assert len(set(tokens)) == 1000

        # Sufficient entropy (at least 128 bits)
        for token in tokens:
            assert len(token) >= 32  # 256 bits in base64

        # No detectable patterns
        # (More sophisticated: check with statistical randomness tests)
```

### A03: Injection

```python
class TestInjection:
    """Verify injection attacks are prevented."""

    @given(st.text())
    def test_sql_injection_impossible(self, malicious_input: str) -> None:
        """SQL injection must be impossible regardless of input."""
        # This should never raise or return unintended data
        result = search_users(query=malicious_input)

        # Result should be empty or valid users, never an error
        # that indicates SQL execution
        assert isinstance(result, list)

    def test_sql_injection_payloads(self, client: TestClient) -> None:
        """Test known SQL injection payloads."""
        payloads = [
            "'; DROP TABLE users; --",
            "1' OR '1'='1",
            "1; SELECT * FROM passwords",
            "' UNION SELECT * FROM users --",
            "admin'--",
            "1' AND 1=1 --",
            "' OR 1=1#",
        ]

        for payload in payloads:
            response = client.get(f"/api/search?q={payload}")
            # Should not cause error or unexpected behavior
            assert response.status_code in [200, 400]

            if response.status_code == 200:
                # Should not return unexpected data
                data = response.json()
                assert "password" not in str(data).lower()

    @given(st.text())
    def test_command_injection_impossible(self, malicious_input: str) -> None:
        """Command injection must be impossible."""
        # Function that uses subprocess
        result = process_filename(malicious_input)

        # Should handle safely, never execute arbitrary commands
        assert result is not None or result is None  # No exception

    def test_command_injection_payloads(self) -> None:
        """Test known command injection payloads."""
        payloads = [
            "; cat /etc/passwd",
            "| cat /etc/passwd",
            "$(cat /etc/passwd)",
            "`cat /etc/passwd`",
            "&& cat /etc/passwd",
            "|| cat /etc/passwd",
            "\n cat /etc/passwd",
        ]

        for payload in payloads:
            result = process_filename(f"test{payload}")
            # Should not contain passwd file contents
            assert "root:" not in str(result)
```

### A04: Insecure Design

```python
class TestSecureDesign:
    """Verify security is built into design."""

    def test_rate_limiting_prevents_brute_force(
        self, client: TestClient
    ) -> None:
        """Brute-force attacks must be prevented."""
        for i in range(10):
            response = client.post("/api/login", json={
                "email": "victim@example.com",
                "password": f"wrong{i}"
            })

        # After 10 attempts, should be rate limited
        response = client.post("/api/login", json={
            "email": "victim@example.com",
            "password": "another_attempt"
        })
        assert response.status_code == 429

    def test_account_enumeration_prevented(
        self, client: TestClient, existing_user: User
    ) -> None:
        """Cannot determine if account exists via error messages."""
        # Login with existing user, wrong password
        response1 = client.post("/api/login", json={
            "email": existing_user.email,
            "password": "wrongpassword"
        })

        # Login with non-existing user
        response2 = client.post("/api/login", json={
            "email": "nonexistent@example.com",
            "password": "wrongpassword"
        })

        # Error messages must be identical
        assert response1.status_code == response2.status_code
        assert response1.json()["error"] == response2.json()["error"]

        # Response times should be similar (timing attack prevention)
        # This requires timing measurement in real test
```

### A05: Security Misconfiguration

```python
class TestSecurityConfiguration:
    """Verify secure configuration."""

    def test_debug_mode_disabled_in_production(self) -> None:
        """Debug mode must be disabled."""
        from app.config import settings

        if settings.environment == "production":
            assert settings.debug is False
            assert settings.testing is False

    def test_security_headers_present(
        self, client: TestClient
    ) -> None:
        """Security headers must be set."""
        response = client.get("/")

        assert response.headers.get("X-Content-Type-Options") == "nosniff"
        assert response.headers.get("X-Frame-Options") in ["DENY", "SAMEORIGIN"]
        assert "strict-origin" in response.headers.get(
            "Referrer-Policy", ""
        ).lower()
        assert response.headers.get("X-XSS-Protection") == "1; mode=block"

    def test_cors_properly_configured(
        self, client: TestClient
    ) -> None:
        """CORS must not allow all origins."""
        response = client.options(
            "/api/data",
            headers={"Origin": "https://evil.com"}
        )

        # Should not reflect arbitrary origin
        assert response.headers.get(
            "Access-Control-Allow-Origin"
        ) != "https://evil.com"
```

---

## Mutation Testing

Mutation testing verifies that tests actually catch bugs.

### Running mutmut

```bash
# Run mutation testing
mutmut run

# View results
mutmut results

# View a specific surviving mutant
mutmut show 42

# Generate HTML report
mutmut html
```

### Interpreting Results

| Result | Meaning | Action |
|--------|---------|--------|
| Killed | Test caught the mutation | Good - test is effective |
| Survived | Test didn't catch mutation | Bad - add/improve test |
| Timeout | Mutation caused infinite loop | Usually acceptable |
| Suspicious | Inconsistent results | Investigate flaky test |

### Fixing Surviving Mutants

```python
# Original code
def is_admin(user: User) -> bool:
    return user.role == "admin"

# Mutant: changed == to !=
def is_admin(user: User) -> bool:
    return user.role != "admin"  # MUTANT

# If this mutant survives, the test is insufficient:
def test_is_admin_weak():
    admin = User(role="admin")
    assert is_admin(admin)  # Passes for both original and mutant!

# Fixed test:
def test_is_admin():
    admin = User(role="admin")
    regular = User(role="user")

    assert is_admin(admin) is True
    assert is_admin(regular) is False  # Catches the mutant!
```

---

## Property-Based Testing with Hypothesis

Find edge cases automatically:

```python
from hypothesis import given, strategies as st, assume, settings

@given(st.text(max_size=10000))
def test_input_sanitization_handles_all_strings(text: str) -> None:
    """Property: sanitization never crashes and removes dangerous content."""
    result = sanitize_html(text)

    assert "<script" not in result.lower()
    assert "javascript:" not in result.lower()
    assert "onerror=" not in result.lower()
    assert "onclick=" not in result.lower()

@given(
    st.integers(min_value=0),
    st.integers(min_value=0)
)
def test_pagination_never_returns_negative_offset(
    page: int, per_page: int
) -> None:
    """Property: pagination calculation never produces negative values."""
    assume(per_page > 0)  # Skip invalid inputs

    offset, limit = calculate_pagination(page, per_page)

    assert offset >= 0
    assert limit >= 0

@given(st.binary(max_size=10000))
def test_encryption_roundtrip(plaintext: bytes) -> None:
    """Property: decryption reverses encryption exactly."""
    key = generate_key()

    ciphertext = encrypt(plaintext, key)
    decrypted = decrypt(ciphertext, key)

    assert decrypted == plaintext
    assert ciphertext != plaintext  # Actually encrypted

@given(st.dictionaries(st.text(), st.text()))
def test_serialization_roundtrip(data: dict[str, str]) -> None:
    """Property: serialization is reversible."""
    serialized = serialize(data)
    deserialized = deserialize(serialized)

    assert deserialized == data
```

---

## Forbidden Pattern Detection

Add these tests to CI to catch forbidden patterns:

```python
import ast
import pathlib

def test_no_forbidden_patterns_in_codebase() -> None:
    """Scan codebase for forbidden security patterns."""
    forbidden = [
        "eval(",
        "exec(",
        "shell=True",
        "verify=False",
        "# TODO",
        "# FIXME",
        "@pytest.mark.skip",
        "@pytest.mark.xfail",
    ]

    src_path = pathlib.Path("src")
    violations = []

    for py_file in src_path.rglob("*.py"):
        content = py_file.read_text()
        for pattern in forbidden:
            if pattern in content:
                violations.append(f"{py_file}: contains '{pattern}'")

    assert not violations, "\n".join(violations)

def test_no_empty_except_blocks() -> None:
    """Detect empty except blocks that swallow errors."""
    src_path = pathlib.Path("src")
    violations = []

    for py_file in src_path.rglob("*.py"):
        tree = ast.parse(py_file.read_text())
        for node in ast.walk(tree):
            if isinstance(node, ast.ExceptHandler):
                if len(node.body) == 1 and isinstance(node.body[0], ast.Pass):
                    violations.append(
                        f"{py_file}:{node.lineno}: empty except block"
                    )

    assert not violations, "\n".join(violations)
```
