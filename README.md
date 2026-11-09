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

### `ai-code-review.yml`

Reviews a pull request with any **OpenAI-compatible** Chat Completions API (`POST {base-url}/chat/completions`).

- Findings are posted as inline review comments on the changed lines, each with a severity: critical, warning or suggestion.
- Findings that cannot be placed on a changed line go into one summary comment, which is updated in place on every run.
- The model receives the diff with head-side line numbers, the full content of small changed files, the PR title and description, and the repository conventions from `context-files` read from the base branch.
- Large PRs are split into several requests by file. Files beyond `max-requests` are listed in the summary as not reviewed.
- Lock, vendor, build, minified, snapshot and generated files are excluded. Add more with `exclude-paths`.
- On a new push, only the commits since the last review are reviewed. A force push or a rerun triggers a full review.
- Draft PRs and PRs opened by bots are skipped. A newer push cancels a running review.
- PR-authored text is marked as untrusted in the prompt, and model output is posted with `@mentions` and HTML comments neutralised.
- A missing `AI_API_KEY` (for example on a fork PR) or a failed API request produces a warning, not a failed check. Requests are retried on network errors, HTTP 429 and HTTP 5xx.
- `fail-on-severity` fails the check when a finding reaches that severity. It is off by default.

**Inputs**

| Input                     | Required | Default                     | Description                                                                                   |
| ------------------------- | -------- | --------------------------- | --------------------------------------------------------------------------------------------- |
| `base-url`                | No       | `https://api.openai.com/v1` | OpenAI-compatible API base URL (no trailing `/chat/completions`)                              |
| `model`                   | No       | `expen`                     | Model id                                                                                      |
| `system-prompt`           | No       | Built-in reviewer prompt    | Reviewer role and rules. The output format and untrusted-input rules are always appended     |
| `max-diff-bytes`          | No       | `60000`                     | Byte budget for one request: diff plus file context                                           |
| `max-requests`            | No       | `6`                         | Maximum model requests per run                                                                |
| `max-file-bytes`          | No       | `24000`                     | Largest changed file whose full content is sent as context                                    |
| `max-findings`            | No       | `20`                        | Maximum findings per run                                                                      |
| `temperature`             | No       | `0.2`                       | Sampling temperature                                                                          |
| `comment-tag`             | No       | `ai-code-review`            | Hidden marker used to find and update the summary comment                                    |
| `context-files`           | No       | `CLAUDE.md,AGENTS.md`       | Comma-separated files sent as repository conventions, read from the base branch               |
| `exclude-paths`           | No       | —                           | Extra git globs to exclude, comma- or newline-separated                                       |
| `review-drafts`           | No       | `false`                     | Review draft PRs                                                                              |
| `review-bots`             | No       | `false`                     | Review PRs opened by bots                                                                     |
| `skip-docs-only`          | No       | `false`                     | Skip PRs that change only Markdown, text or `docs/` files                                     |
| `incremental`             | No       | `true`                      | On a new push, review only the commits since the last review                                  |
| `inline-comments`         | No       | `true`                      | Post findings as inline comments; when `false`, all findings go into the summary             |
| `fail-on-severity`        | No       | `none`                      | `none`, `critical` or `warning`                                                               |
| `request-timeout-seconds` | No       | `240`                       | Timeout for one model request                                                                 |
| `retries`                 | No       | `2`                         | Retries after a network error, HTTP 429 or HTTP 5xx                                           |

**Secrets**

| Secret        | Required | Description                                                                 |
| ------------- | -------- | --------------------------------------------------------------------------- |
| `AI_API_KEY`  | No       | Bearer token for the endpoint. When missing, the review is skipped          |
| `AI_BASE_URL` | No       | Base URL; overrides the `base-url` input. Handy as an org-wide secret       |

**Permissions:** the calling workflow must grant `contents: read` and `pull-requests: write`.

**Usage**

Pin a release tag. `@main` changes under every caller as soon as this repository changes.

Inside `JianyueLab-Org`, with `AI_API_KEY` and `AI_BASE_URL` as org secrets:

```yaml
# .github/workflows/ai-code-review.yml
name: AI Code Review

on:
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review]

permissions:
  contents: read
  pull-requests: write

jobs:
  review:
    uses: JianyueLab-Org/actions/.github/workflows/ai-code-review.yml@v27.1.0
    secrets: inherit
```

From another organization, forward the secrets explicitly:

```yaml
jobs:
  review:
    uses: JianyueLab-Org/actions/.github/workflows/ai-code-review.yml@v27.1.0
    with:
      fail-on-severity: critical
    secrets:
      AI_API_KEY: ${{ secrets.AI_API_KEY }}
      AI_BASE_URL: ${{ secrets.AI_BASE_URL }}
```

## Setup

For each project that uses `deploy-dokploy.yml`, add the following secrets in **GitHub repo Settings → Secrets and variables → Actions**:

- `DOKPLOY_API_KEY` — shared API key for Dokploy
- `APPLICATION_ID` — the Dokploy application ID specific to this project

For `ai-code-review.yml`, add one secret:

- `AI_API_KEY` — Bearer token for your OpenAI-compatible endpoint
