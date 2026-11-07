# layer.new.layer

Create and select a raster layer with optional blending, clipping, label, and neutral fill.

Verified against: `3d60ad01f941519307c755eed8e0896d6274cde6` · 2026-11-06
Navigation: [layer commands](../layer.md#layer) · [Layers/masks actions](../../tools/layers-masks.md#layers-masks) · [command categories](../index.md#categories) · [tool categories](../../tools/index.md#categories)

## Context to use

An open document is required. The active layer determines placement; the command does not require a pixel layer or a selection. A locked open group can receive the new layer, which then inherits its effective lock. [Registration](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/cmd.rs#L498-L507) · [Placement](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/layer_place.rs#L11-L28)

- Layer › New › Layer… or Cmd+Shift+N opens the New Layer dialog; OK runs the command and Cancel leaves the document alone. [Menu](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/menu_catalog.rs#L246) · [Interception](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/menus/invoke.rs#L495-L496) · [OK](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/dialogs/new_layer_ui.rs#L131-L133)
- The Layers footer button creates immediately; Option-click opens the dialog, and Cmd/Ctrl-click passes `position: "below"`. [Footer](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/panels/layers.rs#L415-L428)

Headless callers execute the engine command directly, so empty parameters create a layer immediately without opening a dialog. [Engine entry](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/session/execute.rs#L17-L23)

## Inputs and response

Pass a JSON object (or `null`) to an active session, for example:

```json
{"command":"layer.new.layer","params":{"name":"Shade","mode":"Overlay","opacity":0.4,"fillNeutral":true}}
```

- `name`: optional string; omitted or `null` gets the next `Layer N` name. An explicit empty string remains empty in the engine; the dialog omits a blank name.
- `clipToPrevious`, `fillNeutral`: optional booleans, default `false`.
- `mode`: blend mode name, default `Normal`; `Pass Through` is refused for a raster layer.
- `opacity`: finite number, default `1`; values outside 0–1 are clamped.
- `color`: layer label ID, default `none`.
- `position`: `above` (default) or `below`; below a Background resolves to above it.

The result is JSON such as `{"layer":42}`; that ID is selected and one undoable edit is recorded. [Handler](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/new_layer.rs#L38-L62)

Malformed field types, unknown modes or colors, and `fillNeutral: true` for a mode without a neutral color return parameter errors before mutation. Missing documents and conflicting document jobs also refuse execution. [Parser](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/new_layer.rs#L66-L103) · [Admission](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/session/execute.rs#L96-L117)

## Structures used

The parsed options carry validated values into the edit. Original source: [NewLayer](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/new_layer.rs#L27-L36).

**Adapted Rust** — sample comments show the invocation above; no fields omitted.

```rust
struct NewLayer {
    clip: bool,                 // false: do not force a clipping mask
    mode: BlendMode,            // Overlay
    opacity: f32,               // 0.4
    label: LabelColor,          // None
    neutral: Option<[f32; 3]>, // Some([0.5; 3]): Overlay's neutral gray
    below: bool,                // false: automatic placement
    name: Option<String>,       // Some("Shade")
}
```

The document layer receives the new ID, properties and raster content. Original source: [Layer fields](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/doc/src/lib.rs#L632-L650) · [LayerId](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/doc/src/lib.rs#L60-L61).

**Adapted Rust** — other Layer fields omitted.

```rust
struct LayerId(u64); // 42: returned as JSON number
struct Layer {
    id: LayerId,          // LayerId(42)
    name: String,         // "Shade"
    blend: BlendMode,     // Overlay
    opacity: f32,         // 0.4
    clipped: bool,        // false unless forced or inserted into a clip group
    label: LabelColor,    // None
    content: LayerContent,// Raster surface in the document pixel format
    locks: Locks,         // inherited through a locked parent group when applicable
}
```

## Core logic flow

### Menu or Option-click opens the New Layer dialog

An empty-parameter menu invocation or Option-click on the Layers footer enters the shell route. It opens a dialog with the next layer name and defaults, returning a dialog ID without changing the document. Cancel closes it without dispatch. OK converts the displayed opacity percentage to a 0–1 fraction, omits a blank name, and passes the resulting options to the creation flow below. [Shell route](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/menus/invoke.rs#L495-L496) · [Dialog defaults and conversion](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/dialogs/new_layer_ui.rs#L18-L63) · [OK dispatch](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/ui-egui/src/dialogs/new_layer_ui.rs#L131-L133)

**Adapted Rust** — the shell and dialog hand off to the engine only after confirmation.

```rust
if id == "layer.new.layer" && params.as_object().is_none_or(|o| o.is_empty()) {
    return Ok(json!({"dialog": new_layer_ui::open(app)?})); // Result<u64, String>; e.g. dialog 7
}
// On OK, params(fields) maps 40% to 0.4 and omits a whitespace-only name.
app.run("layer.new.layer", new_layer_ui::params(fields)) // Result<Value, String>; e.g. {"layer":42}
```

### Footer click, dialog OK, or direct dispatch creates and selects a layer

#### Admission and options

The registered command requires a document; precheck validates New Layer options before a pending transform commits. Dispatch also rejects conflicting jobs. [Registration](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/cmd.rs#L498-L507) · [Precheck](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/session/execute.rs#L65-L91)

**Adapted Rust** — focused parser branches; all other fields still validate as described above. This group chooses safe defaults and rejects unsupported fill combinations.

```rust
let mode = match p.get("mode") {
    None | Some(Value::Null) => BlendMode::Normal,
    Some(v) => crate::commands::blend_from_str(
        v.as_str().ok_or_else(|| bad("`mode` must be a blend mode name"))?
    )
        .filter(|m| *m != BlendMode::PassThrough)
        .ok_or_else(|| bad("unknown layer blend mode"))?, // Option<BlendMode> becomes Result<BlendMode, EngineError>; e.g. Overlay
};
let opacity = match p.get("opacity") {
    None | Some(Value::Null) => 1.0,
    Some(v) => v.as_f64().filter(|o| o.is_finite())
        .ok_or_else(|| bad("`opacity` must be a number"))?
        .clamp(0.0, 1.0) as f32,
};
let neutral = if fill_neutral {
    Some(neutral_color(mode).ok_or_else(|| bad("mode has no neutral color"))?.0) // Option<([f32; 3], &str)>; Overlay => gray
} else { None };
```

#### Build and place the layer

The edit uses the document's pixel format, so RGB/Gray and 8/16/32-bit surfaces follow the document. A neutral fill covers document bounds through the raster conversion; an empty layer allocates no filled tiles. [Handler](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/new_layer.rs#L41-L61)

**Adapted Rust** — shortened error propagation and omitted incidental locals. This group constructs, inserts, clips and selects the layer.

```rust
let id = s.edit("New Layer", |doc, active| {
    let mut layer = Layer::raster(name.unwrap_or_else(|| doc.next_layer_name("Layer")), doc.pixel_format());
    layer.blend = mode;
    layer.opacity = opacity;
    layer.label = label;
    if let Some(rgb) = neutral {
        let surface = layer.surface_mut().ok_or_else(|| bad("a new pixel layer has no pixels"))?;
        let px = photocraft_raster::from_rgba(&surface.format(), [rgb[0], rgb[1], rgb[2], 1.0]);
        surface.fill_rect(doc.bounds(), &px);
    }
    let id = layer_place::insert_new(doc, *active, layer, if below { Place::Below } else { Place::Auto }); // LayerId, e.g. 42
    clip_group::settle(doc, &[id], &Default::default(), clip); // forced clip or join clip group at slot
    *active = Some(id);
    Ok(id)
})?; // Result<LayerId, EngineError>; error leaves document uncommitted
Ok(json!({"layer": id.0})) // final JSON: {"layer":42}
```

`Place::Auto` enters an open group or artboard when eligible, otherwise inserts above the anchor; `Below` inserts below except at Background. Insertion also joins an existing clipping run, while `settle` applies the explicit clipping option and releases orphans. [Placement](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/layer_place.rs#L31-L96) · [Clipping](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/clip_group.rs#L100-L122)

#### Commit and history

`Session::edit` works on a document copy; an error leaves the original in place. On success it installs the edited document and active ID, records the `New Layer` history state, and bumps the revision. [Edit transaction](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/session/execute.rs#L245-L284) · [History](https://github.com/storytold/photocraft/blob/3d60ad01f941519307c755eed8e0896d6274cde6/crates/engine/src/session/execute.rs#L319-L337)
