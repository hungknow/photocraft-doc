# file.open

- Open a file or a set of picked files, with each admitted document in its own tab.

Verified against: `3d60ad01f941519307c755eed8e0896d6274cde6` · 2026-11-06 (Rust source clean).
Navigation: [file commands](../file.md#file) · [File document tools](../../tools/file-document.md#file-document) · [Command categories](../index.md#categories) · [Tool categories](../../tools/index.md#categories)

## Context to use

No document or active layer is required. Opening creates a separate document; it does not change the current selection, layer, mask or history of an existing document.

- **File > Open…** and **Cmd+O** invoke `file.open` without a path and request a multiple-file picker. See [menu catalog](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/menu_catalog.rs#L8) and [dialog route](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/file_dialog.rs#L243-L257).
- An explicit `path` goes directly to the shell's path opener. See [invoke route](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/menus/invoke.rs#L233-L239).

For headless document import, use the engine's supported import API: `file.open` is registered as a shell command and has no engine `run` handler. See [shell registry](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/shell_commands.rs#L28) and [job ID](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/jobs.rs#L833-L834).

## Inputs and response

An invocation with a path on the desktop can be sent as:

```json
{"id":"file.open","params":{"path":"/pictures/example.psd"}}
```

- `path`: optional string. Without a string path, the command opens the picker; a wrong-type value also takes that route.
- No format, color, depth, layer or selection parameter is accepted by this shell route. Import uses the file contents and current import/color preferences.

With `path`, the immediate JSON result is `{"warnings":[]}` when no warnings were reported; a background job can still be running. Without `path`, the immediate result is `{"fileDialog":"open"}`; the later picker callback returns JSON `null`. See [return route](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/menus/invoke.rs#L515-L517) and [picker](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/file_dialog.rs#L243-L257).

An empty or unreadable path fails during file reading; web builds reject filesystem paths. A picker with no dialog service, or while another dialog is open, returns an error. Cancel leaves the document set unchanged. Per-file picker failures are reported individually while other selected files continue. See [path handling](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/file_open.rs#L190-L220), [dialog admission](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/file_dialog.rs#L103-L129), and [batch loop](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/file_open.rs#L227-L249).

## Structures used

Original source: [dialog types](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/file_dialog.rs#L24-L52); [open source](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/jobs.rs#L824-L834); [job tab](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/jobs_ui.rs#L265-L285).

**Adapted Rust** — unrelated dialog variants and open-tab fields omitted.

```rust
enum FileDialogRequest {
    Open { multiple: bool, initial_dir: Option<String>, extensions: Option<Vec<String>> },
    // For File > Open: multiple=true; extensions=None means all supported files.
}
enum FileDialogAnswer {
    Paths(Vec<String>),                 // e.g. ["/pictures/example.psd"] on desktop
    ContentsMany(Vec<(String, Vec<u8>)>), // web picker: name and file bytes
    Unavailable(String),               // e.g. picker service failure
}
enum OpenSource {
    Bytes(Arc<Vec<u8>>),                // already selected bytes
    Path(String),                      // worker reads "/pictures/example.psd"
}
```

## Core logic flow

### 1. Route and admit

The shell route is enabled without an active document. Automation authorization and discard interception run before it. See [enabled state](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/menus/state.rs#L64) and [invoke guard](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/menus/invoke.rs#L42-L51).

Original source: [file route](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/menus/invoke.rs#L233-L239).

**Adapted Rust** — direct branch only.

Purpose: Choose a path open or defer to the picker.

```rust
match params.get("path").and_then(Value::as_str) {
    Some(path) => open_path(app, path), // Result<Value, String>; e.g. Ok({"warnings":[]})
    None => app.open_dialog_file(),     // Result<Value, String>; e.g. Ok({"fileDialog":"open"})
}
```

### 2. Read, decode and wait for approval

Original source: [path opener](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/file_open.rs#L190-L209); [byte import](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/app/files.rs#L82-L142); [job start](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/jobs_ui.rs#L265-L285).

**Adapted Rust** — combines path, import and job branches; error propagation retained.

Purpose: Avoid duplicate opens, then import with the document's real format and color data.

```rust
if app.show_if_open(path) { return Ok(Vec::new()); } // Focus existing document or pending tab.
if app.background_jobs && !is_preset(path) {
    start_open(app, name, Some(path.into()), OpenSource::Path(path.into()), slot, order)?;
    return Ok(Vec::new()); // Job may finish later or require memory/profile approval.
}
let bytes = photocraft_format::read_file(Path::new(path))?; // Result<Vec<u8>, _>
if let Some(preset_result) = preset_files_ui::open(app, path, &bytes) {
    return preset_result.map(|()| Vec::new()); // Preset library, no document or recent entry.
}
let (doc, warnings) = import_with_options(&bytes, app.session.prefs())?;
// PSD/TIFF may instead start a job for memory approval; color policy, damage
// confirmation and Camera Raw may hold the decoded document before admission.
open_prompt::arrive(app, held_document(doc, path, warnings.clone()));
Ok(warnings)
```

The `import_with_options` and `held_document` calls above are **explanatory pseudocode**, condensing the typed PSD/TIFF and generic importer branches in the linked source.

### 3. Complete admission

Original source: [job completion](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/jobs_ui.rs#L662-L719); [after open](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/jobs_ui.rs#L725-L759); [recent/path update](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/file_open.rs#L158-L170).

**Adapted Rust** — omits tab placement and notice formatting.

Purpose: Admit the completed document and record its source after any approval prompts.

```rust
if let Some(preflight) = result.get("pendingPreflight") {
    arrive_memory_prompt(preflight); // User may reject; no document admitted yet.
} else if let Some(held) = result.get("held") {
    arrive_profile_or_damage_prompt(held); // Approval or cancel determines admission.
} else {
    let doc_id = result.get("docId").and_then(Value::as_u64).ok_or("missing document identity")?;
    let index = find_document(doc_id).ok_or("opened document is missing")?;
    app.session.set_active(index);
    if let Some(path) = path { app.opened_from(path); } // Template/read-only source is not Save target.
    app.sync_views();
    show_warnings_and_camera_raw_if_needed();
    document_opened(&mut app.session); // Fires configured Open Document script events.
}
```

These helper calls are **explanatory pseudocode** for the linked completion code. Import warnings, source path and Open Recent update only after admission; no edit is made to an existing document.
