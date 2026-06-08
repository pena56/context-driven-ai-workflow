# Reference

## Complexity rubric

Score the unit on four signals, then map to a route. Keep it cheap — this is a quick gate,
not a reasoning exercise.

| Signal | trivial = 0 | standard = 1 | complex = 2 |
|---|---|---|---|
| Files / boundaries touched | 1 file | 1 boundary, few files | crosses boundaries |
| New dependency? | no | no | yes (new dep = new surface) |
| Net-new logic | none (rename/format/move) | contained, well-trodden | novel / algorithmic |
| Spec ambiguity | none | fully resolved | open questions remain |

Route:
- **Any signal = 2, or two+ signals ≥1 with ambiguity** → **complex**: split or grill first.
- **Mostly 1s, ambiguity = 0** → **standard**: Sonnet executor + Opus reviewer.
- **All 0s** → **inline**: build now on the cheap model, no subagents.

Why this exists: routing a `complex` (underspecified) unit to a cheap executor is the single
most expensive mistake — it re-reads everything, guesses, and produces drift. Spend Opus
reasoning *up front* on the spec; then execution is cheap and safe.

### Model routing summary
- Plan / spec / review → Opus (low token volume, high leverage).
- Execute → Sonnet (high token volume — biggest dollar lever).
- Pure mechanical → Haiku.
- Review in a FRESH context window — a reviewer that didn't write the code catches more.

---

## Spec file template

Save as `context/specs/NN-feature-name.md`.

```markdown
# Unit NN: [Feature Name]

Issue: #N · Branch: feat/N-feature-name · Boundary: [folder from architecture.md]
Route: [inline | standard | complex] — [one-line reason]

## Goal
[1–2 sentences. The concrete, verifiable output of this unit. Specific, not "create the X".]

## Design   ← presence of this section marks a UI unit → frontend-design / impeccable run at build
[Visual/structural decisions specific to this unit. Reference ui-context.md tokens where
relevant. OMIT this section entirely for non-UI units.]

## Implementation
### [Component / sub-section, one per file or concern]
[Exactly what to build and how — enough that "done" is unambiguous. No cross-boundary work.]

### [Next sub-section]
[…]

## Dependencies
- [package@VERSION (reason)] — only what THIS unit first needs; install just-in-time.
  Resolve VERSION from the live registry (`npm view <pkg> version`, `pip index versions`,
  `cargo search`, etc.) — never a version recalled from memory. "none" is valid.

## Verify when done
- [ ] [behavioural condition specific to this unit]
- [ ] [BUILD CMD] passes        ← from router CLAUDE.md, stack-specific
- [ ] [TEST CMD] passes
- [ ] [LINT/TYPECHECK CMD] clean
- [ ] No invariant in architecture.md violated
```

Notes:
- One feature may need one spec or three — let complexity decide, not a fixed rule.
- The verify checklist mirrors what CI will enforce, so a green CI == a satisfied checklist.
- Keep the spec tight: it is the handoff payload. Every extra paragraph is tokens the
  executor pays. Precise, not verbose.
