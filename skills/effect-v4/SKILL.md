---
name: effect-v4
description: Write, review, migrate, and debug current Effect v4 / effect-smol TypeScript code. Use when working with Effect services, layers, Schema data modeling, typed errors, resources/scopes, HTTP/RPC APIs, streams, or Effect-aware tests.
---

# Effect v4 / effect-smol

Effect v4 APIs move quickly. Treat the local project and the current `effect-smol` codebase as the source of truth; do not rely on older Effect examples or memory.

## When to Apply

Use this skill when:

- Writing TypeScript code with `Effect`, `Context`, `Layer`, `Schema`, `Scope`, streams, resources, or schedules
- Creating or reviewing Effect services and layers
- Modeling data, IDs, validation, and typed errors with `Schema`
- Migrating promise, callback, filesystem, process, HTTP, or database boundaries into Effect
- Building HTTP API or RPC contracts with the current unstable Effect packages
- Writing tests for Effect code with `@effect/vitest`
- Debugging Effect type errors, missing services, layer wiring, resource leaks, or error-channel behavior

Do not use this skill for generic TypeScript work that does not involve Effect APIs.

## Source of Truth Workflow

Before implementing or giving API-specific advice:

1. Inspect the target repo's local instructions, package scripts, nearby implementation, and tests.
2. Inspect the current `effect-smol` codebase, especially:
   - `AGENTS.md`
   - `.patterns/effect.md`
   - `.patterns/testing.md`
   - `ai-docs/README.md`
   - `ai-docs/src/**`
   - `packages/effect/src/**`
   - `packages/*/test/**`
   - `packages/*/typetest/**`
3. Prefer source, tests, and generated AI docs from `effect-smol` over blog posts, old snippets, or assumptions.
4. Make the smallest change that satisfies the task.
5. Run the target repo's validation commands before claiming completion.

If the task is specifically inside `effect-smol`, follow its repo guidance: use `pnpm`, keep changes surgical, read relevant `.patterns/` files first, and run the narrowest validation that covers the change.

## Core Imports and Type Shape

Common core imports come from `effect`:

```ts
import { Context, Effect, Layer, Schema, Scope } from "effect"
```

Effect values use this type shape:

```ts
Effect.Effect<A, E, R>
```

| Type parameter | Meaning |
|---|---|
| `A` | success value |
| `E` | typed failure channel |
| `R` | required services/context |

Prefer making service methods return effects with `R = never`; acquire dependencies in the service layer instead of leaking them through every method call.

## Choosing Effect Constructors

| Need | Preferred primitive |
|---|---|
| Pure value | `Effect.succeed(value)` |
| Synchronous side effect | `Effect.sync(() => value)` |
| Synchronous throwable API | `Effect.try({ try, catch })` |
| Promise-returning API | `Effect.tryPromise({ try, catch })` |
| Callback API | `Effect.callback` |
| Sequential composition | `Effect.gen(function* () { ... })` |
| Reusable named workflow | `Effect.fn("Name")(function* (...) { ... })` |
| Library/internal hot path | `Effect.fnUntraced(function* (...) { ... })` |
| Shared in-flight computation | `Effect.cached` |
| Recover by tag | `Effect.catchTag` / `Effect.catchTags` |
| Inspect success/failure | `Effect.result(effect)` |

Use exact API names from current `effect-smol` source/tests when in doubt.

## Generator Rules

### Do not use `try/catch` to handle Effect errors

Effect failures live in the Effect error channel. JavaScript `try/catch` does not catch normal Effect failures yielded inside `Effect.gen`.

```ts
// Bad: do not handle Effect failures this way
Effect.gen(function* () {
  try {
    return yield* loadUser(id)
  } catch (error) {
    return fallbackUser
  }
})
```

Use Effect combinators instead:

