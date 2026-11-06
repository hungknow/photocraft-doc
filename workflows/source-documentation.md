# Source documentation contract

Read this for every finding or command-documentation run. Paths below are relative to the documentation repository unless explicitly called source paths.

## Inputs and evidence

- Default source: sibling `../photocraft`; accept an explicit override. Locate the documentation root by this file and `AGENTS.md`, not by assuming the current working directory. Verify the source Cargo workspace before scanning. If absent, request its path without changing existing inventories.
- Read existing indexes, detail pages, and the latest scan report first. Record source revision (`git rev-parse HEAD`), dirty state (`git status --short`), date, scope, platform/features considered, and source root. Include untracked Rust source when relevant; distinguish it from committed evidence.
- Discover current module paths with `rg --files` and follow `mod`, `pub use`, macros, registration functions, tables, and call sites. Documentation and string search are starting points; neither establishes that an action is registered or reachable. Test-only references are evidence of tests, not production callers.
- Use source paths, symbols, and line spans for every nontrivial claim. Link local source files relative to the output page; describe line spans in text because local Markdown viewers may ignore line fragments. Optionally add immutable remote revision links only when the remote and committed content are verified. Dirty excerpts need local evidence, not misleading commit links.
- Do not execute editing commands against user documents to discover behavior. Read-only registry enumeration is optional; label its platform/feature limitations. No build is required merely to read source.

## Current source map (verify on each run)

The inspected checkout uses `crates/engine/src/cmd.rs` (`CommandSpec`, `command_specs`), `cmd/*_cmds.rs`, and `lib.rs` re-exporting `cmd as commands`. Older project prose refers to `commands.rs`; do not assume it still exists. Registrations also come from modules outside `cmd/`.

Trace engine admission/execution in `crates/engine/src/session/`, shell metadata in `crates/engine/src/shell_commands.rs`, and actual shell handling in `crates/ui-egui/src/app/commands.rs` and `menus/`. Inspect `menu_catalog.rs`, `menus.rs`, `panels/toolbar.rs`, `tools.rs`, `tools/`, `panels/`, `input/`, dialogs, context menus, and control dispatch. Search all Rust workspace sources for additional call sites and generated/dynamic routes. Check CLI, automation, and control entrypoints for non-UI callers.

## Identity and relationships

Commands use the exact case-sensitive ID. Do not normalize dots or camelCase, merge aliases, or invent IDs for direct UI state changes. A single command record can describe multiple engine/shell routes, with their dispatch precedence. Record aliases separately and link their verified target.

Encode each ID as UTF-8 bytes: keep ASCII letters, digits, `_`, `-`, and `.`, and replace every other byte with `~HH` (uppercase hex, including `~` itself). Prefix filenames with `cmd-` and anchors with `cmd-`. Thus `edit.undo` uses `docs/commands/cmd-edit.undo.md` and anchor `cmd-edit.undo`. This is reversible, handles separators safely, and prevents slug collisions. Detect case-insensitive filesystem collisions before creating pages; if one occurs, encode the colliding IDs entirely as `~HH` bytes and update links together.

A tool record represents a discoverable UI action/location, not every decorative widget. Include toolbar tools/flyouts, menu items, panel actions, options controls, context menus, dialogs, and keyboard-only actions. Use a persistent key such as `toolbar:Brush`, `menu:Edit/Undo`, or `panel:Layers/delete`; choose a stable discriminator for duplicate labels. Keep the original key when an item moves or is relabeled if source evidence establishes continuity. Tool anchors use `tool-` plus the same byte encoding. Never derive identity from line numbers.

Relationships are many-to-many. Each binding records tool key, exact command ID, trigger/phase (select, gesture, preview, commit, cancel), params or parameter derivation, conditions, and source evidence. Preserve separate bindings when parameters or phases differ. The command record links back to the exact tool anchor. Generic `command_run(id)` transports are generic callers, not evidence of a specific UI tool. Follow computed IDs and macro/table expansions; list unresolved dynamic routes explicitly instead of guessing.

Use separate fields:

