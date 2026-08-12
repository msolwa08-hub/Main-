# STATE.md

_Last audited: 2026-08-12, by running the actual test suites, security scans, and four parallel structural sweeps of the repo — not by reading docs and assuming they're current (several aren't; see below). Written for someone returning to this repo after months away. Pairs with `ARCHITECTURE.md`._

## Quick verdict

The **content layer** (agents/skills/commands/rules) and the **JS tooling** that validates and ships it are in genuinely good shape — well-tested, internally consistent, clean security scan. The **multi-harness promise** ("works the same in 13 different AI coding tools") is real for 3–4 tools and thin-to-nonexistent for the rest. Two side-projects (`ecc2/` Rust TUI, `src/llm/` Python package) are solid in isolation but disconnected from everything else — they're passengers in this repo, not integrated parts of it.

## Finished / solid

- **Content layer**: 67 agents, 94 commands, 302 skills, 21 rule dirs. All structurally validated (`scripts/ci/validate-*.js` all pass), frontmatter conventions match what the docs claim, catalog counts are self-consistent.
- **JS test suite**: 3,438 / 3,440 passing (`node tests/run-all.js`). See "Broken" below for the 2 failures.
- **Python test suite**: 94 / 94 passing (`src/llm/`, via `uv run pytest`) — note this requires `uv` to install dev dependencies first; a bare `pytest` fails with missing-module errors because `openai`/`pyyaml` aren't preinstalled.
- **Lint**: clean (`npm run lint` — ESLint + markdownlint).
- **Security / supply-chain scan**: clean (`npm run security:ioc-scan` — 207 files inspected, no known malicious-package indicators).
- **Harness audit**: 80/80, a perfect score (`npm run harness:audit`).
- **`.opencode/` adapter**: fully built and tested — TypeScript source compiles via `scripts/build-opencode.js`, output is gitignored (built fresh at pack time, not stale), and a dedicated regression test confirms the published package actually contains the compiled output (this was previously broken and got a permanent test guard).
- **`.codex/` adapter**: real sync script (`scripts/sync-ecc-to-codex.sh`) with several passing tests.
- **Hook system**: functional and actively used for real automation (auto-format, GateGuard fact-forcing, session persistence, background pattern-learning). It has a documented history of real bugs — runaway background processes, non-session-aware lifecycle, Windows path handling — but each one has a merged fix *and* a permanent regression test. Read as "a system that broke in real ways and was hardened," not "untested."
- **Installer core**: `install.sh` → `install-plan.js` → `install-apply.js`, with per-tool adapters for 10+ targets. This is the best-tested part of the whole install layer.

## Half-built

- **ECC 2.0 (`ecc2/`, the Rust TUI control plane)**: compiles cleanly, has ~428 inline tests across 13 of 17 source files — this is real, not a stub. But it is explicitly self-described (its own roadmap doc) as **alpha, not GA**, and it is a standalone control plane that supervises Claude Code sessions from outside — nothing in the rest of the repo calls into it or depends on it. If you're auditing "does the toolkit work," you can ignore `ecc2/` entirely; if you're auditing "is ECC 2.0 shippable," it's not there yet.
- **Multi-harness coverage is uneven**, despite the repo's dot-directory for nearly every AI tool suggesting otherwise:
  - **Fully wired** (sync script + tests + real generated content): `.opencode/`, `.codex/`.
  - **Curated subset, not a full mirror**: `.cursor/` (11 of 302 skills), `.kiro/` (43 of 302, and it bypasses the shared adapter system with its own installer), `.agents/` (39 of 302, no dedicated sync script — looks hand-maintained).
  - **Thin stub — the repo folder itself has almost nothing**, because the real install target is the user's home directory, populated only at install time: `.gemini/` (has an adapter + test, but the repo dir itself is just one file), `.zed/`, `.hermes/`, `.kimi/`, `.qwen/`, `.openclaw/`, `.trae/` (has an installer + test, but no content mirror), `.codex-plugin/` (literally just a README and a plugin.json — the least developed of all of them).
  - Net effect: the "write once, works everywhere" pitch is proven out for 2 tools, plausible-but-thin for 3, and aspirational for the remaining 6+.
