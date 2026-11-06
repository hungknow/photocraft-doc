# Source documentation contract

Read this for every finding or command-documentation run. Paths are relative to this documentation repository unless marked as source paths.

## Scope and evidence

Default source is sibling `../photocraft`; accept an explicit override. Verify its Cargo workspace and read its AGENTS.md and relevant docs. Treat source as read-only. Read the category directories, relevant tables, existing detail pages, before updating documents. Record revision, dirty state, date, scope, platforms/features, and unresolved areas in the affected category/index metadata or command page. A partial scan must never be presented as complete.

Discover current paths with `rg --files`. Follow module exports, registration functions, macros, factories, tables and actual call sites. A string occurrence or menu declaration does not prove a command is registered or reachable. Test references are test evidence, not production callers. Do not execute editing commands against user documents to discover behavior.

Current starting points (verify each run): `crates/engine/src/cmd.rs` (`CommandSpec`, `command_specs`), `cmd/*_cmds.rs`, `lib.rs` (`cmd as commands`), `session/`, `shell_commands.rs`; UI `app/commands.rs`, `menus/`, `menu_catalog.rs`, `panels/toolbar.rs`, `tools.rs`, `tools/`, `panels/`, `input/`, dialogs and control dispatch. Search the entire Rust workspace for generated routes, CLI, automation and additional callers. Older prose referring to `commands.rs` may be stale.

## Index layout: tables only

`docs/tools/index.md` and `docs/commands/index.md` are short category directories. Use the tool-index and command-index templates: brief scan/coverage metadata and a table with Category and Entries only. Category links include the target heading fragment.

`docs/tools/<category-slug>.md` and `docs/commands/<category-slug>.md` contain a category heading, brief scan/coverage metadata, directory backlink, and one summary table. Use the tool-category and command-category templates:

- Tools: Tool/action, Surface and location, Commands.
- Commands: Command, Purpose, Invoking UI actions.

Do not add Lifecycle or Implementation columns, per-item headings, detailed records, parameter tables, source excerpts, Historical bindings, Notes, or other detail sections to any index/category file. Keep purpose to one short sentence. Keep coverage and unresolved findings brief in the affected category/index metadata; full explanation belongs exclusively in a requested doctify-cmd page. In those pages, favor bullets and short sentences. Show only essential code; adapted Rust is allowed when clearly labeled and behavior is preserved. Put Input, Response and Explanation comments in every shown code snippet. Use pinned GitHub source links with verified line fragments for all committed code excerpts. Do not generate detail pages just to give an index row a link.

Use coherent categories such as File/document, Editing/history, Selection, Layers/masks, Painting/retouching, Transform/crop, Color/adjustments, Filters, Type, Vector/shapes, View/workspace, Automation and Other. Preserve existing assignments unless evidence supports a move. Derive filenames from category names using lowercase ASCII words separated by hyphens, for example `layers-masks.md`. Preserve existing filenames. Resolve empty/reserved names (`index`, `cmd-` prefix) and collisions with a unique descriptive slug rather than overwriting another file.

Give category pages a plain heading derived from the slug, such as `# Layers masks`, whose standard Markdown anchor is `#layers-masks`. One item has one canonical category row. Category labels in directory tables may retain their readable spelling, such as Layers/masks. Retain category-specific source/coverage metadata when only other categories are refreshed.

## Identity and relationships

Use the exact case-sensitive command ID. Do not normalize or merge aliases. A UI item is an actionable toolbar tool/flyout, menu entry, panel action, options control, context-menu action, dialog control or keyboard action; exclude decoration/separators. Identify tools by stable source keys (for example `toolbar:Brush`, `menu:Edit/Undo`, `panel:Layers/delete`), not line numbers. Preserve these keys in a compact HTML comment inside the tool-name cell (`<!-- tool-key: toolbar:Brush -->`); comments must remain within the row and use safely escaped text without raw `--` or angle brackets. This is identity metadata, not a Notes section. Do not add hidden detailed records.

Relationships are many-to-many. Verify selection versus gesture/preview/commit/cancel, parameter derivation, dispatch interception and feature conditions from code. Summarize only the relevant tool/command links in tables; record material uncertainties concisely in the affected category coverage line. Generic ID-dispatch transports are not specific UI callers. Direct UI state changes may have no command; say `Direct UI action` rather than inventing an ID. Keep unresolved routes explicitly uncertain. Distinguish engine, shell and catalog-only behavior during investigation without adding implementation columns to indexes.

