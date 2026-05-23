# OPEN — New decision

Mission: **retrieve the field's consensus** + **cite real sources** on the user's behalf. Responsible = specific source + three confidence tiers (see SKILL.md).

Every Step is bound by SKILL.md's three meta-rules (every-word-earns-its-place / citation freshness / Step 2 default-confirm, no fake questions). **Step 9.5 self-check enforces this.**

---

## Step 0 — Read the personal playbook (includes user decision patterns)

```
Read ~/.claude/projects/<slug>/memory/code_playbook.md
```

Playbook merges two sections:

- **Section A — User decision patterns**: required reading — helps you spot the user's blind spots / preferred angles / advice they reject
- **Section B — Decision technique playbook**: check **whether a same problem-class has been handled before** — if so, cite the prior pattern + outcome as prior

If playbook doesn't exist: skip, must create after this session (Step 11).

## Step 0.5 — Stakes judgment (early-exit gate)

Not every question is worth opening council. 3 quick checks:

1. **Stakes**: if wrong, can it be rolled back in < 1 day?
2. **Domain**: is it a simple bug / implementation detail / one-line choice?
3. **Consensus**: does the domain already have an obvious default (90% pick it)?

At least 2 of 3 "yes" → **abort directly**, do not enter Step 1-11. One-line recommendation + one URL-verified source + tell the user why council didn't convene.

Abort template:

> Not worth opening council: [stakes / domain / consensus — which one matched].
> Recommendation: [one-line].
> Source: [one WebFetch-verified URL].
> If you still want a full council, say "convene."

Abort does not write a decision object, does not update playbook. Only appears in main chat.

**Boundaries that don't abort**:
- Irreversible decisions (even if they look small)
- Multi-person collaboration / team alignment has value
- User explicitly says "open council" / "formal convene"
- Step 0 playbook shows same problem-class had a failed outcome last time — worth running the full process again

## Step 1 — Identify problem phase

Judge which phase the user is in (state the judgment to user):

- **A. Don't know the options**: "I want to do X, how do I approach it?"
  → Main task: **show canonical options map** (Step 3 is the heavyweight)
- **B. Picking among known options**: "How do I pick between A vs B vs C?"
  → Main task: **trade-off + red team** (Step 5 / 6 are heavyweight)
- **C. Already implementing, want a review**: "I built it as X — is that right?"
  → Main task: **compare to canonical + find known traps** (Step 3 + Step 5 are heavyweight)

Different phases shift the weight of subsequent Steps.

## Step 2 — Abstract to problem class (default-confirm, not forced STOP)

Abstract the user's concrete question to a **problem domain**. Wrong abstraction = everything downstream is wrong.

Examples:
- "Should the oral trainer split ASR / eval / dialog?" → "low-latency multimodal pipeline architecture under small-team constraint"
- "Cache keeps OOMing" → "in-process cache eviction policy for read-heavy workload"
- "Microservices or monolith?" → "service decomposition under <N>-engineer team"
- "How to store agent memory?" → "long-term episodic memory for autonomous agents"

### Default-confirm mode

Use default-confirm to reduce interruptions (not forced STOP):

> "I'm abstracting this as: **[one-line problem class]**.
> Key constraints: [1-3, e.g. solo dev / 10 routes / latency < 200ms].
> No objection, I proceed. **If you object, cut me off immediately and say 'no, it should be X'**."

Then **immediately enter Step 3**, don't wait for confirmation. User interrupts → **before retrieving anything in Step 3**, return to Step 2, absorb the correction.

### 3 cases that still require a real STOP

Although default is confirm, these still require a real stop:
1. The question has **multiple independent sub-problems**, no single problem-class can cover them → stop, ask "which one first"
2. The abstraction is **obviously diverging from user's wording** (user said X but abstraction is Y) → stop, confirm
3. Step 0 playbook shows same problem-class had a failed outcome last time → stop, confirm "is this same essential problem as last time"

