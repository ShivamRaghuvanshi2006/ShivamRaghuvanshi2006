---
description: |
  This workflow creates concise daily repo status reports from recent issues,
  pull requests, commits, and releases.

on:
  schedule: daily
  workflow_dispatch:

permissions:
  contents: read
  issues: read
  pull-requests: read

network:
  allowed:
    - defaults
    - api.groq.com

tools:
  bash: ["cat", "ls", "find", "grep", "head", "tail", "wc"]
  github:
    allowed:
      - name: list_issues
        max-calls: 1
      - name: list_pull_requests
        max-calls: 1
      - name: list_commits
        max-calls: 1
      - name: list_releases
        max-calls: 1
      - name: issue_read
        max-calls: 1
      - name: pull_request_read
        max-calls: 1
    allowed-repos: ${{ github.repository }}
    min-integrity: none

safe-outputs:
  mentions: false
  allowed-github-references: []
  create-issue:
    title-prefix: "[repo-status] "
    labels: [report, daily-status]
    close-older-issues: true

engine:
  id: copilot
  env:
    COPILOT_PROVIDER_BASE_URL: https://api.groq.com/openai/v1
    COPILOT_MODEL: openai/gpt-oss-20b
    COPILOT_PROVIDER_MODEL_ID: openai/gpt-oss-20b
    COPILOT_PROVIDER_API_KEY: ${{ secrets.GEMINI_API_KEY }}
    COPILOT_PROVIDER_TYPE: openai

source: githubnext/agentics/workflows/repo-status.md@139fa09ef17cde4135e696ff8e7af09bc5ddfa6e
---

# Repo Status

Create a concise daily status report for this repository as a GitHub issue.

## What to include

- Activity from the last 7 days: issues, pull requests, commits, and releases.
- Meaningful progress, blockers, and notable changes.
- Two or three practical next steps for maintainers.

## Scope and context limits

- Use only the six explicitly available GitHub tools.
- Review one page/list per activity type; do not paginate or repeatedly fetch data.
- Read an individual issue or pull request only when its summary is necessary to understand a significant event.
- Do not scan the whole repository or fetch large files.
- If an activity type has no recent entries, say so briefly rather than searching further.
- Keep the final report compact and factual. Do not invent activity, progress, or recommendations unsupported by repository evidence.

## Process

1. Gather recent activity using the available list tools.
2. Inspect at most one important issue and one important pull request if needed.
3. Summarize verified activity, blockers, and next steps.
4. Create one GitHub issue with the report.
