# tools.select.line

Select the Line tool so a later canvas drag can create a vector shape or extend the work path.

Verified against: `f68e227026701cf24e0e182530694240e2d3cfe4` · 2026-11-07
Navigation: [tools commands](../tools.md#tools) · [Vector shapes actions](../../tools/vector-shapes.md#vector-shapes) · [command categories](../index.md#categories) · [tool categories](../../tools/index.md#categories)

## Context to use

Selection works without an open document. Drawing requires an open document; Shape mode creates a new vector layer, while Path mode writes the work path. The options bar controls mode, fill, stroke, line weight and arrowheads. [Options](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/state/tool_options.rs#L99-L120)

- Assign or invoke the Line Tool shortcut in Edit › Keyboard Shortcuts. Its default is `U`, shared with the other shape tools; Shift cycles the group when default bindings are used. The shortcut ID is `tools.select.line`. [Catalog](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/engine/src/shell_commands.rs#L168-L173) · [ID mapping](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/input/tool_shortcuts.rs#L6-L25)
- Click the Line entry in the shape-tool toolbar flyout, or Alt-click its slot until Line is selected. These call `tool_memory::select` directly; they do not dispatch this ID. [Toolbar](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/panels/toolbar.rs#L266-L275) · [Flyout](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/panels/toolbar.rs#L385-L392)
- Drag on the canvas after selecting Line. This is a later pointer flow, not execution of `tools.select.line`; it ultimately runs `shape.create` or `path.set`. [Release](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas/gesture.rs#L102-L109) · [Finish](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/vector_ui/shapes.rs#L101-L150)

Headless `Session::execute` has no selected UI tool and cannot execute this shell ID. A caller with a UI app can invoke it through the UI command route; headless drawing uses `shape.create` or `path.set` directly. [Shell route](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/menus/invoke.rs#L77-L82)

## Inputs and response

Pass a JSON object to the UI command route:

```json
{"command":"tools.select.line","params":{}}
```

There are no required fields. The shell selection route does not read parameter fields. It returns `{"tool":"Line"}` (the serialized `Tool` enum) and updates UI tool state; no document edit or History step occurs. Unknown or differently cased IDs fail lookup rather than selecting Line. [Route](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/menus/invoke.rs#L77-L82) · [Exact ID check](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/input/tool_shortcuts.rs#L14-L18)

A later valid Shape mode drag returns a `shape.info` JSON object internally, including the new layer ID; the UI release handler reports errors in the status bar. In Path mode, `path.set` returns the work path JSON internally. A zero-length or subpixel drag, or a refused press, makes no edit. A short drag of at least one document pixel can create a Line even when it meets the click tolerance. [Shape result](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/engine/src/cmd/vector_cmds/shape.rs#L369-L385) · [Path result](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/engine/src/cmd/vector_cmds/paths.rs#L76-L114)

## Structures used

Original source: [Tool and ToolOptions](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/state/tool_options.rs#L99-L120) · [Tool variant](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/state/tool.rs#L69-L75).

**Adapted Rust** — only Line-relevant options shown.

```rust
enum Tool { Line } // The selected UI tool; serialized as "Line".
struct ToolOptions {
    shape_mode: String, // "shape" creates a layer; "path" edits the work path.
    shape_fill: bool,   // true: use foreground or chosen gradient as fill.
    stroke_width: f32,  // 2.0 px: separate outline stroke; 0 disables it.
    line_weight: f32,   // 5.0 px: thickness of the line geometry.
    arrow_start: bool,  // false: no arrow at the press endpoint.
    arrow_end: bool,    // true: add an arrow at the release endpoint.
    arrow_width: f32,   // 20 px across the head in the worked example.
    arrow_length: f32,  // 12 px along the line.
    arrow_concavity: f32, // 50%: a rear notch halfway through the head.
}
```

Original source: [Drag](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas/drag.rs#L4-L25) · [Owner capture scope](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas/drag.rs#L75-L84).

**Adapted Rust** — only gesture fields used here. The Line tool does not capture a vector owner.

```rust
struct Drag {
    tool: Tool,            // Line: the tool at pointer press.
    start: [f64; 2],       // [10.0, 150.0] document pixels.
    points: Vec<[f64; 3]>, // Last sample [90.0, 152.0, 1.0], including pressure.
    live: egui::Modifiers, // Shift at release constrains angle to 45° steps.
    vector_owner: Option<VectorGestureOwner>, // None for Line; owner capture covers other vector tools.
}
```

Original source: [ShapeLayer fields](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/doc/src/vector.rs#L336-L354) · [Creation](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/engine/src/cmd/vector_cmds/shape.rs#L425-L462).

**Adapted Rust** — the stored vector fields relevant to this flow.

```rust
struct ShapeLayer {
    path: Path,               // Line or arrow outline from (10,150) to (90,150).
    fill: Option<Fill>,       // Some(Solid(...)) in filled Shape mode.
    stroke: Option<ShapeStroke>, // Optional separate outline stroke.
    live: Option<LiveShape>,  // Editable Line or ArrowLine geometry.
}
```

Original source: [Shape box state](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/shape_bounds_box.rs#L27-L46) · [Pointer updates](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/shape_bounds_box.rs#L180-L232).

**Adapted Rust** — fields that identify and scale a selected shape box.

```rust
struct ShapeBoxDrag {
    doc: DocId,            // The selected shape’s document at press.
    revision: u64,         // Its revision; a change cancels the handle drag.
    layers: Vec<u64>,      // [42]: selected shape layer IDs.
    rect: [f64; 4],        // [10, 20, 90, 60]: starting path bounds.
    handle: [i8; 2],      // [1, 1]: bottom-right corner.
    scale: [f64; 2],      // [1.25, 1.25]: current preview scale.
}
```

## Algorithm

The Line ID selects a tool. A later canvas drag supplies the geometry. The example below uses document pixels and assumes snapping leaves the endpoints unchanged. [Geometry](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/vector_ui/shapes.rs#L48-L65)

- Press at (10, 150) and drag to (90, 152), holding Shift. The raw vector is (80, 2) px. Its length is √(80² + 2²) = 80.025 px and its angle is about 1.43°. Rounding to the nearest 45° gives 0°; the resulting endpoint is (90.025, 150) px. A drag shorter than one document pixel returns no geometry, so it has no preview or commit. [Constraint](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/vector_ui/shapes.rs#L51-L61)
- Choose a 5 px weight, an end arrow 20 px wide and 12 px long, and 50% concavity. The geometry value contains the adjusted endpoints, weight and arrow options. The parser takes those parameters and returns Result<LiveShape, EngineError>, here an editable ArrowLine; invalid arrow flag types or dimensions return an error. [Parameters](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/vector_ui/shapes.rs#L62-L65) · [Parser](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/engine/src/cmd/vector_cmds/params.rs#L358-L405)
- The vector algorithm returns a Path with a 5 px wide shaft from x = 10 to x = 84.025, between y = 147.5 and 152.5. The arrow tip is (90.025, 150); its rear corners are (78.025, 140) and (78.025, 160). The 50% rear notch lands at (84.025, 150). Thus the user sees a thin bar ending in a 20 px wide head. With both arrow flags false, the same function constructs a four-corner filled bar. Non-finite coordinates produce an empty Path. [Bar](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/vector/src/shapes.rs#L209-L216) · [Arrow](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/vector/src/shapes.rs#L224-L288)
- In a separate paint pass, the preview path gets the current fill and optional stroke in Shape mode. The view transform maps it to screen coordinates. At 2× zoom with screen origin (0, 0), the tip appears near (180.05, 300) screen pixels, and the head spans about 40 screen pixels vertically. The preview renderer returns no value: it compiles the shape, rasterizes its visible bounds to RGBA, caches the texture and paints it. Empty bounds or invalid clip draw nothing. It limits oversized previews to a 2048 px proxy. Path mode shows only the accent outline. [Preview construction](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/vector_ui/shapes.rs#L195-L241) · [Preview raster](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/shape_stroke_ui.rs#L90-L131)
- On release in Shape mode, the engine parses and validates the same geometry. It rasterizes the stored layer cache in the document pixel format and canvas bounds, then inserts the vector layer. The screen RGBA preview does not set the document color model or depth. In Path mode, the engine stores the generated vector subpath in the work path instead. [Commit](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/engine/src/cmd/vector_cmds/shape.rs#L425-L462) · [Document renderer](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/vector/src/lib.rs#L695-L699) · [Path mode](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/vector_ui/shapes.rs#L101-L124)
- A shape-box handle uses a separate scale algorithm. For bounds [10, 20, 90, 60], a bottom-right handle drag by (20, 10) px has diagonal (80, 40) px. Its projected scale is 1 + (20×80 + 10×40)/(80² + 40²) = 1.25 on both axes. The opposite corner (10, 20) stays fixed, so the preview box becomes [10, 20, 110, 70]. Release passes 125% on both axes to edit.transform, which updates the selected paths. [Scale](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/shape_bounds_box.rs#L93-L126) · [Preview and commit](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/shape_bounds_box.rs#L202-L251)

## Core logic flow

### Read the execution route before the individual flows

1. The shortcut dispatcher invokes the shell command; `menus::invoke` reaches `invoke_early_commands`, whose exact ID parser resolves `Tool::Line`. `tool_memory::select` changes the selected UI tool and returns `{"tool":"Line"}`. The toolbar calls the same selector directly. [Shell selection](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/menus/invoke.rs#L77-L82) · [Selector](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tool_memory.rs#L24-L33)
2. On a later canvas press, `tool_event` runs canvas interceptors, including an existing shape box handle, then creates `Drag` for an ordinary Line press. The move-lock, hidden-paint-target and rasterize-paint checks do not block ordinary Line creation. Pointer moves update the drag. Line does not capture a `VectorGestureOwner`, so its release has no revision/target ownership check. [Input preflight](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas/tool_input.rs#L349-L395) · [Handle interceptor](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas/tool_input.rs#L169-L178) · [Capture](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas/tool_input.rs#L527-L536)
3. In a separate paint pass, `canvas::paint` calls `draw_shape_preview`, which derives line geometry, turns it into a vector path and paints a screen overlay. This does not mutate the document. [Paint call](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas/paint.rs#L644) · [Preview](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/vector_ui/shapes.rs#L210-L241)
4. On release, `finish_release` calls `finish_shape` for a Line drag; its owner check is inactive because Line has no captured owner. Depending on mode, it runs `shape.create` or `path.set`; the engine validates and records one edit in History. [Release](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas/gesture.rs#L16-L32) · [Finish](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/vector_ui/shapes.rs#L101-L150)

`shape_geometry` takes tool options, endpoints and live modifiers; it returns `Option<Value>` (`None` below one pixel, otherwise JSON geometry). `shape_path` takes that JSON and returns `Result<Path, EngineError>`; invalid geometry is an error, which the preview converts to no overlay. `create_shape` returns `Result<Value, String>`; an engine failure becomes status text. [Geometry](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/vector_ui/shapes.rs#L48-L65) · [Path](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/engine/src/cmd/vector_cmds/shape.rs#L70-L74) · [Create](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/vector_ui/shapes.rs#L145-L150)

The flows below are alternatives, not one top-to-bottom execution sequence.

### Invoke the ID to select Line

The UI command route admits this tool ID without a document. Selection clears temporary spring and stylus overrides, updates `ui.tool`, and does not add a History entry. Direct toolbar selection reaches `tool_memory::select` without running the ID. [Enable check](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/menus/state.rs#L5-L11) · [Selection](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tool_memory.rs#L24-L33)

**Adapted Rust** — the shell route and selector are condensed. This group changes UI state only.

```rust
if let Some(tool) = tool_shortcuts::tool(id) { // Option<Tool>; Some(Line) only for the exact ID.
    tool_memory::select(app, tool);            // Clears temporary tool overrides, sets ui.tool.
    return Ok(json!({"tool": tool}));          // Result<Value, String>; {"tool":"Line"}.
}
```

### Drag in Shape mode to create a Line layer

At press, shape-box handles and other canvas overlays may take the gesture. The ordinary Line press is not blocked by the active layer’s paint or move lock, because it creates a new shape layer. Line drags do not capture document or target ownership; unlike Freeform Pen and Path Selection, a changed document, layer or mode is not caught by the vector-owner check. The options bar affects the current preview and release geometry. [Input](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas/tool_input.rs#L349-L395) · [Move-lock scope](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/move_lock.rs#L18-L43) · [Owner capture scope](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas/drag.rs#L75-L84)

Holding Space repositions the whole gesture; snapping can adjust its bounds. Shift snaps the line angle to 45° increments. Alt does not change Line geometry: the centre-draw Alt branch applies to the other shapes. [Reposition](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/input/hold_keys.rs#L222-L229) · [Snapping](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas/snap_ui.rs#L516-L536) · [Geometry](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/vector_ui/shapes.rs#L48-L74)

Original source: [Line geometry](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/vector_ui/shapes.rs#L48-L65) · [Live line parsing](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/engine/src/cmd/vector_cmds/params.rs#L355-L405) · [Path generation](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/vector/src/shapes.rs#L285-L288).

**Adapted Rust** — endpoint calculations and parsing are shortened. This group derives the same geometry for preview and commit.

```rust
let (mut dx, mut dy) = (end[0] - start[0], end[1] - start[1]);
if mods.shift {
    let angle = (dy.atan2(dx) / FRAC_PI_4).round() * FRAC_PI_4; // Shift snaps angle to 45°.
    let length = dx.hypot(dy);
    (dx, dy) = (length * angle.cos(), length * angle.sin());
}
if dx.hypot(dy) < 1.0 { return None; } // Option<Value>: no geometry, no edit.
let geometry = json!({
    "kind":"line", "from":start, "to":[start[0]+dx,start[1]+dy],
    "weight":options.line_weight.max(1.0),
    "arrowStart":options.arrow_start, "arrowEnd":options.arrow_end,
    "arrowWidth":options.arrow_width, "arrowLength":options.arrow_length,
    "arrowConcavity":options.arrow_concavity
}); // Value; e.g. from [10,150], to [90,150], weight 5.
```

The separate render pass calls `shape_preview`: `shape_outline` turns geometry into a `Path` using `shape_path`; Shape mode adds the chosen fill and optional stroke. `draw_shape_preview` transforms that path into screen coordinates and paints both shape and contrast outline. A missing or invalid outline yields no overlay. [Build](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/vector_ui/shapes.rs#L197-L220) · [Paint](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/vector_ui/shapes.rs#L222-L241)

On release, `finish_shape` reruns geometry using live modifiers, adds fill and stroke parameters, and calls `shape.create`. `live_from_params` validates endpoints, weight and arrow dimensions; `live_path` makes the stored outline. The engine validates the path, renders its cache in the document's pixel format and profile path, inserts and selects a Shape layer, and records a `New Shape Layer` History step. It returns `shape.info` with layer, path and bounds. Failure leaves an error in UI status. [Commit](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/vector_ui/shapes.rs#L125-L150) · [Engine](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/engine/src/cmd/vector_cmds/shape.rs#L425-L462)

**Adapted Rust** — the UI handoff and engine branch are condensed. This group commits an undoable vector layer.

```rust
let Some(mut p) = shape_geometry(options, Tool::Line, start, end, live_mods) else { return; }; // Option<Value>; None aborts.
p["fill"] = fill_param(app);     // Value: foreground/gradient, or null.
p["stroke"] = stroke_param(app); // Value: outline stroke, or null.
if let Err(e) = app.run("shape.create", p) { // Result<Value, String>; Ok has {"layer":42,"kind":"line",...}.
    app.ui.status = e; app.ui.status_error = true; // A rejected edit is shown in the status bar.
}
// Engine success: s.edit("New Shape Layer", ...) inserts/selects the layer.
```

A click within the shape dialog's three-screen-pixel tolerance calls `shape_dialog::open`, which returns `None` for Line. It falls through to `finish_shape`: a drag below one document pixel is discarded, while a short drag at or above that threshold can create a Line. No Create Line dialog opens. [Click](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas/gesture.rs#L102-L108) · [Dialog exclusion](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/dialogs/shape_dialog.rs#L50-L59)

### Drag in Path mode to extend the work path

The same canvas interceptors apply. Line has no captured owner in either mode. The paint pass uses the same line outline but suppresses fill and stroke, so only the vector outline previews. On release, `finish_shape` appends the outline's subpaths to a copy of the existing work path, or starts a work path with that fill rule. It runs `path.set` with the complete path, selects `work` in the UI, and records a `Work Path` History step. It does not create a layer or change the active layer. Path validation errors are shown in status. A click in Path mode adds nothing. [Preview](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/vector_ui/shapes.rs#L210-L219) · [UI finish](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/vector_ui/shapes.rs#L101-L124) · [Engine path](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/engine/src/cmd/vector_cmds/paths.rs#L76-L114)

**Adapted Rust** — the existing-path and error branches are shortened. This group commits a work path rather than a layer.

```rust
let Some(outline) = shape_outline(app, Tool::Line, start, end, live_mods) else { return };
let mut path = active_doc.work_path.clone().unwrap_or_else(|| Path { fill_rule: outline.fill_rule, ..Default::default() }); // Path; empty with matching fill rule when none existed.
path.subpaths.extend(outline.subpaths); // Keep earlier components; add the line.
match app.run("path.set", json!({"name":"work","path":path_json(&path)})) {
    Ok(_) => app.ui.selected_path = Some("work".into()), // Result<Value, String>; work path JSON.
    Err(e) => { app.ui.status = e; app.ui.status_error = true; } // No successful edit.
}
```

### Drag a selected shape box handle to transform existing paths

With Extras and Target Path visible, selected shape layers show a bounding box under the Line tool. A press on one of its eight handles, unless Command is held, is consumed by `shape_bounds_box::pointer` before the ordinary Line drag begins. `hit` returns `Option<[i8; 2]>` for the nearest handle within screen-scaled grab distance; `selection` returns `Option<(Vec<u64>, [f64; 4])>` for all selected shape layers and their combined path bounds. A mixed or empty selection has no box, so the press can draw a new Line instead. [Visibility and selection](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/shape_bounds_box.rs#L48-L75) · [Hit and press](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/shape_bounds_box.rs#L133-L140) · [Interceptor](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas/tool_input.rs#L169-L178)

Move updates `ShapeBoxDrag.scale`: a corner scales both axes equally; an edge handle scales one axis, with finite bounds enforced. A separate draw pass transforms the existing paths for an outline preview without editing them. Escape, tool/document/revision changes, or hiding the box cancel the drag; its remaining release is consumed. [Scale](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/shape_bounds_box.rs#L103-L118) · [Preview](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/shape_bounds_box.rs#L236-L251) · [Cancel](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/shape_bounds_box.rs#L162-L177)

**Adapted Rust** — the release branch is shortened. This group dispatches a transform of existing shape paths, with no new Line.

```rust
if drag.scale == [1.0, 1.0] { return true; } // A handle click has no edit.
let fixed = fixed_point(drag.rect, drag.handle); // [f64; 2]; e.g. opposite corner [10,20].
let params = json!({
    "layers":drag.layers, "rect":drag.rect,
    "scaleX":drag.scale[0] * 100.0, "scaleY":drag.scale[1] * 100.0,
    "reference":fixed, "stepName":TRANSFORM_PATH
});
if let Err(e) = app.run("edit.transform", params) { // Result<Value, String>; error is shown.
    app.ui.status = e; app.ui.status_error = true;
}
```

The engine’s `edit.transform` validates the listed layers and transform budget, applies the transform to the shape content, and records its `Transform Path` History step; a refused transform reports an error. [Validation](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/engine/src/cmd/transform_cmds.rs#L855-L896) · [Apply](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/engine/src/cmd/transform_cmds.rs#L990-L1011)
