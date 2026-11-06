---
name: cmd-finding
description: Scan PhotoCraft Rust command registrations and handlers to create or refresh a grouped Markdown command index with purpose, parameters, implementation evidence, and reverse links to invoking tools and menus.
---

# Find commands and their callers

Input: optional source checkout and feature scope. Output: `docs/commands/index.md`, reciprocal tool records in `docs/tools/index.md`, and a scan report.

Read [the shared contract](../../../workflows/source-documentation.md). Use [the command index template](../../../templates/command-index.md), [tool index template](../../../templates/tool-index.md), and [scan report template](../../../templates/scan-report.md).

1. Locate the actual registry through public exports. Follow every registration source, including `CommandSpec` literals, `cmd!` macros, `specs()`/`spec()` factories, loops, aliases, and feature/platform branches. Include shell commands and catalog-only IDs as distinct implementation classes. Inspect declarations that never reach registration and report them as unresolved/unregistered findings, not live commands.
2. Derive a concise purpose from each handler and the helpers it calls. Capture label, group, menu metadata, shortcut, parameter contract/defaults, enabled conditions, handler symbols, result/effects, and history/journal behavior. Do not infer behavior merely from an ID or equate journal recording with undo support. The index is a summary; detailed Rust walkthroughs belong to doctify-cmd.
3. Reverse-trace callers across the entire Rust workspace: literal IDs, computed dispatch tables, macros, helper wrappers, shell interception, tools/gestures, menu construction, panels, dialogs, shortcuts, CLI, control channel, automation, and command-to-command calls. Verify reachability before calling a string occurrence a caller. Mark tests as test evidence. Record generic ID-dispatch transports separately from concrete bindings. A command with no proven UI caller remains indexed as having none found in the stated scope.
4. Reconcile commands and affected tool records in both indexes. For each verified UI caller, create/update its tool anchor and the reciprocal binding, preserving the exact phase, params, and conditions. Shared protocol governs removals, restorations, details becoming stale, and human Notes. Do not assert a full UI inventory from a command-focused scan.
5. Compare discovered IDs against every registry/factory and shell/catalog source visited. Explain count differences (aliases, placeholders, dynamic registration, cfg branches). Write a scan report with unresolved and unvisited areas. Only a complete relevant scan can mark an absent command deleted.

Validate identity uniqueness, reciprocal bindings, local links, and lifecycle history. Report output links and added/changed/deleted/restored counts, with coverage limitations. Repeating on unchanged source must preserve records and links without duplication.
