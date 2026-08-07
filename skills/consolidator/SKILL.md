---
name: consolidator
description: Before scoping or starting new work on a large, long-running, or multi-repo project, builds a ground-truth ledger of what already exists — code, planning docs, status files, sibling repos, decision records — so work isn't duplicated, redone, or planned against a stale claim. Use before any "what's left" assessment, before creating new backlog items, or whenever a task references something ("the eval," "the report," "the tool") that might already exist elsewhere.
metadata:
  origin: ECC
---

# Consolidator

Stops a specific, expensive failure mode: confidently telling someone work is
missing, or redoing it, when it already exists somewhere you didn't look. On a
project that spans months and multiple repos, the person asking "what's left"
usually knows less about the current state than the last five sessions
combined — which means the honest answer requires reading, not remembering.

This skill exists because of a real miss: a session told its owner an AI
safety eval "doesn't exist yet," quoting a stale code comment, while the
measured results sat two lines above it in the same file and in a status
report in a sibling repo neither had opened. The grep that "confirmed" it was
missing only checked exact keyword phrasing. Grep found nothing wrong;
reading would have.

## When to Use

- Someone asks "what's done vs what's left" on a project with any real history
- About to create a new backlog item, task, or plan for a "missing" feature
- A task or conversation references something — a report, a tool, a decision,
  a prior attempt — that might already exist in this repo or a sibling one
- Multiple repos are in play (a research/builder/UI split, a monorepo with
  satellite repos, a "planner" coordination pattern) and no single repo has
  the full picture
- Resuming work after a long gap or a context compaction, before trusting a
  prior summary's account of "what's done"

## How It Works

### Phase 1 — Map the territory

List every repo/workspace actually in play, not just the one open right now.
Check for a coordination layer: a `docs/<project>/` folder mirrored across
repos, `STATUS.md` files, an `outbox/` with `reports/`, `recommendations/`,
`decisions/`. If one exists, it is the highest-signal place to look —
someone already wrote down what happened, in a format meant to be read.

### Phase 2 — Read the load-bearing docs, don't grep them

Open the actual planning documents in full: the standing document, the
sequencing plan, work orders, dated reports. A narrow keyword search only
finds your exact phrasing — if the thing you're looking for was described in
different words, grep reports it absent and you believe it. Reserve grep for
*finding where to look*, never for concluding something isn't there. If a
grep comes back empty and the stakes matter, that's the cue to read the
file, not the answer.

### Phase 3 — Cross-check code claims against evidence

Treat a code comment, a TODO, or a doc string that asserts a state
("doesn't exist yet," "not implemented," "pending") as a hypothesis, not a
fact — especially if it's inside a large or old file. Verify it against:

- other content in the *same file* (a routing table, a rationale string, a
  constant) that might already cite real measured results
- sibling repos' status/outbox/decision files
- git log / file dates — a comment can go stale the moment the thing it
  describes ships, if nobody remembers to update the comment
- whether tests or a build artifact already exercise the "missing" thing

### Phase 4 — Build the ledger

For every item under discussion, assign one status, with the file/line or
report that justifies it:

| Status | Means |
|---|---|
| **DONE** | Built, and verified — tests pass, or you ran it and watched it work |
| **BUILT, NOT VERIFIED** | Code/artifact exists; nobody has confirmed it runs correctly |
| **PARTIALLY BUILT** | Some of it exists; name exactly what's missing |
| **PLANNED, NOT STARTED** | Written down as intended; no code yet |
| **STALE CLAIM** | A comment or doc says one thing; evidence elsewhere contradicts it — flag both sides |

### Phase 5 — Report the ledger before recommending anything

Never propose new work on an item without stating its ledger status and the
evidence for it first. If the status is uncertain, say so plainly — "not yet
verifiable" is a legitimate answer and a better one than a guess dressed as
fact.

## Examples

**Multi-repo status check**
```
User: What's left on the diagnosis reasoning layer?
Skill: Maps 4 connected repos → finds docs/meridian/outbox/STATUS.md in two of
       them → reads (not greps) the R1 eval report → finds it already shipped
       and ran against three models, contradicting a stale "doesn't exist yet"
       comment in the code being read → reports BUILT AND VERIFIED, with the
       real numbers, instead of proposing to rebuild it.
```

**Before opening a new backlog item**
```
User: We need a tool that scores diagnosis completeness.
Skill: Searches sibling repos' outbox/reports for "completeness" →
       finds a delivered eval corpus already doing exactly this →
       reports it as PARTIALLY BUILT (corpus + scorer exist, live baseline
       pending an API key) instead of adding a duplicate task.
```

**Resuming after a gap**
```
User: Pick up where we left off.
Skill: Doesn't trust the last session's summary at face value — re-reads the
       actual STATUS.md/outbox files changed since, flags anywhere the
       summary and the real state have drifted, then resumes from the
       verified state.
```

## Best Practices

- **STATUS/outbox/decision files first, source code second** — if a project
  has a written coordination layer, someone already did the reading; use it
  before re-deriving it from scratch
- **A grep that finds nothing proves nothing** — it proves your search terms
  didn't match; keep it to locating candidates, never to ruling something out
- **Sibling repos are not optional reading** — in a split architecture
  (research / build / UI / plan), the answer to "is X done" is almost never
  fully contained in the repo you happen to have open
- **Every ledger entry needs a citation** — a file path, a line, a report
  date; an assessment with no evidence attached is the same failure mode this
  skill exists to prevent, just one level up
- **Stale claims get surfaced, not silently corrected** — say what the
  comment claimed, what the evidence actually shows, and that the two
  disagree; that visibility is what stops the next person from citing the
  same stale line
