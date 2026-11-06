# PhotoCraft source documentation

Three repository-local agent skills turn the Rust source in `../photocraft` into linked Markdown documentation. Open your agent in this repository to discover `.agents/skills`. If your client does not discover repository skills, ask it to read the relevant `SKILL.md` directly. Keep this repository's shared templates and workflows alongside the skills.

## Usage

```text
$tool-finding Scan ../photocraft and create or refresh the grouped tool index.
$cmd-finding Scan ../photocraft and create or refresh the command index and callers.
$doctify-cmd edit.undo
```

You can specify another source checkout or limit a scan to a feature area. Limited scans are marked partial in the affected index metadata and never mark unseen items deleted. Run either finding skill again to reconcile additions, changes, removals, and restored items. Each finding skill maintains both sides of discovered tool/command relationships; neither requires you to run the other first.

## Outputs

| File | Contents |
|---|---|
| `docs/tools/index.md` | Short category directory with entry counts and coverage |
| `docs/tools/<category-name>.md` | Summary table of tools, locations, and command links |
| `docs/commands/index.md` | Short category directory with entry counts and coverage |
| `docs/commands/<category-name>.md` | Summary table of commands, purpose, and callers |
| `docs/commands/<category-name>/cmd-<encoded-command-id>.md` | Command walkthrough in a folder matching its category file, with Rust structure examples |

Command categories use ID prefixes: `tools.defaultColors` is listed in `docs/commands/tools.md`, and its detailed page goes under `docs/commands/tools/`. Tool categories stay grouped by user task. The main indexes list categories only. Category files contain summary tables without lifecycle/implementation columns, detailed records, Historical bindings, or Notes. Detailed explanations are created only when you invoke `$doctify-cmd <id>`. Finding skills maintain this layout on every scan. Templates live in [templates](templates/), and update semantics live in the [shared workflow](workflows/source-documentation.md).

## Extend

Add `.agents/skills/<new-skill>/SKILL.md`, link to the shared workflow, define its inputs and output ownership, and register it in [AGENTS.md](AGENTS.md). Add a template if it produces a new kind of document. Keep exact command IDs and stable tool keys so new skills can reconcile existing rows. Links target category headings or existing command-detail headings.
