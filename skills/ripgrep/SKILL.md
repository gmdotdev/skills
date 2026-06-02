---
name: ripgrep
description: Install and use ripgrep (`rg`) for fast recursive text search. Use when searching files, code, docs, logs, or replacing slower `grep -R` / `find | xargs grep` workflows.
---

# ripgrep (`rg`)

ripgrep is a fast recursive search tool that respects `.gitignore` by default, searches Unicode text well, and combines the common jobs of `grep`, `find`, and `xargs` into one command.

## When to Apply

Use this skill when:

- Searching source code, docs, config files, logs, or generated text
- Replacing `grep -R`, `find ... -exec grep`, or `find | xargs grep`
- Finding files by glob before inspecting or editing them
- Producing concise search output for another tool or agent
- Auditing a repo for deprecated API names, style patterns, TODOs, secrets placeholders, or docs references

Use plain `grep` only when `rg` is not installed and cannot be installed, or when matching POSIX shell portability matters more than speed and ergonomics.

## Install

macOS with Homebrew:

```bash
brew install ripgrep
```

Debian / Ubuntu:

```bash
sudo apt-get update
sudo apt-get install ripgrep
```

Fedora:

```bash
sudo dnf install ripgrep
```

Arch Linux:

```bash
sudo pacman -S ripgrep
```

Rust / Cargo:

```bash
cargo install ripgrep
```

Verify:

```bash
rg --version
rg --help
```

## Core Usage

Search recursively from the current directory:

```bash
rg 'Context.Service'
```

Search specific paths:

```bash
rg 'Context.Service' packages ai-docs/src .patterns
```

Show only file names with matches:

```bash
rg -l 'Context.Service'
```

Count matches per file:

```bash
rg -c 'Context.Service'
```

Search by file type or glob:

```bash
rg -t ts 'Context.Service'
rg -g '*.md' 'grep -R'
rg -g '!dist/**' -g '!coverage/**' 'TODO|FIXME'
```

List files instead of searching contents:

```bash
rg --files
rg --files -g '*.ts'
rg --files | rg '(^|/)AGENTS\.md$'
```

Include hidden or ignored files only when needed:

```bash
rg --hidden 'pattern'
rg --no-ignore 'pattern'
rg -uuu 'pattern'
```

## Replacing Common grep Patterns

| Old pattern | Prefer |
|---|---|
| `grep -R "foo" -n src tests` | `rg -n 'foo' src tests` |
| `grep -R "foo\|bar" -n .` | `rg -n 'foo|bar'` |
| `find . -name '*.ts' -exec grep -n "foo" {} +` | `rg -n -g '*.ts' 'foo'` |
| `grep -R -l "foo" .` | `rg -l 'foo'` |
| `grep -R -c "foo" .` | `rg -c 'foo'` |

## Agent Tips

- Prefer single quotes around regex patterns in shells so backslashes are not eaten.
- Use `-n` when line numbers matter; ripgrep prints line numbers by default when stdout is a terminal, but `-n` is explicit in scripts.
- Use `-F` for literal strings that contain regex metacharacters.
- Use `-i` for case-insensitive search.
- Use `-C 2`, `-A 3`, or `-B 3` for context lines.
- Use `--json` when another program needs structured results.
- Start with ignored-file defaults; add `--hidden`, `--no-ignore`, or `-uuu` only after deciding ignored files matter.
- For very broad searches, pass explicit paths and globs to keep output focused.

## Common Pitfalls

1. **Searching ignored files accidentally.** By default `rg` respects `.gitignore`; add `--no-ignore` or `-uuu` only when ignored files matter.
2. **Regex escaping differences.** `rg` uses Rust regex syntax. Use `-F` for literal strings.
3. **Too much output.** Narrow by path, `-g`, `-t`, `-l`, or `-m <count>` before reading huge results.
4. **Hidden files skipped.** Add `--hidden` for dotfiles and hidden directories.
5. **Binary files skipped.** Use `-a` only when you intentionally want to search binary-looking files as text.

## Verification Checklist

- [ ] `rg --version` works.
- [ ] Search paths are explicit enough to avoid flooding output.
- [ ] Globs or type filters match the intended file set.
- [ ] Ignored/hidden file behavior is intentional.
- [ ] Literal searches use `-F` when regex interpretation would be wrong.
