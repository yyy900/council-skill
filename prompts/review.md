# REVIEW — Remote agent look-back prompt

## How it's invoked

`/schedule` cron-fires a new session on the specified date. The new session has **no conversation context**, only:

- Arguments: `<project-root>` `<decision-id>`
- Filesystem access

So this prompt must be self-contained.

## Step 1 — Locate the decision object

```bash
DECISION_DIR="<project-root>/.claude/decisions/<decision-id>"
```

If it doesn't exist, check global:

```bash
DECISION_DIR="$HOME/.claude/decisions/<decision-id>"
```

Still not found → this routine is dead (user may have deleted the decision directory). Log error, create no files, exit.

## Step 2 — Read the decision object (2 files post-simplification)

```
Read decision.md
Read council-log.md
Read outcome.md (if exists)
```

## Step 3 — Write review-pending.md

Write to `$DECISION_DIR/review-pending.md`:

```markdown
# Review N — Remote agent preliminary analysis

**Generated**: <ISO timestamp>
**Review point**: <which review_date this is>
**Decision status**: <current status>

## Assumption state inventory

[List every assumption from assumptions.md]
- A1 <content>: state pending your update (agent can't verify things like "product form alignment")
- A2 <content>: agent's proposed check = ...
- ...

## Kill criteria state inventory

[List every kill criterion from kill-criteria.md]
- K1 <content>: agent can't judge directly, please answer yes/no/unknown
- ...

## Agent's judgments

For anything **agent can judge independently** (e.g. codebase commits, content already written in outcome.md, filesystem state), agent gives judgment here.

If completely dependent on human input (most assumptions and kill criteria are), say clearly "agent can't judge, waiting on user".

**Forbidden**: hallucinated judgments. Don't assume benchmarks ran, don't assume co-founder aligned. Only judge facts readable directly from the filesystem.

## Questions for the user

[List specific questions the user must answer, each yes/no or specific data]

How to respond: edit this file directly, answer the questions, then tell Claude "review done" — Claude will integrate answers into council-log.md and update decision.md status.

## Status machine suggestion

Based on this inventory, agent suggests decision.md status be set to:
- `reviewing` — currently waiting for user to fill this file
- or keep current status if user hasn't responded yet
```

## Step 4 — What NOT to do

- Don't modify decision.md / assumptions.md / kill-criteria.md (write permissions belong to the user's main chat)
- Don't open a new council round (no conversation context — new angles would just be generic advice)
- Don't PushNotification to interrupt the user (let surface happen when user organically approaches the topic)

## Step 5 — Optional: push notification

If user enabled push in `~/.claude/council-config.yml` (default off), send one short notification:

```
📋 Decision review ready: <title>. Will surface next time you enter the project.
```

Otherwise drop silently, wait for user to organically approach the topic.
