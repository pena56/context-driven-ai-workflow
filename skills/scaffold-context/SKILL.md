---
name: scaffold-context
description: Scaffold a slim, router-style context-file system for a NEW (greenfield) project from a planning/grilling conversation, then generate a load-on-demand CLAUDE.md and an empty specs/ build-plan folder. Use when starting a new project, after a grill-me/grill-with-docs planning session, when the user mentions "scaffold context", "set up a new project", "new build", or wants the Six-File context system for a project that has no code yet.
---

# Scaffold Context

The greenfield twin of [derive-context](../derive-context/SKILL.md). That one reads code
bottom-up; this one writes context top-down from a planning conversation, *before* any code
exists. Output is identical in shape — **slim, router-based** — so the build loop is the same
whether a project was onboarded or born here.

## Prerequisite
There must be design thinking to consume. If the user hasn't planned, route them to
`grill-me` (or `grill-with-docs`) first — do NOT invent a product. This skill turns a
*resolved* plan into files; it is not the planning step itself.

## Workflow

### 0. Grill brand, naming & design direction (greenfield only)
The product grill (`grill-me`) resolves *what / who / flow / scope / stack*. It does **not**
resolve the **name**, the brand, or the **visual direction** — and the scaffolder needs all
three (the name seeds `project-overview.md`; the direction seeds `ui-context.md`). So before
writing files, run a focused brand grill. Drive `grill-me` with this agenda — keep interviewing
until each is *decided*, not vibed:
- **Name & naming** — product name; candidates considered and why this one; naming convention
  for sub-things (services, packages, features) so they stay consistent later.
- **Positioning & personality** — one-line positioning, who it's for, the 3–5 adjectives the
  brand *is* and the ones it is *not* (these become the voice/tone the UI copy must match).
