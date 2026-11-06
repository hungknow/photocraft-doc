---
name: tool-finding
description: Scan PhotoCraft Rust UI source to create or refresh a grouped Markdown inventory of toolbar tools, menus, panel actions, and their command bindings, preserving deleted items and reciprocal command links.
---

# Find UI tools and actions

Input: optional source checkout and feature scope. Default to the sibling PhotoCraft checkout. Output: category directories at `docs/tools/index.md` and `docs/commands/index.md`, tool records in `docs/tools/<category-slug>.md`, reciprocal command records in `docs/commands/<category-slug>.md`.

Read [the shared contract](../../../workflows/source-documentation.md) first. Use [the tool index template](../../../templates/tool-index.md), [command index template](../../../templates/command-index.md), [tool category template](../../../templates/tool-category.md), [command category template](../../../templates/command-category.md). Paths in the contract are relative to the documentation root; these reference links are relative to this file.

Record source revision, date, coverage, and unresolved scope in affected pages. Do not create `docs/scans/`, `log/`, or separate run reports.

Keep directories and category files compact: category navigation at the root and summary tables in category files. No Lifecycle/Implementation columns, detailed records, Historical bindings, or Notes. Link to real Markdown headings using the shared contract; never use custom HTML record anchors. Keep detailed explanations exclusively in requested doctify-cmd pages. Update both sides of relationships and category entry counts. Migrate old detailed indexes using the shared contract.

1. Discover toolbar groups/flyouts, menus, panels/context menus, options, dialogs, shortcuts and canvas gestures. Follow actual handlers; exclude decoration. Keep hidden/disabled entries with their scan limitations.
2. Trace selection separately from gesture/preview/commit/cancel through wrappers, dialogs and shell interception to command handlers. Verify exact IDs against registrations, macros/factories and conditional routes. Do not infer implementation from catalog strings. Direct UI state changes may have no command.
3. Group tools by purpose and write only the three-column tool summary table. Preserve stable tool keys in compact row comments. Link each command to its existing detail heading or owning command category heading. Keep unresolved scope brief in category metadata; use source links in requested detail pages.
4. Update reciprocal command summary rows and caller links, placing each command under its ID prefix before the first dot (`tools.defaultColors` → `tools`). Create missing category files/directories with partial coverage. A UI scan does not establish command inventory completeness. Preserve existing detail links and follow deletion/restoration rules; do not remove unseen rows during partial scans.
5. Validate row identities, reciprocal relationships, real heading links, and category counts. Report outputs, change counts, and remaining scope to the user. Do not generate detailed command pages unless requested.
