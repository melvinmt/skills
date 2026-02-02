# Pre-commit Hooks Configuration

Enforce security checks before every commit using the `pre-commit` framework.

## Installation

```bash
pip install pre-commit
pre-commit install
```

---

## .pre-commit-config.yaml

```yaml
# .pre-commit-config.yaml
repos:
  # Secret scanning - MUST BE FIRST (fail fast on secrets)
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.4
    hooks:
      - id: gitleaks

  # Basic file checks
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
        args: [--unsafe]  # Allow custom tags
      - id: check-json
      - id: check-toml
      - id: check-added-large-files
        args: [--maxkb=1000]
      - id: check-merge-conflict
      - id: detect-private-key
      - id: no-commit-to-branch
        args: [--branch, main, --branch, master]

  # Ruff - linting and formatting
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.4.10
    hooks:
      - id: ruff
        args: [--fix, --exit-non-zero-on-fix]
      - id: ruff-format

  # Type checking
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.10.0
    hooks:
      - id: mypy
        additional_dependencies: []  # Add your type stubs here
        args: [--strict]

  # Security scanning
  - repo: https://github.com/PyCQA/bandit
    rev: 1.7.9
    hooks:
      - id: bandit
        args: [-r, src/, -c, pyproject.toml]
        exclude: tests/

  # Dependency audit
  - repo: https://github.com/pypa/pip-audit
    rev: v2.7.3
    hooks:
      - id: pip-audit
        args: [--strict, --require-hashes, --disable-pip]

# Hook order matters - fail fast on most critical issues
default_stages: [commit]

# CI configuration
ci:
  autofix_commit_msg: "style: auto-fix from pre-commit hooks"
  autofix_prs: true
  autoupdate_commit_msg: "chore: update pre-commit hook versions"
```

---

## Gitleaks Configuration

Create `.gitleaks.toml` for custom rules:

```toml
# .gitleaks.toml
title = "Custom Gitleaks Configuration"

[extend]
useDefault = true

# Add custom patterns for your specific secrets
[[rules]]
id = "custom-api-key"
description = "Custom API Key Pattern"
regex = '''(?i)(api[_-]?key|apikey)\s*[=:]\s*['"]?([a-zA-Z0-9]{32,})['"]?'''
secretGroup = 2
entropy = 3.5

# Allowlist paths that may contain example/test secrets
[allowlist]
paths = [
    '''tests/fixtures/.*''',
    '''docs/examples/.*''',
]
regexes = [
    '''EXAMPLE_.*''',
    '''test_.*''',
]
```

---

## Running Hooks Manually

```bash
# Run all hooks on all files
pre-commit run --all-files

# Run specific hook
pre-commit run gitleaks --all-files
pre-commit run bandit --all-files
pre-commit run mypy --all-files

# Update hook versions
pre-commit autoupdate

# Skip hooks (EMERGENCY ONLY - never in normal workflow)
# git commit --no-verify  # DO NOT USE
```

---

## Hook Ordering Strategy

Hooks run in order defined. Optimal order for security:

1. **gitleaks** - Block secrets immediately (most critical)
2. **pre-commit-hooks** - Basic file validation
3. **ruff** - Lint and format (auto-fixable)
4. **mypy** - Type checking
5. **bandit** - Security scanning
6. **pip-audit** - Dependency vulnerabilities

This ensures:
- Secrets never reach later stages
- Fast fixes (formatting) happen early
- Slower checks (type checking) run last

---

## Adding Tests to Pre-commit

For full enforcement, add pytest to pre-commit:

```yaml
  # Tests with 100% coverage (add after bandit)
  - repo: local
    hooks:
      - id: pytest-cov
        name: pytest with coverage
        entry: pytest --cov-fail-under=100
        language: system
        pass_filenames: false
        always_run: true
        stages: [commit]
```

**Note:** Running full test suite on every commit may be slow. Consider:
- Running on `pre-push` instead of `pre-commit`
- Using `pytest --last-failed` for faster incremental checks
- Relying on CI for full coverage enforcement

---

## Troubleshooting

### Hook fails with "not found"

```bash
# Ensure virtual environment is activated
source .venv/bin/activate

# Reinstall hooks
pre-commit clean
pre-commit install
```

### Gitleaks false positives

Add to `.gitleaks.toml`:

```toml
[allowlist]
commits = ["<commit-sha>"]  # Allowlist specific commits
paths = ["path/to/file"]     # Allowlist specific paths
regexes = ["EXAMPLE_.*"]     # Allowlist patterns
```

### Slow hooks

```bash
# Run hooks in parallel
pre-commit run --all-files -j 4
```
