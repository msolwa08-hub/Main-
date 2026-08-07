---
name: flow
description: A full project lifecycle for building a large or new app end to end — Brainstorm, a Research & Challenge loop that saturates a topic before trusting it, a deep Plan pass (backend, frontend/UI, assets, with an explicit open-endedness-via-LLM check and a unique-identity check), a Build pass that's allowed to loop back to earlier modes as new things are discovered, a Final Research Pass, and a Test & Polish gate that separates deep adversarial testing from a non-negotiable zero-obvious-defect bar. Scales from a single session to a multi-repo split (dedicated research/UI repos) for genuinely large projects. Use when starting or continuing a large feature, a whole new project, or a long-running build — not for a small fix or a single-file change.
metadata:
  origin: ECC
---

# Flow

A full lifecycle, not a checklist — modes chain into each other's output, and
the chain is allowed to loop backward when a later mode discovers something
an earlier one got wrong. Nothing here replaces `planner`, `tdd-guide`,
`e2e-runner`, `code-reviewer`, `security-reviewer`, or a project's own domain
skills — Flow is the sequence that calls them at the right moment, so a large
build never skips straight from an idea to code with no research pass, ships
with no adversarial pass, or reads as a template with someone else's identity
painted on.

This isn't hypothetical. Every mode below was exercised for real on Meridian
in the same working session this skill was written in — the examples
throughout are what actually happened, not invented illustrations.

## The engine is the product

Say this plainly, because a UI pass makes it easy to lose sight of: **the
algorithm, backed by real research, is usually the actual product — not a
subsystem sitting behind the interface.** TikTok's feed, Instagram's
ranking, any app people describe as "it just gets it" — the thing doing the
work is the algorithm underneath, tuned on real data, not the screen wrapped
around it. A polished interface in front of a shallow or unresearched engine
is a polished wrapper around nothing. Every mode below exists in service of
that engine: Brainstorm and Plan decide its shape, Research & Challenge is
what earns it the right to be trusted, Build wires it to something people
touch, and Test & Polish proves it holds.

This is why **Mode 2 (Research & Challenge) is not a side quest that happens
before the "real" work — the research is what makes the engine worth
having.** Meridian's engine is a deterministic log-odds scoring system; what
makes it trustworthy isn't the math alone, it's that every weight, every
likelihood ratio, every must-not-miss diagnosis in it traces to a cited
source, calibrated to the population it actually serves rather than a
generic one. An engine with confident-looking numbers and no research behind
them is worse than one that says "I don't know" — that's the same failure
Check A (below) exists to catch, one level deeper: not just "is this
open-ended," but "is what's driving it actually earned." **The algorithm
does the deciding; the LLM keeps what it can consider open** — that
combination, not either half alone, is the thing worth building.

Engine quality and the research backing it are load-bearing for the life of
the project, not a phase that ends once Mode 2 closes. If new research
surfaces something that should change the engine mid-build, that is exactly
what Mode 4's loop-back exists for — the engine does not get to go stale
while everything built around it keeps moving.

**Before Brainstorm opens**, run `consolidator` if the project has any real
history — brainstorming new ideas for something partially built already is
how work gets duplicated. Know what exists before inventing what's next.

## When to Use

- Starting or continuing a large feature, a whole new project, or a
  long-running multi-session build — not a small fix or single-file change
- The user describes an outcome without having already decided the
  structure, the plan, or the UI
- Resuming a project after a gap, where "what's actually done" needs
  re-establishing before adding more
- Explicitly invoked by name, or by mode: "brainstorm mode," "plan mode,"
  "run Flow on this"

## How It Works

### Mode 1 — Brainstorm

**Goal: flesh out the full idea space and app structure, before any
implementation decision is made.**

Not "list some ideas." Pull in every skill relevant to the domain — a
healthcare surface pulls the `healthcare-*` skills, a UI-heavy feature pulls
`frontend-design-direction` and `design-references`, a data-heavy one pulls
the relevant backend/database skills — and use each one's lens to surface
ideas, requirements, and structural detail a single generic pass would miss.
Use `SearchSkills`/`ListSkills` to find what's relevant rather than guessing
from memory; skills are added faster than anyone's memory of the catalogue
stays current.

Output: an idea/structure brief — every feature, edge case, and structural
piece worth considering, organized by area. Nothing here commits to what
ships; it establishes what *could*.

### Mode 2 — Research & Challenge

**Goal: don't trust Mode 1's brief until it survives real scrutiny. Research
until the topic stops giving up new information, not until it feels done.**

This is the mode most builds skip, and skipping it is how a plan ships
confidently wrong. Three passes, on every non-trivial claim or area in the
brief:

