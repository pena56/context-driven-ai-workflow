# Templates

Slim, stack-agnostic versions. Fill in from the codebase. Delete bracketed guidance.
Keep them short — every line is a token paid on the sessions that load it.

---

## Router `CLAUDE.md` (project root)

This is the key token-optimization artifact. It is a **router**, not a loader: it tells the
agent *which* context file to read for *which* kind of task, instead of loading all of them.

```markdown
# Project Context

Always read first: `context/progress-tracker.md` (small — current state, open questions).

Read on demand — load a file only when the task touches its concern:

| If the task involves… | Read |
|---|---|
| data model, auth, layers, system boundaries, an architectural choice | `context/architecture.md` |
| a new feature, scope question, "should we build X" | `context/project-overview.md` |
| writing/changing code (conventions a linter doesn't cover) | `context/code-standards.md` |
| UI / visual / component work | `context/ui-context.md` |
| how to scope, split, or verify a unit | `context/ai-workflow-rules.md` |

Do not read files outside the task's concern — that wastes tokens.

Verify before done: `[BUILD CMD]` · `[TEST CMD]` · `[LINT/TYPECHECK CMD]`

After meaningful changes: update `context/progress-tracker.md`. If a change alters
architecture/scope/standards, update that file too. Promote durable rules to invariants
in `architecture.md`.
```

---

## `context/project-overview.md`

```markdown
# [Project Name]

## Overview
[One paragraph: what it does, who for, problem solved — derived from README/entry point.]

## Core User Flow
1. [Step] 2. [Step] 3. [Step]

## Features (current, as observed in code)
- [Feature] — `[where it lives]`

## Scope
### In scope
- [observed]
### Out of scope
- [stated in README/docs, or "unknown — confirm"]

## Success Criteria
- [verifiable condition; mark "(inferred — confirm)" if not stated anywhere]
```

---

## `context/architecture.md`

```markdown
# Architecture

## Stack
| Layer | Technology | Role |
|---|---|---|
| [layer] | [from manifest] | [role] |

## System Boundaries
- `[folder]/` — [what it owns]

## Storage Model
- [store] — [what lives here]   (omit if none)

## Auth & Access
- [how auth/ownership works, or "none / confirm"]

## Invariants
1. [Rule the code enforces — cite where] 
2. [Rule] (inferred — confirm)
```

---

## `context/code-standards.md`

```markdown
# Code Standards

Enforced automatically (do not restate — see config):
- [linter/formatter/type config path, e.g. `clippy.toml`, `.eslintrc`, `pyproject [tool.ruff]`]

Conventions NOT machine-enforced (these are why this file exists):
- [observed convention the agent must follow]

## File Organization
- `[folder]/` — [what belongs here]
```
Keep only judgment rules. If a tool already enforces it, link the config and move on.

---

## `context/ui-context.md`  *(skip entirely for non-UI projects)*

```markdown
# UI Context
## Theme / Tokens
| Role | Token | Value |
|---|---|---|
## Component Library
[library + where components live]
## Layout Patterns
- [pattern]
```

---

## `context/progress-tracker.md`

```markdown
# Progress Tracker
Update after every meaningful change.

## Current Phase
- onboarding

## Current Goal
- [first thing to work on, or blank]

## Completed
- (deriving context) baseline context files generated from codebase

## Next Up
- [from TODOs/issues, or blank]

## Open Questions
- [everything the code could not answer — auth model? scope? invariants to confirm]

## Architecture Decisions
- [observed decisions worth recording; promote rules to architecture.md invariants]

## Session Notes
- [how to resume]
```
