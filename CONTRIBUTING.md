# Agent Contribution Notes

This repository contains `skills.sh`-compatible AI coding agent skills. These notes are for humans and agents contributing new skills or maintaining existing ones.

## Repository layout

```text
README.md
CONTRIBUTING.md
skills/
  <skill-name>/
    SKILL.md
    references/
      *.md
```

- `README.md` is the public index of available skills.
- `skills/<skill-name>/SKILL.md` is the required skill entrypoint.
- `skills/<skill-name>/references/*.md` can hold longer examples, troubleshooting, API notes, and detailed recipes.

## Adding a skill

1. Start from the repository root and confirm the worktree state:

   ```bash
   git status --short --branch
   ```

2. Create the skill folder:

   ```bash
   mkdir -p skills/<skill-name>/references
   ```

3. Add `skills/<skill-name>/SKILL.md` with YAML frontmatter:

   ```yaml
   ---
   name: <skill-name>
   description: One sentence describing what the skill is for and when to use it.
   ---
   ```

4. Keep `SKILL.md` focused on activation-time guidance. Put long examples, API details, troubleshooting, and expanded recipes in `references/*.md`.

5. Update `README.md`:
   - add the skill to the Available Skills table
   - update the tree in the Structure section if needed
   - keep installation examples accurate

6. Validate the skill structure:

   ```bash
   python3 - <<'PY'
   from pathlib import Path
   import sys

   root = Path('skills')
   problems = []
   for path in sorted(p for p in root.iterdir() if p.is_dir()):
       skill = path / 'SKILL.md'
       if not skill.exists():
           problems.append(str(skill))
           continue
       text = skill.read_text()
       if not text.startswith('---\n') or '\n---\n' not in text[4:]:
           problems.append(f'{skill}: missing YAML frontmatter')
   if problems:
       print('\n'.join(problems), file=sys.stderr)
       raise SystemExit(1)
   print('skill structure ok')
   PY
   ```

7. Review the final diff:

   ```bash
   git diff -- README.md CONTRIBUTING.md skills/
   git diff --check
   ```

## Contribution style

- Prefer concise, agent-actionable instructions over long prose.
- Include concrete commands and repo-relative file paths.
- Include pitfalls and verification steps for workflows that break easily.
- Keep public docs portable: avoid local machine paths, usernames, private hostnames, private tokens, and maintainer-specific workflow assumptions.
- Do not include secrets or credentials in skill files, examples, docs, or commit history.

## Agent guidance

When an AI agent contributes to this repo:

1. Inspect `README.md`, `CONTRIBUTING.md`, existing `skills/*/SKILL.md`, and `git status` before editing.
2. Match the style of existing skills.
3. Keep public docs generic and portable.
4. Put long-form supporting material in `references/*.md` rather than bloating `SKILL.md`.
5. Run the validation snippet above and `git diff --check` before reporting completion.
6. Report exact changed paths and verification output.
