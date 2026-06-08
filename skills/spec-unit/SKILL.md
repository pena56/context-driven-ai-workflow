---
name: spec-unit
description: Turn "I want to build X" into a scoped, verifiable spec file for one unit of work, score its complexity, link it to a GitHub issue, and auto-route execution to the right model (inline / Sonnet executor + Opus reviewer / back to grilling). Use when starting a feature unit, breaking a build-plan item into a buildable spec, mentions "spec this", "write a spec", "build unit NN", or wants the spec-driven build loop.
---

# Spec Unit

Convert one unit of work into a complete spec, then dispatch it. The spec is BOTH the
quality artifact and the token-optimization payload: a tight spec lets a cheaper executor
build the unit reading only the spec + the one or two boundary files it names — not all the
context files. A vague spec makes the executor flail and re-read everything, losing the win.

Reads context via the router `AGENTS.md` — load only the files the unit touches.

## Workflow

### 1. Establish the unit
A unit = one visible, verifiable result inside ONE system boundary, buildable in one
session. If the request is a phase ("build the dashboard"), split it first and spec the
first slice only. Confirm the unit's boundary against `context/architecture.md`.

### 2. Resolve ambiguity BEFORE writing the spec
If product/behaviour is undefined: do not invent it. Either ask the user, or record it as an
Open Question in `progress-tracker.md`. A spec built on guesses routes to "complex" and stalls
the executor. Resolve, then spec.

### 3. Score complexity → pick the route
Score with the rubric in [REFERENCE.md](REFERENCE.md). Print the route and the one-line reason.

| Route | When | Dispatch |
|---|---|---|
| **inline** | 1 file, mechanical, zero ambiguity | Build inline now (cheap model); no subagents |
| **standard** | 1 boundary, spec fully resolved | Sonnet executor subagent → Opus fresh-context reviewer |
| **complex** | multi-boundary OR open questions remain | STOP — split the unit or run grill-me/grill-with-docs first |

Always state the route + why. The user may override (`force: inline` / `force: split`).

### 4. Write the spec file
Use the five-section template in [REFERENCE.md](REFERENCE.md). Save to
`context/specs/NN-feature-name.md` where NN is the next unit number. Pull verify commands
from the router `AGENTS.md` (stack-agnostic — never hardcode `npm run build`).

### 5. Link to the issue (issue = disk, tracker = RAM)
If `gh` is available and the repo has a remote:
- Create/find the GitHub issue for this unit; capture its number `#N`.
- Thread the convention so the whole chain is greppable:
  `Unit NN  ⇄  Issue #N  ⇄  branch feat/N-feature-name  ⇄  spec specs/NN-feature-name.md  ⇄  PR "Closes #N"`
- Put `Issue: #N` in the spec header.
- In `progress-tracker.md`, set Current Goal to the active issue number only (durable backlog
  lives in Issues, not the tracker).
If no remote: skip silently, use Unit NN as the id, note "local-only" in the tracker.

### 6. Dispatch
- **inline**: implement directly, then run verify commands.
- **standard**: hand the spec to the executor (prefer the `tdd` skill — build test-first).
  After green, hand the diff to a fresh-context reviewer (the `code-review` skill). Reviewer
  must check the unit against the invariants in `architecture.md`.
- **complex**: do not code. Return the split or the grilling outcome.

**If the unit builds or changes any user interface** (a page, component, screen, form, layout
— anything visual), design quality is part of "done", not an afterthought:
- **Net-new UI** → generate it with `frontend-design` (distinctive, production-grade output),
  then polish/audit with `impeccable` before review. Do not hand-roll UI when these exist.
- **Changing existing UI** → run `impeccable` on it for UX/visual/a11y quality.
- Both must honour `ui-context.md` tokens — design skills choose *quality*, the context file
  constrains *consistency*. Run them in the executor step, before `code-review`.
This is automatic for UI units — the user should not have to ask for `impeccable` each time.

### 7. Close
On done + verified: mark Unit NN complete in `progress-tracker.md`, branch
`feat/N-feature-name`, PR body `Closes #N`. Promote any new durable rule to an invariant in
`architecture.md`; promote any resolved decision to an ADR.

## Do not
- Do not write a spec on unresolved ambiguity — resolve or flag first.
- Do not combine boundaries in one unit (UI + DB + background = three units).
- Do not hand a "complex" unit to an executor — that is the failure mode this prevents.
- Do not let `progress-tracker.md` become a second backlog — Issues are the source of truth.

## Credits
Orchestrates third-party skills: `tdd` ([Matt Pocock](https://github.com/mattpocock/skills/tree/main/skills/engineering/tdd)),
`grill-me`/`grill-with-docs` ([Matt Pocock](https://github.com/mattpocock/skills)),
`impeccable` ([Paul Bakaus](https://github.com/pbakaus/impeccable)),
`frontend-design` ([Anthropic](https://github.com/anthropics/claude-plugins-public/tree/main/plugins/frontend-design)),
`code-review` (Claude Code built-in).
