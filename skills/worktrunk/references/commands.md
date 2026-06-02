# Worktrunk Command Reference

## Install

```bash
brew install worktrunk
wt config shell install
```

Alternative:

```bash
cargo install worktrunk
wt config shell install
```

Restart the shell after installing shell integration.

## Core Commands

| Command | Purpose |
|---|---|
| `wt switch --create <branch>` | Create a new branch and worktree from the default branch |
| `wt switch <branch>` | Switch to an existing worktree or create one for an existing branch |
| `wt switch ^` | Switch to the default branch worktree |
| `wt switch -` | Switch to the previous worktree |
| `wt list` | List worktrees and compact status |
| `wt list --full` | Show fuller worktree status |
| `wt switch pr:<number>` | Check out a GitHub PR into a worktree |
| `wt remove` | Remove current worktree and delete the branch if merged |
| `wt --help` | Show top-level help |
| `wt <command> --help` | Show command-specific help |

## Automation Pattern

Agents and scripts should prefer structured output when shell integration cannot affect the current process:

```bash
wt switch --create docs/example --format json --no-cd
```

Example output includes:

```json
{
  "action": "created",
  "branch": "docs/example",
  "path": "/path/to/repo.docs-example",
  "created_branch": true,
  "base_branch": "origin/main"
}
```

Then run subsequent commands with the returned path as the working directory.

## Simple PR Recipe

```bash
git fetch origin --prune
git checkout main
git pull --ff-only origin main
wt switch --create docs/example --format json --no-cd
cd /path/from/json
# make changes
git diff --check
git add <files>
git commit -m "docs: add example"
git push -u origin HEAD
gh pr create --base main --head docs/example --title "docs: add example" --body "..."
```

After the PR is merged:

```bash
wt switch docs/example
wt remove
```
