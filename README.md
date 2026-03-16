# RAIO Infra DevSecOps Workflows

This repository contains **centrally managed, reusable GitHub Actions workflows** for automated security scanning—covering both SAST (code/static analysis) and SCA (dependency vulnerability)—using [Gitleaks](https://github.com/gitleaks/gitleaks), [Semgrep](https://semgrep.dev/), and [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/). These workflows are designed for use across all RAIO-TECHNOLOGIES organization repositories, ensuring consistent and easy security enforcement with minimal setup.

## Contents

- `.github/workflows/sast-gitleaks.yml` — Reusable workflow for secret scanning with Gitleaks
- `.github/workflows/sast-semgrep.yml` — Reusable workflow for SAST/static code analysis with Semgrep
- `.github/workflows/sast-dependencycheck.yml` — Reusable workflow for SCA/dependency analysis with OWASP Dependency-Check
- `.github/workflows/sast.yml` — Central orchestrator to run all the above security scans together

## How to Use in Other Repositories

To apply these security scans in any repository of your organization, simply create a workflow file (for example: `.github/workflows/devsecops.yml`) containing:

```
name: devsecops

on:
  push:
  pull_request:
  workflow_dispatch:

jobs:
  sast:
    uses: RAIO-TECHNOLOGIES/raio-infra-devsecops/.github/workflows/sast.yml@main
```

- No need to copy/paste jobs – always use the centralized logic via the `uses:` field.
- For branch protection, add the `sast` workflow as a required status check in repository settings.

## How It Works

- The central orchestrator (`sast.yml`) calls each specialized workflow (`sast-gitleaks.yml', `sast-semgrep.yml`, `sast-dependencycheck.yml`) using `workflow_call`.
- **Each workflow runs independently** and will fail the pipeline if high-risk issues or vulnerabilities are found.
- **No tokens or extra secrets required:** All tools run open-source mode and require no licenses for public repositories.

### New: Dependency-Check (SCA)

`.github/workflows/sast-dependencycheck.yml`:

```
name: sast-dependency-check

on:
  workflow_call:

jobs:
  dependency-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run OWASP Dependency-Check
        uses: dependency-check/Dependency-Check_Action@main
        with:
          project: "repo-sast"
          format: "HTML"
          args: "--failOnCVSS 6.5"
```
- **Automatically scans all supported dependency manifests** (`package.json`, `.csproj`, etc.) and fails the job if vulnerabilities with CVSS > 6.5 are detected.
- Results are visible directly in the GitHub Actions log.

## Updating Central Workflows

Any update pushed to these workflows becomes immediately available to all consuming repositories using the current branch (e.g., `main`).  
For reproducible workflows, you can pin to a specific commit SHA instead of `@main`.

## References

- [GitHub Docs: Reusing workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [Gitleaks](https://github.com/gitleaks/gitleaks)
- [Semgrep](https://semgrep.dev/)
- [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/)

---

**Tip:**  
With this approach, your entire organization maintains strong security standards—centrally managed, easy to update, and instantly enforced organization-wide!
```
