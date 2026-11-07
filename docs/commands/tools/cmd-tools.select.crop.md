# tools.select.crop

Selects the Crop tool in the PhotoCraft UI so later canvas, options-bar, and keyboard gestures can frame, turn, and commit a crop.

Verified against: `3d60ad01f941519307c755eed8e0896d6274cde6` · 2026-11-07
Navigation: [tools commands](../tools.md#tools) · [Transform/crop actions](../../tools/transform-crop.md#transform-crop)

## Context to use

The shell can select Crop without an open document. The selection itself changes UI state only; a document edit happens only when a later gesture commits the pending frame through `image.crop`.

- Invoke `tools.select.crop` explicitly through a command palette, menu/automation route, or the app's control channel. The shell resolves the exact ID to `Tool::Crop`. See [tool ID route](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/tool_shortcuts.rs#L14-L18), [shell dispatch](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/menus/invoke.rs#L77-L81), and [control dispatch](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/control/dispatch.rs#L89-L121).
- Press the default `C` group to choose or cycle Crop, Perspective Crop, Slice, and Slice Select. This default path assigns `ui.tool` directly and does not dispatch this ID. See [default C group](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/state/tool.rs#L262-L290) and [tool-letter dispatch](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/shortcuts.rs#L896-L925).
- Press a user-assigned shortcut for `tools.select.crop`. A non-empty assignment for this ID enters the shortcut command table and dispatches the exact ID. See [override table](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/shortcut_dispatch.rs#L73-L113) and [shortcut dispatch](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/shortcut_dispatch.rs#L238-L278).
- Click the Crop toolbar slot, Alt-click it to cycle the Crop flyout, or choose Crop in the flyout. These paths call tool selection directly and do not dispatch this ID. See [toolbar group](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/panels/toolbar.rs#L6-L18), [slot click](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/panels/toolbar.rs#L195-L212), and [flyout row](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/panels/toolbar.rs#L320-L328).
- After Crop is active, each canvas frame creates a default frame from selection bounds or the whole canvas. Pointer drags edit the frame, ratio/output fields constrain it, and Enter, a double-click inside it, the options-bar commit button, or Image ▸ Crop eventually runs `image.crop`. See [frame creation](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L405-L473) and [commit construction](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/gesture.rs#L253-L298).

In a headless engine session, the ID is listed for shortcut settings but has no engine `CommandSpec`, so direct `Session::execute("tools.select.crop", ..)` returns `UnknownCommand`. See [bindable tools](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/shell_commands.rs#L102-L117), [bindable item list](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/prefs/shortcuts.rs#L184-L193), and [engine command lookup](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/jobs.rs#L1509-L1519).

## Inputs and response

A control-channel request selects the tool through the UI shell:

```json
{"method":"engine.execute","params":{"command":"tools.select.crop","params":{}}}
```

- No fields are read by the tool branch; `{}` is the expected shape. Unexpected fields are ignored because this branch does not parse the params object.
- The default shortcut is `C`, shared with Perspective Crop, Slice, and Slice Select. A user assignment dispatches this exact ID; the default `C` path includes only tools that still use their default assignments.
- A document is not required to select the tool. A document is required for frame creation, crop commits, and most option actions that read document size or resolution.

The successful UI invocation returns `{"tool":"Crop"}`, sets `ui.tool` to `Tool::Crop`, clears temporary tool ownership, and makes no document or history edit. The selected tool and changed tool options are persisted later when the pointer is not down. Later gestures keep their own results and errors; the eventual commit is one `Crop` history edit.

- An automation authorization callback can reject an explicit or control-channel request before the tool changes. See [guarded invoke](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/menus/invoke.rs#L41-L56) and [control authorization](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/control/dispatch.rs#L107-L121).
- A differently cased or misspelled ID does not resolve to `Tool::Crop`. See [canonical ID match](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/tool_shortcuts.rs#L6-L18).
- Direct engine execution returns `UnknownCommand`; only the UI shell implements the tool-selection route. See [`Session::execute`](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/session/execute.rs#L8-L23).
- Later commit failures include an absent document, non-positive or over-large target dimensions, an angle without an explicit frame, and out-of-range geometry. `image.crop` reports these before its history edit; the UI restores the pending frame. See [commit restore](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/gesture.rs#L253-L298) and [engine validation](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/cmd/image_cmds.rs#L662-L734).

## Structures used

Original source: [bindable tool tuple](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/shell_commands.rs#L102-L117), [Tool enum](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/state/tool.rs#L3-L24), and [default key](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/state/tool.rs#L262-L290).

**Adapted Rust** — only the Crop variant and its C-key group are shown from the larger tool table and enum.

```rust
type BindableTool = (&'static str, &'static str, Option<&'static str>);

const CROP: BindableTool = ("tools.select.crop", "Crop Tool", Some("C"));

enum Tool {
    Crop,             // The target of this command; the toolbar shows glyph "C".
    PerspectiveCrop,  // Also defaults to C, but remains a distinct selected tool.
    Slice,
    SliceSelect,
}

impl Tool {
    fn key(self) -> char {
        match self {
            Tool::Crop | Tool::PerspectiveCrop | Tool::Slice | Tool::SliceSelect => 'C',
            // Other tools omitted.
        }
    }
}
```

Original source: [UI state](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/state.rs#L205-L216) and [crop fields](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/state.rs#L485-L503).

**Adapted Rust** — unrelated UI fields are omitted.

```rust
struct UiState {
    tool: Tool,                         // Tool::Crop after this command succeeds.
    group_tools: Vec<Tool>,             // Last-used tool per default-key group, e.g. [Tool::Crop].
    spring: Option<Spring>,             // None after an explicit tool choice.
    crop_rect: Option<[f64; 4]>,        // Some([20.0, 10.0, 80.0, 50.0]): x0,y0,x1,y1 document px.
    crop_angle: f64,                    // 15.0: pending frame turned 15° clockwise on screen.
    status: String,                     // Holds an image.crop error if commit is refused.
}
```

Original source: [CropState](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L43-L76), [CropStep](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L78-L88), [CropDrag](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L90-L114), and [Hit](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L116-L122).

**Adapted Rust** — private ownership fields are summarized and unrelated comments omitted.

```rust
struct CropState {
    drag: Option<CropDrag>,             // Some(CropDrag::Draw { .. }) during a frame drag.
    space: bool,                        // true while Space repositions an in-progress draw.
    default_frame: bool,                // true before the whole-canvas frame is edited.
    frame_from_selection: bool,         // true when the default frame is selection bounds.
    frame_for: Option<(usize, u32, u32)>, // Tab index and size the default frame belongs to.
    dismissed: Option<(DocId, u32, u32)>, // Document whose frame stays hidden after commit/Escape.
    reset_to: Option<[f64; 4]>,         // Frame Reset should return to, e.g. W×H×Resolution.
    editing: bool,                      // true after a press or other real edit.
    doc: Option<DocId>,                 // Owning document of a pending gesture/frame.
    straighten: Straighten,             // Straighten button, Command-held, and line state.
    undo: Vec<CropStep>,                // Crop-box-local undo stack.
    redo: Vec<CropStep>,                // Crop-box-local redo stack.
    press_step: Option<CropStep>,       // Frame state saved at pointer down.
}

struct CropStep {
    rect: Option<[f64; 4]>,             // Some([20.0, 10.0, 80.0, 50.0]).
    angle: f64,                         // 15.0 degrees.
    default_frame: bool,                // false after a drawn or edited frame.
    frame_from_selection: bool,         // false once the selection frame is edited.
    editing: bool,                      // true while pixels beyond the frame are shown.
}

enum CropDrag {
    Draw { anchor: [f64; 2], cur: [f64; 2], last: [f64; 2], prev: Option<[f64; 4]> },
    Move { start: [f64; 2], rect: [f64; 4] },
    Resize { hx: i8, hy: i8, start: [f64; 2], rect: [f64; 4], view: Option<PressView> },
    Rotate { center: [f64; 2], grab: f64, from: f64 },
    MoveImage { start: [f64; 2], rect: [f64; 4], view: Option<PressView> },
    TurnImage { center: [f64; 2], grab: f64, from: f64, rect: [f64; 4], view: Option<PressView> },
}

enum Hit {
    Handle(i8, i8), // Handle(-1, 1): left/bottom handle in the frame's own axes.
    Inside,         // A press inside the frame.
    Outside,        // A press outside the frame and its handles.
}
```

Original source: [Crop options](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/state/tool_options.rs#L161-L201) and [CropShield](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_shield.rs#L34-L69).

**Adapted Rust** — only fields used by Crop are retained from `ToolOptions`.

```rust
struct ToolOptions {
    crop_ratio: String,                   // "", "4:5", "original", or "whr".
    crop_custom_ratio_width: f32,         // 16.0 while a custom ratio is being typed.
    crop_custom_ratio_height: f32,        // 0.0 until the second side is entered.
    crop_delete: bool,                    // true by default: Delete Cropped Pixels.
    crop_content_aware: bool,             // true selects Content-Aware Fill.
    crop_width: String,                   // "4 in" or "1024 px"; empty means unset.
    crop_height: String,                  // "5 in" or "768 px"; empty means unset.
    crop_resolution: String,              // "300"; empty uses/keeps the document resolution.
    crop_resolution_unit: String,         // "px/in" (default) or "px/cm".
    crop_overlay: CropOverlay,            // CropOverlay::Thirds by default.
    crop_overlay_show: OverlayShow,       // OverlayShow::Auto by default.
    crop_overlay_orientation: u8,         // 0..=3, meaningful for Triangle/Golden Spiral.
    crop_shield: CropShield,              // Preview mode and shield settings.
}

struct CropShield {
    classic_mode: bool,                   // false: upright screen box, moving image.
    auto_center_preview: bool,            // true by default in default mode.
    show_cropped_area: bool,              // true by default; ignored as false in Classic Mode.
    enabled: bool,                        // true: dim the area outside the frame.
    color: ShieldColor,                   // MatchCanvas by default.
    custom_color: [u8; 3],                // [40, 80, 120] when Custom is chosen.
    opacity: f32,                         // 75.0 percent by default.
    auto_adjust: bool,                    // true: lighter while editing.
}
```

Original source: [Straighten state](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_straighten.rs#L25-L37) and [default-mode drag state](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_mode.rs#L186-L199).

**Adapted Rust** — constructors and private field visibility are omitted.

```rust
struct Straighten {
    armed: bool,                     // true after the options-bar button, until one line ends.
    held: bool,                      // true while Command/Ctrl is held over the canvas.
    line: Option<[[f64; 2]; 2]>,     // Some([[20.0,30.0],[70.0,32.0]]) while drawing.
}

struct LiveResize {
    rect: [f64; 4],                  // Frame at pointer down.
    deg: f64,                        // Frame angle at pointer down.
    hx: i8,                          // -1 for a left handle.
    hy: i8,                          // 1 for a bottom handle.
    start: [f64; 2],                 // Press point in the press-time view.
    q: [f64; 2],                     // Current point in the press-time view.
    ratio: Option<f64>,              // Some(1.6) for 16:10, or Shift's current ratio.
    alt: bool,                       // true to resize about the centre.
}

struct PressView {
    rotation: f32,                   // -15.0 while default mode shows a 15° frame upright.
    center: [f32; 2],                // Document-space camera centre at press.
}
```

## Core logic flow

Tool selection is UI state. Only the exact-ID and custom-shortcut routes dispatch `tools.select.crop`; default `C` and toolbar choices bypass the ID but select the same `Tool::Crop`. Every later crop gesture is driven by that state, and only its commit dispatches `image.crop`.

### Invoke `tools.select.crop` to select Crop

Original source: [canonical ID](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/tool_shortcuts.rs#L6-L18), [guarded invoke](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/menus/invoke.rs#L41-L56), [tool branch](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/menus/invoke.rs#L76-L81), [control channel](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/control/dispatch.rs#L89-L121), and [explicit tool selection](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tool_memory.rs#L24-L33).

**Adapted Rust pseudocode** — ID parsing, authorization, and state mutation are combined across functions with distinct signatures.

Purpose: trace one exact-ID request from authorization through canonical tool resolution to the JSON result.

```rust
let name = id.strip_prefix("tools.select.")?;       // Option<&str>: Some("crop") for this ID.
let tool = Tool::from_name(name)?;                  // Option<Tool>: None for "Crop" or "unknown".
let tool = (id == tool_id(tool)).then_some(tool)?;  // Rejects every noncanonical spelling.

if app.automation_input {
    authorize(id, &params)?;                        // Result<(), String>; Err selects nothing.
}
if req_method == "engine.execute" && engine_commands::find(id).is_none() {
    menus::invoke(app, ctx, id, params)?;           // Falls through to the UI shell route.
}

tool_memory::select(app, Tool::Crop);               // Takes ownership of the selected tool.
Ok(json!({"tool": "Crop"}))                         // Result<Value, String>; no document edit.
```

`tool_memory::select` clears the pen eraser return tool and spring-loaded state, cancels Frame-tool setup when leaving Frame, and assigns `app.ui.tool = Tool::Crop`. Selection is available without a document. The eventual preference write occurs from `tool_memory::persist` once the pointer is up, not as a document history step.

### Press the default `C` key to choose or cycle the Crop group

Original source: [C group](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/state/tool.rs#L262-L290), [default assignment gate](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/tool_shortcuts.rs#L20-L30), [repeat suppression](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/shortcuts.rs#L879-L887), and [group selection](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/shortcuts.rs#L896-L925).

**Adapted Rust** — the loop over all tools is reduced to the C group and spring-tool bookkeeping is summarized.

Purpose: show how the default key chooses, keeps, or cycles a tool without dispatching this command ID.

```rust
let shift_switch = prefs.tools.use_shift_key_for_tool_switch;
let group: Vec<Tool> = [Crop, PerspectiveCrop, Slice, SliceSelect]
    .into_iter()
    .filter(|tool| tool_shortcuts::uses_default(app, *tool)) // Custom IDs leave the default group.
    .collect();

let cycle_shift = shift_switch && consume(Modifiers::SHIFT, 'C');
if cycle_shift || consume(Modifiers::NONE, 'C') {
    app.ui.tool = match group.iter().position(|tool| *tool == app.ui.tool) {
        Some(i) if shift_switch && !cycle_shift => group[i], // Plain C keeps the current C tool.
        Some(i) => group[(i + 1) % group.len()],             // Otherwise cycle the group.
        None => app.ui.group_tool('C')
            .filter(|last| group.contains(last))
            .unwrap_or(group[0]),                            // Restore last used or first tool.
    };
    spring_tools::pressed(app, 'C', before, plain, now);     // Plain key may be spring-loaded.
}
```

Held-key repeats are removed before this selection, so holding `C` does not continuously cycle. If the plain key is spring-loaded, the old tool returns when it is released; a permanent choice clears the pen-eraser marker. No part of this default path invokes `tools.select.crop` or `menus::invoke`.

### Press a custom shortcut assigned to `tools.select.crop`

Original source: [override table](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/shortcut_dispatch.rs#L73-L113), [matching dispatch](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/shortcut_dispatch.rs#L238-L278), and [ordinary command dispatch](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/shortcut_dispatch.rs#L369-L403).

**Adapted Rust pseudocode** — table construction is represented by the relevant override entry.

Purpose: show that a user binding follows the exact command route rather than the default tool-letter path.

```rust
bindings.push(("tools.select.crop", assigned_shortcut));  // Non-empty prefs.shortcuts entry.
if key_matches(assigned_shortcut, key, modifiers) {
    if !menus::is_enabled(app, "tools.select.crop") {
        show_disabled_reason(app, "tools.select.crop");    // Status/error outcome; no selection.
    } else {
        let result: Result<Value, String> =
            menus::invoke(app, ctx, "tools.select.crop", json!({}));
        report_result(app, result);                        // Ok selects Crop; Err sets status.
    }
}
```

Because the dispatch goes through `menus::invoke`, authorization and the normal tool branch apply. The default `C` group skips this tool once an assignment exists in preferences.

### Click the Crop toolbar button or flyout row

Original source: [toolbar group](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/panels/toolbar.rs#L6-L18), [slot click](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/panels/toolbar.rs#L195-L212), and [flyout row](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/panels/toolbar.rs#L320-L328).

**Adapted Rust** — the toolbar UI and modifier-history details are omitted.

Purpose: show the direct toolbar paths that select Crop without dispatching the command ID.

```rust
const CROP_SLOT: &[Tool] = &[Tool::Crop, Tool::PerspectiveCrop, Tool::Slice, Tool::SliceSelect];

if button_clicked {
    let pick = if alt_click && CROP_SLOT.len() > 1 {
        next_slot_tool(CROP_SLOT, current_tool_in_slot()) // Alt/Option cycles the flyout.
    } else {
        shown_slot_tool                                  // Usually Crop when Crop was last shown.
    };
    tool_memory::select(app, pick);                      // Direct call; no menus::invoke.
}
if flyout_row_clicked_or_drag_released {
    tool_memory::select(app, Tool::Crop);                 // Direct flyout choice.
}
```

These paths activate the same Crop behavior and use the same explicit tool-selection cleanup, but they do not authorize or dispatch `tools.select.crop`.

### Drag to draw a new crop frame

Original source: [canvas routing](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/tool_input.rs#L187-L190), [early edit preview](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas.rs#L827-L835), [pointer admission](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L952-L968), [press branch](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L1012-L1033), [draw geometry](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L344-L351), [draw tracking](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L1146-L1167), and [release](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L1058-L1103).

**Adapted Rust pseudocode** — hit testing, drawing, tracking, and release are joined; unrelated drag variants are omitted.

Purpose: show the press-to-release path that replaces an untouched or absent frame with a drawn frame.

```rust
if app.ui.tool != Tool::Crop { app.crop.drag = None; return false; }
let point = [event.x, event.y];
if active_document().is_none() || !point.iter().all(|v| v.is_finite()) {
    return true;                                      // Event consumed; no frame or document edit.
}

on_down {
    show_hidden_frame();
    claim(active_doc_id);
    crop.editing = true;                              // Shows pixels beyond the frame immediately.
    crop.press_step = Some(step_of(app));
    if no_frame || hit_turned(frame) == Hit::Inside && crop.default_frame {
        app.ui.crop_angle = 0.0;                      // A newly drawn frame starts upright.
        crop.drag = Some(CropDrag::Draw {
            anchor: point, cur: point, last: point,
            prev: app.ui.crop_rect,                   // Restored if the release is too small.
        });
    }
}

on_move {
    if crop.space {
        anchor += pointer_delta; cur += pointer_delta; // Space pans the frame being drawn.
    } else {
        cur = point;
    }
    let ratio = preset_ratio(app);                     // None for free Ratio.
    let end = if ratio.is_some() {
        marquee_fixed_ratio_end(anchor, cur, ratio)    // A preset outranks Shift's square constraint.
    } else {
        marquee_end(anchor, cur, shift)                // Shift constrains a free draw to a square.
    };
    app.ui.crop_rect = Some(if alt {
        centered_rect(anchor, end)                     // Option/Alt draws outward from the anchor.
    } else {
        corner_rect(anchor, end)
    });
}

on_up {
    let valid = frame_width >= 1.0 && frame_height >= 1.0;
    if valid {
        crop.default_frame = false;                    // A drawn frame is a pending crop.
        sync_frame(); auto_center_if_enabled();        // Auto Center applies to a newly drawn box.
    } else {
        app.ui.crop_rect = draw.prev;                  // A click/collapse restores the prior frame.
    }
    if step_of(app) != crop.press_step {
        crop.undo.push(press_step); crop.redo.clear(); // One local box-undo step per gesture.
    }
}
```

The canvas marks `editing` on the initial press, before egui's drag threshold, so the beyond-canvas preview and shield appear immediately. A frame smaller than one document pixel in either dimension is never left behind.

### Drag a handle to resize the crop frame

Original source: [press branch](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L1012-L1033), [resize geometry](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L353-L390), [turned resize](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L208-L243), [live-centred resize](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_mode.rs#L186-L235), and [ordinary resize tracking](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L1172-L1188).

**Adapted Rust pseudocode** — the axis-algebra implementation is summarized while its constraints and outputs remain explicit.

Purpose: show how an edge or corner drag changes frame size under ratio, Shift, Option, turn, and Auto Center rules.

```rust
on_down if hit_turned(frame, angle, point) == Hit::Handle(hx, hy) {
    let press_view = if default_mode && auto_center_preview { Some(camera_at_press()) } else { None };
    crop.drag = Some(CropDrag::Resize { hx, hy, start: point, rect: frame, view: press_view });
}

let preset = preset_ratio(app);
let keep_current = (rect_width > 0.0 && rect_height > 0.0)
    .then_some(rect_width / rect_height);
let ratio = preset.or_else(|| if shift { keep_current } else { None });

if default_mode && auto_center_preview {
    let local_point = turn(pointer - start, -frame_angle);      // Work in the frame's own axes.
    let frame = resized_about_half_extents(frame, hx, hy, local_point, ratio, alt);
    let frame_center = opposite_edge_anchor_plus_growth();       // Option uses the old centre instead.
    app.ui.crop_rect = Some(frame);
    view.center = press_view.center + frame_center_shift;        // Box stays fixed on screen.
} else {
    let local_drag = turn(pointer - start, -frame_angle);
    app.ui.crop_rect = Some(resized_turned(
        rect, frame_angle, hx, hy, local_drag, ratio, alt,
    ));
}

on_release if width < 1.0 || height < 1.0 {
    restore_frame_at_press();                                    // Also restores a live-centred view.
}
```

A ratio preset always constrains the frame. Without a preset, Shift preserves the frame's current aspect ratio; Option/Alt resizes about its centre. A turned frame is resized in its own axes. The release uses the same one-document-pixel minimum and crop-box Undo step as drawing.

### Drag inside the frame to move the image or frame

Original source: [inside press](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L1012-L1044), [image offset](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_mode.rs#L318-L332), [move tracking](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L1129-L1133), [ordinary frame move](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L1168-L1171), and [mode choice](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_mode.rs#L37-L54).

**Adapted Rust** — mode selection and the two drag variants are combined.

Purpose: distinguish the default mode's moving image from Classic Mode's moving frame.

```rust
let drag = if moves_image(app) {                    // Crop selected and Classic Mode off.
    CropDrag::MoveImage { start: point, rect: frame, view: press_view() }
} else {
    CropDrag::Move { start: point, rect: frame }
};

match drag {
    CropDrag::MoveImage { start, rect, view } => {
        let p = read_pointer_in_press_view(app, view, point);
        let d = [p[0] - start[0], p[1] - start[1]];
        app.ui.crop_rect = Some([
            rect[0] - d[0], rect[1] - d[1],        // The frame moves opposite the image.
            rect[2] - d[0], rect[3] - d[1],
        ]);
        view.center -= d;                           // Camera follows, so the box stays on screen.
    }
    CropDrag::Move { start, rect } => {
        let d = point - start;
        app.ui.crop_rect = Some(rect + d);          // Classic Mode moves the turned frame on the page.
    }
}
```

An untouched default frame does not enter this branch: pressing inside it starts a new Draw gesture instead. A valid changed inside-drag makes the frame real and records one crop-box Undo step on release.

### Drag outside the frame to turn the image or frame

Original source: [outside press](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L1046-L1055), [angle snapping](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L191-L206), [turn tracking](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L1115-L1128), [default-mode camera](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_mode.rs#L73-L97), and [keep-inside geometry](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_mode.rs#L122-L184).

**Adapted Rust pseudocode** — the default and Classic branches are shown together, with `keep_inside` summarized.

Purpose: show how an outside drag changes `crop_angle` and which object visually turns in each mode.

```rust
let grab = atan2(point.y - center.y, point.x - center.x);
let drag = if moves_image(app) {
    CropDrag::TurnImage { center, grab, from: old_angle, rect: frame, view: press_view() }
} else {
    CropDrag::Rotate { center, grab, from: old_angle }
};

match drag {
    CropDrag::Rotate { center, grab, from } => {
        app.ui.crop_angle = turned_angle(center, grab, from, point, shift);
        // Shift snaps to 15° increments; Classic Mode shows the frame turning over the image.
    }
    CropDrag::TurnImage { center, grab, from, rect, view } => {
        let p = read_pointer_in_press_view(app, view, point);
        let sweep = atan2(p.y - center.y, p.x - center.x) - grab;
        let turned = from - sweep.to_degrees();     // The frame turns opposite the on-screen image.
        app.ui.crop_angle = if shift { snap_15_degrees(turned) } else { normalize(turned) };
        sync_frame();                               // Camera rotation becomes -crop_angle.
        keep_inside(rect, from, command_held);      // Move/shrink frame unless Command, CAF, or already outside.
    }
}
```

Both modes commit the same `crop_rect` plus `crop_angle`; the mode changes only the preview camera and which object appears to move. In default mode, an outside drag keeps the frame inside the turned image unless Command is held, Content-Aware Fill is selected, or the frame already extended beyond the image at press.

### Click Straighten and draw a line that fits the frame

Original source: [Straighten modes](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_straighten.rs#L25-L68), [line pointer](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_straighten.rs#L70-L100), [angle](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_straighten.rs#L103-L114), [fit](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_straighten.rs#L116-L159), and [Crop integration](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L984-L1009).

**Adapted Rust** — line drawing and frame application are condensed.

Purpose: trace the one-shot options-button flow that levels the image and fits a new frame.

```rust
if straighten_button.clicked { crop.straighten.toggle(); } // Arms for one line; a second click disarms.

on_down if crop.straighten.armed {
    crop.straighten.line = Some([point, point]);
    crop.editing = frame.is_some();
    show_hidden_frame(); claim(active_doc_id);
    crop.press_step = Some(step_of(app));             // The line result is one box-Undo step.
}
on_move { line.end = point; sync_frame(); }
on_up {
    let fit = take(crop.straighten.armed);            // Only the button's line fits the frame.
    let min_document_len = 3.0 / point_zoom();        // Shorter than 3 screen points does nothing.
    let Some(deg) = frame_angle(line.start, line.end, min_document_len) else { return; };

    let (w, h) = existing_frame.map_or(canvas_size, frame_size); // Keep frame/canvas proportions.
    let fitted = largest_centered_fitting_frame(w, h, canvas_width, canvas_height, deg)?;
    app.ui.crop_rect = Some(fitted);
    app.ui.crop_angle = deg;                           // Horizontal, or vertical when nearer plumb.
    crop.default_frame = false;
    crop.editing = true;
}
```

The line itself and its angle readout are preview-only. The resulting frame remains pending: Enter commits it, Escape cancels it, and the local crop-box Undo stack can step it back.

### Hold Command and draw a Straighten line without fitting

Original source: [held state](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_straighten.rs#L39-L49), [canvas key-state update](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas.rs#L1957-L1966), [pointer admission](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_straighten.rs#L70-L100), and [fit choice](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_straighten.rs#L136-L159).

**Adapted Rust** — the common angle calculation is omitted because only frame placement differs.

Purpose: show the temporary Command-held Straighten that preserves frame size and position.

```rust
each_canvas_frame {
    crop.straighten.held = !text_edit_focused() && modifiers.command;
}

on_down if crop.straighten.held && crop.drag.is_none() {
    crop.straighten.line = Some([point, point]);      // Temporary mode; no one-shot button state.
}
on_up {
    let fit = false;                                  // A Command-held line never fits the frame.
    let deg = frame_angle(line.start, line.end, 3.0 / zoom)?;
    let frame = existing_frame.unwrap_or([0.0, 0.0, canvas_w, canvas_h]);
    app.ui.crop_rect = Some(frame);                   // Same size and position.
    app.ui.crop_angle = deg;
    crop.default_frame = false;
    crop.editing = true;
}
```

Because uncovered corners are not removed by fitting, a later commit uses the selected Fill: Background/Transparent normally, or Content-Aware Fill when that option is selected.

### Choose a ratio or W×H×Resolution output size

Original source: [options bar](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/panels/options_bar.rs#L471-L565), [preset ratio](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L255-L317), [fit and default-frame rules](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L319-L342), [target parsing](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_size.rs#L121-L149), and [commit params](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_size.rs#L189-L208).

**Adapted Rust** — options-bar widget creation is represented by the state changes it produces.

Purpose: show how a selected ratio or output size updates the pending preview and later commit parameters.

```rust
if crop_ratio == "whr" {
    let (w, h, resolution) = target(&options, document_dpi, point_size_per_inch)?;
    // w and h are pixels converted from "4 in", "10 cm", or "1024 px";
    // an empty resolution falls back to document_dpi.
} else if custom_ratio_changed || swap_clicked {
    set_custom_ratio_fields(&mut options, width, height); // Both positive sides make "w:h".
}
if effective_ratio_changed {
    apply_ratio(app);                                     // Reveals a hidden frame, then fits it.
}
```

The fit is the largest centered rectangle of that ratio inside the current frame; it never adds area. While a custom ratio is being typed, every intermediate value is fitted from the frame captured when the edit began, so moving from `1:3` toward `16:3` does not progressively shrink the box.

**Adapted Rust** — the distinction between a plain ratio and an explicit W×H×Resolution target is retained.

Purpose: explain when fitting an untouched default frame becomes a real crop.

```rust
let sized = crop_size::target(&options, document_dpi, point_size_per_inch).is_some();
if let Some(fitted) = fitted_ratio(current_frame, ratio) {
    if fitted != current_frame && !sized && crop.default_frame {
        app.ui.crop_rect = Some(fitted);
        crop.frame_from_selection = false;       // Plain ratio still is not a pending crop.
    } else if fitted != current_frame || sized {
        make_real_edited_frame(fitted);          // W×H×Resolution is an explicit output instruction.
    }
}
sync_default_mode_camera();
```

On commit, W×H×Resolution adds both `targetWidth` and `targetHeight` when both fields parse, plus `resample` from the Image Interpolation preference. A nonempty resolution adds `resolution` in ppi. With one of W/H empty there is no target ratio or resampling; a resolution alone may still change document resolution.

### Press arrow keys to nudge the frame or image

Original source: [key ownership](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L837-L840), [key dispatch](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L932-L950), [nudge](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L859-L878), and [shortcut precedence](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/shortcuts.rs#L614-L618).

**Adapted Rust** — the four arrow mappings are represented by one signed vector.

Purpose: show the crop-specific arrow branch that outranks selection and layer nudges.

```rust
if ui.tool != Tool::Crop || crop.drag.is_some()
    || ui.transform.is_some() || !finite_frame() {
    return false;                                  // Other shortcuts may handle the arrow.
}
if arrow_pressed {
    let distance = if shift { 10.0 } else { 1.0 }; // Document pixels.
    let d = arrow_vector * distance;
    if default_mode {
        crop.default_frame = false;
        crop.editing = true;
        sync_frame();
        shift_image(screen_vector_to_document(d)); // Image moves under the upright box.
    } else {
        make_real_edited_frame(rect + d);          // Classic Mode moves the frame on the page.
    }
    return true;
}
```

This key path does not dispatch a document command and does not create document history; it changes only the pending crop state used by a later commit.

### Press `X` to swap orientation and ratio

Original source: [swap geometry](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L880-L929), [W×H swap](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_size.rs#L240-L253), and [shortcut precedence](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/shortcuts.rs#L614-L618).

**Adapted Rust** — frame and options updates are shown in their source order.

Purpose: show why X swaps the crop frame instead of foreground/background colours while Crop owns a frame.

```rust
if !takes_crop_keys(app) { return false; }         // Crop, finite frame, no drag/Free Transform.
if !crop_size::swap(&mut options) {                // True only in W×H×Resolution mode.
    if let Some(swapped) = swapped_ratio(options.crop_ratio, doc_w, doc_h) {
        options.crop_ratio = swapped;               // "4:5" becomes "5:4"; "original" turns too.
    }
}
if swapped_width_height(rect) != rect {
    make_real_edited_frame(swapped_width_height(rect)); // Turn frame about its centre, keep angle.
}
if preset_ratio(app).is_some() {
    apply_ratio(app);                               // Fit from the already-turned frame.
} else {
    sync_frame();
    auto_center_if_default_mode();
}
true;                                              // X must not reach tools.swapColors.
```

A square frame does not change, so an untouched square default frame remains untouched.

### Press `O` to change the composition overlay

Original source: [overlay state](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_overlay.rs#L18-L44), [orientation count](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_overlay.rs#L75-L89), [visible guides](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_overlay.rs#L250-L275), and [keys](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_overlay.rs#L350-L370).

**Adapted Rust** — guide geometry is represented by the selected options only.

Purpose: show the Crop-owned `O` shortcuts that change preview guides without touching the frame.

```rust
if ui.tool != Tool::Crop || ui.crop_rect.is_none() || crop.drag.is_some() {
    return false;                                  // O remains available to Dodge/Burn/Sponge.
}
if shift_o_pressed {
    options.crop_overlay_orientation =
        (options.crop_overlay_orientation + 1) % options.crop_overlay.orientations().max(1);
} else if o_pressed {
    options.crop_overlay = options.crop_overlay.next(); // Thirds → Grid → … → Golden Spiral.
}
// Auto shows only during a drag; Always shows without one; Never shows none.
let show = options.crop_overlay_show.shows(crop.drag.is_some());
// During default-mode turning, the chosen guide is replaced by a turn grid.
```

Triangle has two meaningful orientations and Golden Spiral four; symmetric overlays effectively ignore orientation. These are UI preferences, not document edits.

### Press `H` or `P` to change cropped-area preview

Original source: [shield defaults](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_shield.rs#L34-L83), [acting shield](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_shield.rs#L112-L148), and [default-mode camera](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_mode.rs#L73-L97).

**Adapted Rust** — only the two keyboard branches are retained.

Purpose: show when Crop owns H/P and what preview state each changes.

```rust
if ui.tool != Tool::Crop || ui.crop_rect.is_none()
    || crop.default_frame || crop.drag.is_some() {
    return false;                                  // H/P remain Hand/Pen tool keys.
}
if h_pressed {
    if !shield.classic_mode {
        shield.show_cropped_area = !shield.show_cropped_area; // Hide/reveal outside the frame.
    }
    return true;                                   // Classic Mode greys out H, so it is consumed.
}
if p_pressed {
    shield.classic_mode = !shield.classic_mode;     // Switch preview model.
    crop_mode::sync(app, None);                     // Camera becomes -angle, or upright in Classic.
    return true;
}
```

Both keys act only after the default frame has become real. They do not alter `crop_rect`, `crop_angle`, or document history.

### Press `I` to use the active image as W×H×Resolution

Original source: [Front Image](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_size.rs#L210-L220), [preset choice](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_size.rs#L255-L270), and [key](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_size.rs#L273-L285).

**Adapted Rust** — the options-bar preset selector is omitted.

Purpose: show how `I` fills the size fields from the current document and immediately constrains the frame.

```rust
if ui.tool != Tool::Crop || crop.drag.is_some() || active_document().is_none() {
    return false;                                  // I remains the Eyedropper group key.
}
if i_pressed {
    options.crop_ratio = "whr";
    options.crop_width = format_pixels(document.width);       // e.g. "1920 px".
    options.crop_height = format_pixels(document.height);     // e.g. "1080 px".
    options.crop_resolution = format_resolution(document_dpi, options.resolution_unit);
    apply_ratio(app);                              // Uses the new target ratio immediately.
    return true;
}
```

The resulting target ratio can fit or reshape the pending frame; the document itself is changed only on commit.

### Pick up a default crop frame on the next canvas frame

Original source: [canvas frame update](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas.rs#L1957-L1966), [frame entry](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L405-L411), [default rectangle](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L413-L424), and [frame ownership/hidden state](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L426-L473).

**Adapted Rust** — per-frame view synchronization and unrelated document switches are omitted.

Purpose: derive the frame shown immediately after Crop becomes active.

```rust
let canvas = active_doc.bounds();
let selection = active_doc.selection
    .as_ref()
    .map(|s| s.content_bounds().intersect(&canvas))
    .filter(|b| !b.is_empty());                       // Empty intersections use the canvas.
let (rect, from_selection) = match selection {
    Some(b) => (b, true),                             // Selection bounds become a live candidate.
    None => (canvas, false),                          // Otherwise the whole canvas is default.
};

ui.crop_rect = Some(rect);
crop.default_frame = true;                            // Untouched whole canvas commits nothing.
crop.frame_from_selection = from_selection;           // Selection bounds can be committed unchanged.
crop.frame_for = active_tab_index_and_size;           // Another tab/size gets its own frame.
crop.editing = false;
clear_crop_history();
claim(active_doc_id);
fit_frame_for_selected_ratio_or_size();
```

With no document, the state simply has no frame. After a successful commit or Escape, the frame remains hidden for that document until a canvas press, Straighten press, or ratio action calls `show_frame`; selecting Crop again does not automatically reveal it. A selection-bounds default frame is meaningful: pressing Enter or otherwise making it live can crop to those bounds.

### Undo or Redo the crop box while a crop is pending

Original source: [box-history gate](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L637-L665), [box step](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L667-L692), [shell interception](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/app/commands.rs#L88-L93), and [menu enablement](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/menus/state.rs#L49-L51).

**Adapted Rust pseudocode** — Undo and Redo share one stepping routine; the source's view-center arithmetic is retained.

Purpose: show why Edit ▸ Undo and Redo edit the pending crop box instead of document history.

```rust
if tool != Tool::Crop || (crop.undo.is_empty() && crop.redo.is_empty()) {
    normal_document_history(id);                       // Not Crop-owned box state.
}

match id {
    "edit.undo" if !crop.undo.is_empty() => {
        let previous = crop.undo.pop()?;               // Option<CropStep>.
        let current = step_of(app);
        apply_step(app, previous);                     // Restores rect, angle, default/selection/edit state.
        crop.redo.push(current);
    }
    "edit.redo" if !crop.redo.is_empty() => {
        let next = crop.redo.pop()?;
        let current = step_of(app);
        apply_step(app, next);
        crop.undo.push(current);
    }
    "edit.undo" | "edit.redo" => {}                    // Intercepted but empty: greyed in menus, Null if run.
    "edit.toggleLastState" => return Err("Commit or cancel the crop first"),
    _ => normal_document_history(id),
}

if default_mode && current.rect.is_some() && target.rect.is_some() {
    view.center += center(target.rect) - center(current.rect); // The image follows the restored box.
}
sync_frame_and_auto_center(app);
Ok(json!(null))                                        // Result<Value, String>; no document revision changes.
```

The interception is installed before normal command dispatch in `PhotocraftApp::run`, so menus, shortcuts, the control channel, and MCP callers take the same branch. Committing, cancelling, or leaving the crop clears both box stacks; document Undo/Redo then applies to the resulting document history.

### Try another command while a crop is pending

Original source: [pending definition](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L694-L706), [allowed commands](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L708-L741), [menu gate](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/menus.rs#L41-L50), and [control-channel gate](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/control/dispatch.rs#L97-L121).

**Adapted Rust** — the four prefix rules and exact-ID allow list are unchanged in shape.

Purpose: define exactly when another command can run while the Crop tool owns a real pending frame.

```rust
let pending = tool == Tool::Crop
    && crop_rect.is_some()
    && !default_frame
    && (owner.is_none() || owner == active_document_id()); // A drawn, edited, or turned frame.

let allowed = match id {
    i if i.starts_with("view.") => ![
        "proofSetup.", "pixelAspectRatio", "thirtyTwoBitPreviewOptions",
        "flipHorizontal", "screenMode.",
    ].iter().any(|greyed| view_suffix.starts_with(greyed)),
    i if i.starts_with("window.") => {
        !(window_suffix.starts_with("workspace.") || window_suffix == "panel.adjustments")
    }
    i if i.starts_with("help.") => i != "help.systemInfo",
    _ => matches!(
        id,
        "file.close" | "file.save" | "file.saveAs" | "file.saveACopy" | "file.exit"
            | "edit.undo" | "edit.redo" | "edit.toggleLastState" | "edit.search"
            | "image.crop"
    ),
};

let blocked = pending && !allowed;                       // Menus grey it; menu/control dispatch refuses it.
```

The pending definition excludes an untouched default frame, an in-progress drag whose old default frame has not yet been replaced, and a frame owned by another document. `image.crop` is allowed because its menu route commits the frame; Undo and Redo are allowed because the crop-box interceptor handles them first.

### Switch documents while a crop is pending

Original source: [ownership cancellation](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L743-L756), [per-frame call](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas.rs#L1957-L1966), [command sync](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/app/commands.rs#L302-L334), and [pre-commit call](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/gesture.rs#L253-L263).

**Adapted Rust** — only the document-id comparison and reset are shown.

Purpose: prevent a frame or active gesture from one tab being applied to another document.

```rust
let active = session.active().map(|st| st.doc.id);       // Option<DocId>.
let pending = crop_rect.is_some() || drag.is_some();
if pending && owner.is_some() && owner != active {
    cancel(app);                                          // Uprights the view and clears all crop state/history.
}
owner = active;                                           // A frame created before ownership was claimed belongs here.
```

This check runs on every canvas frame, during command/view synchronization after document changes such as New, Open, tab switching, or Close, and again immediately before commit. The newly active document can then receive its own default Crop frame; the old document's frame is never transferred to it.

### Press Enter, double-click inside the frame, or click the options-bar check button to commit

Original source: [Enter route](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/shortcuts.rs#L810-L831), [double-click route](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas.rs#L1258-L1260), [inside-frame test](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L790-L796), and [options-bar buttons](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/panels/options_bar.rs#L544-L558).

**Adapted Rust** — the three trigger sites are combined.

Purpose: show the distinct UI entries that all call the same crop-frame commit routine.

```rust
if crop_rect.is_some() && !popup_open && key_pressed(Enter) {
    commit_crop(app);                                   // commit_crop handles engine errors and rollback.
} else if tool == Tool::Crop
    && double_clicked
    && commits_at(app, document_point)                  // bool: Crop active, no drag, point inside the frame.
{
    commit_crop(app);
} else if options_check_button.clicked() {
    commit_crop(app);
}
```

#### Build and dispatch `image.crop`

Original source: [commit routine](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/gesture.rs#L253-L298), [state kept for rollback](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L613-L635), and [output-size parameters](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_size.rs#L620-L665).

**Adapted Rust pseudocode** — JSON construction and rollback are shown in source order while unrelated view code is omitted.

Purpose: trace the pending frame into the exact `image.crop` request and show what survives a refusal.

```rust
cancel_stale(app);                                      // Never commits another document's frame.
let kept = keep(app);                                   // CropStep, box Undo/Redo, and view rotation/center.
clear_history(app);                                     // After commit, Undo means document history.
crop.drag = None;                                       // Enter during a drag ends that drag.
let Some(r) = ui.crop_rect.take() else { return; };     // [f64; 4], document px.
crop_mode::upright(app);

if take(crop.default_frame) && active_document_bounds() == Some(r) {
    return;                                             // Untouched whole-canvas frame crops nothing.
}

let (x, y) = (r[0].round(), r[1].round());
let (w, h) = ((r[2] - r[0]).round().max(1.0), (r[3] - r[1]).round().max(1.0));
let mut params = json!({
    "x": x, "y": y, "width": w, "height": h,
    "deleteCroppedPixels": options.crop_delete,
});
if options.crop_content_aware {
    params["contentAware"] = json!(true);               // Fill uncovered area from surroundings.
}
let angle = normalized_crop_angle(ui.crop_angle);       // e.g. 15.0.
ui.crop_angle = 0.0;
if angle != 0.0 { params["angle"] = json!(angle); }
crop_size::commit_params(app, &mut params);             // May add targetWidth/Height, resolution, resample.

if app.run("image.crop", params).is_err() {
    crop_ui::restore(app, kept);                        // Frame, angle, local history, and view come back.
} else {
    active_view.fit_pending = true;
    crop_ui::hide_frame(app);                           // Returning to Crop does not immediately reframe.
}
```

For example, a turned frame may produce:

```json
{
  "x": 20.0,
  "y": 10.0,
  "width": 60.0,
  "height": 40.0,
  "deleteCroppedPixels": true,
  "angle": 15.0
}
```

#### Apply an unrotated `image.crop`

Original source: [parameter validation](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/cmd/image_cmds.rs#L638-L734), [target validation](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/cmd/image_cmds.rs#L737-L829), [angle validation](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/cmd/image_cmds.rs#L831-L847), [document crop](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/cmd/image_cmds.rs#L444-L497), and [compound history](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/cmd/analysis_cmds.rs#L174-L205).

**Adapted Rust pseudocode** — parameter coercions and lower-level surface iteration are summarized without changing admission rules or mutations.

Purpose: show the engine-side checks, document mutation, one-step history, and JSON result for the Crop tool's commit.

```rust
let delete = params.bool_or("deleteCroppedPixels", true);
let content_aware = params.bool_or_null("contentAware", false)?; // Non-bool/null is Err.
let explicit = parse_rect_i32(params)?                // All four fields or None; w/h > 0.
    .with_max_side(300_000)?
    .with_checked_far_edges()?;                       // x+width and y+height must remain i32.
let target = parse_crop_target(params)?;              // Target sides, resolution, and resample filter.
check_resample_budget_if_target(document, target)?;   // Refuses excessive output before mutation.

let angle = parse_angle(params)?;                     // Explicit rectangle required; ±3600 max.
if angle.is_some() {
    return rotated_crop(session, explicit, angle, delete, content_aware, target);
}

let rect = explicit
    .or_else(|| selection_bounds_intersected_with_canvas()) // Menu fallback when no explicit frame.
    .ok_or("nothing to crop: pass x/y/width/height or make a selection")?;
let unlock_background = !delete && !content_aware;    // Crop-tool route does not keep Background special.

compound(session, "Crop", |s| {
    if unlock_background { unlock_background_for_crop(document); } // Renames it to Layer 0.
    crop_doc(document, rect, delete, session.tools.background)?;   // Delete/retain pixels and extend canvas.
    fit_canvas_artboard_if_sole_board_covers_canvas(document, explicit.is_some());
    if content_aware {
        crop_fill::fill_uncovered(document, active_layer, coverage(rect))?;
    }
    document.selection = None;
    target.apply(document);                           // Optional resample and resolution in this same step.
})?;                                                  // One history edit named Crop; failure rolls it back.

Ok(json!({
    "x": rect.x0, "y": rect.y0, "width": rect.width(), "height": rect.height(),
    "documentWidth": document.width, "documentHeight": document.height,
    "resolution": document.resolution_dpi,
}))
```

`targetWidth` and `targetHeight` must be supplied together as whole pixels from 1 to 300,000; `resolution` must be finite from 1 to 10,000 ppi. A target size defaults to Bicubic resampling and rejects `resample: "none"`. Delete Cropped Pixels defaults to true; when it is off, pixels outside the new canvas are retained and a Background normally becomes `Layer 0`, except with Content-Aware Fill. An extending straight crop fills the Background's new area with the background colour while other layers gain transparency. The active layer matters mainly as Content-Aware Fill's fallback fill target.

#### Apply a turned frame

Original source: [rotated crop](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/cmd/image_cmds.rs#L849-L945), [source rectangle](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/cmd/image_cmds.rs#L947-L965), and [angle normalization](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/cmd/image_cmds.rs#L831-L847).

**Adapted Rust pseudocode** — checked geometry is summarized, but the order of document mutations remains explicit.

Purpose: explain how a nonzero `angle` rotates the document into the frame and still records one Crop history step.

```rust
let angle = normalize_to_open_minus_180_closed_180(angle); // 375.0 becomes 15.0.
let rotated_canvas = rotated_size(old_canvas, -angle);
let upright_frame = map_frame_centre_with_checked_i32_edges(frame, angle, rotated_canvas)?;
let source_keep = source_pixels_needed_by_frame_plus_kernel_reach(
    old_canvas, rotated_canvas, angle, upright_frame,
);
let coverage = content_aware_coverage_before_rotation(active_layer);

compound(session, "Crop", |s| {
    if delete_cropped_pixels {
        edit_crop("Crop", |doc| {
            delete_cropped_pixels(&mut doc.layers, source_keep); // Only pixels the frame can sample.
            crop_layer_masks(doc, source_keep);
            crop_channels_and_quick_mask(doc, source_keep);
            doc.selection = None;
        })?;
    }
    if unlock_background {
        edit_crop("Crop", unlock_background_for_crop)?; // Delete-off CAF excepted by the caller.
    }
    rotate_arbitrary(s, angle, CounterClockwise)?;       // Makes the requested frame upright.
    map_color_samplers(target.sampler_map(upright_frame));
    edit_crop("Crop", |doc, active| {
        crop_doc(doc, upright_frame, delete, background_colour)?;
        fit_canvas_artboard(doc, board);
        if content_aware { fill_uncovered(doc, *active, coverage)?; }
        doc.selection = None;
        target.apply(doc);                               // Optional resample and resolution.
    })
})?;

Ok(crop_result(json!({
    "x": frame.x0, "y": frame.y0,
    "width": frame.width(), "height": frame.height(), "angle": angle,
})))
```

The rotated route first checks finite frame coordinates and i32-safe far edges. It then combines pixel deletion, mask/channel cropping, optional Background unlock, rotation, crop, optional Content-Aware Fill, sampler mapping, resampling, and resolution into one compound `Crop` history result. A failed inner edit restores the prior document and history.

### Choose Image ▸ Crop while the Crop frame is pending

Original source: [menu interception](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/menus/invoke.rs#L102-L106), [pending allowance](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L717-L740), and [control route split](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/control/dispatch.rs#L115-L121).

**Adapted Rust** — the menu-shell branch is unchanged in effect.

Purpose: show the menu/command-palette route that commits a pending frame instead of invoking the engine crop directly.

```rust
if id == "image.crop" && crop_ui::pending(app) {
    canvas::commit_crop(app);                          // Uses the pending Crop-tool frame.
    return Some(Ok(json!({"committed": true})));       // Result<Value, String>.
}
```

This interception covers native menu clicks and control-channel `ui.menu.invoke` requests. It bypasses the engine handler's selection-bounds fallback for that invocation. In contrast, a control-channel `engine.execute` request for `image.crop` follows the direct engine route with its supplied params rather than this menu branch.

### Leave the Crop tool with a drawn frame

Original source: [tool-change frame handling](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L426-L480) and [commit-on-leave](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L482-L495).

**Adapted Rust** — the routine runs whenever `ui.tool` has changed on a later canvas frame.

Purpose: distinguish dropping an untouched default frame from committing or cancelling a drawn frame when another tool becomes active.

```rust
if tool != Tool::Crop {
    crop.editing = false;
    crop_straighten::reset(app);
    if crop.default_frame {
        ui.crop_rect = None;                           // Untouched default frame is discarded.
        crop.default_frame = false;
        crop.frame_from_selection = false;
    }
    if ui.crop_rect.is_some() && !keeps_pending(app) {
        let new_tool = replace(ui.tool, Tool::Crop);   // Commit routine expects Crop.
        commit_crop(app);                              // Same construction and image.crop path as Enter.
        if ui.crop_rect.is_some() {
            cancel(app);                               // Refused crop is not retried by the new tool.
        }
        ui.tool = new_tool;
        crop.dismissed = None;                         // Returning to Crop shows a fresh default frame.
    }
}
```

A direct toolbar/flyout tool change can reach this path without dispatching `tools.select.crop`. The frame also stays pending if its drag is still active or the stylus eraser end is temporarily holding Crop for the pointer; otherwise, leaving the tool either commits a drawn frame or drops the untouched default.

### Reset the crop box

Original source: [options-bar Reset](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/panels/options_bar.rs#L559-L563), [modified test](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L546-L553), and [reset implementation](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L555-L584).

**Adapted Rust** — ratio fitting is represented by its effect on `default_frame` and `reset_to`.

Purpose: show how Reset restores the starting frame while retaining a local crop-box Undo step.

```rust
let changed = tool == Tool::Crop
    && crop_rect.is_some()
    && drag.is_none()
    && (crop_angle != 0.0 || (!default_frame && crop_rect != reset_to));
options_reset_button.enabled = changed;                 // No drag and no real modification: disabled.

if clicked && changed {
    let (rect, from_selection) = default_rect(app)?;    // Selection bounds or whole canvas.
    let before = step_of(app);
    ui.crop_rect = Some(rect);
    ui.crop_angle = 0.0;                                // Reset is upright.
    crop.default_frame = true;
    crop.frame_from_selection = from_selection;
    crop.frame_for = active_tab_and_size;
    crop.editing = false;
    crop.press_step = None;
    claim(active_document_id);
    crop_straighten::reset(app);
    fit_frame(app);                                     // Reapplies ratio or W×H×Resolution preset.
    crop.reset_to = crop_rect.filter(|_| !default_frame); // WHR may leave a real pending crop.
    if step_of(app) != before {
        crop.undo.push(before);
        crop.redo.clear();                              // Reset is one new branch of box history.
    }
    sync_frame_and_auto_center(app);
}
```

Reset is not document Undo. A W×H×Resolution preset can fit the “default” frame to its output size, making it a real pending crop again; `reset_to` remembers that frame so the button disables until it changes.

### Cancel or dismiss the crop

Original source: [cancel and hide state](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L502-L539), [options-bar Cancel](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/panels/options_bar.rs#L556-L558), [Escape route](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/shortcuts.rs#L810-L831), and [inside-frame test](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/crop_ui.rs#L790-L796).

**Adapted Rust** — cancellation, dismissal, and the Escape special case are combined.

Purpose: show what is cleared and when Escape is consumed instead of leaving full-screen mode.

```rust
fn cancel(app: &mut PhotocraftApp) {
    crop_mode::upright(app);
    ui.crop_rect = None;
    ui.crop_angle = 0.0;
    crop.drag = None;
    crop.default_frame = false;
    crop.frame_for = None;
    crop.editing = false;
    clear_history(app);                                 // Box Undo/Redo no longer applies.
    crop_straighten::reset(app);
}

fn dismiss(app: &mut PhotocraftApp) {
    cancel(app);
    hide_frame(app);                                    // Escape/options Cancel keeps the frame hidden.
}

if crop_rect.is_some() && !popup_open && escape_pressed {
    let untouched_fullscreen_default = screen_mode != "standard"
        && crop.default_frame
        && !crop_straighten::mode(app);
    if !untouched_fullscreen_default {
        dismiss(app);                                   // Options Cancel does this unconditionally when clicked.
        return true;                                    // Escape does not also leave full screen.
    }
    // Otherwise Escape continues to the full-screen exit handler.
}
```

With Crop still selected and no hidden-frame key, the next canvas frame normally creates a new default frame. Escape and the options-bar Cancel instead call `dismiss`, which records the active document/size as hidden, so no default frame reappears until a canvas press, Straighten press, or ratio action. An open Crop options popup receives Escape first and closes without dismissing the frame.

