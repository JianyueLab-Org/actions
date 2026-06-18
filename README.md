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

Reviews a pull request diff with any **OpenAI-compatible** Chat Completions API and posts the result as a PR comment (updating its previous comment on each push). Works with OpenAI, Azure OpenAI, OpenRouter, DeepSeek, Groq, Together, or a self-hosted vLLM/Ollama gateway — anything that exposes `POST {base-url}/chat/completions`.

**Inputs**

| Input            | Required | Default                     | Description                                                  |
| ---------------- | -------- | --------------------------- | ------------------------------------------------------------ |
| `base-url`       | No       | `https://api.openai.com/v1` | OpenAI-compatible API base URL (no trailing `/chat/completions`) |
| `model`          | No       | `gpt-5.5`                   | Model id to use                                              |
| `system-prompt`  | No       | Built-in reviewer prompt    | Overrides the reviewer's role and rules                     |
| `max-diff-bytes` | No       | `60000`                     | Max diff size sent to the model; larger diffs are truncated |
| `temperature`    | No       | `0.2`                       | Sampling temperature                                        |
| `comment-tag`    | No       | `ai-code-review`            | Hidden marker used to update the bot's previous comment     |

**Secrets**

| Secret       | Required | Description                                        |
| ------------ | -------- | -------------------------------------------------- |
| `AI_API_KEY` | Yes      | Bearer token for the OpenAI-compatible endpoint    |

**Usage**

```yaml
# .github/workflows/review.yml
name: AI Review

on:
  pull_request:

jobs:
  review:
    uses: JianyueLab-Org/actions/.github/workflows/ai-code-review.yml@main
    with:
      base-url: https://api.deepseek.com/v1
      model: deepseek-chat
    secrets:
      AI_API_KEY: ${{ secrets.AI_API_KEY }}
```

## Setup

For each project that uses `deploy-dokploy.yml`, add the following secrets in **GitHub repo Settings → Secrets and variables → Actions**:

- `DOKPLOY_API_KEY` — shared API key for Dokploy
- `APPLICATION_ID` — the Dokploy application ID specific to this project

For `ai-code-review.yml`, add one secret:

- `AI_API_KEY` — Bearer token for your OpenAI-compatible endpoint
