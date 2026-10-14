---
name: review-and-ship
description: Land a change on GitHub end to end - branch, test-first implementation, validation, PR, preview, review gates, squash merge. Explicit invocation includes merge unless narrowed; choosing it yourself grants no PR or merge authority.
---

# Review and Ship

When the user explicitly invokes `review-and-ship` ("review and ship this", `/review-and-ship`), carry the change through review, fixes, validation, commit, push, PR, required checks, and squash merge. That invocation authorizes both the PR-open and merge gates. Default terminal state is a verified merged PR.

Honor a narrower endpoint ("local only", "draft PR only", "don't merge"). Choosing this skill yourself grants no authority. Discussing or editing the skill is not an invocation.

Stop only for a genuine blocker: ambiguous ownership, a product decision, missing credentials, accepted risk, destructive recovery, or an unsatisfied required gate. Full-workflow authority does not waive CI or review, and never authorizes a production release.

## 1. Establish the change

1. Read the request, the repo's `AGENTS.md`, and the smallest matching skills.
2. Inspect the full intended change against base: `git status`, `git diff main...HEAD`, staged and untracked files. Preserve pre-existing user work.
3. Check for an open PR on the branch: `gh pr list --head "$(git branch --show-current)"`. Record base and head SHAs, draft state, mergeability, checks, and unresolved threads.
4. Never commit to `main`. Create a short branch (`feat/...`, `fix/...`, `chore/...`) first. Do not switch a dirty checkout or rewrite remote history for convenience.

After compaction or a resumed session, re-read the request, the diff, and live PR state before editing further.

## 2. Implement, test, self-review

1. Trace the existing boundary before editing. Prefer installed sources and reference clones over memory.
2. Follow `testing-guidelines`: failing test first, minimum code, refactor.
3. Make the smallest coherent change. Do not weaken lint rules to pass.
4. Self-review the whole diff in context: callers, failure paths, lifetimes, secrets, workflow and deploy effects.
5. Fix every substantiated in-scope finding, then re-review the whole diff.

Independent reviews run only at the gates below, not after every fix.

## 3. Qualify

Run the commands CI runs (usually check, test, build; the repo's `AGENTS.md` and `.github/workflows` say which), from the repo root, and record exact commands and outcomes. Narrow while iterating, full set before pushing. Any content change invalidates an earlier pass. Rerun a clearly transient failure once before diagnosing.

- User-visible changes: run `verify-web` against the local build.
- Infra changes: read the deploy plan (`alchemy`) and the preview build (`cloudflare-previews`).

## 4. Open the PR

1. Run one independent `code-review` on the stable diff immediately before the PR write. Fix valid findings, requalify, repeat until `APPROVE`.
2. Review the staged patch, commit only intended files with a Conventional Commit message, push, and confirm the remote head matches (`git rev-parse HEAD` vs `gh pr view --json headRefOid`).
3. `gh pr create --base main` with a Conventional Commit title and a short body (what and why; non-obvious validation only).
4. The first non-draft PR head gets exactly one `conformance-review`. Address valid findings; do not rerun unless a human asks.

## 5. Preview and checks

- Wait for all required checks on the exact current head: `gh pr checks <N> --watch`.
- If the repo deploys PR previews, confirm the preview matches the head SHA, open it, and run `verify-web` there for user-visible changes. Preview success is not production proof.
- On failure read the logs (`gh run view <id> --log-failed`), fix with a focused commit, requalify.

Handle valid in-scope review feedback with normal follow-up commits.

## 6. Merge gate

1. Record the head SHA and run one independent `code-review` on it. Reuse an approval only for the unchanged SHA.
2. Immediately before merging confirm: head SHA unchanged, PR open and non-draft, all required checks green, mergeable, threads handled.
3. Squash merge pinned to the reviewed head:

   ```sh
   gh pr merge <N> --squash --match-head-commit <reviewed-head-sha>
   ```

4. Re-read the PR, verify `MERGED` and the merge commit, and check that post-merge workflows on `main` go green.

A missing, pending, skipped, canceled, or failed required gate is not a pass. Report the exact blocker.

## Production

Merging does not deploy production unless the repo says otherwise. Publish releases and run production deploys only when the user asks.

## Completion report

The merged PR (URL, merge commit) or the concrete blocker. Include: change summary, findings repaired, exact validation commands and outcomes, head and base SHAs, review verdicts, check state, preview URL and what was verified on it, and omissions.
