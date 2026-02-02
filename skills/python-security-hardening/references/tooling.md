# Python Security Tooling Configuration

Complete configuration for security-hardened Python projects.

## pyproject.toml

```toml
[project]
name = "your-project"
version = "0.1.0"
requires-python = ">=3.11"

[project.optional-dependencies]
dev = [
    "ruff>=0.4.0",
    "mypy>=1.10.0",
    "pytest>=8.0.0",
    "pytest-cov>=5.0.0",
    "bandit>=1.7.0",
    "mutmut>=2.4.0",
    "hypothesis>=6.100.0",
    "pip-audit>=2.7.0",
    "pip-licenses>=4.4.0",
]

[tool.ruff]
target-version = "py311"
line-length = 88

[tool.ruff.lint]
select = [
    "E",      # pycodestyle errors
    "W",      # pycodestyle warnings
    "F",      # pyflakes
    "I",      # isort
    "B",      # flake8-bugbear
    "C4",     # flake8-comprehensions
    "UP",     # pyupgrade
    "S",      # flake8-bandit (security)
    "A",      # flake8-builtins
    "C90",    # mccabe complexity
    "N",      # pep8-naming
    "RUF",    # ruff-specific
    "PTH",    # flake8-use-pathlib
    "PL",     # pylint
    "PERF",   # perflint
]

[tool.ruff.lint.mccabe]
max-complexity = 10

[tool.ruff.lint.pylint]
max-args = 5
max-branches = 12
max-returns = 6
max-statements = 50

[tool.ruff.lint.per-file-ignores]
"tests/**/*.py" = ["S101"]  # Allow assert in tests

[tool.mypy]
python_version = "3.11"
strict = true
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
disallow_incomplete_defs = true
check_untyped_defs = true
disallow_untyped_decorators = true
no_implicit_optional = true
warn_redundant_casts = true
warn_unused_ignores = true
warn_no_return = true
follow_imports = "normal"
show_error_codes = true

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = [
    "--cov=src",
    "--cov-report=term-missing",
    "--cov-report=xml",
    "--cov-fail-under=100",
    "--strict-markers",
    "-v",
]
filterwarnings = ["error"]

[tool.coverage.run]
branch = true
source = ["src"]
omit = ["tests/*"]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "if TYPE_CHECKING:",
    "if __name__ == .__main__.:",
]
fail_under = 100
show_missing = true

[tool.bandit]
exclude_dirs = ["tests", "venv", ".venv"]
skips = []  # Do not skip any checks

[tool.mutmut]
paths_to_mutate = "src/"
tests_dir = "tests/"
runner = "pytest -x --tb=no -q"
```

---

## Ruff Security Rules (S - flake8-bandit)

Key security rules enabled:

| Rule | Description |
|------|-------------|
| S101 | Assert used (disabled in tests) |
| S102 | exec() used |
| S103 | Bad file permissions |
| S104 | Binding to all interfaces |
| S105 | Hardcoded password |
| S106 | Hardcoded password in function arg |
| S107 | Hardcoded password default |
| S108 | Insecure temp file |
| S110 | try-except-pass |
| S112 | try-except-continue |
| S113 | Request without timeout |
| S301 | pickle usage |
| S302 | marshal usage |
| S303 | Insecure hash function (MD5) |
| S304 | Insecure cipher |
| S305 | Insecure cipher mode |
| S306 | mktemp usage |
| S307 | eval() usage |
| S308 | mark_safe usage |
| S310 | URL open audit |
| S311 | random for security |
| S312 | telnet usage |
| S313 | xml.etree.ElementTree |
| S314 | xml.etree.cElementTree |
| S320 | lxml usage |
| S324 | Insecure hash (SHA1) |
| S501 | Request with no cert validation |
| S502 | SSL with bad version |
| S503 | SSL with bad defaults |
| S504 | SSL with no version |
| S505 | Weak cryptographic key |
| S506 | Unsafe YAML load |
| S507 | SSH no host key verification |
| S508 | Snmp insecure version |
| S509 | Snmp weak crypto |
| S601 | Paramiko shell |
| S602 | subprocess popen shell |
| S603 | subprocess without shell |
| S604 | Function call with shell=True |
| S605 | Start process with shell |
| S606 | Start process with no shell |
| S607 | Start process with partial path |
| S608 | SQL injection |
| S609 | Wildcard injection |
| S610 | Django extra used |
| S611 | Django rawsql used |
| S612 | Logging config listen |
| S701 | Jinja2 autoescape false |
| S702 | Mako templates |

---

## Bandit Configuration

Create `bandit.yaml` for additional configuration:

```yaml
# bandit.yaml
skips: []  # Never skip security checks

exclude_dirs:
  - tests
  - venv
  - .venv
  - .git

# Severity and confidence thresholds
severity:
  - low
  - medium
  - high

confidence:
  - low
  - medium
  - high
```

Run bandit:

```bash
bandit -r src/ -c bandit.yaml -f json -o bandit-report.json
```

---

## Mutation Testing with mutmut

Run mutation testing:

```bash
# Run all mutations
mutmut run

# Show results
mutmut results

# Show surviving mutants (tests that didn't catch the mutation)
mutmut show <id>
```

**Target: 80%+ mutation score**

Surviving mutants indicate tests that pass even when code is broken. Fix by:
1. Adding assertions for the mutated behavior
2. Adding edge case tests
3. Using hypothesis for property-based coverage

---

## Property-Based Testing with Hypothesis

Example patterns:

```python
from hypothesis import given, strategies as st, settings, assume

@given(st.text())
def test_input_sanitization_never_contains_script_tags(user_input: str) -> None:
    """Property: sanitized output never contains script tags."""
    result = sanitize_input(user_input)
    assert "<script" not in result.lower()
    assert "javascript:" not in result.lower()

@given(st.binary())
def test_encryption_roundtrip(plaintext: bytes) -> None:
    """Property: decrypt(encrypt(x)) == x for all x."""
    key = generate_key()
    ciphertext = encrypt(plaintext, key)
    decrypted = decrypt(ciphertext, key)
    assert decrypted == plaintext

@given(st.emails())
def test_email_validation_accepts_valid_emails(email: str) -> None:
    """Property: valid emails are accepted."""
    assert validate_email(email) is True

@given(st.text())
def test_sql_injection_impossible(user_input: str) -> None:
    """Property: queries are parameterized, injection impossible."""
    query = build_query(user_input)
    # Query should use parameterized format, not string interpolation
    assert user_input not in query or "?" in query or "%s" in query
```

---

## Running All Checks

```bash
# Lint and format
ruff check . --fix
ruff format .

# Type check
mypy src/

# Security scan
bandit -r src/

# Run tests with coverage
pytest --cov-fail-under=100

# Dependency audit
pip-audit

# License check
pip-licenses --fail-on="GPL;AGPL"

# Mutation testing
mutmut run
```

---

## VS Code Settings

Add to `.vscode/settings.json`:

```json
{
  "[python]": {
    "editor.formatOnSave": true,
    "editor.defaultFormatter": "charliermarsh.ruff",
    "editor.codeActionsOnSave": {
      "source.fixAll.ruff": "explicit",
      "source.organizeImports.ruff": "explicit"
    }
  },
  "python.analysis.typeCheckingMode": "strict",
  "mypy.runUsingActiveInterpreter": true
}
```
