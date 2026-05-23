---
name: council
description: |
  Prior-art consultant — when you face a decision under domain uncertainty
  (coding architecture, growth strategy, business positioning, system design,
  content positioning), this skill retrieves the canonical patterns and
  master framings for that problem class, paired with a Codex cross-model
  red team to catch single-model overconfidence.

  Solves: any single decision-maker's experience radius is limited. This
  skill is your domain-knowledge entrypoint — it lets AI retrieve the
  field's consensus on your behalf and cite real sources.

  When to call:
  - Architecture choice / system design / data structure selection (coding)
  - "What's the best practice in this problem domain?" (any domain)
  - Performance / security / maintainability tradeoffs
  - Rewrite vs incremental / split vs unify / which DB/queue/cache
  - Non-coding: growth strategy, business positioning, team design,
    content positioning — anything "what does the field do, whose
    shoulders should I stand on"

  Proactively self-invoke (lightweight inline mode) when about to make a
  choice in a domain with known prior art AND own confidence < 7/10.
  Trigger on yourself, not just on user.

  When NOT to call:
  - Implementation detail with no architectural choice (pure syntax /
    known API call) → just write it
  - Product form / is-it-the-right-problem → /office-hours
  - UI/UX → /design-* skills
  - PR / single-file sanity → /review or /codex
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - WebFetch
  - WebSearch
---

# Council

## Core belief

> You're writing code while standing on giants' shoulders. This skill finds which giants you should stand on.

Not Claude impersonating a master. Claude **retrieving the field's consensus and citing real sources on your behalf**.

---

## Response language

Match the user's input language. Chinese input → Chinese output. English input → English output. Don't mix languages within a single response.

---

## Default depth: inline mode

By default, council runs in **inline mode**: 1-line canonical citation + 1-line trap + continue. No decision object, no ritual.

**Heavy mode** (full 11-step OPEN flow + decision.md + council-log.md + red team) is opt-in. Triggers — any of:

- **External blast**: decision affects others / public commitment / users already perceive (publish, hire, brand, partnership)
- **Production data**: decision touches data that's already generated (schema migration, user-generated content)
- **Load-bearing**: ≥3 downstream decisions will be built on this one, changing it cascades
- **User explicit invoke**: user says "open council" / "formal council" / "正式召开"

In the vibe-coding era, "code-to-revert" is no longer a valid criterion — AI rewrites code in hours. The reversibility test is about **non-code consequences**: data, public commitment, follow-up decisions, other people.

### Inline mode output template

```
[inline canonical check]
Choice: <the choice being made, 1 sentence>
Canonical: <pattern name + 🟢/🟡/🔴 + 1 URL if applicable>
Trap: <most common failure mode of this pattern, 1 sentence>
Going with: <which option + 1-line reason>
```

3-5 lines, then continue writing code. Don't break flow.

---

## Three confidence tiers (the most important rule)

Every technical claim must be tagged with one tier:

| Tier | When to use | Must be able to cite | Verification |
|---|---|---|---|
| **🟢 Canonical** | Field-recognized | **Verifiable URL**: paper DOI / GitHub repo / company eng blog / known talk recording | URLs batch-verified by the red team fact-check pass (not inline in main flow) |
| **🟡 Common** | Community-common, not unique | **Multiple adopters** or **mainstream framework default** | At least 2 named adopters; URL not required |
| **🔴 Claude inference** | No domain consensus | **Explicitly state**: "based on first-principles reasoning, not domain canon" | No source required |

**Never mix tiers.** Dressing 🔴 up as 🟢 is the most lethal failure mode — once trust collapses, the whole skill is dead.
If you can't find 🟢 / 🟡 → honestly mark it 🔴. That's 1000× better than faking canonical.

**🟢 URLs are verified by the red team fact-check pass**: main flow doesn't inline-verify (too slow). When you write 🟢 you must include a URL; verification happens centrally in the red team — see "Red team" section + `prompts/codex_red_team.md` subtask A. Claude's training distribution ≠ verified truth; this gate cannot be skipped.

---

## Three working modes

| Mode | Trigger | Flow file |
|---|---|---|
| OPEN | New problem / new decision | `prompts/open.md` |
| CONTINUE | Existing decision, adding info | `prompts/continue.md` |
| REVIEW | Post-implementation look-back (user-triggered, not scheduled) | `prompts/review.md` |

State simplified: `open → done`. Major reversals go through `reversed`.

---

## Decision object

