# GitHub Actions Workflows — intech-newguys

Reusable workflows that replace Jenkins shared libraries.

## Structure

```
.github/
  workflows/
    backend-build.yml          # Replaces newguysBackendPipeline.groovy
    frontend-ssr-build.yml     # Replaces newguysBettingFrontendSSR.groovy
    frontend-static-build.yml  # Replaces newguysFrontend.groovy
    qa-cypress.yml             # Replaces newguysQApipelinek8s.groovy
  actions/
    slack-notify/              # Reusable Slack notification action
    jira-update/               # Reusable Jira ticket update action

examples/
    backend-service-build.yml      # Drop into any backend repo
    frontend-ssr-build.yml         # Drop into SSR frontend repos
    frontend-static-build.yml      # Drop into static frontend repos
    qa-cypress-tests.yml           # Drop into QA automation repo
```

## Jenkins → GitHub Actions Mapping

| Jenkins Pipeline | GitHub Workflow | Trigger |
|---|---|---|
| newguysBackendPipeline.groovy | backend-build.yml | push to dev/qa/stage/prod |
| newguysBettingFrontendSSR.groovy | frontend-ssr-build.yml | push to dev/qa/stage/prod |
| newguysFrontend.groovy | frontend-static-build.yml | push to dev/qa/stage/prod |
| newguysQApipelinek8s.groovy | qa-cypress.yml | push to stage + daily cron |

## Prerequisites

1. **AWS OIDC Provider** — configured in IAM for GitHub Actions
2. **GitHub Org Secrets:**
   - `AWS_ROLE_ARN` — IAM role for OIDC
   - `SLACK_TOKEN` — Slack API token
   - `SENTRY_AUTH_TOKEN` — Sentry (frontend only)
   - `CLOUDFLARE_TOKEN` — Cloudflare (SSR frontend only)
   - `CYPRESS_RECORD_KEY` — Cypress Cloud (QA only)
   - `JIRA_WEBHOOKS_JSON` — Jira webhook config (optional)
3. **Self-hosted runners** — ARC on dedicated CI cluster

## Usage

Each service repo needs a single `.github/workflows/build.yml` file.
See `examples/` for templates.

## AWS OIDC Setup (replaces static credentials)

Instead of storing AWS access keys as secrets, use OIDC federation:

```bash
# Create OIDC provider in AWS (one-time)
aws iam create-open-id-connect-provider \
  --url "https://token.actions.githubusercontent.com" \
  --thumbprint-list "6938fd4d98bab03faadb97b34396831e3780aea1" \
  --client-id-list "sts.amazonaws.com"

# Create IAM role with trust policy for the GitHub org
# Trust policy should allow: repo:intech-newguys/*:*
```
