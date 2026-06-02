# ripgrep Recipes

## Fast Repo Audit

```bash
rg -n 'deprecatedApi|oldFunction|TODO' src tests docs
```

## Find Markdown References

```bash
rg -n -g '*.md' 'grep -R|find .*grep|xargs grep'
```

## Find TypeScript Symbols

```bash
rg -n -t ts 'class .* extends Context\.Service|static readonly layerNoDeps|Effect\.fnUntraced'
```

## File Discovery

```bash
rg --files -g 'SKILL.md'
rg --files -g '*.ts' -g '!dist/**'
```

## Literal Strings

```bash
rg -n -F 'Effect.fn("Domain.method")'
```

## Context Around Matches

```bash
rg -n -C 2 'Layer\.provide' packages
```

## Structured Output

```bash
rg --json -n 'Context.Service' packages > matches.jsonl
```
