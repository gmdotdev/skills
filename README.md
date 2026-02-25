# gmdotdev/skills

Curated AI coding agent skills for [OpenCode](https://opencode.ai) and compatible skill loaders.

## Structure

```
skills/
└── cobra-cli/         # Cobra CLI scaffolding and best practices
    ├── SKILL.md       # Primary skill definition (loaded on activation)
    └── references/    # Detailed reference docs (loaded on demand)
```

## Installation

Using the OpenCode skill installer:

```
/install gmdotdev/skills/cobra-cli
```

Or clone and symlink:

```bash
git clone https://github.com/gmdotdev/skills.git
ln -s $(pwd)/skills/skills/cobra-cli ~/.agents/skills/cobra-cli
```

## Adding Skills

Each skill lives in `skills/<skill-name>/` and must contain a `SKILL.md` with YAML frontmatter (`name`, `description`). See any existing skill for the pattern.

## License

MIT
