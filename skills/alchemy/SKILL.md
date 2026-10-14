---
name: alchemy
description: Alchemy v2 (alchemy 2.0.0-beta) infrastructure-as-Effects on Cloudflare - Stack, Cloudflare.Website.*, stages, the state store, adoption, plan-before-deploy, and release-driven production deploys. Use when touching alchemy.run.ts or deploying with Alchemy.
---

# Alchemy v2

Alchemy v2 is Infrastructure-as-Effects: the stack is an Effect program. It is not v1 (`alchemy()` scope, top-level await). Do not apply v1 patterns from memory; verify every API in the installed `node_modules/alchemy/src`, or a clone of `alchemy-run/alchemy` at the installed version.

## Read first

- The repo's `alchemy.run.ts`.
- `AGENTS.md` in the Alchemy repo: Resource, Stage, Reconcile, Adopt, Drift.
- `packages/alchemy/src/Cloudflare/Website/*.ts`: props and JSDoc for the website resources.
- `examples/cloudflare-website-*/alchemy.run.ts`: canonical examples.
- Docs: `cli/{deploy,plan,adopting-resources,inspecting-state}`, `environments/stages`, `state-store`.

## A static site stack

```ts
export default Alchemy.Stack(
  "my-site",
  { providers: Cloudflare.providers(), state: Cloudflare.state() },
  Effect.gen(function* () {
    const stage = yield* Alchemy.Stage;
    const web = yield* Cloudflare.Website.Foldkit("Web", {
      env: { VERSION: process.env.VERSION ?? "development" },
      name: "my-site-web",
      assets: { htmlHandling: "drop-trailing-slash", notFoundHandling: "none" },
      domain:
        stage === "production"
          ? { aliases: ["www.example.com"], name: "example.com", zoneName: "example.com" }
          : null,
      memo: { include: ["**/*", ".version"], lockfile: true },
    });
    return { url: web.url.as<string>() };
  })
);
```

- Website resources drive the app's own `vite build` with an injected Cloudflare plugin, then deploy the client output as static assets. No wrangler config. If the repo also uses `cf` previews, keep `cloudflare()` behind `CF_BUILD=1` (see `cloudflare-previews`).
- A prerendered site sets `notFoundHandling: "none"` so unknown paths 404 instead of falling back to the SPA shell.
- Custom domains only on `production`; other stages get a `workers.dev` URL. Alchemy manages only the hostnames you give it. Other records in the zone (mail, other services) are never touched by it, so do not add them to the stack.
- `memo` hashes inputs to skip unchanged builds. An env var is not a file: write it to a file `memo` includes (`.version` above) so a re-release of an unchanged tree still rebuilds.
- Several apps in one repo: one stack per app, each with its own name, so they deploy and plan independently.

## Versions and runtime

- Pin `alchemy` to one exact version everywhere the stack deploys (laptops, CI) and upgrade them together.
- The CLI needs Node 24.11.1+ or Bun.
- A hostname attached to another Worker fails the deploy (`Cannot attach hostname ... already attached`). Detach it first.

## Plan before deploy

```sh
bunx alchemy plan --stage production      # read-only; --detailed for property diffs
bunx alchemy deploy --stage production
```

1. `+` create, `~` update, `-` delete, `•` no-op. Read every line. Any delete or replace of a Worker, domain, or DNS record is stop-and-ask. A no-op plan is expected for a code-only change.
2. `alchemy deploy` prompts unless there are no changes or `--yes` is set. Never pass `--yes` locally without having read the plan.
3. Production should deploy from CI on a published release, not from a laptop: check out the tag, require it on `main`, run check and test, bootstrap the state store, plan, deploy with `--yes`, then poll a `version.json` until it reports the released version.

## Stages

A stage is an isolated instance with its own state and physical names. Resolution: `--stage`, then `$ALCHEMY_STAGE`, then `live_$USER`. A bare `alchemy deploy` goes to a personal sandbox; clean it up with `alchemy destroy --stage live_<user>`. Never run `destroy --stage production`. If previews are `cf` Worker Previews, do not also create `pr-*` Alchemy stages.

## State

`Cloudflare.state()` keeps state in a Durable Object Worker on the account, with its keys in the Cloudflare Secrets Store. `bunx alchemy provider cloudflare bootstrap --no-input` deploys or adopts it (run it in CI before planning). State is what the next plan diffs against, not the live cloud; `alchemy drift` compares with reality.

```sh
bunx alchemy state
bunx alchemy state list <stack>/production
```

## Credentials

`CLOUDFLARE_API_TOKEN` and `CLOUDFLARE_ACCOUNT_ID`, from CI secrets; never commit or echo them. A production token needs:

- Account: Workers Scripts edit, Account Settings read, Secrets Store edit and read. Without Secrets Store access the bootstrap fails with `Unauthorized`.
- Zone (only the zones the stack attaches domains in): Zone read, DNS edit, Workers Routes edit.

## Adopting existing resources

When state is missing for a resource that exists: `alchemy plan --adopt` and read it. Resources the provider can prove are ours recover silently; foreign ones fail with `OwnedBySomeoneElse` unless `--adopt` is set. `--adopt` is a whole-deploy takeover; run it only after the user approves the plan.

## Verify

Typecheck, then plan. A passing typecheck does not prove the plan is safe; read the plan.
