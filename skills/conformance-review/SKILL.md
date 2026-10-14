---
name: conformance-review
description: Read-only review of architecture and engineering quality against the repo's canonical shape (owners, boundaries, names, amount of machinery). Run on request, and exactly once on the first non-draft PR head.
---

# Conformance Review

Raise code from merely working to the clearest current shape for the repo. `code-review` asks whether a change is safe and correct. This pass asks whether it uses the right owners, primitives, boundaries, names, and amount of machinery.

## Invocation contract

- Run when the user asks for conformance, architecture, or refactor-quality review.
- In delivery, run exactly once per PR, on the first non-draft head. Do not rerun on every push; rerun only when a human asks.
- A conformance approval is not a correctness approval, and vice versa. Both gates stay.
- Read-only. No edits, formatting, commits, pushes, comments, merges, or scope widening.

## Establish the evidence boundary

1. Pin exact base and head. Inventory every changed path and the PR intent. Include uncommitted changes for a local branch.
2. Read the repo's `AGENTS.md` and the smallest matching skills for each changed surface. Repo guidance is evidence, not mutation authority.
3. Start from the diff, then inspect immediate callers, composition roots (entry points, infra stacks, workflows), tests, and old surfaces the diff materially changes.
4. Classify each touched surface: canonical shape, documented divergence, or legacy to migrate. Neighboring code is evidence of behavior, not the target design.
5. For large diffs keep a coverage ledger. Do not approve a skimmed surface; list it under `Not reviewed` and use `NEEDS HUMAN REVIEW` if the audit could not finish.

## Canonical-first standard

- Prefer upstream sources over memory: the installed packages, and any read-only reference clones the repo keeps.
- Write net-new code in the target shape. Do not copy a legacy pattern to match a neighbor.
- When a change substantially alters a legacy surface, migrating that surface is in scope. Remove superseded helpers rather than adding compat wrappers. Incidental edits do not authorize cleanup.
- A deviation needs a real constraint and a matching doc or lint note. "Existing code does it" is not a reason.
- Build only the layers the behavior needs. No abstractions for symmetry, no just-in-case code. Wait for the third use before abstracting.
- Colocate. No barrel files unless genuinely necessary.

## Code shape

- Names express stable domain meaning, not implementation phase or control flow.
- Functions take minimal, predictable parameters. No option bags that disable required behavior.
- Types infer from schemas and library APIs. Flag duplicated aliases, `any`, pass-through `unknown`, type assertions, and casts hiding a broken boundary. Discriminated unions over optional properties; make impossible states impossible.
- Untrusted input is parsed once, at its owner. Parse, don't validate.
- Imports, exports, and literals are deterministic where order carries no meaning.
- Abstractions earn existence through reusable policy, a substantial pure algorithm, external I/O, or resource lifetime. Fold forwarding wrappers back into their owner.

## Framework and infra audits

Apply the matching skill's checklist to every touched surface and its immediate callers:

- UI framework (for example the upstream `foldkit` / `audit-program` skills): purity of update and view, effects in their owners, message naming, routing through the router.
- Effect (the upstream `effect-ts` skill): requirements and layer wiring, lifetime ownership, cleanup on success, failure, and interruption, stream backpressure. Typechecking is only the start.
- Infra (`alchemy`, `cloudflare-previews`): stack, stage, and resource names unchanged unless intended; only production gets custom domains; secrets come from the CI secret store; anything that alters production is visible in the plan and called out in the PR.

## Tests and proof

- Tests prove the real boundary and invariant, not an implementation-shaped mock. Fake only true external seams. No module mocking.
- Check unhappy paths, stale state, and cleanup, not only the happy path.
- Reject greenwashing: weakened lint config, broadened types, swallowed failures, inflated timeouts, tests that mirror the implementation.

## Finding bar

Reportable even when behavior passes, but it must name a concrete long-term defect: split ownership, canonical-shape violation, hidden dependency, unsafe lifecycle, stale abstraction, incomplete migration of a touched surface, or missing proof for a consequential invariant. No formatting, taste, or hypothetical rewrites.

```text
changed code -> violated canonical invariant -> concrete consequence -> smallest coherent target shape
```

`high`: architecture or lifecycle violation that sets the wrong durable boundary. `medium`: material quality or migration defect that should block the PR. `low`: useful non-blocking advisory. A fix that needs a product decision or large scope is `NEEDS HUMAN REVIEW`.

## Output

```md
## Conformance Findings

- [medium] `path/to/file.ts:42` - Evidence, violated invariant, consequence, smallest target shape.

## Coverage Ledger

- Canonical shape and touched legacy: reviewed
- Framework architecture: reviewed / not applicable
- Effect and lifecycle: reviewed / not applicable
- Infra and deploy: reviewed / not applicable
- Tests and proof: reviewed / not applicable

## Not reviewed

- Material scope not inspected, and why.

## Verdict

REQUEST CHANGES
```

Omit `Not reviewed` when empty. End with exactly one verdict: `APPROVE`, `REQUEST CHANGES`, or `NEEDS HUMAN REVIEW`. Approval requires every applicable ledger item to be substantively reviewed; it does not require inventing findings.
