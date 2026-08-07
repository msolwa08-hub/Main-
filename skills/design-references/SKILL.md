---
name: design-references
description: Seventy-four teardowns of real production design systems — Apple, Stripe, Linear, Uber, Vercel, Notion, Figma, Airbnb, Tesla, Nike, Spotify and more — each documenting the actual colour ramps, type scales, spacing, motion and component anatomy that make it feel the way it does. Use when a UI needs a concrete reference bar, when borrowing an interaction language deliberately, or when qualitative feedback ("doesn't feel premium", "no identity") needs turning into specific mechanisms.
origin: community
---

# Design References

Seventy-four real products, taken apart. Each entry under
[`references/`](references/) carries a `DESIGN.md` documenting how that product
actually achieves its look — colour ramps with hex values, type scales, spacing
systems, motion timings, component anatomy — plus a `README.md` summarising it.

## When to Use

- A UI needs a **reference bar** and "make it look good" is not actionable.
- You are deliberately **borrowing an interaction language** (Apple's deference
  and depth, Stripe's density, Linear's speed, Uber's wayfinding) and want the
  mechanisms rather than a vague impression.
- Qualitative feedback needs decomposing: *"nothing feels premium"*, *"no
  identity"*, *"reads poorly"* → find two or three products that solve the same
  problem well, read what they actually do, extract the devices.
- Building a token layer and you want to see how a shipped system structures
  ramps, scales and elevation.

## How It Works

### 1. Pick references by PROBLEM, not by taste

Choose products that solve *your* problem, not ones you like the look of.

| Problem | Look at |
|---|---|
| Dense data, must stay calm | `linear.app`, `stripe`, `sentry`, `posthog`, `clickhouse` |
| Depth, restraint, hierarchy | `apple`, `notion`, `superhuman`, `raycast` |
| Wayfinding under pressure | `uber`, `airbnb`, `revolut`, `wise` |
| Developer tooling | `vercel`, `warp`, `cursor`, `resend`, `supabase`, `railway`-likes |
| Trust with money | `stripe`, `coinbase`, `kraken`, `mastercard`, `binance` |
| Brand-forward, cinematic | `tesla`, `bmw-m`, `ferrari`, `spacex`, `nike`, `playstation` |
| Editorial / content | `theverge`, `wired`, `pinterest`, `medium`-likes |
| AI products | `claude`, `cohere`, `mistral.ai`, `elevenlabs`, `runwayml`, `x.ai` |

Full list: `ls references/`.

### 2. Extract devices, not skins

The value is the **mechanism**. From each reference pull the specific move:

- How many greys, and how far apart?
- Is the accent used once per screen or everywhere?
- What is the type scale ratio, and which weights are reserved for what?
- What elevates — shadow, border, background shift, or blur?
- What animates, over how long, and what does it communicate?
- How is density handled: spacing, rules, or grouping?

Then re-express those in **your own tokens**. Copying the surface produces a
product that looks like someone else's and behaves like neither.

### 3. Triangulate — never single-source

One reference produces pastiche. Read two or three that solve the same problem
differently, note where they *agree* (that convergence is usually the principle)
and where they *diverge* (that is the taste choice you now get to make
deliberately).

### 4. Close the loop

Extracted a device? Apply it, render at real widths, screenshot it, **look at
it**, critique in writing, fix. A reference does not verify your implementation.

## Examples

**"Our clinical tool feels flat and generic."**
Read `linear.app` (calm density), `apple` (depth and deference) and `stripe`
(hierarchy in data-heavy surfaces). Note what all three do about elevation and
restraint in accent use. Rebuild the token layer around those two mechanisms
before touching any component.

**"Borrow Apple's UI and Uber's UI."**
`references/apple/DESIGN.md` for deference, depth and typographic hierarchy;
`references/uber/DESIGN.md` for wayfinding, decisive primary actions and
high-contrast state. They solve different halves of the problem — take the
structure from one and the urgency from the other.

**"What should our landing page do?"**
`resend`, `vercel`, `clay`, `framer`, `lovable` — read how each sequences hero →
proof → depth, and what they leave out.

## Notes

- These are observational teardowns of public products, not official design
  systems. Treat them as well-researched documentation, not specification.
- Every product here has a brand identity of its own. The goal is to build
  *yours*, informed by how they built theirs — the failure mode is producing a
  convincing imitation of a company you are not.
