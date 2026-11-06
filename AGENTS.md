# PhotoCraft documentation workspace

This repository contains reusable agent skills and source-based documentation for the sibling `../photocraft` Rust project. Read its `AGENTS.md` and relevant architecture/development documentation when examining source. Treat the source checkout as read-only for documentation tasks.

## Skills

- [tool-finding](.agents/skills/tool-finding/SKILL.md): inventory UI tools and actions, group them by purpose, and link their command bindings.
- [cmd-finding](.agents/skills/cmd-finding/SKILL.md): inventory commands, their purpose, and callers.
- [doctify-cmd](.agents/skills/doctify-cmd/SKILL.md): explain one exact command ID using Rust excerpts and update both indexes.

All three use the [scan and reconciliation contract](workflows/source-documentation.md) and [templates](templates/). Follow that contract before changing generated records. Preserve human notes and historical deleted records. Do not claim a complete inventory from a partial scan.

New skills go in `.agents/skills/<name>/SKILL.md` with `name` and `description` YAML frontmatter. Reuse the shared contract and add a template only for a new document type. Register the skill here and add an invocation example to the README. Keep shared conventions in the contract rather than copying them into each skill.

For documentation-only changes, validate skill frontmatter, links/anchors, and reconciliation behavior; Rust build and UI snapshot gates apply only if Rust or UI source changes. Record completed work and remaining limitations in `log/devlog.md`.