Path (preserves the original convention):
- Default (project-scoped): `<project>/.claude/decisions/<id>/`
- Global: `~/.claude/decisions/<id>/`, with a symlink left in the original location
- ID: `YYYY-MM-DD_<slug>`

Each decision directory (2 files, optionally + outcome):

```
decision.md      One-stop: frontmatter + problem + candidates (with confidence/source/trap)
                 + tradeoffs + recommendation (detailed plan) + non-recommended + reading list
                 + kill criteria
council-log.md   Process audit: problem-class confirmation, anti-council 4 questions,
                 codex red team, chairman's ruling path (including superseded early rulings)
outcome.md       Filled in after implementation, optional
```

**No longer separate files (folded into decision.md)**: canonical_patterns / master_framings / tradeoff_table / reading_list / kill-criteria.

**learning_notes don't go in the decision directory**: appended directly to `~/.claude/projects/<slug>/memory/code_playbook.md` (single source, accumulates across decisions).

frontmatter schema (slimmed to 6 fields):

```yaml
---
id: 2026-04-30_ai-architecture
title: <human-readable>
status: open | done | reversed
problem_class: <e.g. "low-latency multimodal pipeline architecture">
topics: [keyword list, used for surface matching]
red_team:
  codex: true   # default true under new positioning (cross-source check on canonical claims)
---
```

