# Council

A multi-agent decision skill for Claude Code. Hit it with an architecture call, it runs a panel of canonical patterns from prior art + a Codex red team on a different model, and outputs a chairman ruling tagged 🟢 canonical / 🟡 common / 🔴 inference.

Built for solo dev workflow. No infra.

> 中文版 / Chinese version: **[SKILL.zh.md](./SKILL.zh.md)**

## What it does

When you face a real architecture decision — system design, data structure choice, rewrite-vs-incremental, which DB/queue/cache to pick — Council retrieves the domain's canonical patterns and master framings, runs an adversarial cross-model check, and gives you a recommendation with explicit confidence tiers.

The point isn't Claude impersonating an expert. The point is forcing Claude to cite verifiable sources (paper DOIs, GitHub repos, talks) and to admit when it's reasoning from first principles instead of from canon.

## How it's structured

Each decision is two files:

```
.claude/decisions/YYYY-MM-DD_<slug>/
  decision.md      # frontmatter + problem + candidates + tradeoffs + ruling + kill criteria
  council-log.md   # audit: problem-class confirm, anti-council 4 questions, red team, chairman path
```

Cross-decision learnings get appended to a single `code_playbook.md` instead of per-decision files — they need to be read every session, so they live separately from per-decision state.

See [`SKILL.md`](./SKILL.md) for the full schema, output rules, and red-team flow.

## Why two files instead of six

The first version generated five files per decision (assumptions, kill-criteria, review-points, council-log, decision). Two weeks in, three of them never got reopened after the decision closed.

The dead files all answered "what did we think at decision time?" But post-decision the only consumer is the next council session asking "what's the pattern here?" One question, not three files of context. Designed for an auditor that didn't exist.

The principle: separate write-once state from read-every-session state. Most multi-agent designs collapse the two and produce ceremony.

## Install

Drop this directory into `~/.claude/skills/council/` and invoke via `/council` in Claude Code.

## License

MIT.
