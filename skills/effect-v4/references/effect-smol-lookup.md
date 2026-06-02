# effect-smol Lookup Guide

Use the current `effect-smol` codebase as the source of truth for Effect v4 APIs.

## High-Value Paths

| Path | Use |
|---|---|
| `AGENTS.md` | Repo workflow, validation, and style rules |
| `.patterns/effect.md` | Core Effect coding rules |
| `.patterns/testing.md` | Test framework and assertion rules |
| `ai-docs/README.md` | How AI docs are authored and generated |
| `ai-docs/src/01_effect/` | Core Effect examples |
| `ai-docs/src/09_testing/` | Testing examples |
| `ai-docs/src/50_http-client/` | HTTP client examples |
| `ai-docs/src/51_http-server/` | HTTP server examples |
| `packages/effect/src/` | Source for exact API names and types |
| `packages/effect/test/` | Runtime behavior examples |
| `packages/*/typetest/` | Type-level API expectations |

## Search Patterns

```bash
# service patterns
grep -R "Context.Service" -n packages ai-docs/src .patterns

# generator and function patterns
grep -R "Effect.fnUntraced\|Effect.fn(\|Effect.gen" -n packages/effect/src packages/effect/test ai-docs/src

# Schema modeling
grep -R "Schema.Class\|Schema.TaggedErrorClass\|Schema.brand" -n packages ai-docs/src

# HTTP/RPC APIs
grep -R "HttpApi\|HttpApiGroup\|RpcGroup\|RpcServer" -n packages ai-docs/src

# Effect tests
grep -R "it.effect\|@effect/vitest\|TestClock" -n packages ai-docs/src .patterns
```

Prefer a structural search tool if the target environment has one, but simple text search is enough for initial discovery.

## effect-smol Validation Hints

For changes inside `effect-smol`, use the repo's own guidance and run the narrowest validation that covers the change:

| Change type | Typical validation |
|---|---|
| Code changes | `pnpm lint-fix`, targeted `pnpm test <test_file.ts>`, `pnpm check:tsgo` |
| Tests-only changes | `pnpm lint-fix`, targeted `pnpm test <test_file.ts>`, `pnpm check:tsgo` |
| Type/API changes | targeted `pnpm test-types <filename>`, plus `pnpm check:tsgo` when source types changed |
| JSDoc text/category/link changes | `pnpm lint` |
| JSDoc example changes | `pnpm lint`; package-local `pnpm docgen` when needed |
| AI docs | `pnpm ai-docgen` when generated docs must be refreshed |

Always report which commands ran and what failed or was skipped.