1. **Full research** — not a skim. Read primary sources, not just enough to
   pattern-match a keyword. A narrow search that finds nothing is not
   evidence of absence (see `consolidator`'s whole premise) — go deep enough
   that the *absence* of something is itself a finding, not a blind spot.
2. **Challenge it like an expert would** — for every non-trivial claim in
   the brief, actively try to find where a domain expert would push back.
   What's the strongest objection? What edge case breaks the assumption?
   Where would this be *wrong* in the specific population/context/scale this
   project actually targets, not the general case? (Meridian's research
   corpus does exactly this: it's calibrated to a high-HIV/TB South African
   district-hospital population specifically *because* a generic corpus
   would pass its own bar while being unsafe for the population that
   actually matters here.)
3. **Test for saturation** — keep researching until a round turns up nothing
   new, not once, but for real: if the last research pass still surfaced
   new material, the topic isn't saturated yet, keep going. This is a
   loop-until-dry pattern, not a fixed number of passes.

**Output: a revision, not a rewrite.** Come back to the Mode 1 brief and say
plainly what changes, what gets added, what gets improved, and what gets cut
— each with the research finding behind it. A research pass that confirms
everything unchanged is a red flag, not a clean result; it usually means the
challenge step wasn't adversarial enough.

**This loops.** If the revision is large enough to open new structural
questions, go back to Mode 1 with the new brief. Don't force a single pass
to be sufficient when it isn't.

### Mode 3 — Plan

**Goal: the concrete, buildable plan for the researched brief — what the
tool needs to know, split across backend, frontend/UI, and assets — plus two
checks that decide the character of the whole build.**

Three tracks, each producing its own section of one plan:

- **Backend** — data model, business logic, API surface, what's computed vs.
  stored. Use `planner`/`architect`.
- **Frontend / UI** — the screens, the states each screen can be in, the
  interaction model. Use `frontend-design-direction`, `design-references`,
  `code-architect`, or the project's own design agent if it has one.
- **Assets** — what needs generating or sourcing, and which tool produces
  each (Gemini for imagery, a design system, licensed sources). Name the
  generator and the fallback per asset up front, not mid-build.

**Check A — open-endedness.** For every subsystem that resembles a closed
menu, a fixed list, or a finite set of hardcoded cases, ask explicitly:
*should this be open, with an algorithm scoring whatever comes in and an LLM
widening what's considered, instead of a gate that only recognizes what's on
the list?* Deterministic algorithms are still where the actual scoring and
decisions should live — the LLM's job is to keep the input space open, not
to replace the algorithm. This is not a style preference; it was a LAW-level
fix on Meridian: `routeComplaint` originally matched a complaint against a
fixed presentation list and silently fell back to a generic board on no
match. The fix wasn't "add more presentations," it was making the fallback
honest and giving the system a documented path to reason beyond the fixed
list. Find this pattern before it ships, not after a user hits the edge of
the menu. This check is where "the engine is the product" (above) becomes a
concrete decision rather than a stated value — the algorithm-plus-LLM
combination this check protects is usually the actual thing being built.

**Check B — identity.** A plan that says "make it like similar apps in this
field" produces something that falls short of the best thing in that field,
by construction — the field's mediocre middle is easier to copy than its
best outlier, and copying the field means inheriting its ceiling. Establish
a reference class and an explicit point of difference instead — what makes
this feel like *its own* thing the way Uber, Apple, or any product with a
real signature does, not a competent instance of its category. Use
`design-references`, `taste-skill`, or a Claudia-style design agent's own
discipline: design blind to competitors first, name the reference class in
words (not adjectives — "clean," "modern," and "premium" describe a feeling,
not a decision, and are explicitly banned in rigorous design processes for
exactly that reason), and require the plan to name what's being deliberately
NOT done, not just what's included. This is the same failure named directly
mid-build on Meridian: a UI that read as generic, with art that didn't fit
its own frame, was called out as not having "its own identity" — and the fix
was a reference-class and design-system pass, not a incremental polish pass.

**Check C — scale.** Decide, once, whether this project fits in one repo and
one working context, or whether it needs to split — a dedicated research
repo, a dedicated UI/design repo, each with its own agent and its own
coordination layer. This is not a hypothetical option; it is a proven,
working pattern (four repos: a builder, a research repo, a UI-design repo,
and a planner that writes work orders to each and reads a `STATUS.md` +
`outbox/` from each). Split when the research or design surface is genuinely
too large to hold in one context alongside the build work; stay in one repo
and one session when it isn't. Don't split by default and don't refuse to
split by default — decide from the actual size of the work.

Output: one plan with all three tracks plus the results of Checks A, B, and
C. Enough detail that Mode 4 doesn't stall on an unanswered question.

### Mode 4 — Build

**Goal: build backend, frontend, and assets together, as one integrating
system — not backend first with UI bolted on after.**

Combine as you go: wire a piece of backend to the screen that uses it and
the asset it displays as each is ready, rather than building three
disconnected stacks and hoping they meet cleanly at the end. Seams between
pieces built separately are where real bugs concentrate — catching a seam
break while both sides are still fresh is cheap; catching it after both are
"done" is not.

**This mode is explicitly allowed to loop backward.** The plan from Mode 3
is a plan, not a contract with reality — building always turns up things
research didn't anticipate: a new requirement, a piece of research that
turns out to matter, an idea that only makes sense once part of the system
exists. When that happens, the right move is back to Mode 1 or Mode 2 for
that piece, not a workaround bolted onto Mode 4. An "exhaustive" brief from
Mode 1 is allowed to stop being exhaustive as the build teaches you more —
that's not a Flow failure, that's Flow working. What *is* a failure is
treating Mode 4 as strictly linear and letting a real gap get built around
instead of fed back.

### Mode 5 — Final Research Pass

**Goal: catch drift between what was researched at the start and what's true
by the time the build is mostly real.**

A build that took real time can outlive its own research: content researched
early can be superseded, an assumption from Mode 2 can turn out to have
changed, and the build itself will have surfaced questions Mode 2 never
asked because the system didn't exist yet to ask them of. Re-run the same
discipline as Mode 2 — full research, adversarial challenge, saturation —
scoped to what actually changed or what the build revealed, not the whole
brief from scratch. Feed anything that moves back into the build.

### Mode 6 — Test & Polish

**Goal: prove the system holds up, and separately, prove nobody would look
at it and immediately spot something wrong.**

These are two different bars and both are unconditional — passing one does
not substitute for the other.

**The deep bar:**
1. **End-to-end test** — the real user journey, start to finish, on the
   actual integrated system. Use `e2e-runner`.
2. **Stress test** — load, volume, or scale conditions past normal use.
3. **Critical / adversarial test** — actively try to break it: bad input,
   edge cases, security-sensitive paths (`security-reviewer`), and the
   specific failure modes that matter most for this domain, not generic
   ones.
4. **Code review** — `code-reviewer` over the full diff.

**The baseline bar — zero tolerance, no severity scale:**
Basic, obvious mistakes are not a low-severity bug category; a shipped
product with visible spacing errors, misaligned elements, typos, or
copy that reads like a placeholder has failed regardless of how well the
deep bar scored. This bar is checked by actually looking, not by reasoning
about the code: render the real UI, screenshot it, and read what's on
screen — the exact method used to catch Meridian's own age-field alignment
bug and a bisected sticky-footer bug this session, neither of which any
amount of code review alone would have caught. If there's no way to render
it, say so plainly rather than claim the pass happened.

**Output: a statistics report** — test coverage, pass/fail counts by
category, what was stress-tested and at what scale, what's flagged NOT YET
VERIFIABLE rather than claimed, and confirmation the baseline bar was
checked by rendering, not inference. This report is what closes Flow — a
final pass with no numbers attached is an opinion, not a result.

## Examples

**Continuing a large, long-running project (Meridian, as it actually ran)**
```
consolidator first — before assessing "what's left," re-reads sibling repos'
STATUS.md/outbox files rather than trusting a stale code comment; catches a
safety eval that had already shipped and been measured, contradicting what
the code comment claimed.
Mode 1 (Brainstorm) — a new surface (e.g. the problem-solving chat tool) gets
its full idea space fleshed out: two modes, safety rules, what makes it beat
a bare chatbot.
Mode 2 (Research & Challenge) — the spec is checked against what's actually
built vs. only planned; a gap between "routing exists" and "no UI exists at
all" surfaces here, not after work starts on the wrong assumption.
Mode 3 (Plan) — Check A catches a closed-menu presentation router as the
LAW-level "open engine" bug; Check B is the literal reason a UI pass got
redirected away from "similar to other apps in this space" toward its own
visual identity; Check C matches the real four-repo split already in use
(builder, research, UI, planner) for this exact project.
Mode 4 (Build) — a UI honesty-layer fix surfaces mid-build that a whole-board
desaturation state also needed extending to a rail and a glance view not
originally scoped — fed back rather than left half-done.
Mode 6 (Test & Polish) — a rail flag rendering in an unreadable color, and a
production bundler bug that silently corrupted every single-file export,
were both caught by actually rendering and looking, not by code review alone.
```

**A brand-new project, single repo**
```
No consolidator needed — nothing exists yet.
Mode 1 → Mode 2 (until the topic stops giving up new material) → Mode 3
(all three tracks + Checks A/B/C — Check C concludes one repo is enough) →
Mode 4 (built as one integrating system, looping back once when the build
reveals a gap) → Mode 5 (a scoped re-check, not a full re-research) →
Mode 6 (deep bar + baseline bar, both unconditional).
```

## Best Practices

- **Research isn't done because it feels done — it's done when it stops
  producing anything new.** A saturation loop that returns after one pass
  because "that seems like enough" isn't a saturation loop.
- **The open-endedness check and the identity check are structural, not
  aesthetic** — they change what gets built (an algorithm plus an LLM
  widening layer, a reference class instead of a category template), not
  how it's described afterward.
- **Loop-back is a feature of Mode 4, not a failure of Mode 1** — an
  exhaustive brief that later needs revising because the build taught you
  something is Flow working correctly.
- **The baseline bar is checked by looking, never by reasoning** — render
  it, screenshot it, read it. A code review that never rendered the output
  cannot certify the baseline bar, no matter how thorough the review was.
- **The statistics report is not optional** — an unverified "it works" is
  exactly the kind of claim `consolidator` exists to catch later; produce
  the numbers at the end of Flow instead of leaving them for someone else
  to reconstruct.
- **Don't split into multiple repos by default, and don't refuse to split by
  default** — Check C is a sizing decision made from the actual scope of the
  work, not a fixed preference either way.
