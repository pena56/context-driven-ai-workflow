# The Workflow, End to End

How the skills, your existing skills, and the git/CI pipeline fit into one loop.

---

## The map

```
NEW project:        grill-me ─→ scaffold-context ─→ scaffold-delivery ─┐
EXISTING project:   derive-context ─→ scaffold-delivery ──────────────┤
                                                                       ▼
                              per unit:  spec-unit  (scores complexity)
                                          ├─ trivial  → build inline (cheap model)
                                          ├─ standard → tdd (Sonnet)
                                          │              → frontend-design / impeccable (if UI)
                                          │              → code-review (Opus, fresh context)
                                          └─ complex  → split / grill-me first
                                                  ▼
                              CI verifies → PR "Closes #N" → merge → staging
                                                  ▼
                                       git tag vX.Y.Z → production
```

---

## Phase 1 — Plan (your thinking, externalized)

Use **`grill-me`** (or **`grill-with-docs`** against an existing domain) until you can answer,
without hesitation: what it does, who it's for, the core flow, the hard parts, what's out of
scope, the stack and why. Don't scaffold before this is resolved — the scaffolder turns a
*resolved* plan into files; it is not the planning step.

The product grill stops at *function* — it deliberately doesn't settle the **name**, the brand
personality, or the **visual direction**. Those are a second, focused grill that `scaffold-context`
runs at its top (step 0), since the name seeds the overview and the direction seeds the UI tokens.

## Phase 2 — Establish context

- **New project → `scaffold-context`.** Runs the **brand/naming/design grill**, asks your stack,
  **pins latest-stable versions from the live registry**, adopts the stack's **official lint/format
  config** (and notes unit 01 will wire it), asks your **convention preferences** (file-naming case,
  identifier naming, imports — routing each to config or `code-standards.md`), writes the slim
  context files (skipping `ui-context.md` for non-UI), emits the router `AGENTS.md`, and seeds
  `context/specs/` with a build plan (via `to-issues`).
- **Existing project → `derive-context`.** Infers the stack from manifests, maps boundaries by
  *sampling* (not reading the whole repo), writes the same slim files bottom-up, and flags any
  inferred invariants for you to confirm.

Either way you end with: `context/` (the slim files), a router `AGENTS.md`, and a build plan.

## Phase 3 — Wire delivery — `scaffold-delivery`

Branch convention, the issue⇄spec⇄PR link, CI that runs your stack's verify commands, a PR
template mirroring the spec checklist, tag-gated production deploy, and (optionally, via
`update-config`) a doc-sync hook that reminds you to update context files when a boundary
changes. Do this once per repo.

## Phase 4 — Build, one unit at a time — `spec-unit`

This is the loop you'll spend most of your time in:

1. **Establish the unit** — one visible result, one boundary. If it's a phase, split first.
2. **Resolve ambiguity** — undefined behavior becomes a question to you or an Open Question in
   the tracker. Never spec on a guess.
3. **Score complexity → route** (see rubric below).
4. **Write the spec** — five sections (Goal, Design, Implementation, Dependencies, Verify),
   saved to `context/specs/NN-name.md`. Per-unit deps are pinned to live-resolved versions.
5. **Link the issue** — `Unit NN ⇄ #N ⇄ feat/N-name ⇄ specs/NN-name.md ⇄ PR "Closes #N"`.
6. **Dispatch:**
   - inline → build directly.
   - standard → **`tdd`** (Sonnet) builds test-first; if the unit has a `## Design` section,
     **`frontend-design`**/**`impeccable`** handle the UI; then **`code-review`** (Opus, fresh
     context) checks the diff against the invariants.
   - complex → stop; split or grill.
7. **Close** — mark the unit complete in the tracker, branch, PR `Closes #N`, promote durable
   rules to invariants / decisions to ADRs.

## Phase 5 — Ship

Merge → CI → staging. When you're ready for users: `git tag vX.Y.Z && git push --tags` →
production. The tag is the gate.

---

## Model-routing rubric (inside `spec-unit`)

Score four signals, cheaply:

| Signal | trivial = 0 | standard = 1 | complex = 2 |
|---|---|---|---|
| Files / boundaries | 1 file | 1 boundary | crosses boundaries |
| New dependency? | no | no | yes |
| Net-new logic | none | contained | novel/algorithmic |
| Spec ambiguity | none | resolved | open questions remain |

- any 2, or multiple ≥1 with ambiguity → **complex** (split/grill)
- mostly 1s, no ambiguity → **standard** (Sonnet build, Opus review)
- all 0s → **inline** (cheap model)

**Model assignment:** plan/spec/review → Opus (low volume, high leverage); execute → Sonnet
(high volume, biggest cost lever); pure mechanical → Haiku. Review always in a **fresh**
context window.

---

## Why this beats "just prompt the agent"

- The agent never starts from zero — the router `AGENTS.md` + tracker restore exactly the
  context the task needs, and nothing it doesn't.
- Drift is caught by CI and a fresh-context reviewer, not by hoping the agent remembers.
- Cost scales with task difficulty, not with a flat "always use the biggest model".
- Design quality, the product's name/brand, current dependency versions, the stack's official
  lint config, and your naming conventions are all defaults captured up front — not things you
  remember to ask for.
