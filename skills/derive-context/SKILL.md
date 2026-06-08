---
name: derive-context
description: Derive a slim, router-style context-file system from an EXISTING codebase by reading the code bottom-up, then generate a load-on-demand CLAUDE.md. Use when onboarding to a brownfield repo, when the user wants context/docs generated from existing code, mentions "derive context", "map this codebase", "set up context files", or wants the Six-File context system without authoring it by hand.
---

# Derive Context

Generate context files *from* an existing codebase (bottom-up), not from a planning
conversation. Output is **slim and router-based**: a tiny `CLAUDE.md` that loads each
context file *only when a task touches it*, so sessions don't pay tokens for context
they don't use.

Stack-agnostic. Detect the stack — never assume Next.js/JS.

## Workflow

### 1. Detect the stack (cheap, no full reads)
Glob for manifests and infer. Do NOT read source files yet.

| Manifest | Stack | Verify commands (confirm in scripts) |
|---|---|---|
| `Cargo.toml` | Rust | `cargo build`, `cargo test`, `cargo clippy` |
| `go.mod` | Go | `go build ./...`, `go test ./...`, `go vet` |
| `pyproject.toml`/`setup.py` | Python | `pytest`, `ruff`/`mypy` |
| `package.json` | JS/TS | read `scripts`: `build`/`test`/`lint`/`typecheck` |
| `pom.xml`/`build.gradle` | Java/Kotlin | `mvn verify` / `./gradlew build` |
| `*.csproj` | .NET | `dotnet build`, `dotnet test` |
| `Gemfile` | Ruby | `bundle exec rspec`, `rubocop` |
| `composer.json` | PHP | per `scripts` |

If multiple, it's a monorepo/polyglot — note each and its root. Read the *actual* verify
commands from the manifest (e.g. `package.json` scripts), don't guess them.

### 2. Map boundaries (sample, don't read everything — this is the token discipline)
- Glob the top 2 levels of the source tree to see the folder structure.
- Read only: the entry point(s), one representative file per top-level boundary, the
  router/route table or main wiring, and any existing `README`/`docs`.
- Infer what each top-level folder *owns*. Token budget: prefer Grep over Read; read whole
  files only when a sample is genuinely needed.

### 3. Generate context files into `context/`
Use the templates in [TEMPLATES.md](TEMPLATES.md). Rules:
- **Slim**: pointers and rules, not prose dumps. Each file earns its tokens.
- **Skip what doesn't apply**: no `ui-context.md` for a CLI/library/backend service. Six
  files may become four. State which you skipped and why.
- **Invariants**: infer them, but mark each `(inferred — confirm)` unless the code proves
  it (e.g. a check enforced at every boundary). Never assert an invariant you can't point at.
- **code-standards.md**: only capture rules a linter/formatter/types do NOT already
  enforce. If `clippy`/`ruff`/`eslint`/`tsconfig` enforces it, point at the config instead
  of restating it — that rule is then free and never drifts.
- **progress-tracker.md**: seed `Current Phase: onboarding`, list real next-up items if
  obvious from TODOs/issues, otherwise leave for the user.

### 4. Generate the router `CLAUDE.md`
Use the router template in [TEMPLATES.md](TEMPLATES.md). It must load context **on demand
by task type**, not all-at-once. Only `progress-tracker.md` is always-read (it's small and
task-relevant). Fill in the detected verify commands.

### 5. Report
- Stack detected + confidence.
- Files generated / skipped (with reason).
- Inferred invariants flagged for confirmation.
- Anything you couldn't determine from code (→ leave as open questions in the tracker).

## Do not
- Do not read the whole codebase to "be thorough" — that defeats the token goal.
- Do not hardcode JS/Next.js idioms into a non-JS project.
- Do not invent invariants or product intent the code doesn't support — flag gaps instead.
