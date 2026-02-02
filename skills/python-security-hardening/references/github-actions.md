# GitHub Actions CI Configuration

Complete CI workflow for security-hardened Python projects.

## .github/workflows/ci.yml

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read
  security-events: write

jobs:
  security-scan:
    name: Security Scanning
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for gitleaks

      - name: Gitleaks Secret Scan
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: python

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3

  lint-and-type-check:
    name: Lint & Type Check
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install ruff mypy bandit
          pip install -e ".[dev]"

      - name: Ruff lint
        run: ruff check . --output-format=github

      - name: Ruff format check
        run: ruff format --check .

      - name: Mypy type check
        run: mypy src/ --strict

      - name: Bandit security scan
        run: bandit -r src/ -f json -o bandit-report.json || true

      - name: Upload Bandit report
        uses: actions/upload-artifact@v4
        with:
          name: bandit-report
          path: bandit-report.json

  test:
    name: Test with Coverage
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -e ".[dev]"

      - name: Run tests with coverage
        run: pytest --cov-fail-under=100 --cov-report=xml

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v4
        with:
          file: ./coverage.xml
          fail_ci_if_error: true

  mutation-testing:
    name: Mutation Testing
    runs-on: ubuntu-latest
    needs: test  # Only run if tests pass
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -e ".[dev]"

      - name: Run mutation testing
        run: |
          mutmut run --no-progress
          mutmut results
          # Fail if mutation score is below 80%
          SURVIVED=$(mutmut results | grep -oP 'Survived: \K\d+' || echo "0")
          KILLED=$(mutmut results | grep -oP 'Killed: \K\d+' || echo "0")
          TOTAL=$((SURVIVED + KILLED))
          if [ "$TOTAL" -gt 0 ]; then
            SCORE=$((KILLED * 100 / TOTAL))
            echo "Mutation score: $SCORE%"
            if [ "$SCORE" -lt 80 ]; then
              echo "Mutation score below 80% threshold"
              exit 1
            fi
          fi

  dependency-audit:
    name: Dependency Audit
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install pip-audit
        run: pip install pip-audit pip-licenses

      - name: Audit dependencies
        run: pip-audit --strict

      - name: Check licenses
        run: |
          pip install -e ".[dev]"
          pip-licenses --fail-on="GPL-3.0;AGPL-3.0"

  all-checks-passed:
    name: All Checks Passed
    runs-on: ubuntu-latest
    needs:
      - security-scan
      - lint-and-type-check
      - test
      - mutation-testing
      - dependency-audit
    steps:
      - run: echo "All security and quality checks passed!"
```

---

## .github/dependabot.yml

```yaml
# .github/dependabot.yml
version: 2
updates:
  # Python dependencies
  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "daily"
    open-pull-requests-limit: 10
    groups:
      security:
        applies-to: security-updates
        patterns:
          - "*"
      development:
        applies-to: version-updates
        patterns:
          - "*"
    commit-message:
      prefix: "chore(deps)"

  # GitHub Actions
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
    commit-message:
      prefix: "chore(ci)"
```

---

## .github/CODEOWNERS

```
# .github/CODEOWNERS
# Security-sensitive files require security team review

# Default owners for everything
* @your-org/developers

# Security-critical paths
/src/auth/ @your-org/security
/src/crypto/ @your-org/security
/src/api/permissions/ @your-org/security
*.pem @your-org/security
*.key @your-org/security

# Configuration files
pyproject.toml @your-org/security
.pre-commit-config.yaml @your-org/security
.github/workflows/ @your-org/security

# Security documentation
SECURITY.md @your-org/security
```

---

## Branch Protection Rules

Configure in GitHub repository settings:

### Required for `main` branch:

- [x] Require a pull request before merging
  - [x] Require approvals: 1
  - [x] Dismiss stale pull request approvals when new commits are pushed
  - [x] Require review from Code Owners
- [x] Require status checks to pass before merging
  - [x] Require branches to be up to date before merging
  - Required checks:
    - `All Checks Passed`
    - `security-scan`
    - `lint-and-type-check`
    - `test`
    - `mutation-testing`
    - `dependency-audit`
- [x] Require signed commits
- [x] Do not allow bypassing the above settings
- [x] Restrict who can push to matching branches

---

## Required Status Checks

Ensure these jobs are marked as required in branch protection:

| Job Name | Purpose |
|----------|---------|
| `security-scan` | Gitleaks + CodeQL |
| `lint-and-type-check` | Ruff + Mypy + Bandit |
| `test` | Pytest with 100% coverage |
| `mutation-testing` | 80%+ mutation score |
| `dependency-audit` | pip-audit + license check |
| `all-checks-passed` | Gate job for all checks |

---

## Caching Dependencies

Add caching for faster CI:

```yaml
      - name: Cache pip dependencies
        uses: actions/cache@v4
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('**/pyproject.toml') }}
          restore-keys: |
            ${{ runner.os }}-pip-
```

---

## Secrets Required

| Secret | Purpose |
|--------|---------|
| `GITHUB_TOKEN` | Auto-provided, for gitleaks and CodeQL |
| `CODECOV_TOKEN` | Coverage reporting (optional, for private repos) |
