# ARCHITECTURE.md

_Last audited: 2026-08-12. Written for someone returning to this repo after months away._

## What this project is

This repo is **ECC** ("Everything Claude Code", package name `ecc-universal`) — a large, pre-packaged bundle of instructions, automation, and conventions for AI coding assistants. It is not an application with a UI that end users click through. It's closer to a **style guide + toolbox that installs itself into other tools**: Claude Code, Cursor, Codex/OpenAI, Gemini, OpenCode, and about ten other AI coding harnesses.

The core idea: write a piece of guidance once (e.g. "how to review Python code," "how to do TDD," "how to run an E2E test"), and have an installer copy it, in the right shape, into whichever AI coding tool a developer happens to be using — so behavior stays consistent no matter which tool someone opens.

**Important context on origin:** this is a fork/mirror of an actively developed open-source project (`affaan-m/ECC`, MIT licensed, run by a single maintainer with sponsors and a paid "ECC Pro" tier). The overwhelming majority of the content here — the 302 skills, 94 commands, 67 agents, the roadmap docs, the business/sponsorship docs — is **upstream-authored**, not written by the account this fork lives under (`msolwa08-hub/Main-`). Treat this repo as "a large piece of borrowed infrastructure," not a from-scratch personal project.

## The mental model

```
   authored guidance                validation                 per-tool packaging              installed
   (agents/skills/commands/  ──▶   (scripts/ci/*,     ──▶      (scripts/lib/install-  ──▶      (~/.claude,
    rules/hooks — the "what")       npm test)                   targets/*, one per tool)         ~/.cursor,
                                                                                                   ~/.codex, …)
```

1. **Content is written once**, in plain Markdown/JSON, at the repo root.
2. **A validation layer** checks that content is well-formed before it ships (this is most of what `npm test` runs).
3. **A packaging/adapter layer** reshapes that content per target tool — some tools want YAML frontmatter, some want `.mdc` instead of `.md`, some want a single merged config file.
4. **An installer** copies the packaged output into the user's actual tool config directories (their home folder, or the current project).
5. **At runtime**, a separate hook system wires some of that content into the tool's actual lifecycle (e.g. "run this check before every Bash command").

## The major parts

### 1. The content layer — what the toolkit actually says

| Directory | Count | What it is |
|---|---|---|
| `agents/` | 67 files | Specialist personas an AI can delegate to (e.g. `planner.md`, `security-reviewer.md`). Each is Markdown with YAML frontmatter: `name`, `description` (including *when* to invoke it), `tools`, `model`. |
| `skills/` | 302 directories | Reusable how-to guidance (e.g. `tdd-workflow/`, `security-scan/`). Each has a `SKILL.md` with `name`, `description`, `origin`. **91% of skills are just that one Markdown file** — no scripts, no templates. Skills here are prompt documentation, not executable tooling, with a few (28) exceptions that bundle real scripts/templates. |
| `commands/` | 94 files | Slash commands (`/tdd`, `/plan`, `/code-review`, etc.) — full standalone workflows, typically 150–200+ lines each. |
| `rules/` | `common/` + 20 per-language dirs | Always-on conventions (coding style, testing, security, git workflow), layered so language-specific rules override common ones. |

All four are grouped into **install modules** (`manifests/install-modules.json`, `install-components.json`) so a user can selectively install only what their stack needs (e.g. `lang:typescript`) instead of the whole bundle.

### 2. The validation layer — the quality-control line

`scripts/ci/` (13 scripts) is what `npm test` actually runs, in order: unicode/emoji safety, agent/command/rule/skill/hook structural validation, install-manifest validation, no-personal-paths check, catalog sync check (do the documented counts match reality?), command-registry sync check — and only then the real unit tests (`tests/run-all.js`).

### 3. The multi-harness adapter layer — one bundle, many targets

The repo has a dot-directory for nearly every AI coding tool (`.claude/`, `.cursor/`, `.codex/`, `.codex-plugin/`, `.gemini/`, `.opencode/`, `.kiro/`, `.trae/`, `.zed/`, `.hermes/`, `.kimi/`, `.qwen/`, `.openclaw/`, `.agents/`). These are **not equally real** — see STATE.md for which are fully built vs. stubs. The mechanism, where it exists, is `scripts/lib/install-targets/*.js` — one adapter file per tool, each responsible for translating canonical content into that tool's expected shape, invoked by the installer.

### 4. The installer

`install.sh` is a thin shell shim (handles symlinks, Git-Bash path quirks, auto-runs `npm install`) that hands off to `scripts/install-apply.js`, which executes a plan built by `scripts/install-plan.js` using the adapters above. `scripts/uninstall.js` reverses it. A few tools (`.kiro/`, `.trae/`) have their own bespoke installer instead of using the shared adapter registry.

### 5. The hook system — runtime automation

`hooks/hooks.json` wires specific lifecycle events (SessionStart, PreToolUse, PostToolUse, Stop, PreCompact, SessionEnd) to real scripts in `scripts/hooks/` (51 files). Examples: a "GateGuard" that blocks the first edit to a file until facts are stated (this is the gate that fired while writing this very file), an auto-formatter after edits, a background "observer" that watches session activity and extracts reusable patterns (`continuous-learning-v2`). Every hook entry is wrapped in a defensive bootstrapper that resolves the real plugin install path first — this matters because ECC itself is installed as a Claude Code plugin, so a hook can't assume a fixed file path.

### 6. Two side-subsystems living in the same repo

- **`ecc2/`** — a separate Rust project (`ecc-tui`, ~54,000 lines), a terminal-UI "control plane" for supervising many concurrent AI coding sessions at once (start/stop/resume, SQLite-backed session history, worktree awareness, a background daemon). It compiles and has real test coverage, but it is a standalone experiment — it is not called by, or wired into, anything else in this repo. Think of it as a second, parallel product living in the same house.
- **`src/llm/`** — a Python package (`llm-abstraction`) that picks a language-model provider (Claude, OpenAI, Ollama, and a couple of custom ones) behind one common interface, with its own CLI (`llm-select`). Fully self-contained and fully tested, but — like `ecc2/` — nothing else in the repo imports or calls it.

### 7. Tests

Two independent test suites: `tests/*.test.js` (Node's built-in test runner, ~3,440 tests, driven by `node tests/run-all.js`) covering the JS/scripts layer, and `tests/test_*.py` (pytest, 94 tests, run via `uv run pytest`) covering `src/llm/` only. They do not share a runner — you have to know to run both.

### 8. Docs

`docs/` (23+ files/subdirs) is a mix of two very different things: genuine how-to guides (`SKILL-DEVELOPMENT-GUIDE.md`, `TROUBLESHOOTING.md`) and the **upstream maintainer's own planning artifacts** — Linear-mirrored roadmaps, PR review notes, business/sponsorship copy. The latter describes the upstream project's operations, not this fork's.

## How a change actually flows through the system

1. Someone edits/adds a file under `agents/`, `skills/`, `commands/`, or `rules/`.
2. `npm test` validates structure and keeps the catalog counts honest.
3. `npm run lint` checks code style and Markdown formatting.
4. At install time, the relevant `install-targets/*.js` adapter reshapes the content for each target tool the user has selected.
5. `install.sh` copies the result into that tool's real config location (usually somewhere under the user's home directory).
6. If the change touches `hooks/hooks.json` or `scripts/hooks/*`, it also needs `scripts/ci/validate-hooks.js` to pass and, ideally, a matching test under `tests/hooks/`.
