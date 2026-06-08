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
- Stack + which files created/skipped.
- The build plan (unit list in order).
- Open Questions that still need the user before unit 01.
- Next step: `spec-unit` on unit 01.

## Do not
- Do not scaffold without a resolved plan — send the user to grill first.
- Do not write a `ui-context.md` for a non-UI project.
- Do not hardcode JS idioms or `npm run build` — use the chosen stack's commands.
- Do not pre-install or pre-document dependencies for units not yet reached.

## Credits
Orchestrates third-party skills by [Matt Pocock](https://github.com/mattpocock/skills):
`grill-me`, `grill-with-docs`, and `to-issues`.