- Lifecycle: `ACTIVE` or `DELETED`, with first seen, last seen, and deletion scan/date. An unseen item in a partial scan retains its prior lifecycle.
- Implementation: `ENGINE`, `SHELL`, `ENGINE+SHELL`, `DIRECT_UI`, `CATALOG_ONLY`, or `UNRESOLVED`. A catalog entry without a handler is not implemented and is not deleted. For a tool, classify the actual route and describe multiple routes per binding.
- Availability: enabled predicate, platform/feature conditions, document prerequisites, and any unresolved reachability. A disabled or hidden item still exists.

Use coherent feature groups (File/document, Editing/history, Selection, Layers/masks, Painting/retouching, Transform/crop, Color/adjustments, Filters, Type, Vector/shapes, View/workspace, Automation, Other). Preserve established group assignments unless evidence supports moving them. Add groups as needed; do not group solely by source filename.

## Reconcile without losing history

1. Collect findings and coverage before modifying indexes. Use the index templates for new files and retain established record anchors in existing files. Both finding skills create/update both indexes sufficiently to preserve reciprocal relationships; share this procedure without recursively invoking one another.
2. Match by exact command ID or persistent tool key. Add new records, update evidence and bindings, preserve human-authored Notes, existing detail-page links, and first-seen values. A command-ID rename is a new record plus an old tombstone, with a replacement link only if verified.
3. Replace current bindings only within verified scan coverage. Move removed bindings into a historical-bindings subsection on both records, with a removal scan and reason; do not leave them looking active. Missing evidence is `UNRESOLVED`, not evidence of removal.
4. Mark an absent record **DELETED** only after completing discovery over all relevant source modules, registries, feature/platform branches, dynamic factories, and handlers. Confirm absence by searching its ID/key and tracing possible replacements. A failed search, unavailable dependency source, incomplete scan, or single-platform registry dump cannot establish deletion. Preserve its anchor, previous purpose, historical bindings, and detail page; annotate that page as deleted. Restored records reuse their key/anchor and keep lifecycle history.
5. Missing command handlers behind still-present menu entries become `CATALOG_ONLY`; the UI entry remains active. Handler removal is recorded in history. Never equate loss of a caller with removal of the command.
6. Write a scan report using the template. If a full scan cannot finish, publish only verified partial results, list remaining modules/routes, and provide a continuation point. Never silently label a partial inventory complete. A scan ID is `YYYY-MM-DD-NN`; choose the next unused numeric suffix.
7. On finding-skill refresh, mark an existing detail page `STALE` when its documented implementation, parameters, or bindings changed. Preserve its verified-at revision until doctify-cmd actually refreshes the explanation. An unrelated source revision change alone does not establish staleness. Record uncertainty if comparison is unavailable.
8. Sort groups and records deterministically. Repeating a scan on unchanged source must not duplicate records, append redundant lifecycle events, or change content beyond scan metadata. Review both indexes and changed pages together before finishing.

## Links and validation

Use explicit HTML anchors (`<a id="cmd-edit.undo"></a>`) immediately before record headings. Within `docs/tools/index.md`, link commands as `../commands/index.md#cmd-edit.undo`; within the command index, link tools as `../tools/index.md#<tool-anchor>`. Link detail pages only when they exist. Keep both the command-index link and the detail link in tool records so readers can navigate either route. Detail pages link back to both indexes and all specific callers.

Before finishing, verify unique keys/anchors, existing local link targets and fragments, exact reciprocal current bindings, separated historical bindings, preserved notes and tombstones, and no unresolved template tokens in generated documents. Check that counts reconcile with record lifecycles and implementation classes. Review examples: one tool with several commands, one command with several tools, a shell-only action, a catalog placeholder, and a dynamic caller. State which checks were performed and what remains unresolved.

## Template use

Replace every `{{...}}` slot; omit sample rows and repeat record sections as needed. Unknown facts must say `Unresolved` with the evidence gap. `Not applicable` needs a reason. Keep human Notes separate from source-derived sections. Templates are structural guides, not permission to manufacture missing facts.
