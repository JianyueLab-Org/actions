# actions

Reusable GitHub Actions workflows for JianyueLab projects.

## Workflows

### `web-check.yml`

Runs format, typecheck, and build checks for Bun-based web projects.

**Inputs**

| Input         | Required | Default | Description        |
| ------------- | -------- | ------- | ------------------ |
| `bun-version` | No       | `1.3`   | Bun version to use |

**Usage**

```yaml
# .github/workflows/ci.yml
name: CI

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  web-check:
    uses: JianyueLab-Org/actions/.github/workflows/web-check.yml@main
```

---

### `deploy-dokploy.yml`

Deploys to [Dokploy](https://dokploy.com) and updates GitHub deployment status.

**Inputs**

| Input             | Required | Default               | Description                                 |
| ----------------- | -------- | --------------------- | ------------------------------------------- |
| `environment_url` | Yes      | —                     | Public URL of the deployed environment      |
| `sha`             | No       | Triggering commit SHA | Commit SHA to associate with the deployment |

**Secrets**

| Secret            | Required | Description                             |
| ----------------- | -------- | --------------------------------------- |
| `DOKPLOY_API_KEY` | Yes      | Dokploy API key                         |
| `APPLICATION_ID`  | Yes      | Dokploy application ID for this project |

**Usage**

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  workflow_run:
    workflows: ["CI"]
    types:
      - completed
    branches:
      - main

jobs:
  deploy:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    uses: JianyueLab-Org/actions/.github/workflows/deploy-dokploy.yml@main
    with:
      environment_url: "https://myapp.example.com"
      sha: ${{ github.event.workflow_run.head_sha }}
    secrets:
      DOKPLOY_API_KEY: ${{ secrets.DOKPLOY_API_KEY }}
      APPLICATION_ID: ${{ secrets.APPLICATION_ID }}
```

## Setup

For each project that uses `deploy-dokploy.yml`, add the following secrets in **GitHub repo Settings → Secrets and variables → Actions**:

- `DOKPLOY_API_KEY` — shared API key for Dokploy
- `APPLICATION_ID` — the Dokploy application ID specific to this project
