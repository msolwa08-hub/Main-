---
name: ui-ux-pro-max
description: Searchable design-intelligence datasets — 84 UI styles, 192 colour palettes, 74 font pairings, 98 UX guidelines, 25 chart types, motion and icon libraries — queried through a local BM25 CLI across 22 tech stacks. Use when choosing a visual style, colour system, type pairing or chart type, when auditing a UI against accessibility and interaction rules, or when generating a complete design system for a new project.
origin: community
license: MIT
---

# UI/UX Pro Max

Design decisions, backed by searchable data instead of vibes. Fourteen CSV
datasets queried by a local BM25 search — no network, Python standard library
only.

## When to Use

- Choosing a **visual style** for a product type (dashboard, landing, admin,
  mobile app) and wanting the concrete parameters, not an adjective.
- Picking a **colour palette**, **font pairing**, **chart type**, **icon set**
  or **motion timing** with rationale attached.
- **Auditing** a UI against accessibility, touch, layout, typography and
  animation rules — especially when it "looks unprofessional" and nobody can say
  why.
- **Generating a whole design system** for a new project in one pass.
- Working in a specific stack (React, Next, Vue, Svelte, SwiftUI, Flutter,
  Compose, Tailwind, shadcn and 13 more) and wanting stack-shaped guidance.

Skip it for backend, API, database, infrastructure or non-visual work.

## How It Works

### Prerequisites

Python 3, standard library only. No packages to install, no network calls.

```bash
python3 --version
```

If Python is unavailable, do **not** install it — say so and fall back to
[`templates/base/quick-reference.md`](templates/base/quick-reference.md), which
carries the rule tables in prose.

### Search

```bash
# style for a product type
python3 scripts/search.py "clinical dashboard calm dense" --domain style

# palette, typography, charts, motion, icons
python3 scripts/search.py "trustworthy medical" --domain color
python3 scripts/search.py "editorial serif pairing" --domain typography
python3 scripts/search.py "trend over time" --domain chart

# stack-shaped
python3 scripts/search.py "data table" --stack react

# machine-readable / untruncated
python3 scripts/search.py "bento grid" --domain style --json --full
```

Domains: `style`, `color`, `chart`, `landing`, `product`, `ux`, `typography`,
`icons`, `gsap`, `react`, `web`, `google-fonts`.

### Generate a design system

```bash
python3 scripts/search.py "bedside clinical tool" --design-system \
  --project-name meridian --variance 4 --motion 3 --density 8
```

`--variance`, `--motion` and `--density` (each 1–10) steer how adventurous,
animated and information-dense the result is. `--persist` writes it out;
`--format markdown` for a document rather than terminal output.

### Read the result as INPUT, not output

A row returns style keywords, primary colours, effects, framework fit,
complexity, an implementation checklist and suggested CSS variables. That is a
**starting position** to argue with — it does not know your product, your users
or your constraints. Take the parameters, apply them in your own token layer,
then render and judge with your eyes.

## Examples

**"This dashboard looks unprofessional but I can't say why."**
Run the `ux` domain, work the priority table in
[`templates/base/quick-reference.md`](templates/base/quick-reference.md) top-down
— accessibility, touch, performance, style consistency, layout — and fix in that
order. The ranking is deliberate: contrast and touch targets outrank prettiness.

**"Pick a palette that reads trustworthy for a health product."**
`--domain color`, then verify every pair you actually ship against a contrast
checker in both light and dark. The dataset suggests; it does not certify.

**"Set up a new project's design system."**
`--design-system` with a low `--variance` and low `--motion` for a clinical or
financial tool; higher for consumer or marketing surfaces.

## Notes

- `templates/base/quick-reference.md` is bilingual (Chinese section headings,
  English rule tables). The rule content is fully usable either way.
- Datasets are opinionated aggregations, not standards. Where a row conflicts
  with a platform guideline (Apple HIG, Material) or with a WCAG requirement,
  the platform guideline and WCAG win.
- Pairs well with `design-references` (real teardowns to triangulate against)
  and `ui-design-tools` (the tools that verify what this suggests).
