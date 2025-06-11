# 📦 OWASP Dependency-Check CI/CD Pipeline

This guide outlines a best-practice CI/CD setup using GitHub Actions to automate Software Composition Analysis (SCA) with OWASP Dependency-Check.

---

## 🎯 Objective

- Automatically scan dependencies for known vulnerabilities (CVEs)
- Generate machine-readable (JSON) and human-readable (HTML) reports
- Upload results as GitHub Actions artifacts

---

## 🔐 NVD API Key

> ⚠️ Required for faster database syncs from NVD (National Vulnerability Database)

1. Request your key at: <https://nvd.nist.gov/developers/request-an-api-key>  
2. Add it to your GitHub repo secrets:
   - Go to `Settings > Secrets and variables > Actions`
   - Add new secret: `NVD_API_KEY`

---

## 🛠️ GitHub Actions Workflow

Create this file:

```text
.github/workflows/dependency-check.yml
```

### ✅ Example Workflow

```yaml
name: OWASP Dependency-Check (SCA)

on:
  push:
    branches: [main]
  pull_request:

jobs:
  sca-scan:
    runs-on: ubuntu-latest
    env:
      NVD_API_KEY: ${{ secrets.NVD_API_KEY }}

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Set up Docker
        uses: docker/setup-buildx-action@v2

      - name: Run Dependency-Check scan
        run: |
          docker run --rm \
            -v "$(pwd)/juice-shop:/src" \
            -v "$(pwd)/reports:/report" \
            -e NVD_API_KEY=${{ env.NVD_API_KEY }} \
            owasp/dependency-check \
              --scan /src \
              --format ALL \
              --out /report \
              --project juice-shop \
              --nvdApiKey $NVD_API_KEY

      - name: Upload HTML Report
        uses: actions/upload-artifact@v4
        with:
          name: dependency-check-html
          path: reports/dependency-check-report.html
```

---

## 📁 Output

- `reports/dependency-check-report.json` — machine readable
- `reports/dependency-check-report.html` — UI-friendly

Reports will be available under the GitHub Actions > Artifacts section after the run.

---

## 🧠 Pro Tips

- Use as a required check in PRs for vulnerability enforcement
- Run nightly scans on `main` to catch new CVEs proactively
- Monitor CVE changes using Dependabot or Renovate

---

## 📚 Resources

- [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/)
- [GitHub Actions](https://docs.github.com/en/actions)
- [NVD API Key](https://nvd.nist.gov/developers/request-an-api-key)
