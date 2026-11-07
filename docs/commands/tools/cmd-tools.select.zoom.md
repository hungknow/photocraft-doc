# tools.select.zoom

Selects the Zoom tool for changing the canvas view.

Verified against: `f68e227026701cf24e0e182530694240e2d3cfe4` · 2026-11-07
Navigation: [tools commands](../tools.md#tools) · [View/workspace tools](../../tools/view-workspace.md#view-workspace)

## Context to use

The Zoom tool changes the view camera and works without a document; with a document, its anchor and zoom range use that document's dimensions. It does not edit pixels, selections, layers, masks, color, or bit depth.

- The `Z` binding is associated with this command ID. In the interactive UI, pressing it selects Zoom through the shortcut handler's direct state update; it does not dispatch the ID. [Registration](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/engine/src/shell_commands.rs#L141) · [Key handler](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/input/shortcuts.rs#L908-L936)
- Choose Zoom in the toolbar to set the same UI tool directly; this bypasses the command ID. [Toolbar selection](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/panels/toolbar.rs#L235-L275)
- After selection, click or drag on the canvas to zoom. A plain double-click on the Zoom toolbar button selects it and runs `view.actualPixels`. [Double-click behavior](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/panels/toolbar.rs#L286-L289)

Headless command dispatch does not select a UI tool or change the document; the interactive shortcut handler sets `app.ui.tool` directly.

## Inputs and response

There are no command parameters. This is a valid empty-parameter invocation:

```json
{}
```

The shell returns its normal successful command response and sets the selected tool to Zoom. A bad or unknown command ID is reported as an error by normal command dispatch. Canvas gestures that follow change only view state.

## Structures used

The drag state lives temporarily in egui context data; `rect_to` distinguishes rectangle zoom from scrubby zoom.

Original source: [ZoomDrag and View](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/zoom_tool.rs#L18-L28).

**Adapted Rust** — focused fields; `View` is the existing view state.

```rust
struct ZoomDrag {
    anchor: Pos2,          // (240, 180) screen point where press began
    doc: [f64; 2],         // [640.0, 480.0] document point kept under anchor
    zoom0: f32,            // 1.0 starting zoom
    rect_to: Option<Pos2>, // Some((400, 300)) rectangle corner, None means scrubby mode
}
```

## Core logic flow

Representative route: `Z` is associated with the ID in `shell_commands.rs`; `input::shortcuts::select_tool_by_key` handles the key directly, resolves its tool group and changes `app.ui.tool` without dispatching the ID; a later canvas event calls the Zoom handling in `canvas.rs`; `zoom_tool::drag` updates view state during drag and release, and a separate canvas paint pass calls `zoom_tool::draw` for rectangle feedback. A click follows the canvas click branch instead. These following flow subsections are alternatives, not a top-to-bottom sequence.

### Invoke the ID to select Zoom

This flow is keyboard shortcut selection; it directly sets UI state and does not dispatch the command ID. The toolbar button is a separate direct selection route. Explicit command dispatch is exposed through the shell command catalog, but it is not the route used by either interactive shortcut or toolbar selection.

Original source: [registration and key handling](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/engine/src/shell_commands.rs#L105-L142), [key dispatch](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/input/shortcuts.rs#L908-L936).

Purpose: This adapted fragment shows the shortcut selection result.

**Adapted Rust** — narrowed to the relevant key group and resulting state.

```rust
// tools.select.zoom is catalogued with default key Z; this UI handler handles the key directly.
if pressed(Key::Z) {
    app.ui.tool = Tool::Zoom; // no document mutation or history entry
    tool_memory::ended_by_tool(app);
    return true;
}
```

### Click or drag the canvas to change zoom

This later gesture runs after tool selection. Canvas processing yields to an open Free Transform control; animated Zoom can consume the press first. Otherwise `zoom_tool::drag` captures the anchor and consumes drag events. `scrub_zoom` maps horizontal screen distance to powers of two, clamped to document-supported zoom levels; `anchor_view` updates zoom and center while preserving the anchored document point. It returns a plain `f32` zoom, not a fallible result.

Original source: [canvas dispatch and click](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas.rs#L973-L987), [click branch](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas.rs#L1180-L1188), [drag algorithm](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/tools/zoom_tool.rs#L34-L86).

Purpose: This adapted fragment follows scrubby drag from press through release.

**Adapted Rust** — shortened to the state-changing branches.

```rust
if buttons.started {
    let anchor = pointer_inside_canvas()?;
    let z = ZoomDrag { anchor, doc: xf.to_doc(anchor), zoom0: view.zoom, rect_to: None };
    ctx.data_mut(|d| d.insert_temp(id(), z));
}
if buttons.dragged || buttons.stopped {
    let zoom = scrub_zoom(z.zoom0, pointer.x - z.anchor.x, view.doc_size); // f32, e.g. 2.0
    anchor_view(view, xf, z.anchor, z.doc, zoom, app.ppp);
}
if buttons.stopped {
    ctx.data_mut(|d| d.remove::<ZoomDrag>(id())); // release ends drag; no history entry
}
```

With scrubby Zoom disabled, the drag stores `rect_to` instead. The later paint pass calls `draw`, which renders marching-ants feedback. On release, `zoom_to_rect` fits the dragged rectangle; a rectangle under four screen points in either dimension steps one level at the press anchor. Holding Alt at release steps out. No history entry is created.

### Click to step zoom in or out

A click with no consumed drag takes the canvas click match in a later event. Alt selects a step out; otherwise it steps in. Preferences decide whether the clicked point stays anchored or becomes the view center, and animated zoom may interpolate the camera. `zoom_levels::step` returns a clamped `f32`; the view state records the animation anchor when needed.

Original source: [click handling](https://github.com/storytold/photocraft/blob/f68e227026701cf24e0e182530694240e2d3cfe4/crates/ui-egui/src/canvas.rs#L1180-L1188).

Purpose: This adapted fragment shows click direction and animation setup.

**Adapted Rust** — omitted neighboring tool branches.

```rust
let next = zoom_levels::step(view.zoom, if alt { -1 } else { 1 }, view.doc_size); // f32, e.g. 1.25
zoom_about(view, xf, pointer, next, prefs.zoom_clicked_point_to_center, ppp);
view.request_zoom_animation(prefs.animated_zoom && next != display_zoom);
```
