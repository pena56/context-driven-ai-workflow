---
name: scaffold-delivery
description: Scaffold the git/CI/deploy workflow for a project — branch convention, issue↔spec↔PR linking, CI that runs the stack's verify commands, a PR template derived from the spec checklist, manual-prod-via-release-tag deploys, and a doc-sync hook. Use when setting up CI/CD, GitHub workflow, branching, deployments, "set up the pipeline", "wire up CI", or connecting the spec/issue/PR loop. Stack-agnostic.
---

# Scaffold Delivery

Turn the context/spec workflow into a real delivery pipeline. Stack-agnostic: read the verify
commands from the router `AGENTS.md` (or detect them) — never hardcode `npm run build`.

Two sources of truth, kept distinct: **GitHub Issues = disk** (durable backlog, unit
tracking), **`progress-tracker.md` = RAM** (live session state). They link by number; they do
not duplicate.

## Workflow

### 1. Confirm the model
Defaults (override on request):
- **Trunk-based.** `main` is always-releasable. Short-lived `feat/N-name` branches per unit.
- **Environments via deploy targets, not long-lived branches.** PR → preview/staging deploy;
  `main` → staging; **release tag `vX.Y.Z` → production** (the human prod gate).

### 2. Branch + linking convention
Document the greppable chain in the repo (e.g. `CONTRIBUTING.md` or context):
`Unit NN ⇄ Issue #N ⇄ branch feat/N-name ⇄ spec specs/NN-name.md ⇄ PR "Closes #N"`
Branch from `main`; PR body must contain `Closes #N` so merge auto-closes the issue and Git
records the commit↔issue link.

### 3. CI — make verification deterministic and token-free
Generate the platform's CI config (GitHub Actions `.github/workflows/` unless told otherwise).
Two workflows:
- **on PR + on push to `main`** → run verify: `[TYPECHECK] · [LINT] · [TEST] · [BUILD]` (from
  router CLAUDE.md), then deploy to **staging/preview**.
- **on tag `v*`** → run verify, then deploy to **production**.
CI runs exactly the spec's "Verify when done" checklist, so green CI == satisfied checklist.
The agent reads CI failures and fixes; it never merely attests the build passes.

### 4. PR template from the spec
Generate `.github/pull_request_template.md` that mirrors the spec's verify checklist + a
`Closes #` line + "invariants in architecture.md respected". The reviewer (use `code-review`
in a fresh context) checks the diff against it.

### 5. Issue template
Generate `.github/ISSUE_TEMPLATE/unit.md` matching the unit shape (goal, boundary, spec link)
so `to-issues` output and manual issues share a structure.

### 6. Doc-sync hook (enforcement, not hope)
Offer to wire a project hook via the `update-config` skill (project `.claude/settings.json`):
- **Stop hook**: if files in a tracked boundary changed, remind to update the matching
  context file + `progress-tracker.md`.
- Optionally, a reminder on merge to close the issue + trim the tracker entry.
This replaces the methodology's "remember to update the docs" with something that actually
fires. Keep it a reminder, not a blocker, unless the user wants it strict.

### 7. Report
Files created, the two CI triggers, the prod-gate (tag), and the hook (if wired). Show the
first release command: `git tag vX.Y.Z && git push --tags` → prod.

## Do not
- Do not create long-lived `dev`/`staging`/`prod` branches unless explicitly asked — targets,
  not branches.
- Do not auto-deploy `main` to production — production is gated behind the release tag.
- Do not hardcode stack commands — resolve them from the router CLAUDE.md / manifest.
- Do not let the tracker and Issues both own the backlog — Issues are canonical.

## Credits
Orchestrates `to-issues` ([Matt Pocock](https://github.com/mattpocock/skills/tree/main/skills/engineering/to-issues))
and the Claude Code built-ins `code-review` and `update-config`.
