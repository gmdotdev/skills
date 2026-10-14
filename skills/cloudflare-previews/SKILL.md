---
name: cloudflare-previews
description: Cloudflare Worker Previews with the cf CLI - cloudflare.config.ts, the CF_BUILD guard for Vite, cf previews deploy/delete, Node vs Bun constraints, and a GitHub Actions workflow for PR and main previews. Use when setting up or debugging Worker previews.
---

# Cloudflare Worker Previews (`cf`)

PR previews (`pr-<N>`) and a `main` preview are Worker Previews of the production Worker, deployed with the `cf` CLI (`1.0.0-beta.x`). Production can deploy separately (for example through Alchemy; see `alchemy`). Previews and production must use the same Worker name.

## Moving parts

| File | Role |
| --- | --- |
| `cloudflare.config.ts` | `defineConfig` from `cf/config`: `worker.name`, `compatibilityDate`, observability |
| `vite.config.ts` | adds `cloudflare()` from `@cloudflare/vite-plugin` only when `CF_BUILD=1` |
| `package.json` | `build:cf` = `CF_BUILD=1 cf build`; `preview:deploy` = `CF_BUILD=1 cf previews deploy` |
| `.github/workflows/preview.yml` | deploy on PR open/sync/reopen and push to `main`; delete on PR close |

## Why `CF_BUILD=1`

`cf build` and `cf previews deploy` need the Cloudflare Vite plugin to emit Build Output. Tools that inject their own copy for production (Alchemy does) collide with it, so keep it out of the default config:

```ts
const cfPlugins = process.env["CF_BUILD"] === "1" ? [cloudflare()] : [];
```

- Never remove the guard or set `CF_BUILD` outside preview jobs.
- Any `cf` invocation that builds sets `CF_BUILD=1`.
- `--prebuilt` skips building and reuses existing Build Output.

## Runtime constraints

- `cf` requires Node >= 22.18. In CI, install Node with `actions/setup-node` from `.nvmrc`.
- Bun cannot load `cloudflare.config.ts` for `cf`. Install with Bun, but let `cf` run under Node: its bin has a `#!/usr/bin/env node` shebang, so `bun run preview:deploy` works as long as Node is on PATH and you never pass `bun --bun`.
- Auth: `CLOUDFLARE_API_TOKEN` and `CLOUDFLARE_ACCOUNT_ID` in env (CI secrets), or `cf auth login` locally. A preview token needs only Workers Scripts edit and Account Settings read; keep it separate from the production token, because preview jobs run branch code.

## Commands

```sh
bun run build:cf                                   # local Build Output only
bun run preview:deploy pr-123                      # deploy a named preview
CF_BUILD=1 bunx cf previews deploy pr-123 --dry-run
bunx cf previews delete pr-123 --worker <worker-name> --force
```

`cf` discovery: `cf cli search "<task>"`, then `<command> --help`. Do not chain nested `--help`.

## CI shape

- `PREVIEW_NAME` = `pr-<number>` for pull requests, `main` for pushes to main. Export a `VERSION` like `${PREVIEW_NAME}-${GITHUB_SHA::7}` so the deployed page can say what it is.
- Concurrency group per preview name, with cancel-in-progress.
- Read the URL from the deploy output and upsert one PR comment (an HTML marker such as `<!-- preview -->` finds it again).
- A `cleanup` job deletes `pr-<N>` when the PR closes, and treats "not found" as success.
- Fork PRs get no secrets; skip them.
- Several Workers in one repo: one matrix entry per app (working directory and Worker name), with a separate comment marker per app.

## Troubleshooting

| Symptom | Check |
| --- | --- |
| Build Output missing | `CF_BUILD=1` not set, so the plugin is absent |
| Config fails to load | running under Bun, or Node < 22.18 |
| 401 / 403 | token scope or `CLOUDFLARE_ACCOUNT_ID` wrong |
| Preview hits the wrong Worker | `worker.name` drifted from the production name |
| Stale PR previews pile up | the cleanup job failed; delete by hand |
| Production build suddenly differs | `cloudflare()` leaked outside the `CF_BUILD` guard |

## Verify a change

1. The plain build still works without the plugin.
2. `bun run build:cf` under Node >= 22.18.
3. Push, confirm the preview run is green and the URL serves the new commit. A local `--dry-run` is not proof the preview deploys.