## Links to actual headings

Use ordinary relative Markdown links to real Markdown headings. Do not emit `<a id=...>` anchors, `~HH` record fragments, or self-links to nonexistent per-item headings.

- Directory category link: `[Editing/history](editing-history.md#editing-history)`.
- Tool → command without a detail page: `[edit.undo](../commands/editing-history.md#editing-history)`; the label identifies the row to find within the table.
- Command → caller: `[Undo](../tools/editing-history.md#editing-history)`; disambiguate repeated tool labels with their surface/location.
- Command detail page: use `# edit.undo` as its title, with the human-readable label in the introduction. The standard heading anchor is `#editundo`. Link as `[edit.undo](cmd-edit.undo.md#editundo)` from the command category and `../commands/cmd-edit.undo.md#editundo` from tool categories. For other IDs, derive the fragment from the actual heading using the viewer's standard GitHub-style heading rules, not from the filename. Prefer a plain file link if a reliable fragment cannot be established.
- An undocumented command name in its own category table is plain text (or inline code), not a fabricated link. When doctify-cmd creates its detail page, replace that name with the detail-heading link and update every corresponding command link in tool tables.

Keep detail filenames stable: encode ID UTF-8 bytes, retaining ASCII letters/digits/`_`/`-`/`.` and replacing other bytes with `~HH`, then prefix `cmd-`. This filename encoding is not a heading-anchor algorithm. Handle case-insensitive filename collisions by fully byte-encoding the colliding IDs and updating links together. Link only to existing pages. Command detail pages link back to both category directories and the relevant category headings for their command/callers.

## Reconciliation

Both finding skills maintain reciprocal relationships in both sets of category tables without recursively invoking each other. Match exact command IDs and stable tool keys. Add new rows, refresh summaries/current links, preserve existing detail links, and keep unrelated rows during scoped scans. For verified label/location changes retain the tool key. A command-ID rename is a new row plus a deleted old row, with a replacement link only if verified.

Mark a missing item **DELETED** beside its name only after a complete relevant scan confirms absence across registries, handlers, dynamic factories and feature/platform branches. Do not add a lifecycle column. Disabled/hidden entries remain; an unimplemented but present catalog item is not deleted. A partial scan, failed search or single-platform dump cannot establish removal. Retain deleted rows and existing detail pages; annotate affected detail pages. A restored item reuses its identity and loses the DELETED marker. Record the verified source revision and date in the affected category metadata.

Replace removed current relationships on both sides only within verified scope; keep category coverage and revision metadata accurate. Do not retain Historical bindings or Notes sections. Mark an existing detail page STALE when its implementation/params/bindings demonstrably changed; a source revision change alone does not establish staleness. Findings skills do not rewrite detailed walkthroughs. Repeated scans of unchanged source must not duplicate rows or generate needless content changes.

Update only the requested indexes, category pages, and command detail pages. Keep revision, date, coverage, and a short unresolved-scope statement in affected page metadata. If a full scan cannot finish, mark affected pages PARTIAL and give a brief continuation point there. Do not create `docs/scans/`, `log/`, or any separate run report.

## Migrating existing detailed indexes

Extract the existing summary rows and stable tool keys before removing detailed sections. Keep every tool/command row, category assignment, DELETED marker and existing detail link. Remove lifecycle/implementation columns, full records, old custom anchors, Historical bindings and Notes. Do not turn removed record prose into unsolicited detail pages. Rewrite all repository links to category headings or existing detail headings. Preserve the recorded coverage: layout cleanup is not a source scan. For legacy monolithic indexes, split groups into category tables first and replace the root indexes with category directories. Repeating a completed migration must make no changes.

## Validation

Check unique command IDs/tool keys, one canonical row per item, reciprocal current relationships, category entry totals, every local target and actual heading fragment, no old custom record links, no unresolved template tokens in generated docs, and no forbidden detailed sections/columns in indexes. During migration compare before/after identity and relationship sets. Check heading links for duplicate labels and commands with punctuation. Validate that undocumented commands stay plain text and documented commands link to their actual detail heading. Report checks actually performed and remaining gaps. No Rust build is needed for documentation-only changes.
