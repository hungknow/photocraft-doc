# tools.select.zoom

Selects the Zoom tool for changing the canvas view.

Verified against: `f68e227026701cf24e0e182530694240e2d3cfe4` · 2026-11-07
Navigation: [tools commands](../tools.md#tools) · [View/workspace tools](../../tools/view-workspace.md#view-workspace)

## Context to use

- Zoom changes the view camera, not document pixels, selections, layers, masks, color, or bit depth.
- It works without an open document; when a document is open, its dimensions set the supported zoom range.
- The `Z` key is associated with this ID, but the interactive shortcut handler changes UI state directly instead of dispatching the ID. [Registration](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/engine/src/shell_commands.rs#L141) · [Key handler](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/input/shortcuts.rs#L908-L936)
- Choosing Zoom in the toolbar also sets the tool directly, bypassing this ID. [Toolbar selection](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/panels/toolbar.rs#L235-L275)
- After selecting Zoom, click or drag the canvas to change magnification. A plain double-click on the toolbar Zoom button selects it and runs `view.actualPixels`. [Toolbar double-click](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/panels/toolbar.rs#L286-L289)
- Headless dispatch does not select a UI tool or change the document.

## Inputs and response

- Parameters: none. A valid empty invocation is:

```json
{}
```

- Success: shell command dispatch returns its normal success response; interactive selection sets the UI tool to Zoom.
- Error: unknown IDs follow normal dispatch error handling.
- Later canvas gestures change view state only and do not create document history.

## Structures used

- `ZoomDrag` is stored temporarily in egui context data while a drag is active.
- `rect_to: Some(point)` tracks rectangle-zoom mode; `None` tracks scrubby mode.

Original source: [ZoomDrag](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/zoom_tool.rs#L18-L28).

**Adapted Rust** — focused fields with representative values.

```rust
struct ZoomDrag {
    anchor: Pos2,          // (240, 180) screen point where the press began
    doc: [f64; 2],         // [640.0, 480.0] document point kept at that anchor
    zoom0: f32,            // 1.0 starting magnification
    rect_to: Option<Pos2>, // Some((400, 300)) rectangle corner; None means scrubby zoom
}
```

## Algorithm

### Map document pixels onto the rendered canvas

- `canvas.rs` builds `ViewXform` with `view.zoom / ppp`, converting stored physical-pixel zoom to egui points per document pixel. [Renderer setup](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas.rs#L1912-L1924)
- `ViewXform::to_screen` subtracts the camera center, stretches x by pixel aspect, rotates, optionally flips x, scales by zoom, and offsets by the canvas center. `doc_rect` maps all four corners to get screen bounds. [ViewXform](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas/view_xform.rs#L101-L145)
- The GPU shader applies the corresponding camera scale when mapping document positions to render pixels. [GPU mapping](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas/gpu_canvas/shaders.rs#L53-L76)

Purpose: This adapted calculation maps a sample document point with no rotation, flip, or pixel-aspect correction.

**Adapted Rust** — simplified from `ViewXform::to_screen`; coordinates are egui points and document pixels.

```rust
let zoom = 2.0_f32;            // 2 physical display pixels per document pixel (200%)
let ppp = 2.0_f32;             // 2 physical pixels per egui point
let scale = zoom / ppp;        // 1 egui point per document pixel
let canvas_center = (500.0, 350.0); // screen points
let camera_center = (1000.0, 750.0); // document pixels
let doc_point = (1200.0, 800.0); // document pixels
let screen_point = (
    canvas_center.0 + (doc_point.0 - camera_center.0) * scale,
    canvas_center.1 + (doc_point.1 - camera_center.1) * scale,
); // (700, 400) points = (1400, 800) physical pixels
```

- Worked result: a 2000×1500 document at 200% occupies 4000×3000 physical pixels, or 2000×1500 egui points at 2 physical pixels per point.
- At 50%, it occupies 1000×750 physical pixels.
- Pixel aspect multiplies the x scale; rotation mixes the x/y offsets; horizontal flip reverses the final x offset.
- `to_doc` reverses the mapping so the document point under the cursor can stay fixed during zoom.

## Core logic flow

Representative route:

1. The shell catalog associates `tools.select.zoom` with `Z`; interactive `input::shortcuts::select_tool_by_key` handles the key directly and sets `app.ui.tool`. [Catalog](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/engine/src/shell_commands.rs#L105-L142) · [Shortcut handler](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/input/shortcuts.rs#L908-L936)
2. On a later canvas event, `canvas.rs` gives an open Free Transform control priority; animated Zoom may consume the press before ordinary Zoom drag/click handling. [Canvas dispatch](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas.rs#L973-L987)
3. `zoom_tool::drag` updates view state during drag/release. Rendering occurs in the canvas pass; rectangle feedback is drawn separately by `zoom_tool::draw`.
4. Clicks use the canvas click branch. These flow subsections are alternatives, not a single top-to-bottom execution sequence.

### Press Z to select Zoom

- This shortcut route directly sets UI state; it does not dispatch the command ID or mutate the document.
- Toolbar selection is a separate direct state update.

Original source: [Shortcut handler](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/input/shortcuts.rs#L908-L936).

Purpose: This adapted fragment shows the selected-tool state change.

**Adapted Rust** — narrowed to the Zoom key result.

```rust
if pressed(Key::Z) {
    app.ui.tool = Tool::Zoom; // UI state only; no document history
    tool_memory::ended_by_tool(app);
    return true;
}
```

### Drag on canvas to scrub or fit a rectangle

- `canvas.rs` calls `zoom_tool::drag` only when Zoom is selected and Free Transform does not own the pointer.
- On press, `drag` saves the screen anchor, its document coordinate, and starting zoom in `ZoomDrag`.
- In scrubby mode, horizontal distance `dx` produces `zoom0 * 2^(dx/100)`. The exponent is bounded to ±40 and `zoom_levels::clamp` limits the result to the document's allowed range.
- `anchor_view` stores the new zoom and adjusts the view center to keep the original document point under the screen anchor.
- On release, the temporary drag state is removed. There is no document commit or history entry.

Original source: [Drag routing and math](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/zoom_tool.rs#L34-L86).

Purpose: This adapted fragment shows scrubby zoom calculation and anchor preservation.

**Adapted Rust** — focused excerpt; `pointer` is the current canvas position.

```rust
// Press handling stores ZoomDrag { anchor, doc, zoom0, rect_to: None } in egui temp data.
let dx = pointer.x - z.anchor.x;       // e.g. 100 screen points
let exponent = (dx / 100.0).clamp(-40.0, 40.0);
let zoom = zoom_levels::clamp(z.zoom0 * 2.0_f32.powf(exponent), view.doc_size); // f32; 2.0 at dx=100
anchor_view(view, xf, z.anchor, z.doc, zoom, app.ppp); // updates view zoom and center
// Release removes the temporary ZoomDrag; no document pixels or history are changed.
```

- When scrubby zoom is off, `rect_to` stores the drag corner instead. A later canvas paint pass calls `draw` to render marching-ants feedback.
- On release, `zoom_to_rect` fits a rectangle of at least 4×4 screen points. A smaller rectangle steps one zoom level at the press anchor; Alt steps out. [Rectangle logic](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/zoom_tool.rs#L89-L110)

### Click canvas to step zoom

- A click not consumed by the drag path reaches the `Tool::Zoom` click branch in `canvas.rs`.
- Alt steps down one zoom level; otherwise it steps up. `zoom_levels::step` returns a clamped `f32` zoom.
- `zoom_about` applies the configured clicked-point centering behavior. Animated zoom may interpolate the view camera to the new value.

Original source: [Zoom click handling](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas.rs#L1180-L1188).

Purpose: This adapted fragment shows click direction and animation setup.

**Adapted Rust** — neighboring tool cases omitted.

```rust
let next = zoom_levels::step(view.zoom, if alt { -1 } else { 1 }, view.doc_size); // f32; e.g. 1.25
zoom_about(view, xf, pointer, next, prefs.zoom_clicked_point_to_center, ppp);
view.request_zoom_animation(prefs.animated_zoom && next != display_zoom);
```
