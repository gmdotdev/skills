---
name: worktrunk
description: Set up and use Worktrunk (`wt`) for git worktree-based development. Use when starting isolated feature work, switching between worktrees, opening pull requests from worktree branches, or cleaning up worktrees after merge.
---

# Worktrunk Worktree Workflow

Worktrunk (`wt`) is a git worktree manager for parallel development. It makes the safe default easy: create a separate worktree for each new piece of work, keep the main checkout clean, open a pull request from the worktree branch, and remove the worktree once the PR is merged.

## When to Apply

Use this skill when:

- Setting up Worktrunk on a development machine
- Starting new feature, fix, docs, or refactor work in a git repository
- Running multiple agent or human workstreams in parallel
- Switching between worktrees without reusing a dirty checkout
- Creating a GitHub pull request from a worktree branch
- Cleaning up a worktree and branch after a PR has merged

Do not use this skill for one-off inspection in a repo where no files will change.

## Install Worktrunk

### macOS or Linux with Homebrew

```bash
brew install worktrunk
wt config shell install
```

Restart the shell after installing shell integration. The integration lets `wt switch` change the current directory. Without it, use `wt switch --format json --no-cd` and manually `cd` to the reported `path`.

### Rust / Cargo

```bash
cargo install worktrunk
wt config shell install
```

### Other package managers

Worktrunk also documents installs via pacman, conda-forge, pixi, and winget. On Windows, the command may be installed as `git-wt` to avoid conflicting with Windows Terminal's `wt` command.

Verify:

```bash
wt --version
wt --help
```

## Starting New Work

Always start from a clean, current base branch.

```bash
git status --short --branch
git fetch origin --prune
git checkout main
git pull --ff-only origin main
```

Create a worktree and branch:

```bash
wt switch --create docs/add-worktrunk-skill
```

If shell integration is unavailable or the command is running in automation:

```bash
wt switch --create docs/add-worktrunk-skill --format json --no-cd
cd <path-from-json>
```

Branch naming suggestions:

| Work type | Example branch |
|---|---|
| Feature | `feat/add-auth` |
| Bug fix | `fix/login-redirect` |
| Refactor | `refactor/extract-cache` |
| Tests | `test/add-api-coverage` |
| Docs/skills | `docs/worktrunk-skill` |

## Daily Worktree Commands

```bash
wt list                 # show active worktrees and status
wt list --full          # include more status columns
wt switch <branch>      # switch to an existing worktree or create one for a branch
wt switch ^             # switch to the default branch worktree
wt switch -             # switch to the previous worktree
wt switch pr:123        # check out a GitHub PR branch into a worktree
```

Useful automation pattern:

```bash
wt switch --create feat/example --format json --no-cd
```

The JSON includes the created worktree path and branch name, which is easier for agents and scripts to consume than interactive directory switching.

## Pull Request Workflow

From inside the worktree:

```bash
git status --short --branch
# edit files, then run project validation
git diff --check
git add <changed-files>
git commit -m "docs: add worktrunk workflow skill"
git push -u origin HEAD
gh pr create --base main --head docs/worktrunk-skill   --title "docs: add worktrunk workflow skill"   --body "## Summary
- add a Worktrunk skill
- document setup and worktree PR workflow

## Verification
- git diff --check
- skill structure validation"
```

Use the target repo's own tests, linters, typecheckers, or validation scripts before pushing. For documentation-only skill repos, at minimum validate frontmatter and run `git diff --check`.

## Cleanup After Merge

After the PR is merged, remove the worktree and branch:

```bash
wt remove
```

If you are not currently inside the worktree, switch to it first:

```bash
wt switch docs/worktrunk-skill
wt remove
```

`wt remove` removes the worktree and, by default, deletes the local branch when it is merged. If cleanup fails because the branch is not merged or has uncommitted changes, inspect with:

```bash
git status --short --branch
wt list --full
```

Do not force-remove a worktree with unpushed or unmerged work unless the owner explicitly confirms it can be discarded.

## Agent Workflow Guidelines

1. For any new development task that changes a git repo, create a new `wt` worktree from the current default branch.
2. Do not stack unrelated changes in an existing worktree.
3. Inspect repo instructions and `git status` inside the worktree before editing.
4. Make focused changes and run repo-defined validation.
5. Commit and open a PR from the worktree branch.
6. Leave the worktree in place while the PR is under review.
7. After the PR is merged, remove the worktree with `wt remove`.

## Common Pitfalls

1. **Editing the main checkout.** New work belongs in a `wt` worktree, not on the default branch checkout.
2. **Skipping shell integration.** Without shell integration, `wt switch` cannot change the parent shell directory. Use JSON output and manual `cd` in automation.
3. **Creating a worktree from a stale base.** Fetch and update the default branch before `wt switch --create`.
4. **Mixing tasks in one worktree.** Each task gets its own branch/worktree and PR.
5. **Deleting work before merge.** Keep the worktree until the PR is merged or the branch is intentionally abandoned.
6. **Forcing cleanup blindly.** `wt remove` protects unmerged or dirty branches; inspect before overriding.

## Verification Checklist

- [ ] `wt --version` works
- [ ] shell integration installed or automation uses `--format json --no-cd`
- [ ] branch/worktree created from the intended base
- [ ] repo instructions and `git status` checked inside the worktree
- [ ] repo validation passed
- [ ] branch pushed and PR opened
- [ ] worktree removed after PR merge
