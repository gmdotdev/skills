---
name: code-review
description: Read-only review of the current diff, branch, or PR for correctness, safety, and fit. Use when asked to review code, and at the PR-open and merge gates of review-and-ship.
---

# Code Review

One consistent, evidence-backed review. A request for a "code review" never implies multiple agents, edits, PR comments, or a fix loop.

## Default review

1. Confirm scope. If none is given, review the branch against `git merge-base HEAD main`, including uncommitted changes.
2. Read the task or PR intent, the repo's `AGENTS.md`, and only the skills that govern the diff (framework, infra, testing).
3. Start from the diff. Identify the intended behavior, changed invariants, affected callers, and realistic failures the change introduces. Review the code, not the description of it.
4. Form a short ranked list of falsifiable failure hypotheses. Investigate each with targeted reads of source, callers, tests, and history. No file-by-file commentary pass.
5. Try to disprove each concern: caller guarantees, validation, framework behavior, tests that make it false.
6. Use the smallest read-only deterministic check that settles a consequential hypothesis.
7. Report only surviving findings, by severity, with `path:line`. End with exactly one verdict: `APPROVE`, `REQUEST CHANGES`, or `NEEDS HUMAN REVIEW`.

For an explicit PR-open or merge gate, use exactly one independent reviewer. If none is available, do the same pass yourself and say so.

## Authority

- Read-only. No edits, writing formatters, pushes, GitHub comments, deploys, or extra reviewers.
- The implementer may fix valid findings and ask for another review; that does not make the reviewer mutation-capable.
- Product decisions, scope expansion, accepted risk, or destructive actions return `NEEDS HUMAN REVIEW`.
- Never merge.

## What deserves a finding

- Mismatch with the task or an established contract.
- Correctness, ordering, lifecycle, cleanup, or retry defects.
- Secrets or credentials leaking into code, workflows, logs, or built assets.
- Deploy hazards: a preview path touching production, resource-name drift between deploy tools, DNS or domain changes beyond what the repo manages.
- Framework rule violations that lint cannot see (impure update/view, effects outside their owner, untyped failures).
- Swallowed failures, widened types, weakened lint config, greenwashed tests.
- Missing tests for a realistic regression path.
- Unnecessary complexity that obscures ownership.

No findings for formatting the formatter handles, taste, speculative risk without a concrete path, or legacy code outside the changed boundary. A concise approval is a valid result.

## Finding threshold

A reportable finding establishes:

```text
changed code -> failure mechanism -> realistic trigger -> incorrect outcome
```

Separate severity from confidence. A high-impact guess with weak evidence is investigated further, suppressed, or listed under `Limits`. Before reporting, answer: what exact code causes it, what input or state triggers it, what evidence proves it, what counterevidence did you check, and is it worth interrupting delivery?

Scale effort to consequence: a mechanical diff gets a small review; lifecycle, deploy, state, or public-contract changes get focused evidence gathering.

## Evidence

```sh
git status --short
git merge-base HEAD main
git diff --stat main...HEAD
git diff main...HEAD
git diff
gh pr view <n> --json headRefOid,baseRefName,isDraft,mergeable,statusCheckRollup
```

For a PR, inspect the exact head SHA, base, review threads, and checks. Do not call CI, a preview, or production verified unless that exact environment and revision were observed. Prefer running the repo's check and test commands over speculating. Reproductions go in a temp dir outside the worktree.

Dependency, lockfile, or workflow changes: also check version pins, newly added install scripts, and that action refs and secrets follow the existing conventions.

## Output

```md
## Findings

- [high] `path/to/file.ts:42` - Problem, failure path, smallest safe correction.

## Limits

- Anything material not verified.

## Verdict

REQUEST CHANGES
```

Omit `Limits` when empty. With no findings say so and return `APPROVE`.

`conformance-review` is a separate pass that runs once per PR; it is not a second correctness reviewer.
