# github-workflows

Reusable GitHub Actions workflows for use across projects.

## Workflows

### `slack-notify.yml` — CI/CD completion → Slack

Posts a Block Kit message to a Slack channel via an incoming webhook, summarizing
the status, repo, branch, commit, actor, and a link to the run.

#### Inputs

| Name             | Required | Default | Description |
|------------------|----------|---------|-------------|
| `status`         | no       | `''`    | `success`, `failure`, `cancelled`, or `skipped`. If empty, falls back to the notify job's own `job.status` (usually `success`). Pass explicitly for accurate reporting. |
| `job-name`       | no       | `CI/CD` | Friendly name of the pipeline/job shown in the message. |
| `environment`    | no       | `''`    | Optional environment label (e.g. `staging`, `production`). |
| `custom-message` | no       | `''`    | Optional extra line appended to the Slack message. |

#### Secrets

| Name                | Required | Description |
|---------------------|----------|-------------|
| `SLACK_WEBHOOK_URL` | yes      | Slack incoming webhook URL for the target channel. |

#### Usage

In the calling repo, e.g. `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: make build test

  notify:
    needs: build
    if: always()
    uses: <OWNER>/github-workflows/.github/workflows/slack-notify.yml@main
    with:
      status: ${{ needs.build.result }}
      job-name: Build & Test
      environment: production
    secrets:
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

Replace `<OWNER>` with the GitHub org/user that hosts this repo, and pin to a
tag (e.g. `@v1`) once you cut releases.

#### Notes on status detection

GitHub reusable workflows run in their own job context, so `job.status` inside
this workflow only reflects the notify job itself — not the caller's build job.
Always pass `status: ${{ needs.<job>.result }}` (or `${{ job.status }}` from
within a single-job calling workflow) for accurate reporting, and use
`if: always()` so the notification runs on failure too.
