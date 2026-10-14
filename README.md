# gmdotdev/skills

Opinionated guardrail skills for coding agents: how to review, test, verify, and ship a change, and how to deploy it on Cloudflare. Browse them at [skills.gm.dev](https://skills.gm.dev).

## Install

With the [`skills` CLI](https://skills.sh/docs/cli):

```bash
npx skills add gmdotdev/skills            # all of them
npx skills add gmdotdev/skills --skill code-review testing-guidelines
npx skills update                         # pull the latest versions later
```

## Skills

| Skill | Use it to |
| --- | --- |
| [code-review](skills/code-review/) | Review a diff for correctness and safety, with one verdict |
| [conformance-review](skills/conformance-review/) | Review architecture and engineering quality against the repo's canonical shape |
| [review-and-ship](skills/review-and-ship/) | Take a change from branch to verified squash merge |
| [testing-guidelines](skills/testing-guidelines/) | Red-green-refactor TDD with a mutation-check gate |
| [verify-web](skills/verify-web/) | Drive a web change in a real browser and leave proof |
| [cloudflare-previews](skills/cloudflare-previews/) | PR and `main` Worker Previews with the `cf` CLI |
| [alchemy](skills/alchemy/) | Alchemy v2 infrastructure and release-driven production deploys |

## Upstream skills we use

Framework skills belong to the framework. We don't copy them here; install them from their maintainers so `npx skills update` keeps them current.

```bash
npx skills add Effect-TS/skills --skill effect-ts
npx skills add foldkit/foldkit --skill foldkit generate-program audit-program
```

Alchemy publishes no user-facing skill yet, so [alchemy](skills/alchemy/) lives here until it does.

## Adding skills

See [CONTRIBUTING.md](CONTRIBUTING.md). When you add or remove a skill, update [skills.gm.dev](https://skills.gm.dev) too (`apps/skills-web/src/catalog.ts` in `gmdotdev/dotcom`).
