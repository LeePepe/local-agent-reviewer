# local-agent-reviewer

AI-powered pull request code review using Codex CLI and GitHub Actions.

Runs parallel review jobs on each PR diff and posts findings (CRITICAL / HIGH / MEDIUM / LOW) as PR comments.

## Prerequisites

- Self-hosted GitHub Actions runner: macOS arm64
- Codex CLI installed on the runner at `~/.superset/bin/codex` (or custom path)

## Quick Start

### Option A: Claude Code Skill (recommended)

1. Copy the skill to your Claude Code skills directory:
   ```bash
   mkdir -p ~/.claude/skills/code-review-setup
   cp skill/code-review-setup.md ~/.claude/skills/code-review-setup/SKILL.md
   ```

2. Open a Claude Code session in your target repository and ask:
   > Set up PR code review for this repo

   The skill will ask about your file pattern, topics, and CLI path, then write the workflow file.

3. Commit and push:
   ```bash
   git add .github/workflows/code-review.yml
   git commit -m "ci: add AI PR code review workflow"
   git push
   ```

### Option B: Manual Setup

Create `.github/workflows/code-review.yml` in your repository:

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
    uses: LeePepe/local-agent-reviewer/.github/workflows/review.yml@v1
    with:
      file_pattern: '*.swift'          # adjust for your language
      topics: 'code-quality,performance,security'
      cli_path: '~/.superset/bin/codex'
```

## Supported Topics

| Topic | What it checks |
|-------|----------------|
| `code-quality` | Naming, duplication, complexity, readability, framework best practices |
| `performance` | Recomputation, memory leaks, excessive updates, inefficient algorithms |
| `security` | Hardcoded secrets, insecure storage, input validation, unsafe APIs |
| `<custom>` | Any custom topic — Codex reviews for it generically |

## How It Works

```
PR opened / updated
       ↓
caller workflow (.github/workflows/code-review.yml)
       ↓
uses: LeePepe/local-agent-reviewer/.github/workflows/review.yml@v1
       ↓
setup job: topics string → JSON matrix
       ↓
review jobs (parallel, one per topic)
  ├── get diff (filtered by file_pattern)
  ├── build prompt (topic-specific)
  ├── run: codex -p "$(cat prompt.txt)" --yolo
  └── gh pr comment → posts findings
```

## Permissions

The caller workflow must declare:
```yaml
permissions:
  pull-requests: write
  contents: read
```

## Troubleshooting

**No diff / review says "No changes detected"**
The file pattern didn't match any changed files. Check your `file_pattern` input.

**`jq: command not found` in setup job**
Install jq on your runner: `brew install jq`. Alternatively, replace the jq command with:
```bash
python3 -c "
import json, sys
topics = sys.argv[1].split(',')
print(json.dumps({'topic': topics}))
" "${{ inputs.topics }}"
```

**CLI not found**
Verify `cli_path` matches the actual Codex binary path on the runner. Check with `which codex` on the runner machine.
