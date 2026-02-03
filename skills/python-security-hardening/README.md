# Python Security Hardening Skill

Harden Python projects with security-first development practices. This skill enforces 100% test coverage, static analysis, secret scanning, mutation testing, pre-commit hooks, and SOC 2 compliance.

## What's Included

- **Tooling Stack**: ruff, mypy, pytest-cov, bandit, mutmut, hypothesis, gitleaks, pip-audit
- **Pre-commit Hooks**: Secret scanning, linting, type checking, security scans
- **GitHub Actions**: Complete CI workflow with all security gates
- **SOC 2 Compliance**: Checklist and SECURITY.md template
- **Security Testing**: White-hat methodology with OWASP Top 10 patterns

## Installation

```bash
npx skills add melvinmt/skills --skill python-security-hardening
```

<details>
<summary>bun</summary>

```bash
bunx skills add melvinmt/skills --skill python-security-hardening
```

</details>

<details>
<summary>pnpm</summary>

```bash
pnpm dlx skills add melvinmt/skills --skill python-security-hardening
```

</details>

## Key Features

| Feature | Enforcement |
|---------|-------------|
| Test Coverage | 100% required |
| Mutation Score | 80%+ threshold |
| Secret Scanning | Blocks commits with secrets |
| Dangerous Functions | Blocked by linter (eval, exec, shell=True) |
| Cryptographic Standards | No MD5/SHA1, use secrets module |

## License

MIT
