# Templates

Slim, stack-agnostic versions. Fill in from the codebase. Delete bracketed guidance.
Keep them short — every line is a token paid on the sessions that load it.

---

## Router entry file (project root)

This is the key token-optimization artifact. It is a **router**, not a loader: it tells the
agent *which* context file to read for *which* kind of task, instead of loading all of them.

**Write it as `AGENTS.md`** — the cross-agent standard (Codex/Cursor/Copilot/Claude Code all
read it or an equivalent). Then add a one-line `CLAUDE.md` so Claude Code finds it too:

```markdown
# CLAUDE.md
See `AGENTS.md` — it is the canonical project router. Read it first.
```

The `AGENTS.md` content:

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
- [linter/formatter/type config path, e.g. `clippy.toml`, `eslint.config.js`, `pyproject [tool.ruff]`]
- [name the official/recommended config it extends, e.g. `eslint-config-next`, ruff default set]

Conventions NOT machine-enforced (these are why this file exists):
- File/folder naming — [PascalCase / kebab-case / snake_case; test-file pattern; folder layout]
- Identifier naming — [components, hooks/utils, constants, types — only what the config can't pin]
- Imports — [ordering, path aliases like `@/…`] · [other house rules]

## File Organization
- `[folder]/` — [what belongs here]
```
Keep only judgment rules. If a tool already enforces it (file-case plugin, import-order,
naming-convention rule), configure it there and link the config — don't duplicate it here.

---

## `context/ui-context.md`  *(skip entirely for non-UI projects)*

```markdown
# UI Context
## Brand & Voice
- Name: [product name]   ·   Personality: [3–5 adjectives it IS] · not: [adjectives it's NOT]
- Voice/tone for copy: [from the brand grill]
## Visual Direction
- [mood, light/dark, density, type feel, references to resemble / avoid — drove the tokens below]
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
