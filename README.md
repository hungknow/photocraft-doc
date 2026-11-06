# PhotoCraft source documentation

Three repository-local agent skills turn the Rust source in `../photocraft` into linked Markdown documentation. Open your agent in this repository to discover `.agents/skills`. If your client does not discover repository skills, ask it to read the relevant `SKILL.md` directly. Keep this repository's shared templates and workflows alongside the skills.

## Usage

```text
$tool-finding Scan ../photocraft and create or refresh the grouped tool index.
$cmd-finding Scan ../photocraft and create or refresh the command index and callers.
$doctify-cmd edit.undo
```

You can specify another source checkout or limit a scan to a feature area. Limited scans are recorded as partial and never mark unseen items deleted. Run either finding skill again to reconcile additions, changes, removals, and restored items. Each finding skill maintains both sides of discovered tool/command relationships; neither requires you to run the other first.

## Outputs

| File | Contents |
|---|---|
| `docs/tools/index.md` | Grouped UI tools/actions, locations, command links, and detail-document links |
| `docs/commands/index.md` | Grouped command IDs, purpose, implementations, invoking tools, and detail links |
| `docs/commands/cmd-<encoded-command-id>.md` | Purpose, use context, parameters, core flow, data structures, and actual Rust excerpts |
| `docs/scans/<scan-id>.md` | Source revision, coverage, exclusions, counts, changes, and unresolved findings |

These outputs are created when the skills run; this initial setup does not claim to have inventoried the application. Templates live in [templates](templates/), and update semantics live in the [shared workflow](workflows/source-documentation.md).

## Extend

Add `.agents/skills/<new-skill>/SKILL.md`, link to the shared workflow, define its inputs and output ownership, and register it in [AGENTS.md](AGENTS.md). Add a template if it produces a new kind of document. Existing command IDs and tool anchors must remain stable so new skills can link to them.
