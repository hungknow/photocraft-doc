---
name: tool-finding
description: Scan PhotoCraft Rust UI source to create or refresh a grouped Markdown inventory of toolbar tools, menus, panel actions, and their command bindings, preserving deleted items and reciprocal command links.
---

# Find UI tools and actions

Input: optional source checkout and feature scope. Default to the sibling PhotoCraft checkout. Output: `docs/tools/index.md`, reciprocal command records in `docs/commands/index.md`, and a scan report.

Read [the shared contract](../../../workflows/source-documentation.md) first. Use [the tool index template](../../../templates/tool-index.md), [command index template](../../../templates/command-index.md), and [scan report template](../../../templates/scan-report.md). Paths in the contract are relative to the documentation root; these reference links are relative to this file.

1. Discover UI declarations and actual interaction handlers: toolbar groups/flyouts, menu catalog and dynamic menus, panels and their context menus, options bars, dialogs, keyboard actions, and canvas gestures. Trace tool enum variants, selection dispatch, and pointer/key handling. Exclude separators and decoration. Keep disabled/conditional UI entries with their availability conditions.
2. For each action, trace from the widget/event through wrappers, dialogs, and shell interception to every command invoked. Record selection separately from gesture/preview/commit/cancel. Capture fixed params and how dynamic values are produced. If the handler directly changes UI state, record `DIRECT_UI` and the responsible symbol; do not invent an engine command. If dispatch cannot be resolved, retain the finding as `UNRESOLVED`.
3. Verify each exact command ID against registration and shell handling, including macro/table-generated IDs and conditional branches. A matching catalog string does not prove implementation. Group tools by user task and retain their surface/location so related actions are readable together.
4. Reconcile tool records and affected command records using the shared contract. Create missing command records with verified purpose/implementation and reciprocal caller links. Do not claim the command inventory is complete merely because all UI callers were scanned. Preserve existing detail links; add links to matching detail pages when present.
5. Write coverage and changes to the scan report. Mark deleted tools only after a complete relevant source scan establishes removal. If interrupted, save verified partial results and a continuation point without deleting unseen entries.

Validate both indexes' links, identities, reciprocal bindings, and lifecycle changes. Report added/changed/deleted/restored counts, unresolved bindings, coverage, and links to the outputs. Do not write detailed command explanations unless separately requested.
