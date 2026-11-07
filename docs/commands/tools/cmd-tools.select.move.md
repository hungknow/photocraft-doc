# tools.select.move

Selects the Move tool in the PhotoCraft UI.

Verified against: `3d60ad01f941519307c755eed8e0896d6274cde6`; call route rechecked at `f68e227026701cf24e0e182530694240e2d3cfe4` · 2026-11-07
Navigation: [tools commands](../tools.md#tools) · [Transform/crop actions](../../tools/transform-crop.md#transform-crop)

## Context to use

The shell can select Move without an open document. After Move is active, later canvas and keyboard gestures can select layers, move layers or selected pixels, drag guides, start Free Transform controls, or copy layers to another document.

- Invoke `tools.select.move` explicitly through a menu, command palette, control channel, or another UI command route. See [tool ID route](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/tool_shortcuts.rs#L14-L18) and [shell dispatch](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/menus/invoke.rs#L77-L81).
- Press the default `V` group to choose or cycle Move/Artboard. This default group path assigns `ui.tool` directly and does not dispatch this ID. See [default key group](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/shortcuts.rs#L896-L925).
- Press a user-assigned Move shortcut. A non-empty assignment for this ID enters the shortcut command table and dispatches the exact ID. See [override table](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/shortcut_dispatch.rs#L73-L113).
- Click the Move toolbar slot, Alt-click to cycle its flyout, or choose Move in the flyout. These call tool selection directly and do not dispatch this ID. See [toolbar click](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/panels/toolbar.rs#L195-L212) and [flyout row](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/panels/toolbar.rs#L320-L328).
- Hold Command with an eligible painting tool to make Move effective for that pointer or arrow gesture without changing `ui.tool`; selection tools use their own Command-drag branch. See [temporary-tool rules](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/hold_keys.rs#L69-L75).

In a headless engine session, the ID is listed for shortcut settings but has no engine command handler, so `Session::execute` returns `UnknownCommand`. See [bindable tools](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/prefs/shortcuts.rs#L184-L193) and [engine command lookup](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/jobs.rs#L1509-L1519).

## Inputs and response

The UI command route accepts the exact case-sensitive ID with empty parameters:

```json
{"id":"tools.select.move","params":{}}
```

- No parameters are read; `params` may be `{}` or omitted by an internal caller that already supplies `Value::Null`.
- The default shortcut is `V`, shared with Artboard. A user assignment changes the key that dispatches this ID; the default `V` path only includes tools whose shortcuts still use their defaults.

The successful UI invocation returns `{"tool":"Move"}`, sets `ui.tool` to `Tool::Move`, and makes no document or history edit. Later gestures dispatch their own commands, such as `layer.pickAt`, `layer.translate`, `select.float`, `select.drop`, `view.moveGuide`, `layer.copyToDocument`, and `edit.transform`.

- An automation authorization callback can reject the request before the tool changes. See [guarded invoke](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/menus/invoke.rs#L42-L55).
- A differently cased or misspelled tool ID does not match the canonical route and returns an error. See [canonical ID match](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/tool_shortcuts.rs#L14-L18).
- Direct engine execution fails with `UnknownCommand`; the selection route exists only in the UI shell. See [dispatch](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/jobs.rs#L1509-L1519) and [`Session::execute`](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/session/execute.rs#L17-L23).
- Gesture failures are local to those gestures: unsupported colour-channel targeting, hidden targets, locked targets, and command errors below do not unselect Move.

## Structures used

Original source: [tool enum](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/state/tool.rs#L3-L9), [UI state](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/state.rs#L205-L216), [tool table](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/shell_commands.rs#L102-L108).

**Adapted Rust** — unrelated variants and state fields omitted.

```rust
enum Tool { Move, Artboard /* ... */ } // Move is the persistent selected UI tool.
struct UiState {
    tool: Tool,                 // Tool::Move after selection.
    group_tools: Vec<Tool>,     // e.g. [Tool::Artboard], remembered last choice per key group.
    spring: Option<Spring>,     // None after an explicit selection.
}
const MOVE_BINDING: (&str, &str, Option<&str>) =
    ("tools.select.move", "Move Tool", Some("V"));
```

Original source: [Move options](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/state/tool_options.rs#L144-L152), [defaults](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/state/tool_options.rs#L280-L282), [options bar](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/panels/options_bar.rs#L603-L618).

**Adapted Rust** — only the `ToolOptions` fields owned by Move are shown.

```rust
struct ToolOptions {
    move_auto_select: bool,      // true; Auto-Select is on by default.
    move_target: String,         // "group"; the bar also offers "layer".
    move_show_transform: bool,   // false; controls the visible Move bounding-box handles.
}
```

Original source: [pointer drag](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/drag.rs#L5-L37), [Move modifier state](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/move_mods.rs#L66-L77).

**Adapted Rust** — fields unrelated to Move omitted.

```rust
struct Drag {
    tool: Tool,                  // Tool::Move for a whole-layer or floating-piece drag.
    start: [f64; 2],             // [120.0, 80.0] in document pixels.
    points: Vec<[f64; 3]>,       // [[130.0, 82.0, 1.0]] at the latest pointer sample.
    sel_move: Option<bool>,      // Some(true) while selected pixels float.
}
struct MoveDrag {
    start: Option<[f64; 2]>,     // Some([120.0, 80.0]) after pointer down.
    alt: bool,                   // true if Option/Alt was held at press.
    last: Option<[f64; 2]>,      // Some([10.0, 0.0]) after Shift locks the axis.
    dup_from: Option<usize>,     // Some(4), history length before an Alt duplicate.
}
```

Original source: [Move preview](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/move_ui.rs#L128-L168), [preview construction](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/move_ui.rs#L188-L247).

**Adapted Rust** — helper methods and damage calculation omitted.

```rust
struct MovePreview {
    doc: DocId,                    // source document identity.
    revision: u64,                 // document revision captured for the drag.
    ids: Vec<LayerId>,             // selected roots that translation will move.
    moving: Vec<LayerId>,          // roots plus descendants shown moving.
    bounds: Option<Rect>,          // invalidation area, including off-canvas shape reach.
    canvas: Rect,                  // document bounds.
    offsets: Vec<(i32, i32)>,      // [(12, 0)] whole-pixel offsets already rendered.
    shown: Option<Arc<Document>>,  // translated document at the latest offset.
    floating: bool,                // true for select.float's displayed piece.
    too_big: bool,                 // true above LIVE_LIMIT: outline-only dragging.
}
```

Original source: [floating piece](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/cmd/float_cmds.rs#L236-L257), [cross-document transfer](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/panels/layer_transfer.rs#L21-L30).

**Adapted Rust** — cut-pixel storage is represented by one field.

```rust
struct Floating {
    layer: LayerId,             // 7, the source layer.
    offset: (i32, i32),         // (12, 0), whole-pixel pending offset.
    revision: u64,              // document revision when the piece was cut.
    parts: Arc<CutParts>,       // cut/rest/piece/outline data.
    moved: bool,                // true after a non-zero move, even if returned to 0.
}
struct Transfer {
    source: DocId,              // source document identity.
    layers: Vec<u64>,           // [7, 9] selected source layers to copy.
    grab: Option<[f64; 2]>,     // Some([120.0, 80.0]) for a Move-tool drag.
    label: String,              // "Sky" or "2 layers" for the drag ghost.
}
```

## Core logic flow

Selection changes only UI state. Direct toolbar and default-key choices reach the same `Tool::Move` state without dispatching `tools.select.move`; explicit ID and custom-shortcut routes do dispatch it. Later pointer and arrow-key gestures use that state but dispatch other commands.

### Read the execution route before the individual gestures

The Rust source is split by event type and responsibility. Read a normal whole-layer drag in this order; the canvas calls `display::display_doc` during rendering, between pointer events, rather than from the pointer handler. A selected-pixel drag takes the `select.float` branch at pointer down and the `finish_selection_drag` branch at release instead.

1. **Select the tool.** `menus::invoke` resolves the ID and `tool_memory::select` stores `Tool::Move`. A toolbar click or default `V` key reaches the same tool state directly. This step does not edit a document.
2. **Pointer down.** `canvas.rs` converts the screen point to document coordinates and sends `ToolEvent::Down` to `tool_event`. Its preflight and filters can give the event to a guide, transform handle, layer marquee, or another overlay. `handle_move_pointer` then handles Move-specific picking and selected pixels; otherwise `tool_event` creates a `Drag` for a whole-layer move.
3. **Pointer move and render.** `tool_event` updates the drag's latest point. On a render pass, `display::display_doc` asks `move_ui::display_doc` for a temporary translated document. `Some((document, key))` means the canvas can show that preview; `None` leaves the committed document visible. Neither return value commits an edit.
4. **Pointer up.** `tool_event` takes the drag and calls `finish_release`. For a whole-layer Move drag it calls `move_ui::finish`, which dispatches `layer.translate` once with the rounded offset. The engine changes the document and records the history step; a click at its starting point returns before that command.

Source route: [canvas event emission](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas.rs#L1030-L1050), [tool event gate](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas/tool_input.rs#L350-L385), [Move press](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas/tool_input.rs#L215-L335), [Move release](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas/tool_input.rs#L550-L605), [render choice](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas/display.rs#L145-L153), [preview and commit](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas/move_ui.rs#L205-L255), [release dispatch](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas/gesture.rs#L160-L167).

The method names below have these specific roles:

| Method | What it means here |
|---|---|
| `tool_memory::select` | Save the persistent tool choice in UI state; it does not start a drag. |
| `tool_event` | Route one down, move, or up event through interceptors and the active tool. |
| `move_ui::moves_selected_pixels_with` | Return `bool`: whether this Move gesture should lift the selected pixels instead of moving whole layers. |
| `move_ui::float_selected_for_drag` | Return `bool`: start a floating pixel piece through `select.float`, or report the command error and return `false`. |
| `move_ui::display_doc` | Return `Option<(Arc<Document>, u64)>`: a temporary document and render key for the current offset. `None` also covers outline-only preview. |
| `finish_selection_drag` | Dispatch the selected-pixel move or drop on release. |
| `move_ui::finish` | Dispatch `layer.translate` for a nonzero whole-layer offset; its `()` return is not a success signal. |

The remaining subsections are alternative entry paths and gesture branches. Each starts at its own trigger; they are not one sequence to execute from top to bottom.

### Invoke `tools.select.move` to select Move

Original source: [canonical ID](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/tool_shortcuts.rs#L14-L18), [authorization](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/menus/invoke.rs#L42-L55), [tool route](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/menus/invoke.rs#L77-L81), [explicit selection](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tool_memory.rs#L24-L33).

**Adapted Rust pseudocode** — authorization, ID matching, and state changes are combined across functions with different return types.

```rust
let name = id.strip_prefix("tools.select.")?;
let tool = Tool::from_name(name)?;                 // Option<Tool>; Some(Tool::Move) for "move".
let tool = (id == tool_id(tool)).then_some(tool)?; // Rejects "Move" and other noncanonical IDs.
if app.automation_input {
    authorize(id, &params)?;                       // Result<(), String>; Err leaves the tool unchanged.
}
tool_memory::select(app, tool);                    // Clears spring/eraser ownership and sets ui.tool.
Ok(json!({"tool": tool}))                          // Result<Value, String>; {"tool":"Move"}.
```

The route is live and enabled without a document. It records no document mutation and adds no history.

### Click the toolbar to select Move

Original source: [slot click](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/panels/toolbar.rs#L195-L212), [flyout choice](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/panels/toolbar.rs#L320-L328), [selection](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tool_memory.rs#L24-L33).

**Adapted Rust** — only the decisive tool-choice branches are shown.

```rust
let pick = if flyout_has_multiple_tools && alt_click {
    next_slot_tool(slot, current_tool)             // Tool::Artboard when cycling from Move.
} else {
    shown_tool                                     // Tool::Move for the Move/Artboard slot.
};
tool_memory::select(app, pick);                    // Direct state change; no command ID is invoked.
```

### Press the default `V` key to choose or cycle the Move group

Original source: [default-key behavior](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/shortcuts.rs#L896-L925), [default test](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/tool_shortcuts.rs#L28-L30), [persistence](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tool_memory.rs#L57-L76).

**Adapted Rust pseudocode** — remembered tools and preference persistence are shortened.

```rust
let group: Vec<Tool> = all_tools().filter(|tool| {
    tool.key() == 'V' && !prefs.shortcuts.contains_key(tool_id(*tool)) // [Move, Artboard] by default.
}).collect();
app.ui.tool = match group.iter().position(|tool| *tool == app.ui.tool) {
    Some(i) if shift_switch && !cycle_shift => group[i],       // Plain key retains a current group tool.
    Some(i) => group[(i + 1) % group.len()],                   // Cycle within Move/Artboard.
    None => remembered_group_tool('V').unwrap_or(group[0]),     // Restore the last default-key choice.
};
if !pointer_down { persist_selected_tool(app); }                // Save after release, not each frame.
```

This path never builds a command result and does not invoke `tools.select.move`. A custom Move assignment removes Move from this default group.

### Press a custom shortcut assigned to Move

Original source: [override inclusion](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/shortcut_dispatch.rs#L73-L113), [binding dispatch](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/shortcut_dispatch.rs#L238-L278), [command invoke](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/shortcut_dispatch.rs#L375-L403).

**Adapted Rust pseudocode** — shortcut-table construction is reduced to the custom Move assignment.

```rust
let table = [
    ("tools.select.move", parse("M"))? // User assignment from prefs.shortcuts, ahead of defaults.
];
if let Some((id, _)) = table.find(|(_, binding)| binding.matches(pressed_key, modifiers)) {
    dispatch_as(app, ctx, id, false); // dispatch_as checks enabled state and calls menus::invoke.
}
// Inside dispatch_as for this ordinary command:
menus::invoke(app, ctx, "tools.select.move", json!({})) // Result<Value,String>; selects Move.
```

The custom key therefore follows the exact ID route and can be disabled or rejected like other command dispatch. This walkthrough does not claim what key remains assigned to another command after the user edits shortcuts.

### Hold Command to use Move temporarily

Original source: [eligible tools](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/hold_keys.rs#L69-L75), [event tool](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/tool_input.rs#L652-L661), [selection-tool exception](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/tool_input.rs#L380-L392), [arrow exception](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/hold_keys.rs#L104-L116).

**Adapted Rust** — tool eligibility is represented as a boolean helper.

```rust
let effective_tool = if command_held && cmd_moves(actual_tool) {
    Tool::Move        // Temporary override only; app.ui.tool remains the actual painting tool.
} else {
    actual_tool
};
if selection_tool && command_held && !inside_selection(press) {
    maybe_duplicate_first();                          // Alt duplicates in place, if requested.
    app.drag = Some(Drag::new(Tool::Move, press, ..)); // Selection tool resolves this case itself.
}
```

No variant of this temporary override dispatches `tools.select.move` or persists Move as the selected tool.

### Drag, click, Alt-click, or double-click a guide

Original source: [guide press](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/tool_input.rs#L217-L228), [guide drag](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/tool_input.rs#L287-L299), [guide preferences double-click](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas.rs#L1196-L1201), [guide release](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/rulers.rs#L298-L335).

**Adapted Rust pseudocode** — pointer events and guide state are joined.

```rust
if let Some((vertical, index)) = guide_at(app, x, y) {
    if double_click {
        invoke("edit.preferences.guidesGridAndSlices", json!({}))?; // Opens Preferences; no guide edit.
    } else if alt && !shift && !command {
        flip_guide(app, vertical, index, [x, y]);                  // Reorients through the clicked point.
    } else {
        app.guide_drag = Some(GuideDrag::existing(vertical, index, [x, y]));
    }
    return;                                                          // Guides outrank layer/pixel Move.
}
on_move: guide.track([x, y], alt);
on_release: {
    if screen_distance < 0.5 { select_guide(vertical, index); return; }
    if inside_canvas {
        run("view.moveGuide", guide_position_and_orientation);      // Result<Value,String>; one guide edit.
    } else {
        run("view.deleteGuide", guide_identity);                    // Dropping outside/ruler deletes it.
    }
}
```

### Press an unsupported channel or hidden target

Original source: [multi-channel refusal](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/tool_input.rs#L229-L232), [hidden-target gate](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/tool_input.rs#L366-L379), [Move-to-translate mapping](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/tool_input.rs#L595-L607), [engine refusal](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/hidden_target.rs#L44-L57).

**Adapted Rust** — both immediate press refusals are shown without later drag setup.

```rust
if multiple_colour_channels_targeted(app) {
    app.ui.status = "Move and transforms on several colour channels are not implemented yet".into();
    return true;                    // No Drag, MovePreview, or document command starts.
}
if let Some(reason) = hidden_target::refusal(&app.session, "layer.translate", &json!({})) {
    app.ui.status = reason.into();  // e.g. moving a layer hidden directly or through a hidden group.
    app.ui.status_error = true;
    return;                         // Refuse before layer.translate so no partial gesture commits.
}
```

Channel targeting only bypasses the hidden-target gate when the command target is a channel target rather than multiple colour channels.

### Click or drag empty content into a layer-selection marquee

Original source: [Auto-Select probe](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/tool_input.rs#L233-L270), [marquee state and release](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/move_box.rs#L21-L75).

**Adapted Rust pseudocode** — Auto-Select and the marquee handler are joined.

```rust
if !dragging_selected_pixels && (move_auto_select != command_held) {
    let probe = run("layer.pickAt", {"x":x,"y":y,"target":move_target,"select":false});
    // Result<Value,String>; {"layer":null} at an empty point, or {"layer":7}.
    if locked_hit(&probe) {
        run("layer.pickAt", {"x":x,"y":y,"target":move_target,
            "mode": if shift {"add"} else {"replace"}}); // A click can select the locked layer.
    }
    if empty_hit(&probe) || locked_hit(&probe) {
        layer_box = Some(LayerBox { start:[x,y], end:[x,y], add:shift }); // No Move Drag starts.
    }
}
on_move: layer_box.end = [x, y];
on_release: {
    if zero_size && outside_canvas && !shift { run("select.deselectLayers", {}); return; }
    if width == 0.0 || height == 0.0 { return; }
    run("layer.selectInRect", {"x":start_x,"y":start_y,"width":width,"height":height,
        "target":move_target,"mode": if shift_at_press {"add"} else {"replace"}});
}
```

This marquee selects layers; it does not translate pixels or add a `Move` history step.

### Click a normal layer to select it without moving

Original source: [pick target](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/tool_input.rs#L233-L270), [click release](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/tool_input.rs#L553-L590).

**Adapted Rust pseudocode** — press and unmoved release are combined.

```rust
if !dragging_selected_pixels && (move_auto_select != command_held) {
    run("layer.pickAt", {"x":x,"y":y,"target":move_target,
        "mode": if shift {"add"} else {"replace"}}); // Result<Value,String>; e.g. {"layer":7}.
}
app.drag = Some(Drag::new(Tool::Move, [x, y], initial_point, mods, false));
on_release_if_released_where_pressed: {
    app.move_preview = None;        // A click never snaps or translates the selected layer.
    move_mods::finish(app);         // Discards modifier state; any Alt duplicate remains untriggered.
    return;                         // The layer.pickAt selection remains.
}
```

### Drag selected pixels as a floating piece

Original source: [eligibility](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/move_ui.rs#L19-L33), [drag float start](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas/move_ui.rs#L35-L70), [Move press](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas/tool_input.rs#L275-L305), [modifier filtering](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/move_mods.rs#L144-L159), [floating preview](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas/move_ui.rs#L290-L325), [release](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/selection.rs#L87-L105), [engine float and drop](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/cmd/float_cmds.rs#L289-L333), [automatic drop](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/cmd/float_cmds.rs#L336-L367).

**Adapted Rust pseudocode** — press, preview, release, and drop timing are condensed.

```rust
let pixels_eligible = active_document_has_selection
    && selected_layers.len() <= 1
    && active_layer.visible
    && session.is_enabled("select.float"); // type/shape/group/locked cases are rejected by select.float.
if pixels_eligible {
    if !move_ui::float_selected_for_drag(app, alt_at_press) { return; }
    // bool; false means select.float or select.drop failed and the status bar has the error.
    drag.sel_move = Some(true);
}
on_move: {
    let delta = if shift { constrain_45_degrees(pointer - start, previous) } else { pointer - start };
    // The canvas render calls floating_doc, which asks float_cmds::displayed for a temporary
    // document containing the piece at the pending offset. Moving the pointer alone is not a preview.
}
on_release: {
    if unmoved && floating.offset == (0,0) { run("select.drop", {}); return; } // No edit for an unmoved piece.
    if moved { run("select.float", {"dx":rounded_dx,"dy":rounded_dy}); }       // Result<Value,String>; moves the same floating piece or reports an error.
}
later_command: {
    if id == "select.float" || id == "select.drop" || non_document_prefix(id) { keep_floating(); }
    else if undo_or_history_return { put_piece_back(); }
    else { drop_floating_before_running(id)?; } // One "Move" history step when placement changes pixels.
}
```

The document remains unchanged while the piece floats; `select.drop` places the pixels and translates the selection outline in one history step. An unmoved piece can be dropped without an edit, while a piece moved away and back can still update the selection outline.

### Drag whole layers and commit one translation

Original source: [Auto-Select and lock probe](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/tool_input.rs#L233-L285), [modifier state](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/move_mods.rs#L121-L159), [live preview](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/move_ui.rs#L188-L247), [release](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/tool_input.rs#L553-L590), [commit](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/move_ui.rs#L363-L383), [engine translation](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/cmd/layer_multi_cmds/moves.rs#L113-L190), [Escape ownership](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/shortcuts.rs#L836-L872).

**Adapted Rust pseudocode** — incidental pointer/render setup is omitted.

```rust
app.drag = Some(Drag::new(Tool::Move, [x, y], first_point, mods, false));
on_move: {
    let delta = if shift { constrain_four_way(pointer - start, previous) } else { pointer - start };
    if alt_at_press && !duplicated_yet && screen_distance(delta) >= threshold {
        dup_from = duplicate_layers_in_place(app); // Option<usize>; Some(4) after layer.duplicate.
    }
    // tool_event records the pointer in Drag; it does not construct the image preview here.
}
on_canvas_render: {
    // display::display_doc calls move_ui::display_doc. That computes drag_offset and calls
    // moved_target(...) to build the temporary translated document; Option<(Arc<Document>,u64)>.
    // None for bounding-box-only preference, an oversized move, or a preview error: show outline/readout.
}
on_release: {
    let (dx, dy) = drag_offset.clamp(-1e7, 1e7);   // The same whole-pixel offset the readout showed.
    if dx == 0 && dy == 0 { return; }              // Pointer movement alone was only preview state.
    let result = run("layer.translate", {"dx":dx,"dy":dy,
        "selection": multiple_layers_with_selection, "nest":true}); // Result<Value,String>; Null or {"nested":[...]}.
    if result.is_ok() { fold_alt_copy_into_duplicate_plus_move(app, dup_from); }
    // Escape does not cancel an ordinary layer drag: app.drag.is_some() keeps it from deselect handling.
    // Release is the gesture terminator and commits the displayed offset.
}
```

`layer.translate` resolves selected roots and linked descendants, rejects position locks, moves them in one `Move` history edit, optionally auto-nests artboards, and can carry the selection when multiple layers move. The UI omits `target`, so an unlinked mask moves alone only through an explicit engine `layer.translate {"target":"mask"}` call, not this gesture.

### Drag a locked layer and receive a delayed error

Original source: [lock decision](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/move_lock.rs#L13-L48), [blocked pointer](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/tool_input.rs#L343-L360).

**Adapted Rust** — the press, move, and release branches are combined.

```rust
if move_lock::blocked(app, Tool::Move, [x, y], mods) {
    app.move_blocked = true;       // No Drag or MovePreview starts.
    return;
}
if app.move_blocked && pointer_moved {
    app.move_blocked = false;
    move_lock::prompt(app);        // Error dialog; layer.translate is never called.
}
if app.move_blocked && pointer_released {
    app.move_blocked = false;      // A click reports no lock dialog.
}
```

The locked layer can first be selected by Auto-Select; the later drag follows the layer-selection marquee path rather than translating it.

### Drag whole layers onto another document

Original source: [transfer behavior](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/panels/layer_transfer.rs#L1-L13), [whole-layer eligibility](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/panels/layer_transfer.rs#L62-L68), [tab hover](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/panels/layer_transfer.rs#L79-L106), [other canvas release](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/panels/layer_transfer.rs#L109-L131), [destination commit](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/panels/layer_transfer.rs#L146-L179), [Alt rollback](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/move_mods.rs#L184-L199).

**Adapted Rust pseudocode** — tab activation and destination placement are joined.

```rust
let eligible = drag.tool == Tool::Move && drag.sel_move.is_none()
    && app.ui.transform.is_none() && app.guide_drag.is_none()
    && document.selection.is_none() && session.floating.is_none();
if pointer_over_other_document_tab && eligible {
    app.drag = None;                // The source gesture will not release layer.translate.
    app.move_preview = None;
    move_mods::abandon(app);        // Rolls back an Alt duplicate already made in the source.
    transfer = Some(Transfer { source, selected_layers, grab: Some(drag.start), label });
    session.set_active(destination_index); // Spring-loaded tab shows the destination document.
}
while_pointer_down_over_destination: ghost_and_copy_cursor();
on_release_over_tab_or_canvas: {
    let mut params = {"document":destination,"source":source,"layers":transfer.layers};
    if !shift && release_point.is_some() {
        params["offset"] = [release_point - grabbed_point]; // Move keeps the grabbed pixel under the pointer.
    } else if !same_size {
        params["center"] = true;    // Shift/tab release preserves or centers according to transfer rules.
    }
    run("layer.copyToDocument", params)?; // One history step in the destination; errors show in status.
}
```

The source layers are copied, not translated or removed. A Move drag that begins inside a selection or already carries a floating piece remains an ordinary move and is not converted to this transfer.

### Drag a Move transform control into Free Transform

Original source: [press interception](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas.rs#L1035-L1044), [click fallback](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas.rs#L1203-L1208), [handle hit and session start](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/canvas/transform_controls.rs#L116-L135), [transform pointer](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/transform_tool/pointer.rs#L86-L158), [commit](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/transform_tool/commit.rs#L13-L111), [cancel](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/transform_tool/commit.rs#L255-L273), [Enter and Escape](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/input/shortcuts.rs#L587-L596).

**Adapted Rust pseudocode** — only the Move-to-transform dispatch boundary is shown.

```rust
if active_tool == Tool::Move && app.ui.transform.is_none() {
    begin_transform_controls_at(app, pointer)?; // Only a visible handle or outside rotation band starts.
}
if transform_tool::pointer(app, event, mods) {
    // Interior remains a normal Move target. A pasteboard click can commit; a click outside the box
    // but on the image restores jitter and leaves the box open.
    return;                                      // No ordinary Move Drag or layer.translate starts.
}
on_enter: transform_tool::commit(app);          // edit.transform/edit.transform.warp for a changed box.
on_escape: transform_tool::cancel(app);         // Ends the session and takes back a made layer.
```

An untouched box commits no history edit. A changed box dispatches its transform command; a large warp can return a pending background job and keep the session open until completion. A commit error also keeps the transform state available.

### Press an arrow key to nudge selected pixels

Original source: [arrow dispatch](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/move_mods.rs#L201-L228), [pixel nudge](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/move_mods.rs#L255-L270).

**Adapted Rust** — other arrow-key tool branches omitted.

```rust
let distance = if shift { 10.0 } else { 1.0 };
if text_editing || app.drag.is_some() { return; }       // A pointer gesture owns the arrows.
if move_ui::moves_selected_pixels_with(app, Tool::Move) {
    move_ui::float_selected(app, alt, arrow_dx * distance, arrow_dy * distance);
    return;                                              // Repeated calls move the same floating piece.
}
```

### Press an arrow key to nudge layers

Original source: [arrow dispatch](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/move_mods.rs#L201-L228), [layer nudge](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/tools/move_mods.rs#L255-L285).

**Adapted Rust** — the selected-pixel branch is omitted because it has its own flow above.

```rust
if app.ui.transform.is_some() {
    transform_tool::nudge(app, dx, dy);                  // Free Transform owns arrows while open.
    return;
}
if app.drag.is_some() { return; }
let from = if alt { duplicate_layers_in_place(app) } else { None }; // Option<usize>; Some(4).
let moved = if alt {
    app.run("layer.translate", {"dx":dx,"dy":dy})       // Copy and move are folded below.
} else {
    nudge_bundle::run(app, "layers", "layer.translate", {"dx":dx,"dy":dy})
}; // Result<Value,String>; repeated plain nudges coalesce into one history step.
if let Err(e) = moved { app.ui.status = e; app.ui.status_error = true; }
if let Some(from) = from { fold_history(app, from); }   // One "Duplicate + Move" history step.
```
