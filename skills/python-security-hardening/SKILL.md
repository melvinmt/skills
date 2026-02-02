---
name: python-security-hardening
description: Harden Python projects with security-first development practices. Enforces 100% test coverage, static analysis, secret scanning, mutation testing, pre-commit hooks, and SOC 2 compliance. Use when setting up security tooling, writing security tests, or hardening a Python codebase.
---

# Python Security Hardening

Enforce security-first development with comprehensive tooling and testing practices.

## Quick Start

1. Install the security tooling stack
2. Configure pre-commit hooks
3. Set up GitHub Actions CI
4. Write security tests from an attacker's perspective

---

## Security-First Testing Philosophy

Write tests like a white hat hacker:

1. **Test security premise first** - Define what security guarantees the code must provide
2. **Implementation adapts to tests** - Never weaken tests to match broken implementation
3. **No exceptions** - No skips, no xfails, no TODOs, no documented vulnerabilities

### Forbidden Patterns

These patterns are **never allowed** in tests:

```python
# FORBIDDEN - All of these must fail CI
@pytest.mark.skip
@pytest.mark.xfail
# TODO: fix this later
except:  # Empty except blocks
    pass
```

### Required Test Categories

Every security-sensitive codebase must test:

| Category | What to Test |
|----------|--------------|
| Input Validation | SQL injection, command injection, path traversal |
| Authentication | Bypass attempts, session fixation, brute force |
| Authorization | Privilege escalation, IDOR |
| Data Protection | Encryption, PII exposure |
| Error Handling | Information disclosure, stack traces |

---

## Python Tooling Stack

| Tool | Purpose | Install |
|------|---------|---------|
| ruff | Lint, format, complexity | `pip install ruff` |
| mypy | Static type checking | `pip install mypy` |
| pytest-cov | Coverage with 100% enforcement | `pip install pytest-cov` |
| bandit | Security-focused SAST | `pip install bandit` |
| mutmut | Mutation testing | `pip install mutmut` |
| hypothesis | Property-based testing | `pip install hypothesis` |
| gitleaks | Secret scanning | `brew install gitleaks` |
| pip-audit | Dependency vulnerabilities | `pip install pip-audit` |
| pip-licenses | License compliance | `pip install pip-licenses` |

For complete configuration, see [references/tooling.md](references/tooling.md).

---

## Dangerous Function Blocklist

These functions are blocked by ruff/bandit rules:

| Function | Risk | Alternative |
|----------|------|-------------|
| `eval()`, `exec()` | Code injection | Parse safely or redesign |
| `subprocess(shell=True)` | Command injection | Use `subprocess.run()` with list args |
| `pickle.loads()` | Arbitrary code execution | Use JSON or validated formats |
| `random` | Predictable for security | Use `secrets` module |
| `hashlib.md5/sha1` | Weak for security | Use `hashlib.sha256` or `sha3_256` |
| `yaml.load()` | Code execution | Use `yaml.safe_load()` |
| `os.system()` | Command injection | Use `subprocess.run()` |

---

## Implementation Checklist

```
Security Tooling Setup:
- [ ] Install all tools (ruff, mypy, pytest-cov, bandit, mutmut, hypothesis)
- [ ] Configure pyproject.toml with strict settings
- [ ] Set up pre-commit hooks with gitleaks
- [ ] Configure GitHub Actions CI workflow
- [ ] Add Dependabot for dependency updates
- [ ] Create SECURITY.md with disclosure policy
- [ ] Add CODEOWNERS for security review

Test Coverage:
- [ ] 100% line coverage enforced
- [ ] Mutation testing score >= 80%
- [ ] Property-based tests for all input handling
- [ ] Security test categories covered
- [ ] No forbidden patterns in tests
```

---

## Additional Resources

- [Tooling Configuration](references/tooling.md) - Complete pyproject.toml and tool setup
- [Pre-commit Hooks](references/precommit-hooks.md) - Hook configuration with gitleaks
- [GitHub Actions](references/github-actions.md) - CI workflow and Dependabot setup
- [SOC 2 Checklist](references/soc2-checklist.md) - Compliance requirements and SECURITY.md
- [Security Testing](references/security-testing.md) - White-hat methodology and examples
