# Adversarial QA

Claude Code skill: `adversarial-qa`

## What

Structured product-level adversary tests of **your own** governance gates (omit optional auth, second surface, inverse route, body authority, rollback counters). Matrix of attack × surface × expected refuse × actual. **Not** exploit PoCs or offensive cyber.

## When to use

Before claiming a gate is done; after authz PRs; when adding MCP/CLI/admin surfaces.

## Install

Copy this folder into your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/adversarial-qa
cp SKILL.md ~/.claude/skills/adversarial-qa/
```

Claude Code loads `SKILL.md` from `~/.claude/skills/<name>/`.

## Sibling skills

`fail-closed-review`, `verified-delivery`, `contract-and-compat`, `migration-and-data-safety`, `reachability-audit`, `handoff-faber-rigor`, `bench-loop`, `karpathy-method`, `code-that-holds`

Proposed repos: see [Vorxeo](https://github.com/Vorxeo) `skill-*` packs.

## License

MIT — Copyright (c) 2026 Vorxeo. See [LICENSE](./LICENSE).
