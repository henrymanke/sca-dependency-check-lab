# 📦 OWASP Dependency-Check CI/CD Pipeline

This guide outlines a best-practice CI/CD setup using GitHub Actions to automate Software Composition Analysis (SCA) with OWASP Dependency-Check.

---

## 🎯 Objective

* Automatically scan dependencies for known vulnerabilities (CVEs)
* Generate both machine-readable (JSON) and human-readable (HTML) reports
* Upload results as GitHub Actions artifacts for review

---

## 🔐 NVD API Key

> 🔄 Improves performance and reliability of CVE database syncs

1. Request your free key: [https://nvd.nist.gov/developers/request-an-api-key](https://nvd.nist.gov/developers/request-an-api-key)
2. Add it to your GitHub repository:

   * Go to `Settings > Secrets and variables > Actions`
   * Add secret: `NVD_API_KEY`

---

## 🛠️ GitHub Actions Workflow Setup

Create a workflow file at:

```text
.github/workflows/dependency-check.yml
```

### ✅ Example Workflow

```yaml
name: OWASP Dependency-Check (SCA)

# 📋 Description:
# This workflow performs Software Composition Analysis (SCA) using OWASP Dependency-Check.
# It scans the /src directory for vulnerable dependencies and uploads the results as artifacts.

on:
  push:
    branches: [main]  # Trigger scan on push to the main branch
  pull_request:       # Also trigger on pull requests to ensure new code is scanned

jobs:
  sca-scan:
    name: Run Dependency Check
    runs-on: ubuntu-latest
    env:
      # 🔐 NVD API Key is required for faster CVE syncing
      NVD_API_KEY: ${{ secrets.NVD_API_KEY }}

    steps:
      # 🛎️ Checkout the code from the repository
      - name: Checkout repository
        uses: actions/checkout@v3

      # 🐳 Set up Docker Buildx (recommended for future multi-arch support)
      - name: Set up Docker
        uses: docker/setup-buildx-action@v2

      # 🔎 Run the Dependency-Check scan
      - name: Run Dependency-Check scan
        run: |
          docker run --rm \
            -v "$(pwd)/src:/src" \                         # Source code folder
            -v "$(pwd)/reports:/report" \                  # Report output folder
            -e NVD_API_KEY=${{ env.NVD_API_KEY }} \        # Pass API key into container
            owasp/dependency-check \
              --scan /src \                                # Path to scan
              --format ALL \                               # Generate all supported report formats
              --out /report \                              # Save reports to mounted volume
              --project "${{ github.repository }}" \       # Use repo name as project identifier
              --nvdApiKey "${NVD_API_KEY}"                 # Always use the API key

      # 📤 Upload the HTML report for human review
      - name: Upload HTML Report
        uses: actions/upload-artifact@v4
        with:
          name: dependency-check-html
          path: reports/dependency-check-report.html

```

---

## 📁 Output

After each CI run, the following reports will be available:

* `reports/dependency-check-report.html` — user-friendly report
* `reports/dependency-check-report.json` — structured report for automation
* View the report under **GitHub Actions > Artifacts**

---

## 🧠 Pro Tips

* ✅ Use this as a required check for pull requests to enforce SCA compliance
* 🕑 Schedule nightly or weekly scans on `main` to catch newly disclosed CVEs
* 🔁 Combine with [Dependabot](https://docs.github.com/en/code-security/supply-chain-security/keeping-your-dependencies-updated-automatically) or Renovate for full dependency hygiene

---

## 📚 Resources

* [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/)
* [GitHub Actions Documentation](https://docs.github.com/en/actions)
* [NVD API Key Request](https://nvd.nist.gov/developers/request-an-api-key)
