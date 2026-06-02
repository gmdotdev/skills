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
rg -n 'Context.Service' packages ai-docs/src .patterns
rg -n 'make: Effect.gen|static readonly layer|static readonly layerNoDeps' packages ai-docs/src .patterns

# generator and function patterns
rg -n 'Effect.fnUntraced|Effect.fn\(|Effect.gen' packages/effect/src packages/effect/test ai-docs/src

# Schema modeling
rg -n 'Schema.Class|Schema.TaggedErrorClass|Schema.brand' packages ai-docs/src

# HTTP/RPC APIs
rg -n 'HttpApi|HttpApiGroup|RpcGroup|RpcServer' packages ai-docs/src

# Effect tests
rg -n 'it.effect|@effect/vitest|TestClock' packages ai-docs/src .patterns
```

Prefer `rg` for text search. Use a structural search tool only when syntax-aware matching is needed.

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
