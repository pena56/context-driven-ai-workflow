# Using This Workflow With Other Agents

This system was built for Claude Code, but most of it is not Claude-specific. Here's exactly
what travels to Codex, Cursor, GitHub Copilot, and others — and what doesn't.

---

## Two layers

**Layer 1 — the method (agent-agnostic).** Plain markdown and conventions. Any agent that
reads files can use it:
- the context files (`context/*.md`) and spec files (`context/specs/*.md`)
- the **router** concept — load context on demand by task type, not all at once
- the spec-file pattern and the per-unit build loop
- the git / CI / issues=disk / tracker=RAM / trunk-+-tag workflow
- every decision in [DESIGN-DECISIONS.md](DESIGN-DECISIONS.md)

**Layer 2 — the mechanism (Claude Code-specific).** *How* procedures get triggered:
- **Skills** — the `SKILL.md` format, auto-trigger by description, progressive disclosure
- **Subagent model routing** — Opus/Sonnet/Haiku via the Agent tool
- **Hooks** — `settings.json`

Layer 1 ports as-is. Layer 2 needs translation (below).

---

## The one change that makes it portable: `AGENTS.md`

The scaffolders write the router as **`AGENTS.md`** (the emerging cross-agent standard) plus a
one-line `CLAUDE.md` that points to it. Result: the same router and the same context files
work across Claude Code, Codex, Cursor, and Copilot at once.

- **Codex** reads `AGENTS.md` natively.
- **Cursor** — point its rules (`.cursor/rules/`) at `AGENTS.md`, or paste the router in.
- **Copilot** — reference `AGENTS.md` from `.github/copilot-instructions.md`.
- **Claude Code** — the `CLAUDE.md` pointer sends it to `AGENTS.md`.

---

## Running the "skills" on a non-Claude agent

A `SKILL.md` body is just a **procedure written in markdown**. You don't rewrite it — you
re-house it:

- **Codex:** in your prompt, say *"Follow `skills/spec-unit/SKILL.md` for this unit."* No
  auto-trigger — you invoke it explicitly.
- **Cursor:** convert a `SKILL.md` into a Cursor rule or a `/command` that references the file.
- **Copilot:** add a line to `copilot-instructions.md` pointing at the relevant procedure.

What you lose without Claude Code's skill system:
- **Auto-triggering** — you choose the procedure manually instead of the agent matching it
  from a description.
- **Model routing** — single-model agents keep the *complexity gate* (build inline / split /
  re-plan) but skip the model switch, or map it to that agent's own tiers (e.g. reasoning
  effort). The fresh-context review still applies — run it as a separate pass.
- **Hooks** — replace the doc-sync hook with that agent's equivalent automation, or do it
  manually at unit close.

---

## What's fully portable vs. needs work

| Piece | Portable as-is | Needs translation |
|---|---|---|
| Context files, spec files | ✅ | |
| Router (`AGENTS.md`) | ✅ | |
| Git / CI / issues / deploy workflow | ✅ | |
| Decisions & rationale | ✅ | |
| Skill *procedures* (the markdown) | ✅ (point the agent at the file) | |
| Skill *auto-triggering* | | per-agent (manual invoke) |
| Model routing (Opus/Sonnet/Haiku) | concept ✅ | models + orchestration |
| Hooks | | per-agent equivalent |

**Bottom line:** the thinking, the structure, and the workflow are yours on any agent. Only
the convenience automation is Claude Code-flavored — and where it's missing, you do that step
by hand.