```ts
loadUser(id).pipe(
  Effect.catchTag("UserNotFound", () => Effect.succeed(fallbackUser)),
  Effect.mapError((cause) => new LoadUserFailed({ cause }))
)
```

### Use `return yield*` for terminal branches

When a branch fails, interrupts, or otherwise terminates the generator, make that explicit:

```ts
const program = Effect.gen(function* () {
  const user = yield* findUser(id)
  if (!user) return yield* new UserNotFound({ id })

  return yield* updateUser(user)
})
```

Do not write a bare `yield* Effect.fail(...)` and then continue with unreachable code.

### Prefer `Effect.fnUntraced` in library implementations

If a function only wraps `Effect.gen`, prefer `Effect.fnUntraced` for internal/library code where tracing overhead and noise are not helpful:

```ts
const parseConfig = Effect.fnUntraced(function* (input: unknown) {
  return yield* Schema.decodeUnknownEffect(Config)(input)
})
```

Use `Effect.fn("Domain.operation")` when the name is useful for tracing and diagnostics.

## Services and Layers

Prefer class-based services using `Context.Service`:

```ts
export interface Users {
  readonly findById: (id: UserId) => Effect.Effect<User, UserNotFound>
}

export class Users extends Context.Service<Users, Users>()("@app/Users") {}

export const UsersLive = Layer.effect(
  Users,
  Effect.gen(function* () {
    const db = yield* Database

    const findById = Effect.fn("Users.findById")(function* (id: UserId) {
      const row = yield* db.users.findById(id)
      if (!row) return yield* new UserNotFound({ id })
      return User.make(row)
    })

    return Users.of({ findById })
  })
)
```

Guidelines:

- Use stable, globally unique service identifiers such as `@app/Users`.
- Pull dependencies once in the layer.
- Keep service method `R` types small, usually `never`.
- Use `Layer.succeed` or `Layer.sync` for static/test implementations.
- Use `Layer.effect` for effectful setup.
- Use scoped layers or `Effect.acquireRelease` when cleanup is required.
- Provide layers at application/test edges, not deep inside business logic.
- Reuse layer constants for expensive resources; layer memoization depends on reference identity.

## Schema, Data Modeling, and Errors

Use `Schema` as the source of truth for data crossing boundaries: inputs, outputs, persisted data, decoded JSON, HTTP/RPC contracts, and typed errors.

### Records

```ts
export class User extends Schema.Class<User>("User")({
  id: UserId,
  name: Schema.String,
  email: Schema.String
}) {}
```

### Branded primitives

```ts
export const UserId = Schema.String.pipe(Schema.brand("UserId"))
export type UserId = typeof UserId.Type
```

### Tagged errors

```ts
export class UserNotFound extends Schema.TaggedErrorClass<UserNotFound>()(
  "UserNotFound",
  { id: UserId }
) {}
```

Guidelines:

- Use typed errors for recoverable domain, validation, transport, and external failures.
- Use defects for bugs and impossible invariants, not normal business outcomes.
- Decode untrusted inputs at boundaries with `Schema.decodeUnknownEffect`.
- Encode outputs when leaving a typed boundary with `Schema.encodeEffect`.
- Preserve original unknown causes in typed errors when wrapping external failures.

## Resources, Scope, and Background Work

Use Effect lifetimes instead of ad hoc globals:

- `Effect.acquireRelease` for acquire/use/release lifecycle
- `Effect.addFinalizer` for cleanup inside scoped setup
- `Effect.forkScoped` for background fibers tied to a scope
- `Effect.cached` for shared in-flight work
- `Scope` for explicit lifetime boundaries

Avoid unscoped global maps, manually managed `Promise | undefined`, `Fiber | undefined`, `started` flags, and background work without finalizers.

## HTTP API and RPC

Effect v4 HTTP/RPC APIs currently use unstable imports. Verify exact names against `effect-smol` before writing code.

Common import families:

