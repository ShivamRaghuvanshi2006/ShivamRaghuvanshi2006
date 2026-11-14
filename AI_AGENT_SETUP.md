# AI GitHub Agent Setup

This repository contains a lightweight GitHub automation setup powered by Groq.

## Workflows

| Workflow | What it does | Schedule |
|---|---|---|
| Daily Repo Status | Summarizes recent activity in this profile repository and opens/updates a status issue | Daily, 09:00 IST |
| AI Repository Audit | Reads up to 10 recently updated public, non-fork repositories and posts ranked, evidence-based improvements here | Monday, 09:30 IST |
| AI Coding Agent | Takes one issue from a selected repository, proposes a small change on a new branch, runs available checks, and opens a draft PR | Manual dispatch |

The audit is read-only. It does not execute repository code. The coding workflow never merges a pull request.

## One-time setup for cross-repository coding

The coding workflow needs a fine-grained GitHub Personal Access Token because the default `GITHUB_TOKEN` cannot write to other repositories.

1. On GitHub, open **Settings → Developer settings → Personal access tokens → Fine-grained tokens**.
2. Create a token with your account as the resource owner.
3. Select only the repositories the agent should be allowed to modify.
4. Grant these repository permissions:
   - **Contents: Read and write**
   - **Issues: Read-only**
   - **Pull requests: Read and write**
   - **Metadata: Read-only** (required)
5. Copy the token once. Do not paste it into chat or commit it to a repository.
6. In this profile repository, open **Settings → Secrets and variables → Actions → New repository secret**.
7. Name the secret `GH_AGENT_TOKEN` and paste the token value.

The existing `GEMINI_API_KEY` Actions secret is currently used to hold the Groq API key. Keep it private; the workflow only reads it from Actions secrets.

## Running the coding agent

1. Create a clear GitHub issue in one of the repositories selected in the token.
2. Open this repository's **Actions** tab.
3. Select **AI Coding Agent → Run workflow**.
4. Enter the target repository as `owner/repository` and the issue number.
5. Run it and inspect the resulting draft pull request and its CI checks.

## Safety limits

- The target must belong to `ShivamRaghuvanshi2006`.
- The agent may replace at most two existing source/documentation files.
- It cannot create/delete/rename files or edit GitHub workflows, secrets, dependency manifests, lockfiles, build/deployment configuration, permissions, or licensing.
- Files containing credential-like values are excluded from the code context sent to Groq.
- It runs whitespace checks, available Python/JavaScript syntax checks, and the Python unittest suite when applicable.
- It creates a separate branch and a **draft** pull request. It never merges.
- If the task is ambiguous, unsafe, too large, or the model cannot produce a valid bounded patch, the workflow stops without opening a PR.

## Current scope

The audit workflow reads public repositories only and examines the 10 most recently updated, non-fork repositories. It does not inspect private repositories, execute builds, or claim to be a complete security audit. The coding workflow can write only to repositories explicitly selected in the fine-grained token.