Otherwise default-confirm proceeds. **No fake questions** — writing "right?" then continuing on your own = dereliction. Either use default-confirm with an explicit "no objection, I proceed," or really stop and wait.

## Step 3 — Canonical patterns retrieval

For this problem class, list options. Each option in 3 parts:

```markdown
### [pattern name]

**Confidence**: 🟢 Canonical / 🟡 Common / 🔴 Claude inference

**Source URL** (🟢 mandatory verifiable URL, 🟡 fill in when possible):
- e.g. `https://www.usenix.org/legacy/publications/library/proceedings/fast03/tech/full_papers/megiddo/megiddo.pdf` (ARC FAST 2003)
- e.g. `https://github.com/ben-manes/caffeine`

**Core idea**: one sentence

**Typical adopters**: [actual systems / companies using it]

**When to use**: in what situations is this the first choice

**Known trap**: when does it backfire / common misuses
```

### URL handling (no verification in this step)

- 🟢 must have URL, but **verification happens in Step 6.5 red team fact-check pass, in batch** — don't add latency here
- 🟢 with no URL → demote to 🟡 or 🔴
- URL unknown but pattern name known → WebSearch for `pattern + keywords`, take a real URL from results (verify still done by red team)

**Minimum output**: 🟢 at least 1 (with URL, red team verifies); 🟡 1-3; 🔴 as needed.

If **no 🟢 verified / 🟡 can be found** → tell the user explicitly:

> "I don't know of any verifiable domain consensus for this problem class. Everything below is 🔴 first-principles reasoning. Recommend calling /codex separately to get an OpenAI perspective for cross-check."

Honesty > faking canonical.

## Step 4 — Master framings (top 2-3 patterns)

For the most promising patterns, find **the master framing most helpful for understanding**:

```markdown
### Best lens for [pattern]

**Master**: Hickey / Pike / Carmack / antirez / Linus / Knuth / Dijkstra / ...

**Their framing**: [a paragraph — how they look at this kind of problem]

**Source**: [specific talk / book / commit msg / mailing list / blog]
- e.g. `Hickey "Simple Made Easy" (Strange Loop 2011)`
- e.g. `antirez blog: "The Bug, the Idea, the Cycle" (2013)`

