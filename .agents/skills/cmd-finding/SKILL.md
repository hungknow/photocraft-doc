---
name: cmd-finding
description: Scan PhotoCraft Rust command registrations and handlers to create or refresh a grouped Markdown command index with purpose, parameters, implementation evidence, and reverse links to invoking tools and menus.
---

# Find commands and their callers

Input: optional source checkout and feature scope. Output: category directories at `docs/commands/index.md` and `docs/tools/index.md`, command records in `docs/commands/<category-slug>.md`, reciprocal tool records in `docs/tools/<category-slug>.md`, and a scan report.

Read [the shared contract](../../../workflows/source-documentation.md). Use [the command index template](../../../templates/command-index.md), [tool index template](../../../templates/tool-index.md), [tool category template](../../../templates/tool-category.md), [command category template](../../../templates/command-category.md), and [scan report template](../../../templates/scan-report.md).

Keep directories and category files compact: category navigation at the root and summary tables in category files. No Lifecycle/Implementation columns, detailed records, Historical bindings, or Notes. Link to real Markdown headings using the shared contract; never use custom HTML record anchors. Keep detailed explanations exclusively in requested doctify-cmd pages. Update both sides of relationships and category entry counts. Migrate old detailed indexes using the shared contract.

1. Discover the actual registry through public exports. Follow literals, macros, spec factories, loops, aliases, shell handlers and platform/feature branches. Distinguish registered, shell-only, catalog-only and unresolved cases in investigation/reporting; do not add an implementation column.
2. Derive a one-sentence purpose from each handler and important helpers. Reverse-trace concrete callers through UI widgets, dialogs, gestures, menus, shortcuts, wrappers and generated routes. Generic transports and tests are not specific UI callers. Record evidence and unresolved routes in the scan report.
3. Write the three-column command summary table. The ID links to the actual heading of an existing detail page; otherwise render it as plain text. Caller names link to their owning tool category headings. Do not write parameters, structure explanations, code excerpts or detailed per-command sections in indexes.
4. Update reciprocal tool summary rows and directories. Preserve stable identities, existing detail links and unrelated rows in partial scans. Apply shared deletion/restoration rules; only a complete relevant scan can establish deletion. A command-focused scan does not establish a complete tool inventory.
5. Reconcile coverage against registries/factories and write the scan report. Validate unique rows, reciprocal relationships, actual heading links and category totals. Report output links, changes and unresolved scope.
