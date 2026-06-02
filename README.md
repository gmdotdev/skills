# gmdotdev/skills

Curated AI coding agent skills for [skills.sh](https://skills.sh)-compatible agents.

## Available Skills

| Skill | Description |
|-------|-------------|
| [cobra-cli](skills/cobra-cli/) | Cobra CLI scaffolding and best practices for Go |
| [worktrunk](skills/worktrunk/) | Git worktree workflow with the Worktrunk `wt` CLI |

## Installation

Install any skill with the [`skills` CLI](https://skills.sh/docs/cli):

```bash
npx skills add gmdotdev/skills
```

Or install a specific skill:

```bash
npx skills add gmdotdev/skills/cobra-cli
# or
npx skills add gmdotdev/skills/worktrunk
```

## Structure

```
skills/
├── cobra-cli/         # Cobra CLI scaffolding and best practices
│   ├── SKILL.md       # Primary skill definition (loaded on activation)
│   └── references/    # Detailed reference docs (loaded on demand)
└── worktrunk/         # Git worktree workflow with wt
    ├── SKILL.md
    └── references/
```

## Adding Skills

Each skill lives in `skills/<skill-name>/` and must contain a `SKILL.md` with YAML frontmatter (`name`, `description`). See any existing skill for the pattern.

For contribution workflow notes, see [`CONTRIBUTING.md`](CONTRIBUTING.md).

## License

MIT