**Applied to current problem**: how their framing helps you see the current choice clearly
```

**Master must be one you can cite specific public material for**. Hazy impression → skip this pattern's master framing section, **don't force it**.

## Step 5 — Trade-off table

Horizontal comparison of canonical patterns:

| Pattern | When to use | When not | Implementation complexity | Known trap | Evolution debt |
|---|---|---|---|---|---|

Fill with **real data**, not vague adjectives ("very good" / "slightly worse" don't count).

## Step 6 — Anti-council (fixed 4 questions)

Each question must have a concrete answer ("no obvious issues" also explicit):

1. Is my recommended canonical outdated (any newer consensus in the field in the last 5 years)?
2. Am I confusing "what I'm familiar with" with "canonical"?
3. Under the user's resources / team size / time constraints, is canonical actually unsuitable?
4. Is the user's scenario special enough that no canonical applies, and they should go against the mainstream?

## Step 6.5 — Red team (fact-check + Codex cross-source)

Run the full `prompts/codex_red_team.md` flow (two subtasks).

### Subtask A — Fact-check (default forced run, can't be disabled)

- grep all 🟢 marks from this session's Steps 3-5
- Extract each 🟢's URL
- Batch WebFetch verify (only status 200 passes)
- 4xx / timeout / dead → demote to 🟡 or delete, mark "fact-check failed: <url>" in council-log.md

Main flow doesn't inline-verify for smoothness; fact-check centralized here as forcing function.

### Subtask B — Codex cross-source (toggle-controlled)

Read `decision.md` frontmatter `red_team.codex`:
- **unset / false**: skip (SKILL.md default is true, this should be rare)
- **true**: invoke codex to attack main recommendation / omissions / canonical claims (see codex_red_team.md Section B)

**Strictly forbidden to fake**: when toggle is false, don't have Claude simulate codex output. Either invoke the real model, or it doesn't appear.

## Step 7 — Chairman's ruling (detailed actionable plan, not a summary)

The ruling isn't "I recommend A because it's good." It's **a detailed plan the user can directly follow**.

### Common requirements (any decision type)

1. **Confidence emoji**: 🟢 / 🟡 / 🔴. 🔴 explicitly says "based on first-principles reasoning, not domain canonical."
2. **No false equivalence**: explicit recommendation + explicit non-recommendation, each with 1 concrete reason. "Both sides have merit" is dereliction.
3. **Failure signal**: observable fact that tells you this plan is wrong. Not "if performance is bad" hand-waving.

### Decision-type identification → pick template

- **Implementation decision** (writing code / refactor / pipeline / handler / state machine) → **Template A**
- **Selection decision** (database / queue / cache / toolchain / framework / architecture pattern choice) → **Template B**

#### Template A — Implementation decision

Include:
- **File-level**: which files, LOC budget each (current X lines → target Y lines)
- **Signature-level**: key function / type / handler signatures (pseudocode or type hints)
- **Data-structure-level**: what the core dict / list / dataclass looks like
- **Key code skeleton**: 5-15 lines of core dispatch / loop / decorator (not full implementation)
- **First commit**: which file first, which lines changed, what case verifies it

Example (HTTP routing refactor):

> 🟡 Recommended: HTTP uses `dict[(method, path), Callable]` routing table + handler signature `(request: dict) → (status: int, body: dict)` + middleware `for fn in middlewares: request = fn(request)`.
>
> **Files**: `http_router.py` < 180 lines (currently ~280); `ws_router.py` < 200 lines (currently ~280).
> **Core data structure**:
> ```python
> ROUTES: dict[tuple[str, str], Handler] = {
>     ("POST", "/api/echo"): handle_echo,
> }
> MIDDLEWARES: list[Callable[[dict], dict]] = [auth, parse_json]
> ```
> **Dispatch skeleton**:
> ```python
> def dispatch(scope, body):
>     req = {"path": scope["path"], "method": scope["method"], "body": body}
>     for mw in MIDDLEWARES: req = mw(req)
>     handler = ROUTES.get((req["method"], req["path"]))
>     if not handler: return 404, {}
>     return handler(req)
> ```
> **First commit**: rewrite `http_router.py` to the skeleton above, pass existing 6-endpoint e2e tests → commit.
> **Failure signal**: `/agent/{id}` path params make `dict` lookup fail → fall back to regex or trie.
> **Not recommended**: full sans-IO — 10 routes don't justify it; onion middleware framework — 3 cross-cutting concerns, a for-loop is clearer.

#### Template B — Selection decision

**Don't force a code skeleton**. Include:

- **Why this not that**: 1-3 sentences for each alternative, why not — cite evidence / data / constraints / canonical pattern
- **First validation commit**: the first concrete action that verifies "this choice is right" (observable, 1-2 weeks)
- **Migration path** (if migrating): minimal step sequence from current state → target
- **Lock-in assessment**: effort to roll back from Y to X

Example (PostgreSQL vs SQLite):

> 🟢 Pick SQLite.
> **Why not Postgres**: currently < 100 write req/s, single-machine deployment, no distributed needs — Postgres's MVCC / replication / connection pool all unused, cost without benefit.
> **First validation commit**: existing schema on SQLite + load test 100 req/s × 24h, look at WAL fsync p99.
> **Lock-in**: rolling back to Postgres ≈ 1 week (schema mostly compatible, change connection string + a few SQLite-specific usages).
> **Fail signal**: load test p99 > 200ms or db lock contention > 5% of requests → migrate to Postgres immediately.
> **Not recommended DuckDB**: analytical not OLTP, write profile doesn't match.

### Language style (common)

- **Plain English**: explain terms in one sentence on first appearance. "sans-IO" → "handler doesn't read sockets directly; I/O goes through args and return values."
- **Conservative pitch**: facts not hype.
  - ✅ "Currently 561 lines → target < 400 lines; only two new concepts: route table + handler dict."
  - ❌ "Revolutionary simplification" / "likely easier to maintain"
- **No painted promises**: don't say "future-extensible" or "seamlessly integrates with X" — speculation isn't fact.

### Counter-example (filler the user has flagged)

> "Recommended H1 + H2 lite + H3 lite because they balance simplicity and functionality. Next step: start implementing."

No decision type can be written this way. Dereliction.

## Step 8 — Reading list

At least 3 pieces **the user should actually read**:
- canonical pattern's original paper / book chapter
- 1-2 real-world post-mortems
- a master talk (if applicable)

**Don't list hallucinated URLs**. If you're unsure of a URL, write:

> Search: `["Caffeine cache design" by Ben Manes]`

Better to let the user google than give a fake link.

## Step 9 — Learning notes

Extract **generalizable lessons for the user** (not limited to this specific question):

- Next time you face the same problem class, what pattern should you think of first
- What cognitive blind spot did this council expose
- "Must-know but didn't know" checklist for this domain

This part **appends to code_playbook.md** (not overwrite). This is the skill's real compounding source.

## Step 9.5 — Pre-output self-check (mandatory forcing function)

After Steps 7-9 are written, before Step 10 writes files, **self-grade each item** in the checklist. Any failure → rewrite that part, re-self-check, until all pass.

### Every-word-earns-its-place (8 items)

- Each paragraph asked "if I delete this, does the decision change?" — deleted everything that doesn't change it?
- No framing sentences ("below in three parts" / "first the conclusion" / "in summary" / "specifically for your situation")?
- No symmetric padding (if a candidate has 1 sentence's worth, only 1 sentence, no filler)?
- No hedging clichés ("depends on" / "both sides have merit" / "possibly")?
- Each canonical pattern description ≤ 3 lines?
- Trade-off cells all decision-bearing facts (numbers / yes-no / one-line), no "performs better" filler?
- Chairman's ruling = 1 paragraph + 1 confidence tier + 1 specific next step, no "weighing tradeoffs..." opener?
- No manufactured changes (review-type questions only touched when there's a real issue)?

### Anti-hallucination (6 items)

- Every 🟢 has a URL (Step 6.5 red team fact-checks; here just verify the URL field is filled)?
- Master framings all have specific source + URL/ISBN?
- Didn't say "X would definitely think this" (X = any real person)?
- No invented quotes / no invented URLs?
- Hazy-memory people / patterns all demoted to 🔴?
- No 🔴 dressed as 🟢?

### Citation freshness (2 items)

- Citations all from production within the last 5 years, OR genuinely timeless concepts?
- Hazy old papers / those where you can't say "still in production this year" — demoted or deleted?

### Flow (2 items)

- STOP points are real stops (Step 2 problem-class confirmed by user / default-confirm and user had no objection)?
- Chairman's ruling isn't false equivalence (explicit recommendation + explicit non-recommendation + concrete reasons)?

**Concrete cases of self-check failure**:

- Step 7 chairman's ruling starts with "weighing the tradeoffs, I recommend A because it's good" → rewrite as "🟡 Recommended [pattern] + [file-level / data structure / skeleton]"
- 🟢 ARC paper has no URL → fill URL now; verify by Step 6.5 red team fact-check
- Trade-off table has a "performs better" cell → change to "FAST 2003 benchmark shows hit rate +X%" or delete the column
- Step 8 reading list has a hazy old paper URL → delete, or WebSearch for the real URL first

All pass → proceed to Step 10. Any fail → rewrite until all pass.

## Step 10 — Create decision object

Ask the user:
1. Decision title (default generated from problem class as slug)
2. visibility: project (default) or global
3. red_team.codex (recommend default true unless stakes are tiny — tell user cost is ~$0.01-0.05)
4. Key topics (you propose, user edits)
5. Review point: post-implementation look-back (not a 7-day timer) — ask user when they expect to have implementation data

### Write: 2 files (agent-first design)

```
<project-or-global>/.claude/decisions/YYYY-MM-DD_<slug>/
  decision.md      ~50-80 lines: frontmatter + problem + recommendation + non-recommendation + kill criteria
  council-log.md   Only if there's audit value: anti-council 4 questions substance + codex distill + chairman's path
