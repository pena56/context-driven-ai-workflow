# Context-Driven AI Workflow

A reusable, token-conscious system for building software with AI coding agents — from
planning a brand-new project to dropping into an existing one — and consistently getting
clean, maintainable code out the other side.

It started as an adaptation of JavaScript Mastery's **Six-File Context** methodology, then
fixed that methodology's real weaknesses (docs that rot, no tests in the loop, no progressive
disclosure, greenfield-only). The result is four Claude Code **skills** plus a defined build
loop that wires them — and your existing skills — together.

> **Who this is for:** anyone using Claude Code (or a similar agent) who wants a repeatable
> way to start and run projects without (a) the agent drifting over time, or (b) burning
> tokens re-reading everything every session.

---

## The core ideas (read these first)

1. **You are the architect; the AI is the implementation engine.** The thinking — what
   you're building, how it fits, what the rules are — stays with you. The agent executes it
   at speed. Most AI failures are underspecification, not a bad model.

2. **Context is a *router*, not a *loader*.** The agent reads a context file **only when the
   task touches it** — not all of them, every session. This is the central token fix. (See
   [docs/DESIGN-DECISIONS.md](docs/DESIGN-DECISIONS.md#1-router-not-loader).)

3. **Put truth where it can be *run*.** Tests, types, linters and CI hold what can be
   executed; prose docs hold only what can't (intent, "why", scope). Prose you can't run will
   drift; a test won't.

4. **Spec before code; one verifiable unit at a time.** Each unit is a single visible result
   inside one system boundary, with a checklist for "done".

5. **Route work to the right-cost model.** Plan/review on Opus (low volume, high leverage),
   execute on Sonnet (high volume, biggest cost lever), mechanical on Haiku. The *spec* is
   what makes cheap execution safe.

6. **Issues = disk, progress-tracker = RAM.** GitHub Issues are the durable backlog; the
   tracker is volatile session state. They link by number, never duplicate.

---

## What's in here

```
context-driven-ai-workflow/
├── README.md                       ← you are here
├── docs/
│   ├── DESIGN-DECISIONS.md          ← every decision + why it was made
│   ├── WORKFLOW.md                  ← the end-to-end loop, model routing, git/CI
│   ├── PORTING.md                   ← using it with Codex / Cursor / Copilot
│   └── CRITIQUE.md                  ← the original methodology, critiqued
└── skills/
    ├── derive-context/              ← onboard an EXISTING repo (bottom-up)
    ├── scaffold-context/            ← start a NEW project (top-down)
    ├── spec-unit/                   ← spec + complexity-route + build a unit
    └── scaffold-delivery/           ← git / CI / deploy pipeline
```

## The four skills

| Skill | Use it when | What it produces |
|---|---|---|
| **`derive-context`** | You open an existing/brownfield repo | Slim context files inferred *from the code* + a router `AGENTS.md` |
| **`scaffold-context`** | You're starting something new | Context files from your planning conversation + a build-plan `specs/` folder |
| **`spec-unit`** | You're about to build a feature | A scoped spec file, a complexity score, a GitHub issue link, and the right model route |
| **`scaffold-delivery`** | You're wiring up the repo | Branch convention, CI, PR/issue templates, tag-gated prod deploy, doc-sync hook |

These lean on skills you likely already have: **`grill-me` / `grill-with-docs`** (planning),
**`tdd`** (the executor), **`code-review`** (the fresh-context reviewer), **`to-issues`**
(decompose into units), **`impeccable` / `frontend-design`** (UI quality), **`update-config`**
(hooks).

---

## Getting started

### 1. Install the skills

Copy the four folders into your Claude Code skills directory:

- **User-level (all projects):** `~/.claude/skills/` (Windows: `C:\Users\<you>\.claude\skills\`)
- **Project-level (one repo):** `<repo>/.claude/skills/`

```bash
# from this repo's root
cp -r skills/* ~/.claude/skills/        # macOS/Linux
```
```powershell
Copy-Item -Recurse skills\* "$env:USERPROFILE\.claude\skills\"   # Windows PowerShell
```

Restart Claude Code (or start a new session). The skills are auto-discovered from their
`SKILL.md` front-matter — no registration needed. Confirm by asking Claude to list available
skills, or just trigger one (below).

### 2A. Starting a NEW project

```
1. Plan it.        → run grill-me (or grill-with-docs) until the system is clear
2. Scaffold it.    → run scaffold-context   (asks your stack, pins LATEST stable versions,
                                              writes slim context + router AGENTS.md + specs/)
3. Wire delivery.  → run scaffold-delivery   (branch convention, CI, prod-via-tag, hook)
4. Build, unit by unit:
   → run spec-unit on unit 01  → it scores complexity and routes:
        trivial  → built inline (cheap model)
        standard → tdd (Sonnet) → impeccable/frontend-design if UI → code-review (Opus)
        complex  → it stops and tells you to split or grill first
   → CI verifies → PR "Closes #N" → merge → staging
5. Ship.           → git tag vX.Y.Z → production
```

### 2B. Jumping into an EXISTING project

```
1. Map it.         → run derive-context      (infers stack, writes slim context bottom-up,
                                              flags inferred invariants for you to confirm)
2. (optional) wire delivery if it isn't already → scaffold-delivery
3. Build, unit by unit → same spec-unit loop as above
```

### 3. The build loop in one breath

> **spec-unit** turns intent into a spec → routes it by complexity → **tdd** builds it
> test-first → **impeccable/frontend-design** handle any UI → **code-review** reviews it in a
> fresh context → CI enforces the checklist → PR closes the issue → a release tag ships prod.

Full detail, including the model-routing rubric and the git/CI shape, is in
[docs/WORKFLOW.md](docs/WORKFLOW.md).

---

## Two habits the skills enforce for you

- **Latest-stable versions, never from memory.** Agents pull stale version numbers from
  training data. `scaffold-context` and `spec-unit` resolve current releases from the live
  registry (`npm view`, `pip index versions`, `cargo search`, …) and pin them.
- **Design quality isn't optional.** Any unit that touches an interface automatically routes
  through `frontend-design` (to build) and `impeccable` (to polish) — you don't have to ask.

---

## A note on scope

This is **git + GitHub + some CI runner** flavored, but **stack-agnostic** — nothing assumes
Next.js or any language. Skills detect the stack and skip what doesn't apply (no UI file for a
backend service). It also degrades gracefully: no remote → local-only mode; tiny project →
fewer files. See [docs/DESIGN-DECISIONS.md](docs/DESIGN-DECISIONS.md) for the boundaries.

It's also **agent-portable**: the router is written as `AGENTS.md` (the cross-agent standard)
so the method works on Codex, Cursor, and Copilot too — only Claude Code's convenience
automation (auto-triggered skills, model routing, hooks) is specific. See
[docs/PORTING.md](docs/PORTING.md).

## Credits

Adapted and substantially reworked from JavaScript Mastery's *"From Idea to Product: The
AI-Driven Developer's Playbook"* (the Six-File Context System). The critique that motivated
the changes is in [docs/CRITIQUE.md](docs/CRITIQUE.md).
