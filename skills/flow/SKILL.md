---
name: flow
description: A three-mode project lifecycle for building a large or new feature end to end — Brainstorm Mode (flesh out the full app structure and idea space before any code, drawing on every relevant skill available), Plan Mode (research and produce the concrete backend, frontend/UI, and asset-generation plan), and the final Combine & Test pass (integrate everything, end-to-end test, stress test, adversarially test, and report). Use when starting a large feature or a whole new project surface, not for a small fix or a single-file change.
metadata:
  origin: ECC
---

# Flow

Three modes, in order, each producing a concrete artifact the next mode
consumes. Nothing here replaces `planner`, `tdd-guide`, `e2e-runner`, or
`code-reviewer` — Flow is the sequence that calls them at the right moment,
so a large build doesn't skip straight from an idea to code with no
research pass, or ship with no adversarial testing pass.

**Before Brainstorm Mode opens**, run `consolidator` if the project has any
real history — brainstorming new ideas for something partially built already
is how work gets duplicated. Know what exists before inventing what's next.

## When to Use

- Starting a large feature or a new surface in an existing product, not a
  small fix
- A new project from a blank slate, before any code exists
- The user describes an outcome ("build the X tab", "add Y") without having
  already decided the structure, the plan, or the UI
- Explicitly invoked by name: "run Flow on this," "brainstorm mode," "plan
  mode"

## How It Works

### Mode 1 — Brainstorm

**Goal: flesh out the full idea space and app structure, before any
implementation decision is made.**

This is not "list some ideas." It pulls in every skill relevant to the
domain being brainstormed — a healthcare surface pulls the
`healthcare-*` skills, a UI-heavy feature pulls `frontend-design-direction`
and `make-interfaces-feel-better`, a data-heavy one pulls the relevant
backend/database skills — and uses each one's lens to surface ideas,
requirements, and structural detail a single generic pass would miss. Use
`SearchSkills`/`ListSkills` to find what's relevant to the domain rather than
guessing from memory; skills are added to this repo faster than anyone's
memory of the catalogue stays current.

Output: an idea/structure brief — every feature, edge case, and structural
piece worth considering, organized by area (not yet scoped, not yet
prioritized). Nothing here commits to what ships; it establishes what
*could*.

**Gate before Mode 2:** the person driving the project picks what from the
brief actually gets built. Brainstorm produces options; it does not decide.

### Mode 2 — Plan

**Goal: the concrete, researched plan for what was chosen in Mode 1 — what
the tool needs to know, split across backend, frontend/UI, and assets.**

Three parallel research tracks, each producing its own section of one plan:

- **Backend** — what the tool needs to know and do: data model, business
  logic, API surface, what's computed vs. stored, what's deterministic vs.
  model-driven. Use `planner`/`architect` for this track.
- **Frontend / UI** — what the interface needs to be: the screens, the
  states each screen can be in, the interaction model. Use
  `frontend-design-direction`, `code-architect`, or the project's own design
  agent if it has one (e.g. a Claudia-style UI agent).
- **Assets** — what needs to be generated or sourced to build it: images,
  icons, illustrations, any generated media, and what tool produces each
  (Gemini, a design system, stock/licensed sources). Name the generator and
  the fallback for each asset up front, not mid-build.

Output: one plan document with all three tracks, each with enough detail
that Mode 3's build work doesn't stall on an unanswered question mid-way
through.

**Gate before build starts:** the plan is reviewed before code begins — this
is where a wrong direction is cheapest to catch, per the same principle any
staged design/build process uses (a plan can be rewritten in a paragraph; a
finished build can only be rebuilt).

### Mode 3 — Combine & Test

**Goal: integrate everything built against the plan, then prove it holds up
before calling it done.**

In order:

1. **Combine** — wire the backend, frontend, and assets together into one
   working whole; this is where seams between pieces built separately break,
   so check the joins specifically, not just each piece alone.
2. **End-to-end test** — the real user journey, start to finish, on the
   actual integrated system. Use `e2e-runner`.
3. **Stress test** — load, volume, or scale conditions past normal use;
   what happens at 10x the expected data, concurrent users, or input size.
4. **Critical / adversarial test** — actively try to break it: bad input,
   edge cases, security-sensitive paths (`security-reviewer`), and — for any
   safety-critical domain — the specific failure modes that matter most for
   that domain, not generic ones.
5. **Final pass** — `code-reviewer` over the full diff, plus a **statistics
   report**: test coverage, pass/fail counts by category, what was stress-
   tested and at what scale, what's flagged NOT YET VERIFIABLE rather than
   claimed. This report is the deliverable that closes Flow — a final pass
   with no numbers attached is an opinion, not a result.

## Examples

**A new feature on an existing large project**
```
User: We need an Investigations tab.
Flow: consolidator first — confirms nothing like this exists yet, and surfaces
      any adjacent work (e.g. a research corpus already covering test/imaging
      content) so Plan Mode doesn't re-derive it.
      Brainstorm Mode — pulls healthcare-cdss-patterns, frontend skills, the
      project's own design references; produces the full idea brief (imaging,
      bloods, ingestion paths, what's deterministic vs. generative).
      → owner picks scope from the brief
      Plan Mode — backend track (data model + formulas), UI track (screens +
      states), asset track (what art needs generating and how).
      Combine & Test — build against the plan, e2e test the real flow, stress
      test with a large result set, adversarially test malformed input,
      final pass with a coverage/stats report.
```

**A brand-new project**
```
User: I want to build a new tool for X, starting from nothing.
Flow: Brainstorm Mode alone, first — no consolidator needed, nothing exists
      yet — surfaces the full structural idea space across every relevant
      skill. Then Plan Mode turns the chosen ideas into the three-track plan.
      Then Combine & Test once there's something to integrate and test.
```

## Best Practices

- **Brainstorm is divergent, Plan is convergent** — don't let Mode 1 quietly
  start deciding scope, and don't let Mode 2 quietly start generating new
  ideas instead of researching the chosen ones. Different jobs, different
  modes.
- **Every plan-track answers "what does the tool need to know"** — if Mode 2
  produces a plan and a build still stalls on an unanswered question, that's
  a Plan Mode gap, not a Combine & Test surprise.
- **The stats report is not optional** — "it works" without numbers attached
  is exactly the kind of unverified claim `consolidator` exists to catch
  later; produce the numbers at the end of Flow instead of leaving them for
  someone else to reconstruct.
- **Gate between modes, don't skip them** — a large build that goes straight
  from Brainstorm to code, or from Plan to ship with no adversarial pass, is
  the exact shape of failure Flow exists to prevent.