```

`outcome.md` is **not pre-created**, written only after implementation.
`learning_notes` is **not written to the decision directory**, appended to `code_playbook.md` (Step 11).

### decision.md template (strict adherence)

```markdown
---
id: YYYY-MM-DD_<slug>
title: <human-readable>
status: open
problem_class: <one sentence>
topics: [keywords]
red_team:
  codex: true | false
---

## Problem

- Scenario: <1-2 lines>
- Key constraints: <3-5 bullets, one line each>

## Recommendation

🟢/🟡/🔴 [pattern name]

**Files**: <specific path + LOC budget>
**Core data structure**:
\`\`\`
<5-10 lines type/data structure>
\`\`\`
**Dispatch / entry skeleton**:
\`\`\`
<5-15 lines core code skeleton>
\`\`\`
**First step**: <down to commit level>
**Failure signal**: <observable, not "if bad">

(Optional) authoritative reference:
- <1-2 verified URLs>

## Not recommended

- <pattern X>: <1-line reason — including codex red team nuance, e.g. "Claude mislabeled as canonical; actually not ASGI consensus">
- <pattern Y>: <1 line>

## Kill criteria

- <observable condition 1>
- <observable condition 2>
```

### council-log.md template (only if there's content)

If red team all OK + codex agrees + first-shot final → 5-line stub:

```markdown
problem class (user-confirmed): <>
anti-council 4 questions: no new findings
codex red team: agreed, sources verified
ruling: final on first pass
```

If there's substance (red team found issues / codex changed the ruling / multi-round iteration) → detailed:

```markdown
## problem class (user-confirmed)

<>

## Anti-council 4 questions

Only list questions with **substantive findings**. Skip ones with no findings — don't write "no issues."

## codex red team

- **disagreed**: <codex pushback on Claude's canonical claim>
- **added**: <canonical options Claude missed>
- **verified**: <URLs verified to exist>

(Don't verbatim the full codex output — distill.)

## Chairman's path

Only if the ruling was revised after codex — write early ruling + current ruling + reason for revision.
Otherwise skip this section.
```

Output `/schedule` command for the user to enable cron review (**don't run it for them** — remote agent is a cost behavior).

## Step 11 — Update code_playbook (both sections may need writing)

**If playbook doesn't exist, create it** (with Section A + Section B framework, see SKILL.md "Learning accumulation").

### Section B always appended

```markdown
## YYYY-MM-DD — [problem class]

**Decision ID**: YYYY-MM-DD_<slug>
**Chairman's recommendation**: [pattern] (confidence 🟢/🟡/🔴)
**Key trade-off**: [one-line core]
**Outcome**: pending (filled in after implementation)
**Next time same problem class, think of first**: [pattern name]
```

### Section A updated as needed

If this session revealed **new preferences / decision patterns / recurring blind spots** of the user → update Section A:

- New decision added to decision list
- New effective angle (e.g. steelman pattern) → "Angles effective with this user"
- Direction user explicitly rejected → "Recommendations this user never accepts"
- Decision pattern shorthand → add date + 1-line note

Nothing new, all smooth → don't write.

---

## Forbidden

All "forbidden" items are folded into Step 9.5 self-check + SKILL.md anti-hallucination hard rules. Self-grade against the checklist before output; any violation → rewrite.

Only one not covered by self-check: **out-of-bounds product / business / UI judgments** — proactively suggest routing to office-hours / design-* / etc. (see SKILL.md "Boundary" section).