**Removed fields** (no consumer / self-describing / redundant): `visibility` (path tells you project vs global), `opened` (id prefix already encodes date), `recommendation_confidence` (body's 🟢/🟡/🔴 already expresses this), `chosen_pattern` (no query tool consumes it), `false_match_count` (designed but never enforced).

Old `decision.md` files don't need migration — CONTINUE / REVIEW only consume status / topics / red_team.codex / problem_class, and removed fields don't break.

---

## Learning accumulation (single file, two sections)

`~/.claude/projects/<slug>/memory/code_playbook.md` — single cross-decision learning file. The old `user_decision_profile.md` is now merged in (14 days of staleness showed standalone-file value was lower than the schema cost).

### Section A — User decision patterns (top of file, read on every OPEN)

User judgment patterns / repeated blind spots / effective angles, accumulated across decisions. Schema:

```markdown
## User decision patterns (as of YYYY-MM-DD)

### Decision list
| ID | Title | Status | Recommended direction | Outcome |

### Repeated blind spots
(Don't generalize when data points < 3)

### Angles effective with this user
- e.g. steelman beats defense — default-assume user pushback is valid + upgrade the proposal to absorb it

### Recommendations this user never accepts

### Decision pattern shorthand
- YYYY-MM-DD: note
```

### Section B — Decision technique playbook (appended after each session)

```
## YYYY-MM-DD — [problem class]

**Decision ID**: YYYY-MM-DD_<slug>
**Chairman's recommendation**: [pattern] (confidence 🟢/🟡/🔴)
**Key tradeoff**: [one-line core]
**Outcome**: pending (filled in after implementation)
**Next time this problem class comes up, think of first**: [pattern name]
```

Next time the **same problem class** comes up, **read Section A + Section B entries for the same problem** — your personal technical playbook + user decision-style file.

---

## Anti-hallucination hard rules

1. 🟢 Canonical must come with a **verifiable URL** (paper DOI / GitHub repo / company eng blog / known talk recording) — not just a source name. URL verification happens centrally in the red team fact-check pass; main flow does not inline-verify
2. Master framings must have a specific source (which book's which chapter, which talk, which commit message, which mailing list email) + a locatable URL or ISBN
3. Don't say "X would definitely think this"
4. Don't invent quotes; don't invent URLs — the red team fact-check is the forcing function
5. Hazy-memory people / patterns → demote to 🔴 and say so explicitly

Violate any → output invalid, rewrite.

---

## Citation freshness rule

The council's value is **contemporary, practice-validated consensus**, not archaeology.

- **Prefer**: things still adopted as default in mainstream frameworks/repos within the last 5 years (Werkzeug current docs / Caffeine / axum / FastAPI / Phoenix etc.)
- **Acceptable**: genuinely timeless foundational concepts (actor model, FSM, hash table, CSP, event loop — still in active use even if 30+ years old)
- **Cautious**: pre-2010 specific implementation patterns, **unless** they're still the field's reference implementation or directly inherited by modern mainstream systems
- **Forbidden**: hazy-memory "there must have been" old papers / book chapters where you can't name a system still using them in production → demote to 🔴 or delete

Test: can you name **something in production this year still using it**? If not — don't list it.

---

## Output quality hard rules (every word earns its place)

The council's value rests on **density**. The user pays a ritual cost (explicit trigger + wait + read) for something that's not ordinary tech chat. Every word must do work.

### 8 specific rules

1. **Deletion test**: ask each paragraph "if I cut this, does the decision change?" — if no → delete.
2. **No framing / transitions / summaries**: don't write "below in three parts," "first the conclusion," "in summary," "all things considered," "specifically for your situation." Give the ruling directly.
3. **No symmetric padding**: if a candidate only has one sentence's worth of substance, write one sentence. Don't pad to make the table look balanced.
4. **No hedging clichés**: don't write "depends on the specific scenario," "both sides have merit," "you can choose based on need," "possibly/maybe." Chairman's ruling = ruling, not a disclaimer.
5. **Canonical pattern description ≤ 3 lines**: 1 sentence core idea + 1 sentence source + 1 sentence trap. After reading it, the user can judge whether to dig deeper.
6. **Tradeoff table cells contain only decision-bearing facts**: numbers / yes-no / one-line summary. "Performs better" is filler; "radix lookup is O(prefix), ~10 routes is meaningless" is fact.
7. **Chairman's ruling = 1 paragraph + 1 confidence tier + 1 specific next step**. No "weighing the tradeoffs..." opener.
8. **No manufactured changes**: when reviewing candidates / files / configs, first judge by "does deleting/changing this solve a real problem?" — if not → don't touch. "No actionable changes" is a valid output. Don't produce for the sake of producing.

### Counter-examples (filler the user has flagged in practice)

- "Specifically for your rewrite this time" as a framing header → just give the recommendation
- "Three viewpoints converge on the same refactor" then restating the refactor → already said it, cut
- Explaining H1 by recounting Werkzeug's evolution → user asked for canon, not history

Violate any → output invalid, rewrite.

---

## Red team toggle rules

Council pairs canonical retrieval with a two-part red team to catch single-model overconfidence.

### Section A — Fact-check (always runs)

For every 🟢 canonical claim: batch WebFetch verify the URL.
- 200 → keep 🟢
- 4xx / 5xx / timeout / dead → demote to 🟡 or delete, mark "fact-check failed: <url>" in council-log.md

Cost: ~5-10 seconds, no LLM cost. Always runs in heavy mode. In inline mode, optional but recommended.

### Section B — Codex cross-source (default on)

Independent OpenAI model challenge against main recommendation. Different training distribution = real diversity source.

**Skip Section B when ANY of**:
- Pure-code decision with no external blast, no production data (default for solo vibe-coding)
- Key assumptions all unverified (benchmark not run, data not in, co-founder not aligned) → challenging too early, go gather data first

**Open Section B explicitly when ANY of**:
- Decision has external blast (public / users / partners affected)
- Decision touches production data
- Internal strong consensus → group-think risk, needs external view
- Council's own output is highly self-consistent → suspicious, needs cross-source challenge
- **User strongly inclined toward one direction → THIS IS WHEN RED TEAM MATTERS MOST.** Council's purpose is to challenge, not validate. Don't let strong user conviction skip red team.

Cost reference: Codex single call ~$0.01-0.05 + a few seconds latency.

### Removed criteria (from prior versions)

- ~~"Revert > 1 week"~~ — AI rewrites code in hours; time-to-revert is not a meaningful reversibility test in 2026
- ~~"User already inclined, red team won't change mind"~~ — anti-pattern. Strong conviction is precisely when challenge has value. Council exists to push back, not defer.

See `prompts/codex_red_team.md` for the full two-subtask execution flow.

---

## Topic-relevant surface

After the user's first message, scan decision.md in both locations for any with status `open`:
- grep user message and current file path for topic matches
- match → surface a hint
- no match → silent

**Strictly forbidden**: auto-listing pending on project entry, over-N-days strong reminders.

---

## Boundary — what this skill does NOT do

- Product form / business model → `/office-hours`
- UI/UX visuals → `/design-*` skills
- Implementation details / single-file review → `/codex consult`
- PR diff-level review → `/review` or `/codex review`
- Security audit → `/security-review` or `/cso`

If ≥ 50% of the user's question falls under one of these, **proactively suggest the right skill** instead of forcing it.

---

## Commands

- `/decisions list` — list all decisions in current project
- `/decisions list global` — list global
- `/decisions show <id>` — show full detail
- `/decisions promote <id>` — project → global (creates symlink)
- `/decisions review <id>` — manually trigger review
