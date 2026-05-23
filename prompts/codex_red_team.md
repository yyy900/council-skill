# Red team — Fact-check + Codex cross-source (two subtasks, one prompt)

## Why centralized here

Inline WebFetch verifying every 🟢 in the main flow is too slow. Fact-check (a safety net for 🟢 URLs) + cross-source (a safety net for the main recommendation) are both **safety nets for Claude's single-model overconfidence**. Run them once together — doesn't need to break the rhythm of the main flow.

---

## Section A — Fact-check (Claude self-run, always runs, can't be disabled)

### Step A1 — Extract every 🟢 URL from this session's output

grep the council output currently being written (Step 3 canonical section + Step 4 master framing + Step 8 reading list), list:

```
🟢 ARC: https://www.usenix.org/legacy/publications/library/proceedings/fast03/tech/full_papers/megiddo/megiddo.pdf
🟢 Caffeine: https://github.com/ben-manes/caffeine
🟢 Werkzeug Routing: https://werkzeug.palletsprojects.com/en/latest/routing/
...
```

### Step A2 — Batch WebFetch verify

One WebFetch per URL (same session + same URL gets 15 min cache automatically, no double cost).

Status only:
- **200** → pass, keep 🟢
- **4xx / 5xx / timeout / dead** → demote to 🟡 (or delete this entry), mark "fact-check failed: <url>" in council-log.md

Don't read content (unless you want to — not cost-effective). Status 200 is enough to plug the "Claude invented URL" failure mode.

### Step A3 — Write to council-log.md

```markdown
### ✅ Red team subtask A — Fact-check

**Verify date**: YYYY-MM-DD
**Total verified**: N 🟢 URLs
**Passed**: M
**Demoted**: K

| URL | status | action |
|---|---|---|
| https://... | 200 | 🟢 kept |
| https://... | 404 | 🟡 demoted + source removed |
```

### Step A4 — Effect on Step 7 chairman's ruling

If any 🟢 was demoted by fact-check:
- The main recommendation's supporting source is affected → chairman must **explicitly state**: original recommendation based on [X canonical], but X's URL fact-check failed, so recommendation is downgraded to 🟡 / or direction is changed
- Cannot pretend it didn't happen

---

## Section B — Codex cross-source (`red_team.codex == true` only)

### Why this subtask exists

Council default is Claude playing all roles. All angles share the same model bias — "anti-council" using left hand against right hand can't find real blind spots.

Codex is an OpenAI model, a **real diversity source**. When enabled, it doesn't read all of council-log (to avoid being seeded), only the problem statement + chairman's ruling + key assumptions. Independent challenge.

### Trigger

`decision.md` frontmatter:

```yaml
red_team:
  codex: true
```

`false` or missing → skip Section B (**Section A still runs**).

### When NOT to enable Section B

- Pure-code decision with no external blast and no production data (default for solo vibe-coding — AI rewrites code in hours, code-level reversibility is cheap)
- All key assumptions still pending verification (benchmark not run, data not in, co-founder not aligned) → challenging too early, go gather data first

### When to enable Section B

- Decision has **external blast** (public commitment / users / partners / hire — anything affecting people outside you)
- Decision touches **production data** that's already been generated
- Decision is **load-bearing** (≥3 downstream decisions will build on it)
- You and co-founder have strong internal consensus → easy group-think, needs external view
- Council is internally highly consistent → suspicious, needs cross-source challenge
- Involves major resource commitment (money / time / team energy)
- **User strongly inclined toward one direction → this is precisely when red team matters most.** Council's purpose is to challenge, not validate. Strong conviction is not a reason to skip — it's a reason to run.

### Step B1 — Assemble minimal input for codex

Don't dump the full council-log into it. Only pass:

```
Problem statement: <decision.md "problem" section>
Current recommendation: <council-log.md chairman's ruling direction, 1-2 sentences>
Key assumptions (top 3): <top 3 from assumptions>
Key kill criteria (top 3): <top 3 from kill-criteria>
User identity / background: <1 line, e.g. "AI agent architect, 2-person team", to help codex find identity bias>
```

Reasoning: less context → codex less likely to be seeded → challenge is more independent.

### Step B2 — Invoke /codex consult

```
You are an independent red team, not part of the prior discussion. I have a decision recommendation:

[Step B1 input]

Tasks:

1. Identify the most dangerous hidden assumption in the recommendation (not the listed ones — the one nobody asked about)
2. What's the most likely failure mode for this direction? Give 1-3 concrete scenarios
3. If you fully disagreed with this direction, what would you recommend instead? Why?
4. Is the decision-maker's identity / background biasing this recommendation? How?
5. In 12 months, under what conditions does this recommendation become a liability?
6. The sources Claude marked 🟢 Canonical — have you heard of them? Which obvious canonical options did Claude miss?

Requirements: sharp, specific, no pleasantries, no "both sides have merit" balanced narrative.
Your job is to find holes, not to provide comfort.
```

### Step B3 — Append codex output to council-log.md

```markdown
### 🔴 Red team subtask B — Codex cross-source (real diversity source)

**Model**: codex (OpenAI) via /codex consult
**Date**: YYYY-MM-DD
**Saw only**: problem statement + chairman's ruling + top 3 assumptions + top 3 kill criteria (did NOT read full council-log)

[codex raw output, 6 sections mapping to 6 questions]

---

#### Claude's response to codex's challenges

[For each codex challenge, explicitly: accept / partial accept / rebut. With reasoning.]

#### Chairman's second ruling (if changed)

[If codex's challenge is valid enough to require changing the recommendation, write the new ruling. Otherwise explicitly say "ruling stands" + why.]
```

### Step B4 — Effect on assumptions and kill criteria

If codex exposed an unlisted hidden assumption → add to assumptions, tag "found by Codex red team"
If codex gave new failure scenarios → add to kill-criteria

---

## Forbidden

- Don't skip Section A — fact-check is unconditionally mandatory (toggle only controls Section B)
- Don't treat codex output as gospel (it's also a model, also wrong sometimes)
- Don't skip Claude's response to codex (Claude must give a second judgment)
- Don't silently modify the chairman's ruling (either explicitly say "ruling stands" or explicitly say "changed because X")
- When codex is disabled, don't **fake** codex output (if toggle is false, skip Section B, don't have Claude impersonate codex)
- Don't pretend fact-check passed — the verify table written to council-log.md must be real WebFetch results, not imagined

## Cost note

- **Section A**: one WebFetch per URL (same session + same URL has 15 min auto-cache). Typical 5-10 🟢 → 5-10 seconds latency, no LLM cost
- **Section B**: each codex call ~ $0.01-0.05 (GPT models) + a few seconds latency. Recommended at major decision **first session** + **major assumption changes in continue sessions** — don't run on every continue
