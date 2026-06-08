# Credits

This workflow orchestrates several **existing, third-party skills** rather than reinventing
them. They are not part of this repo — install them from their original sources. Full credit
and thanks to their authors.

## Skills this workflow depends on

| Skill | Author | Source | Role in this workflow |
|---|---|---|---|
| `grill-me` | Matt Pocock | [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/productivity/grill-me) | Planning — stress-test the design before scaffolding |
| `grill-with-docs` | Matt Pocock | [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/engineering/grill-with-docs) | Planning against an existing domain model |
| `tdd` | Matt Pocock | [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/engineering/tdd) | The executor — builds each unit test-first |
| `to-issues` | Matt Pocock | [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/engineering/to-issues) | Decomposes the build plan into tracer-bullet issues |
| `impeccable` | Paul Bakaus | [pbakaus/impeccable](https://github.com/pbakaus/impeccable) | Polishes / audits UI on any interface unit |
| `frontend-design` | Anthropic | [anthropics/claude-plugins-public](https://github.com/anthropics/claude-plugins-public/tree/main/plugins/frontend-design) | Generates net-new, distinctive UI |
| `code-review` | Anthropic (Claude Code built-in) | [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) | The fresh-context reviewer of each diff |
| `update-config` | Anthropic (Claude Code built-in) | [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) | Wires the doc-sync hook into `settings.json` |

Matt Pocock's skills are installable as a set — see
[mattpocock/skills](https://github.com/mattpocock/skills).

## Methodology

The context-file system, spec-driven slices, and three-prompt loop are adapted and reworked
from JavaScript Mastery's *"From Idea to Product: The AI-Driven Developer's Playbook"* (the
Six-File Context System) — see [JavaScript Mastery](https://youtube.com/@javascriptmastery).
The critique that motivated the changes is in [CRITIQUE.md](CRITIQUE.md).

## This repo

The four orchestrating skills (`derive-context`, `scaffold-context`, `spec-unit`,
`scaffold-delivery`) and all docs are original to this repo, MIT-licensed.
