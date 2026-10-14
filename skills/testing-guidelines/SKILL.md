---
name: testing-guidelines
description: Choose and write tests with red-green-refactor TDD and a mutation-check gate, at the smallest boundary that proves the behavior. Use when adding behavior, fixing bugs, or judging whether a test is worth keeping.
---

# Testing Guidelines

## Loop: red, green, refactor

Every feature and every bug fix.

1. **Red.** Write the failing test first. Run it. It must fail for the reason you expect (an assertion on the missing behavior, not an import error or typo). If it passes, the test is wrong.
2. **Green.** Minimum code to pass. Hardcoding is fine.
3. **Refactor** while green. Rerun after each change.
4. **Bug fixes.** Reproduce with a failing test first, then fix.
5. **Legacy code.** Characterization tests before changing anything.

## Mutation-check gate

A green test proves nothing until it can go red for the right reason. Before calling a test done:

1. Break the production code on purpose in the way the test claims to guard (flip the condition, drop the message, return the old value, remove the failure branch).
2. Run the test. It must fail with a message naming the broken behavior.
3. Revert the mutation (`git diff` shows only the intended change) and confirm green.

If the test still passes under mutation, it guards nothing: strengthen the assertion or delete it. Do this for every new test on a consequential invariant, and never leave a mutation in the tree. For a bug fix, the red run on unfixed code is the mutation check.

## Pick the boundary

Prefer the smallest boundary that proves the invariant:

| Behavior | Test |
| --- | --- |
| Pure state transitions (`init`, `update`, reducers) | Story-style tests: given a state, send messages, assert the next state and emitted commands |
| Rendered output, accessibility, links, interaction | Scene-style tests that locate by role, label, or text |
| Services, layers, resources | Effect tests (`@effect/vitest`: `it.effect`, `TestClock`) |
| Prerender, bundling, deploy | The build, the deploy plan, the preview URL; not unit tests |

Mock only true external seams. Don't mock what you don't own, and no module mocking.

## Tests are behavioral specs

Name each test for what the user or system does, give it one flow, and keep its steps readable top to bottom. Prefer one story per user flow over many micro-assertions. A new feature's stories are its spec: write them before the code, and treat a story that is hard to read as a design smell.

- Every fallible operation gets a success test and a failure test.
- Flows get at least one multi-step test.
- Locate UI accessibly (`role('link', { name })`, label, text), not by CSS selector or placeholder.
- Assert attributes that matter (`href`, `aria-*`), not incidental markup or styling.

## Effect tests

`TestClock` instead of sleeps; deterministic layers; cover success, typed failure, and interruption or cleanup when lifetime matters. `it.live` only for real I/O. Scope fixtures so finalizers prove cleanup.

## Test value

Add a test when it would fail on a realistic regression. Skip it when it freezes incidental markup, copies the implementation, or asserts a mock. Do not delete a test on judgment alone; run the mutation check first.

## Report

Exact commands and outcomes, which tests were mutation-checked, and what remains unproven (preview, production).
