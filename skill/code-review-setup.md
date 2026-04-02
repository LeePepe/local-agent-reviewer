---
name: code-review-setup
description: Use when setting up AI-powered PR code review in a repository. Installs a GitHub Actions workflow that runs Codex CLI reviews on pull requests and posts findings as comments.
---

# Code Review Setup

Installs a GitHub Actions workflow that automatically reviews pull request diffs with Codex CLI and posts findings (CRITICAL/HIGH/MEDIUM/LOW) as PR comments.

## When to Use

Use this skill when:
- Setting up AI PR review in a new repository
- The repo has a self-hosted macOS arm64 runner with Codex CLI installed
- You want parallel AI review on code quality, performance, and security

## Prerequisites

Before running, confirm:
- [ ] Self-hosted GitHub Actions runner: macOS arm64
- [ ] Codex CLI installed on the runner (default path: `~/.superset/bin/codex`)
- [ ] Repository has pull requests enabled

## Setup Steps

### Step 1 — Ask the user three questions in a single message

> Here are the defaults I'll use:
>
> - **File pattern**: `*.swift` — which files to diff and review
> - **Topics**: `code-quality,performance,security` — review dimensions (parallel jobs)
> - **CLI path**: `~/.superset/bin/codex` — path to Codex binary on the runner
>
> Accept these defaults, or tell me what to change.

Wait for the response. If the user accepts, proceed with defaults. If they specify changes, update the relevant values.

### Step 2 — Write the workflow file

Create `.github/workflows/code-review.yml` in the target repository with these exact contents, substituting the user's values:

```yaml
name: Code Review (Codex)

on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  pull-requests: write
  contents: read

jobs:
  code-review:
    uses: LeePepe/codex-pr-reviewer/.github/workflows/review.yml@v1
    with:
      file_pattern: '<FILE_PATTERN>'
      topics: '<TOPICS>'
      cli_path: '<CLI_PATH>'
```

Replace `<FILE_PATTERN>`, `<TOPICS>`, and `<CLI_PATH>` with the user's confirmed values.

### Step 3 — Report completion

Tell the user:
1. Which file was written
2. Next step: `git add .github/workflows/code-review.yml && git commit -m "ci: add AI PR code review workflow" && git push`
3. The review will run automatically on the next opened pull request

## Supported Topics

| Topic | What it checks |
|-------|---------------|
| `code-quality` | Naming, duplication, complexity, readability, framework best practices |
| `performance` | Recomputation, memory leaks, excessive updates, inefficient algorithms |
| `security` | Hardcoded secrets, insecure storage, input validation, unsafe APIs |
| `<custom>` | Any custom topic — Codex will review for it generically |

## Notes

- Permissions `pull-requests: write` and `contents: read` are required at the caller level
- The `topics` input is comma-separated with no spaces: `code-quality,performance,security`
- Each topic runs as a separate parallel job; all jobs run even if one fails
