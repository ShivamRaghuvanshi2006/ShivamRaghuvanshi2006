# AI GitHub Agent Setup

This repository contains a cautious GitHub automation system powered by Groq.

## Workflows

| Workflow | Purpose | Schedule |
|---|---|---|
| Daily Repo Status | Summarises activity and health signals for this profile repository; updates one dated issue rather than creating duplicates on reruns | Daily, scheduled at 03:30 UTC (09:00 IST) |
| AI Repository Audit | Audits public, non-fork repositories owned by the account; reviews root structure, README excerpts, open issues, recent commits and recent Actions runs | Monday, scheduled at 04:00 UTC (09:30 IST) |
| AI Coding Agent | Proposes a small issue-scoped code change on a separate branch, runs bounded checks and opens a draft PR | Manual dispatch |

GitHub may delay scheduled workflows. The audit is read-only and does not execute repository code. The coding workflow never merges a pull request.

## One-time setup for cross-repository coding

The coding workflow needs a fine-grained GitHub Personal Access Token because the default `GITHUB_TOKEN` cannot write to other repositories. The workflow now serializes runs per target repository/issue and has a 20-minute timeout.

1. Open **Settings → Developer settings → Personal access tokens → Fine-grained tokens**.
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

The workflows now prefer an Actions secret named `GROQ_API_KEY` and fall back to the legacy `GEMINI_API_KEY` secret. You can keep the existing secret working while adding `GROQ_API_KEY`; both must contain a Groq API key. Never paste either key into chat or commit it.

## Running the coding agent

1. Create a clear GitHub issue in one of the repositories selected in the token.
2. Open this repository's **Actions** tab.
3. Select **AI Coding Agent → Run workflow**.
4. Enter the target repository as `owner/repository` and the issue number.
5. Run it and inspect the resulting draft pull request and its CI checks.

## Safety limits

- The target must belong to `ShivamRaghuvanshi2006`.
- The agent may replace at most two existing source/documentation files.
- It cannot create, rename or delete files, or edit GitHub workflows, secrets, dependency manifests, lockfiles, build/deployment configuration, permissions or licensing.
- Files containing credential-like values are excluded from the code context sent to Groq.
- It runs whitespace checks, available Python/JavaScript syntax checks, and the Python unittest suite when applicable.
- It creates a separate branch and a **draft** pull request. It never merges.
- If the task is ambiguous, unsafe, too large, or the model cannot produce a valid bounded patch, the workflow stops without opening a PR.

## Phase completion tracker

| Phase | Status | Evidence / remaining work |
|---|---|---|
| 1. Daily monitoring | Implemented and live-tested | The enhanced daily health workflow completed successfully on 19 November and created/updated one dated report issue |
| 2. Repository intelligence | Implemented and live-tested | Expanded portfolio audit successfully inspected 11 public, non-fork repositories and published findings in issue #31 |
| 3. Prioritisation | Partially implemented | Portfolio audit produces P1/P2/P3 findings with evidence and confidence; automated labels, deduplication of individual findings and resolution tracking remain |
| 4. Autonomous fixes | Implemented; credential-gated and not yet revalidated after latest workflow edits | Add `GH_AGENT_TOKEN` with the permissions above, then run one small issue through branch push and draft PR creation |
| 5. Verification | Partial | Coding workflow includes whitespace checks, Python/JavaScript syntax checks and Python unittest discovery for `tests/` or `test/`; project-specific build/CI checks remain |
| 6. Continuous improvement | Not implemented | Track accepted/rejected proposals, repeat failures and health trends across runs |

## Current scope and limitations

- The portfolio audit reads repositories visible to the workflow token; the default workflow token reliably covers the current public-repository scope, not arbitrary private repositories.
- The audit inspects root-level file names and selected repository/Actions metadata; it does not read full source files, execute builds, run tests, inspect full dependency trees or perform a complete security review.
- Recent Actions history may be unavailable for repositories with Actions disabled or inaccessible.
- The coding workflow can write only to repositories explicitly selected in the fine-grained token.
- A successful model response is not proof that a code change is correct. Review every draft PR and its CI results before merging.