```ts
import { FetchHttpClient, HttpClient } from "effect/unstable/http"
import { HttpApi, HttpApiBuilder, HttpApiEndpoint, HttpApiGroup } from "effect/unstable/httpapi"
import { Rpc, RpcGroup, RpcClientBuilder, RpcSerialization, RpcServer } from "effect/unstable/rpc"
```

Use HTTP API when:

- endpoints are public or resource-oriented
- OpenAPI documentation matters
- clients may not share Effect/RPC TypeScript contracts
- webhooks, uploads, OAuth, health checks, or status codes are central

Use RPC when:

- client and server can share TypeScript contracts
- app-internal commands and typed errors matter more than REST shape
- a strongly typed client is valuable
- the same group may be mounted over HTTP or another transport

Contract guidelines:

- Keep shared contracts browser/client-safe.
- Define payload, success, and error schemas explicitly.
- Keep handlers/server implementation separate from contract definitions.
- Put auth/session requirements in middleware or annotations rather than scattered callsites.
- Match client/server serialization layers.

## Testing

Prefer `@effect/vitest` for Effect code:

```ts
import { assert, describe, it } from "@effect/vitest"
import { Effect } from "effect"

it.effect("loads a user", () =>
  Effect.gen(function* () {
    const user = yield* Users.findById(userId)
    assert.strictEqual(user.id, userId)
  }))
```

Rules from `effect-smol` testing patterns:

- Use `it.effect` for tests that return Effects.
- Use regular `it` for pure synchronous tests.
- Use `assert` from `@effect/vitest`; do not use Vitest `expect` unless the target repo already requires it.
- Do not use `Effect.runSync` in tests.
- Use `TestClock` for time-dependent behavior.
- Put type-level tests in the target repo's typetest structure and run the relevant type-test command.
- Provide fresh test layers when mutable state must not leak between tests.

## Migration Heuristics

When moving existing TypeScript code into Effect:

1. Identify the boundary: promise API, callback API, filesystem, process, HTTP, database, timer, or global state.
2. Wrap the boundary once in a service or typed abstraction.
3. Decode external/untrusted data with `Schema`.
4. Model recoverable failures as typed errors.
5. Keep business logic inside Effects; avoid bouncing repeatedly between Promise and Effect.
6. Replace manual lifecycle management with Scope, layers, resources, finalizers, and scoped fibers.
7. Migrate tests with the production code.
8. Run the target repo's typecheck and targeted tests.

## Common Pitfalls

1. **Using old Effect examples.** Verify current API names in `effect-smol` source/tests.
2. **Handling Effect failures with `try/catch`.** Use Effect combinators.
3. **Missing `return yield*` on terminal generator branches.** Make failure/interruption explicit.
4. **Leaking dependencies through every service method.** Acquire dependencies in layers.
5. **Providing layers inside business logic.** Provide layers at app/test boundaries.
6. **Creating layer factories repeatedly.** Reuse layer constants when memoization matters.
7. **Skipping Schema decoding at boundaries.** Decode unknown input and encode outgoing data where contracts require it.
8. **Using manual global state for resources.** Prefer scopes, resources, finalizers, cached effects, and scoped fibers.
9. **Leaving tests on ad hoc runtimes.** Use Effect-aware test helpers and layers.
10. **Guessing unstable HTTP/RPC imports.** Check current `effect-smol` code first.

## Verification Checklist

- [ ] Target repo instructions and nearby style inspected.
- [ ] Current `effect-smol` source/docs/tests checked for exact APIs.
- [ ] Services/layers use current `Context.Service` / `Layer` patterns.
- [ ] Data crossing boundaries is modeled with `Schema`.
- [ ] Recoverable failures are typed errors; defects are reserved for bugs/invariants.
- [ ] Resources/background fibers have scoped cleanup.
- [ ] HTTP/RPC contracts stay separate from server handlers.
- [ ] Tests use Effect-aware patterns.
- [ ] Target repo validation commands were run and reported.