- **Visual direction** *(UI projects only — skip for backend/CLI/library)* — mood, light/dark,
  density, color direction (not exact hex yet), typography feel, reference products it should
  (and shouldn't) resemble. Enough to derive real tokens in step 3, not guesses.
For a **non-UI** project, do the name/positioning half and skip visual direction. Record the
outcome — it feeds `project-overview.md` (name, positioning) and `ui-context.md` (direction,
voice). If the user genuinely doesn't care about brand yet, log it as an Open Question rather
than inventing one.

### 1. Resolve the stack (ask — don't assume)
Greenfield has no manifests to read, so ask or confirm: language/framework, datastore, auth,
UI library (or "no UI"). Pick the stack-appropriate verify commands. Never default to
Next.js/JS unless the user chose it.

### 1a. Pin LATEST STABLE versions (never from memory)
Training data is stale — do NOT write a version number you "remember". Resolve each chosen
technology's current stable release from the live source, then pin it:
- npm: `npm view <pkg> version` · PyPI: `pip index versions <pkg>` · crates: `cargo search <pkg>`
- Go: `go list -m -versions <mod>` · others: the registry, or `WebSearch "<tech> latest stable release 2026"`
Prefer the latest **stable** (not alpha/beta/RC) unless the user opts in. Record the resolved
versions in `architecture.md`'s stack table and note "verified <date>". When unit 01 installs
deps, use these pinned versions — don't let the executor reach for a remembered one.

### 1b. Adopt the stack's official lint/format config (don't hand-roll rules)
Most ecosystems ship an **official or de-facto recommended** lint + format config that already
encodes best practice for that technology. Resolve and pin the right one instead of inventing
rules — it does the heavy lifting so `code-standards.md` only carries judgment calls:
- JS/TS: framework config first (`eslint-config-next`, `@vue/eslint-config`, `@angular-eslint`),
  else `@typescript-eslint` recommended; formatter Prettier or **Biome** (lint+format in one).
- Python: **Ruff** (`[tool.ruff]`, with the recommended rule set) + its formatter.
- Go: `gofmt` + **golangci-lint**. · Rust: **rustfmt** + **clippy**. · Others: the registry's
  recommended config, or `WebSearch "<stack> official eslint/lint config best practice 2026"`.
Pin the config package(s) at the same time as the deps above, record them in `architecture.md`,
and have **unit 01 install + wire them** (config files + the verify/lint command in the router).
Don't restate rules a config enforces — `code-standards.md` links the config and stops there.

### 2. Decide which files apply
- Backend service / CLI / library → **skip `ui-context.md`** (four files, not six).
- UI app → include it; tokens come from a design pass, not guessed.
State which files you're creating and which you're skipping, and why.

### 3. Generate the context files
Use the slim templates in [derive-context/TEMPLATES.md](../derive-context/TEMPLATES.md).
Greenfield differences vs derive-context:
- Fill from the **planning conversation**, not from code.
- Unknowns become **decisions to make** (Open Questions in the tracker), not
  `(inferred — confirm)`. If the plan didn't resolve it, don't fabricate it.
- `invariants` are the rules the user *committed to* in planning — state them as firm rules,
  at least four (per the methodology), each one you can defend.
- `code-standards.md` still defers to linter/formatter/type config — note "configure
  [tool] in unit 01" since the configs don't exist yet.

**Ask the user for their convention preferences** (greenfield is the moment to lock these — the
codebase will inherit them forever). Don't assume; ask, then split by who enforces it:
- **File & folder naming** — case for files (PascalCase / kebab-case / snake_case), test-file
  pattern, one-component-per-file, route/feature folder layout.
- **Identifier naming** — components, hooks/utils, constants, types/interfaces, env vars.
- **Import ordering & path aliases** (e.g. `@/…`), and any house rules the official config
  above doesn't cover.
Then route each preference to where it belongs: anything the **official config can enforce**
(via a rule or plugin — e.g. `unicorn/filename-case`, import-order, naming-convention) →
configure it in that config in unit 01, don't duplicate it in prose. Anything **not
machine-enforceable** → record it under "Conventions NOT machine-enforced" in
`code-standards.md` so the agent follows it consistently.

### 4. Generate the router `AGENTS.md` (+ `CLAUDE.md` pointer)
Use the router template in
[derive-context/TEMPLATES.md](../derive-context/TEMPLATES.md). On-demand loading by task
type; only `progress-tracker.md` is always-read. Fill in the chosen verify commands.

### 5. Seed the build plan
- Create `context/specs/` with a `00-build-plan.md` placeholder.
- Hand the feature list to `to-issues` to decompose into ordered, tracer-bullet units, then
  reconcile the numbering into `00-build-plan.md`. Order by: dependencies first, security
  before functionality, backend before frontend wiring, UI shells before real data, deps
  just-in-time.
- `progress-tracker.md`: Current Phase = `scaffolding`, Next Up = unit 01.

### 6. Report
- Name + brand/visual direction captured (or flagged as Open Question).
- Stack + which files created/skipped.
- Official lint/format config chosen + that unit 01 will install/wire it.
- Convention preferences captured, and which went to config vs `code-standards.md`.
- The build plan (unit list in order).
- Open Questions that still need the user before unit 01.
- Next step: `spec-unit` on unit 01.

## Do not
- Do not scaffold without a resolved plan — send the user to grill first.
- Do not invent a product name, brand personality, or visual direction — grill it (step 0) or
  log it as an Open Question. The same goes for convention preferences: ask, don't assume.
- Do not hand-roll lint rules the stack's official config already provides — adopt the config.
- Do not restate in `code-standards.md` any rule the lint/format config enforces — link it.
- Do not write a `ui-context.md` for a non-UI project.
- Do not hardcode JS idioms or `npm run build` — use the chosen stack's commands.
- Do not pre-install or pre-document dependencies for units not yet reached.

## Credits
Orchestrates third-party skills by [Matt Pocock](https://github.com/mattpocock/skills):
`grill-me`, `grill-with-docs`, and `to-issues`.
