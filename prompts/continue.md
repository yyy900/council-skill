# CONTINUE — Continue an existing decision

## Triggers

- User message or current file path matches the `topics` of a decision.md whose status is `open` / `reviewing`
- Or user explicitly says "continue the X decision" / "reopen the council"

## Forbidden

- Don't reopen from scratch, ignoring existing council-log.md
- Don't produce new recommendations that contradict prior conclusions without explanation
- Don't silently mutate decision.md status

## Step 1 — Read the decision object (2 files post-simplification)

```
Read decision.md
Read council-log.md
Read outcome.md (if exists)
Read review-pending.md (if exists — written by remote review agent)
```

Load into working memory. When citing, **state the source explicitly** ("in the first council session, codex red-team point 4 said ...").

### Outcome write-back forcing function (mandatory check)

After reading, immediately judge outcome state (use `stat` for mtime):

| State | Trigger | CONTINUE response must do |
|---|---|---|
| ✅ outcome fresh | outcome.md exists and mtime < 90 days | Cite outcome data |
| ⏰ outcome missing (short) | status=done, outcome.md absent, decision.md mtime < 30 days | Gentle reminder: "Did the last decision finish running? Want to fill in outcome?" |
| ⚠️ outcome missing (medium) | status=done, outcome.md absent, decision.md mtime 30-90 days | Explicit prompt: "Outcome wasn't written. Has data come out and just not been recorded?" |
| 🚨 outcome missing (long) | status=done, outcome.md absent, decision.md mtime > 90 days | Open the CONTINUE response with `🚨 outcome not filled in 90+ days; decision state may be stale` |
| ⏰ outcome aged | outcome.md exists but mtime > 180 days | Prompt: "Worth reviewing — verify the outcome still holds" |

The point of the forcing function: shift outcome write-back from "user-initiated" to "system-prompted." code_playbook compounding only works if outcome actually gets written back.

## Step 2 — Identify the type of this session

Ask or self-identify:

- **Drilling into an existing conclusion**: user wants to discuss one angle in more detail → no new session, just answer specifically
- **New info that updates assumptions**: user says "I talked with my co-founder" / "the benchmark ran" → update assumptions.md state
- **Kill criteria triggered**: check if user's description matches K1-K6 → if matched, enter reversed state machine, reopen council
- **Genuine new angle**: user says "I didn't think about X" → append a new section in council-log.md, dated

## Step 3 — Update files, don't rewrite

All modifications are **append** or **explicit update**:

- council-log.md: append `## YYYY-MM-DD Session N` section
- assumptions.md: update assumption state (pending → verified / disproved), annotate "updated YYYY-MM-DD"
- decision.md frontmatter: update status if needed

## Step 3.5 — Codex red team (conditionally triggered)

If `decision.md` frontmatter has `red_team.codex == true` **and** any of the following:

- This session triggered a kill criterion (any of K1-K6)
- A key assumption had a major state change (any of A1-A3 went from "pending" to "disproved")
- Chairman's ruling direction got modified

→ Run `prompts/codex_red_team.md`, append output to council-log.md.

If conditions aren't met, don't rerun codex (avoid wasted cost).

## Step 4 — User profile sync (write into code_playbook Section A)

`user_decision_profile.md` is now merged into `code_playbook.md` Section A.

If this session reveals **new preferences / decision patterns / recurring blind spots** of the user → append to `~/.claude/projects/<slug>/memory/code_playbook.md` Section A (schema in SKILL.md "Learning accumulation").

If nothing new — don't write.

## Step 5 — Surface message format to user

```
📋 Matched decision: <title>
   Status: <status>
   Last session: <date>
   Unverified assumptions: <count>
   Relevant? (y / not relevant / skip for now)
```

If user says "not relevant," record to decision.md frontmatter `false_match_count`. After 3+ consecutive false matches, lower this decision's topic weight (remove the most easily-mismatched keywords).