- **Cross-harness "control plane" composition** (the work described in `docs/ECC-2.0-GA-ROADMAP.md` — governed contracts for promotion/merge/policy decisions across harnesses): explicitly described in that doc as read-only-first and promotion-gated, i.e. deliberately incremental and not done.
- **Commands-to-skills migration**: `WORKING-CONTEXT.md` lists moving `commands/` to thin skill-pointers as an active goal. It isn't happening yet — 0 of the 94 command files invoke the skill system; they remain full, independent 150–200+ line documents. If you see this described as in-progress anywhere, treat it as intent, not status.

## Broken

- **`tests/hooks/hooks.test.js`** — one sub-test fails: `observe.sh falls back to legacy output fields when tool_response is null`. The fallback path for older-format tool output doesn't currently do what the test expects.
- **`scripts/npm-publish-surface.test.js`** — fails: `package.json files align to the module graph and explicit runtime allowlist`. The list of files `package.json` declares it will publish has drifted out of sync with what the code actually needs at runtime — a packing-list-vs-shipping-box mismatch.
- **`npm test`'s first gate never completes** — `check-unicode-safety.js` fails on ~50 pre-existing emoji occurrences inside *vendored/reference* skill content (`skills/design-references/**/DESIGN.md`, `skills/ui-ux-pro-max/scripts/*.py`, `skills/shadcn/rules/chat.md`, `skills/soft-skill/SKILL.md`). Because `npm test` chains checks with `&&`, this stops the whole suite before anything downstream (including the real unit tests) ever runs. You have to bypass this one gate manually to see the rest of the results — which is how this audit did it.
- **`WORKING-CONTEXT.md` is stale and should not be trusted as current.** It's dated 2026-04-08 and claims catalog counts of 47 agents / 79 commands / 181 skills — the real numbers today are 67 / 94 / 302. Its "active queue" and PR-backlog sections describe a much earlier moment in the upstream project's history. Read it as an archive snapshot, not a dashboard.

## Not started / disconnected

- **`src/llm/` (the Python LLM abstraction package)**: fully self-contained and fully tested (94/94), with a real CLI entry point (`llm-select`). But a repo-wide search for anything importing it outside its own `src/`/`tests/` folders returns **zero hits**. Nothing in the agents/skills/hooks/commands system calls it. It reads as a separate library that happens to be checked into this repo, not a component of the toolkit.
- **`.codex-plugin/`**: contains only a README and a `plugin.json` — no sync mechanism, no content, no tests. The least-developed of any harness surface in the repo.

## How to re-verify any of this yourself

```bash
node scripts/ci/validate-agents.js && node scripts/ci/validate-commands.js && \
  node scripts/ci/validate-rules.js && node scripts/ci/validate-skills.js && \
  node scripts/ci/validate-hooks.js && node tests/run-all.js   # skips the unicode gate — see "Broken" above
uv run --extra dev pytest tests/ -q                            # Python suite
npm run lint                                                   # style
npm run harness:audit                                          # harness config score
npm run security:ioc-scan                                      # supply-chain scan
```

## Audit method and limits

This assessment came from: running every test/lint/audit script that exists in the repo, reading the core identity docs (`CLAUDE.md`, `AGENTS.md`, `SOUL.md`, `RULES.md`, `WORKING-CONTEXT.md`), and four parallel structural sweeps covering the scripts/hooks layer, the content layer, the multi-harness adapter layer, and the two side-subsystems. It did **not** involve reading all 302 skills or all 220 scripts line by line — this is a structural and statistical audit (verified counts, spot-checks, test results), not a full line-by-line code review. Treat specific numbers as accurate as of this date and re-run the commands above before relying on them months from now.
