# Design Decisions

Every significant decision in this workflow, what it means, and *why* it was made. If you're
adapting this system, this is the file that tells you which parts are load-bearing and which
are preference.

---

## 1. Router, not loader

**Decision:** `CLAUDE.md` is a *router* that tells the agent which context file to read for
which kind of task. It does **not** instruct the agent to read all context files every
session.

**Why:** The original methodology says "read all six files in order before doing anything."
That unconditionally loads architecture + UI tokens + code standards on *every* session, even
to rename a variable. Over many sessions that's a large, mostly-wasted token cost with no
quality benefit. Only `progress-tracker.md` (small, always task-relevant) is always-read;
everything else loads on demand.

**Trade-off considered:** Re-reading context that prevents *one* drift-induced rework session
pays for itself many times over. So we don't minimize *tokens-per-session* — we minimize
*tokens-per-shipped-feature*. The router cuts dead-weight reads without starving the agent of
context it actually needs.

---

## 2. Truth lives where it can be run

**Decision:** Tests, types, linters, and CI hold everything executable. Prose context files
hold only what *can't* be executed — intent, "why", scope, invariants. `code-standards.md`
defers to the linter/formatter/type config instead of restating rules a tool already enforces.

**Why:** The original methodology trusts hand-maintained prose to stay aligned with code,
enforced only by the agent's goodwill ("remember to update the docs"). Prose drifts; there's
no mechanism that *fails* when it does. A rule a linter enforces costs zero tokens, never
drifts, and fails loudly. So we moved verification out of prose and into CI + the `tdd` skill.

---

## 3. Spec-driven, one unit at a time

**Decision:** Work is decomposed into **units** — one visible, verifiable result inside one
system boundary, buildable in a single session, each with a "done" checklist. Each unit gets
a spec file before any code.

**Why:** It's the difference between a build that feels fast at the start and chaotic by the
middle, and one that stays predictable end to end. It also makes the agent's surface area
small enough that it can't make sweeping wrong assumptions.

**Ordering rules:** dependencies first; security before functionality; backend before
frontend wiring; UI shells before real data; install dependencies just-in-time.

---

## 4. Auto-route by complexity + model routing

**Decision:** `spec-unit` scores each unit and routes automatically:

| Route | When | Dispatch |
|---|---|---|
| inline | 1 file, mechanical, zero ambiguity | build now on a cheap model |
| standard | 1 boundary, spec fully resolved | **Sonnet** executor → **Opus** fresh-context reviewer |
| complex | multi-boundary OR unresolved ambiguity | stop; split or grill first |

The skill always prints the route + a one-line reason; the user can override
(`force: inline` / `force: split`).

**Why:** Two distinct levers were separated. **Context isolation** (subagents discard their
scratch work; only the conclusion returns) saves *token count* in the main thread.
**Model routing** saves *dollars* by sending the high-volume execution phase to the cheaper
model while reserving Opus for the low-volume, high-leverage plan/review phases. The spec file
is what makes cheap execution safe — it's the compressed handoff payload that lets the
executor build reading only the spec + the boundary files it names, not all the context.

**Why auto (vs. ask each time):** chosen by the user for fewer decisions per unit. The escape
hatch (printed route + override) keeps a mis-scored unit visible and correctable. The worst
mistake this guards against: handing an *underspecified* unit to a cheap executor, which then
re-reads everything, guesses, and produces drift.

**Why review in a fresh context:** a reviewer that never saw the implementation reasoning has
no anchoring bias and catches more.

---

## 5. UI units always route through the design skills

**Decision:** Any unit that touches an interface automatically runs `frontend-design`
(net-new UI) and/or `impeccable` (polish/audit existing UI) during the executor step, before
review. The presence of a `## Design` section in the spec is the trigger.

**Why:** The user was having to invoke `impeccable` manually almost every time. Baking it into
the route makes good UX/UI the default, not an afterthought. Division of labor: the design
skills choose *quality*; `ui-context.md` constrains *consistency* (tokens).

---

## 6. Latest-stable versions, resolved live

**Decision:** Skills never write a version number from memory. `scaffold-context` (at project
birth) and `spec-unit` (per-unit installs) resolve the current stable release from the live
registry (`npm view`, `pip index versions`, `cargo search`, `go list -m -versions`, or web
search) and pin it, recording it in the stack table with a verified date.

**Why:** Training data is stale, so agents default to outdated versions. Resolving live fixes
that at the source. Caveat preserved: "latest stable" isn't always *right* (a brand-new major
may have breaking changes or thin ecosystem support) — so the version is surfaced, not silently
chosen, and the user can override.

---

## 7. Issues = disk, progress-tracker = RAM

**Decision:** GitHub Issues are the canonical, durable backlog (each unit = one issue).
`progress-tracker.md` is volatile session state — current unit, open questions found
mid-session, "resume here" notes. They link by number; they do not both own the backlog.

| | GitHub Issues (disk) | progress-tracker.md (RAM) |
|---|---|---|
| Role | durable backlog, unit tracking | live session scratchpad |
| Lifespan | permanent, linkable to PRs/commits | rewritten constantly |

**The link:** `Unit NN ⇄ Issue #N ⇄ branch feat/N-name ⇄ spec specs/NN-name.md ⇄ PR "Closes #N"`.

**Why:** Two sources of truth for "what's done / next" always drift. Giving them different
*jobs* makes them complementary instead of redundant.

**Decision flow:** open question (tracker) → resolved → ADR in `docs/adr/` (durable) → if it's
now a rule, promote to an invariant in `architecture.md`.

---

## 8. Trunk-based, manual prod via release tag

**Decision:** `main` is always-releasable; short-lived `feat/N-name` branches per unit.
Environments are **deploy targets, not long-lived branches**: PR → preview/staging, `main` →
staging, **release tag `vX.Y.Z` → production** (the human prod gate).

**Why:** Long-lived `dev`/`staging`/`prod` branches are overhead a solo/small team doesn't
need. Per-PR previews give isolated staging without a shared branch to keep in sync. The user
chose a manual prod gate (the tag) over auto-deploying `main`, so shipping to users is always
an explicit act. CI runs exactly the spec's "Verify when done" checklist, so green CI ==
satisfied checklist — verification is deterministic and token-free.

---

## 9. Stack-agnostic by construction

**Decision:** Nothing assumes Next.js or any language. Skills detect the stack from manifests
(or ask, greenfield), resolve verify commands from there, and skip inapplicable files (no
`ui-context.md` for a backend/CLI/library). The only hard dependencies are git + GitHub + some
CI runner, and even those degrade (local-only mode; fewer files for tiny projects).

**Why:** The source methodology's templates were JS-biased because its author is a JS shop.
The *design* is universal; only the leaf-level commands and deploy platform are stack-specific,
and those are inputs, not assumptions.
