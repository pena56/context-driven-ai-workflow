# The Original Methodology, Critiqued

This workflow began as JavaScript Mastery's **Six-File Context System** (from *"From Idea to
Product: The AI-Driven Developer's Playbook"*). That methodology is genuinely good in places
and weak in others. This file records both, so you understand *what* we kept and *why* we
changed the rest. The changes themselves are in [DESIGN-DECISIONS.md](DESIGN-DECISIONS.md).

---

## What the original is

Stripped to essentials, four moves:

1. **Think before you prompt** — a planning conversation with a non-coding AI.
2. **Write the thinking down** — six narrative markdown files in `context/`.
3. **Decompose into vertical slices** — a build plan of "units", each with a spec file.
4. **Run a manual three-prompt loop** — implement / correct / close, re-reading context each
   session.

None of this is new to software engineering — it's RFCs + ADRs + a working-memory doc + story
decomposition, repackaged for people who've never done it. The repackaging *is* the value.

## What we kept (it's genuinely good)

- **"You are the architect; the AI is the implementation engine."** Correct mental model.
  Most AI failures are underspecification.
- **Invariants** — rules the system must never violate, stated so violations become visible.
  The strongest single idea; an informal architecture fitness function.
- **Explicit out-of-scope** — negative space stops the agent helpfully building things you
  didn't ask for.
- **A progress tracker as session memory** — agents are stateless between sessions; a small
  durable state file is the right fix.
- **Vertical slices with a "done" checklist** — clean story decomposition.

## What we changed (and why)

**1. Docs were the source of truth, and docs rot.** The system trusted hand-maintained prose
to stay aligned with code, enforced only by the agent's goodwill. Nothing *fails* when docs
drift. → We moved executable truth into tests/types/linters/CI and kept prose for what can't
be run. (DESIGN-DECISIONS §2)

**2. There were no tests.** "Verification" was a manual checklist plus `npm run build` —
which means *it compiles*, not *it's correct*. Notably, the methodology's own stated failure
mode ("the agent breaks things it shouldn't") is exactly what a regression suite prevents and
prose does not. → The `tdd` skill is the executor; CI enforces the checklist. (§2, §8)

**3. It loaded all context every session.** "Read all six files before doing anything" is a
large, mostly-wasted token cost on most tasks. → Router `CLAUDE.md`: load on demand by task
type. (§1)

**4. It was greenfield-and-solo biased.** Authored top-down for a new app by one person, with
no story for *jumping into existing code* — where the context already exists *as the code*. →
`derive-context` does the reverse direction, bottom-up. (§9)

**5. The manual three-prompt loop ignored automation.** Copy-pasting implement/correct/close
is itself an attention and token tax, and uses one model for everything. → `spec-unit`
automates the loop and routes work to the right-cost model; subagents isolate context. (§4)

**6. Trust calibration.** The original README claims "used in practice, not invented in
theory", yet its `code-standards.md` generation prompt is a copy-paste duplicate of the
`project-overview.md` prompt. A small thing — but it tells you the *templates* were a first
draft even where the *ideas* are field-tested. We treated the ideas as sound and rebuilt the
templates slim and stack-neutral.

---

## The one-line summary

We kept the methodology's thinking-first philosophy, invariants, scope discipline, session
memory, and spec-driven slices — and replaced its weak verification (prose + `npm run build`)
with tests + CI, its unconditional context loading with an on-demand router, its greenfield
blind spot with a brownfield path, and its manual single-model loop with complexity-based
model routing.
