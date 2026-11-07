# tools.select.brush

Select the Brush tool for painting pixels, masks, or channels with subsequent canvas gestures.

Verified against: `3d60ad01f941519307c755eed8e0896d6274cde6` · 2026-11-07. Tracked source is unchanged; two unrelated source-documentation files are untracked.
Navigation: [tools commands](../tools.md#tools) · [Painting/retouching actions](../../tools/painting-retouching.md#painting-retouching) · [command categories](../index.md#categories) · [tool categories](../../tools/index.md#categories)

Source links below open the local checkout. GitHub blob and raw-content requests failed, so remote availability could not be verified. Each cited range was checked against the commit above and the unchanged local Rust file; local links are not immutable permalinks.

## Context to use

Selecting Brush requires the UI shell, but no open document. Painting requires an open document and a writable target. Selection alone does not paint or add document history.

- Invoke `tools.select.brush` through `ui.menu.invoke` or the desktop control server's `engine.execute`. Both reach the shell for this ID. [Control routing (89–122)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/control/dispatch.rs:89)
- Assign the ID a custom shortcut in Keyboard Shortcuts. Default `B` instead chooses the remembered tool in the Brush/Pencil/Mixer Brush/Color Replacement group; cycling depends on the Shift-switch preference. [Custom shortcuts (73–113)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/input/shortcut_dispatch.rs:73) · [Default group (896–925)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/input/shortcuts.rs:896)
- Click the Brush toolbar slot, Alt-click the slot to cycle, or choose Brush from its flyout. These directly select a tool. [Slot (195–212)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/panels/toolbar.rs:195) · [Flyout (320–328)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/panels/toolbar.rs:320)
- Search for Brush in the command palette and choose its tool result. This uses `tool:Brush` and assigns `ui.tool` directly; it does not dispatch `tools.select.brush`. [Tool results (178–188)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/dialogs/palette.rs:178) · [Selection (248–260)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/dialogs/palette.rs:248)
- Click for a dab, drag for a stroke, or Shift-click to connect from the previous endpoint. Release dispatches `paint.stroke`. [Press (494–522)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/tool_input.rs:494) · [Release (82–96)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/gesture.rs:82)
- Alt samples foreground color; Command (Ctrl outside macOS) temporarily uses Move; Control-Alt or Alt-right drag changes brush settings. Right-click opens the preset picker unless another right-button route applies. [Sampling (188–214)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/cursors.rs:188) · [Temporary tools (1–30)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/input/hold_keys.rs:1) · [Right-button routing (393–433)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/paint_mouse.rs:393)

A headless `Session::execute`, CLI run, or engine-backed automation call cannot select a UI tool: this ID is in the shell binding table, not the engine command registry, and engine lookup returns `UnknownCommand`. Use `paint.stroke` directly for headless painting. [Binding (102–126)](/Users/hung/work/aircraft/photocraft/crates/engine/src/shell_commands.rs:102) · [Lookup (1509–1519)](/Users/hung/work/aircraft/photocraft/crates/engine/src/jobs.rs:1509)

## Inputs and response

A complete control-channel selection request is:

```json
{"id":1,"method":"ui.menu.invoke","params":{"command":"tools.select.brush","params":{}}}
```

- No selection parameter is required or read; `{}` is sufficient. The exact case-sensitive spelling matters; `tools.select.Brush` does not resolve. [ID mapping (6–18)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/input/tool_shortcuts.rs:6)
- Brush size, hardness, opacity, flow, mode, dynamics, and smoothing belong to brush settings, not this invocation. A first-time tool inherits the current brush with Normal mode and 10% smoothing; returning tools recover their remembered settings. Foreground/background, erase, and seed remain session-owned. [Brush ownership (14–77)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/paint_mouse.rs:14)
- Later pointer coordinates are document pixels; mouse pressure is 1. Stylus points can add pressure, tilt, rotation, and relative milliseconds. [Point serialization (397–419)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/input/stylus.rs:397)

**Result:** the shell handler returns `Result<Value, String>` with `Ok({"tool":"Brush"})`; the control channel wraps that value in its response. A successful later stroke returns a value such as `{"damage":[110,70,30,30]}`, optionally with `"fromLive":true`, and normally one document history step. [Selection result (77–81)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/menus/invoke.rs:77) · [Damage (242–245)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/brush_cmds.rs:242) · [Stroke result (327–335)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/brush_cmds.rs:327)

**Errors:** control dispatch can reject a modal conflict or automation authorization before selecting. Later painting can fail for no document/target, a hidden layer or ancestor, applicable locks, invalid points, unavailable preset data, excessive brush size/scatter, job conflicts, or unreadable scratch pixels. These are separate from selecting Brush. [Control gates (98–114)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/control/dispatch.rs:98) · [Paint admission (95–113)](/Users/hung/work/aircraft/photocraft/crates/engine/src/session/execute.rs:95) · [Points (56–90)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/brush_cmds.rs:56) · [Brush validation (166–235)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/brush_cmds.rs:166)

For direct `paint.stroke` calls, unparsable point entries are skipped; an empty surviving list or coordinates outside finite ±1,000,000 fail. Scalar hardness/opacity/flow clamp to 0–1, spacing to 0.01–10, and size has a 0.5 minimum before maximum-size validation. The canvas does not set `fill:true`: a closed Brush drag paints its outline, not an enclosed fill. [Point parser (56–90)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/brush_cmds.rs:56) · [Overrides (197–235)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/brush_cmds.rs:197) · [Canvas parameters (295–312)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/strokes.rs:295)

## Structures used

The selected tool and the owner of its remembered settings are separate state.

Original source: [Selected tool (205–207)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/state.rs:205) · [Brush memory (321–326)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/state.rs:321) · [Brush settings (548–610)](/Users/hung/work/aircraft/photocraft/crates/paint/src/brush.rs:548).

**Adapted Rust** — relevant fields only.

```rust
struct UiState {
    tool: Tool,                              // Brush after selection.
    brush_tool: Tool,                        // Pencil until its settings are swapped out.
    tool_brushes: Vec<(Tool, BrushSettings)>, // Remembered Brush size, mode, dynamics, etc.
}
struct BrushSettings {
    size: f32, hardness: f32, spacing: f32,   // 25 px, 0.7 hardness, 0.25 diameters.
    opacity: f32, flow: f32,                 // 0.8 stroke ceiling; 0.2 per dab.
    color: [f32; 4],                         // [1.0, 0.0, 0.0, 1.0], display RGB + alpha.
    erase: bool, behind: bool,               // false; true erases / paints beneath pixels.
    mode: BlendMode,                         // BlendMode::Normal.
    pressure_size: bool, build_up: bool,     // Pen controls size; timed airbrush if enabled.
}
```

Pointer events create a drag; pressure and pose become engine points.

Original source: [Events (9–15)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/tool_input.rs:9) · [Drag (4–22)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/drag.rs:4) · [Stroke point (47–65)](/Users/hung/work/aircraft/photocraft/crates/paint/src/lib.rs:47).

**Adapted Rust** — fields unrelated to Brush omitted.

```rust
enum ToolEvent {
    Down { x: f64, y: f64, pressure: f32 }, // 120.0, 80.0, 0.7.
    Move { x: f64, y: f64, pressure: f32 }, // 125.0, 82.0, 0.8.
    Up { x: f64, y: f64 },                 // Final point inherits the last pressure.
}
struct Drag {
    tool: Tool, start: [f64; 2],           // Brush, [120.0, 80.0].
    points: Vec<[f64; 3]>,                // [[120.0, 80.0, 0.7], [125.0, 82.0, 0.8]].
    erase: bool, clear: bool,             // Latched when the press begins.
    constrain: Option<Axis>,              // Some(axis) while Shift constrains movement.
}
struct StrokePoint {
    x: f64, y: f64, pressure: f32,         // 125.0, 82.0, 0.8.
    tilt_x: f32, tilt_y: f32, rotation: f32,// 15°, 0°, 30°.
    wheel: f32, time: f64,                // 1.0, 16.0 ms after the initial point.
}
```

The UI preview tracks what has been displayed; the engine preview owns a document copy and rendering state. The two `LiveStroke` types are distinct.

Original source: [UI preview (229–242)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/strokes.rs:229) · [Engine preview and commit (521–588)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/brush_cmds.rs:521).

**Adapted Rust** — module wrappers distinguish the types; fields are reduced.

```rust
mod ui {
    struct LiveStroke {
        stroke: EngineStroke,       // EngineStroke::Brush(Box<engine::LiveStroke>).
        doc: DocId, revision: u64,   // Document 7, revision 12 at press.
        fed: usize,                 // 2 gesture points already rendered.
        damage: Vec<Rect>,          // Rectangles changed at each preview step.
    }
}
mod engine {
    struct LiveStroke {
        doc: Arc<Document>,         // Copy-on-write document with provisional pixels.
        pre: Surface,               // Original target pixels for repeated compositing.
        sel: Option<Surface>,       // Some(gray coverage mask), or None for all pixels.
        lock: bool,                 // true preserves pixel-layer transparency.
        renderer: StrokeRenderer,   // Sparse coverage tiles and dab generator.
        points: Vec<StrokePoint>,   // Every input point rendered so far.
        base: (DocId, u64),         // (DocId(7), 12), reuse guard.
        seed: u64, params: Value,   // Deterministic jitter; target/erase/zoom etc.
    }
    struct LiveCommit {
        cmd: String, base: (DocId, u64), // "paint.stroke", (7, 12).
        params: Value, seed: Option<u64>, points: Vec<StrokePoint>,
        surface: Surface, bounds: Rect, // Finished pixels; damaged rectangle.
    }
}
```

Targets and resize state carry the alternate editing destinations and HUD gesture.

Original source: [Target (9–32)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/channel_cmds/targeting.rs:9) · [Resize (27–39)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/tools/brush_resize.rs:27).

**Adapted Rust** — visibility and unused details omitted.

```rust
enum Target {
    Pixels, FilterMask, Mask, // "pixels", "filterMask", "mask".
    Alpha(usize), QuickMask, // {"channel":0}, "quickMask".
}
struct Resize {
    anchor: [f64; 2],        // [120.0, 80.0] document pixels.
    start: (f32, f32),       // (25.0, 0.7): initial size and hardness (or opacity).
    key: u64,               // 3 -> coalesce key "brush-resize:3".
    secondary: bool,        // true for Alt-right rather than Control-Alt drag.
}
```

## Core logic flow

### Invoke the ID or choose Brush to activate it

The exact ID resolves through the shell before engine dispatch. There is no alias or engine `run` handler for it. The shell enabled check returns true without consulting document state; control modal and authorization checks still apply.

Original source: [Enabled (5–11)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/menus/state.rs:5) · [Invoke (41–81)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/menus/invoke.rs:41) · [Exact mapping (6–18)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/input/tool_shortcuts.rs:6) · [Selection (24–32)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/tool_memory.rs:24).

**Adapted Rust** — the early selection route is isolated from other commands.

```rust
if app.automation_input {
    if let Some(authorize) = app.services.automation_command.as_ref() {
        authorize(id, &params)?; // Result<(), String>; Err denies the request.
    }
}
if let Some(tool) = tool_shortcuts::tool(id) { // Option<Tool>; Some(Tool::Brush).
    tool_memory::select(app, tool); // Clears spring/stylus-erasing ownership; sets ui.tool.
    return Some(Ok(json!({"tool": tool}))); // Option<Result<Value, String>> in the early router.
}
```

The toolbar uses `tool_memory::select`; default letter keys and the palette tool result assign `ui.tool` directly. A default `B` press can select another remembered member of the group. A plain letter held for at least 0.3 seconds, or used on the canvas, springs back to the previous tool after release and any ongoing drag; a quick unused tap stays selected. Repeats do not cycle. [Group routing (880–925)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/input/shortcuts.rs:880) · [Spring rules (1–30)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/input/spring_tools.rs:1) · [Release (94–126)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/input/spring_tools.rs:94)

`sync_tool_brush` saves the outgoing brush, loads the incoming tool's remembered settings, and clears the picked preset identity. Tool selection/settings persist after the pointer is up. This changes tool state, not document history. [Swap (23–67)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/paint_mouse.rs:23) · [Persistence (61–86)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/tool_memory.rs:61)

### Press, drag, and release to paint layer pixels

This starts after selection. `ToolEvent` comes from the native canvas or `ui.pointer`; `paint.stroke` is registered separately with `has_paintable` and `paint_stroke`. [Events (9–15)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/tool_input.rs:9) · [Registration (999–1008)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd.rs:999)

#### Press and movement collect samples

A hidden layer/ancestor is refused before preview. An eligible nonraster layer opens the rasterization flow below. Brush selection does not bypass these checks.

Original source: [Admission (366–379)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/tool_input.rs:366) · [Press (494–522)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/tool_input.rs:494) · [Movement (20–50)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/tool_input.rs:20).

**Adapted Rust** — Brush-only branches; modifier-specific flows are expanded below.

```rust
if let Some(why) = hidden_target(app, Tool::Brush) { // Option<&str>; Some("…hidden…").
    app.ui.status = why.into();
    app.ui.status_error = true;
    return;
}
if rasterize_prompt::intercept(app, Tool::Brush, x, y, pressure) { return; }
paint_mouse::sync_tool_brush(app);
let clear = paint_mouse::clears(app, Tool::Brush);
let erase = std::mem::take(&mut app.secondary_erase) || clear;
let mut d = Drag::new(Tool::Brush, [x, y], vec![[x, y, pressure as f64]], mods, erase);
d.clear = clear;
app.drag = Some(d);
app.stylus.begin_stroke();
app.live_stroke = begin_live_stroke(app); // Option<ui::LiveStroke>; None falls back at release.
// On Move, after Shift constraint:
let moved = d.points.last().is_none_or(|p| (p[0] - x).abs() + (p[1] - y).abs() > 0.25);
if moved || held { // `held`: timed repeat due and live renderer wants airbrush/catch-up time.
    d.points.push([x, y, pressure as f64]);
    app.stylus.record_point();
}
feed_live_stroke(app); // Sends only samples after preview.fed; may be batched per frame.
```

A stationary click retains the first point. Timed repeats allow build-up and smoothing catch-up while held; a pressure-only change without movement/time eligibility does not itself append a point. Native input recovers freehand samples between press/release; serialization supplies pose and relative time. [Sample interval (577–597)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/strokes.rs:577) · [Serialization (397–419)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/input/stylus.rs:397)

#### Preview renders dabs on a document copy

The engine resolves target, brush, foreground, deterministic seed, selection and transparency lock, then creates a `StrokeRenderer` using the target's actual pixel format.

Original source: [Preview creation (595–648)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/brush_cmds.rs:595) · [Incremental rendering (694–767)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/brush_cmds.rs:694) · [Displayed document (170–174)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/display.rs:170).

**Adapted Rust** — single-pass preview shown; symmetry uses the union branch described below.

```rust
has_paintable(s).map_err(EngineError::Other)?; // Result<(), EngineError>; missing target fails.
let pts = parse_points(p, cmd)?;              // Result<Vec<StrokePoint>, EngineError>.
let brush = with_blend_mode(resolve_brush(s, p, cmd)?, p);
let (layer, mut brush, zoom) = stroke_target(s, p, brush)?;
let mut doc = (*st.doc).clone();             // COW tiles; session document stays unchanged.
let sel = doc.selection.clone();
let (surf, lock) = channel_cmds::target_surface(&mut doc, layer, p)?;
// Result<(&mut Surface, bool), EngineError>; a locked/missing target returns Err.
erase_locked(&mut brush, lock, s.tools.background);
let renderer = StrokeRenderer::new(&brush, Some(surf.format()), zoom);
// These values populate LiveStroke (construction fields are shown in Structures used).
live.push(&pts)?; // Result<Rect,EngineError>; renders initial points and a provisional finish tail.

// Inside LiveStroke::push, after coordinate validation and renderer.push(pts):
if !old_tail.is_empty() {
    surf.write_region(old_tail, &self.pre.read_region(old_tail));
    self.renderer.mark_dirty(old_tail); // Remove the previous provisional catch-up tail.
}
let mut damage = self.renderer.composite(&self.pre, surf, self.sel.as_ref(), self.lock, false);
if let Some(mut tail) = self.renderer.tail_preview() { // Option<StrokeRenderer>; may be None.
    self.tail = tail.composite(&self.pre, surf, self.sel.as_ref(), self.lock, false);
    damage = damage.union(&self.tail); // Shows what releasing now would paint, including lone dab.
}
// UI display_doc returns the preview Arc<Document> while its doc/revision and drag match.
```

The lower-layer algorithm smooths points, walks the path at brush spacing (or time/speed spacing), interpolates pressure/pose, and produces primary and optional dual dabs. Each dab rasterizes its tip into sparse coverage tiles. Dynamics, scatter and color use seeded generation; opacity is a stroke ceiling and flow accumulates within that stroke.

Original source: [Path walker (346–432)](/Users/hung/work/aircraft/photocraft/crates/paint/src/dynamics.rs:346) · [Dab builder (564–653)](/Users/hung/work/aircraft/photocraft/crates/paint/src/dynamics.rs:564) · [Smoothing and finish (826–852)](/Users/hung/work/aircraft/photocraft/crates/paint/src/dynamics.rs:826) · [Dab rasterization (900–937)](/Users/hung/work/aircraft/photocraft/crates/paint/src/render.rs:900) · [Compositing (1105–1256)](/Users/hung/work/aircraft/photocraft/crates/paint/src/render.rs:1105).

**Adapted Rust** — the decisive per-pixel compositor branches; indexing/setup omitted.

```rust
let m = self.ctx.stroke_mask_wet(coverage, depth, dual_coverage, x, y);
let s = selection_value; // 0..1, or 1 without a selection.
let mut k = (m * brush.opacity.clamp(0.0, 1.0) * s).min(1.0);
if k <= 0.0 { continue; }
if brush.mode == BlendMode::Dissolve {
    k = if hash2(x, y, brush.seed) < k { 1.0 } else { 0.0 };
    if k == 0.0 { continue; }
}
if brush.erase {
    if let (Some(a), false) = (alpha_index, lock_transparency) { px[a] *= 1.0 - k; }
} else if brush.behind {
    if lock_transparency || destination_alpha >= 1.0 { continue; }
    let underneath = source_alpha * k * (1.0 - destination_alpha);
    let alpha = destination_alpha + underneath;
    if alpha > 0.0 {
        for ch in 0..alpha_index_value {
            px[ch] = (px[ch] * destination_alpha + src[ch] * underneath) / alpha;
        }
        px[alpha_index_value] = alpha;
    }
} else if matches!(brush.mode, BlendMode::Normal | BlendMode::Dissolve) {
    over_native(&fmt, px, &src[..channels], k, lock_transparency);
} else {
    // Convert native samples to RGBA, blend (atop when alpha locked), encode back to fmt.
}
// U8 non-aliased results get deterministic sub-level dither; U16/F32 use their own format.
// Dirty tiles read pre-stroke pixels, then write the result: re-previewing cannot double-paint.
```

Normal painting uses native channel values; non-Normal blending converts through the format's RGBA path. The document's CMYK profile scope also reaches parallel tile workers, and a color picked as CMYK retains its ink values where supported. Indexed/Bitmap painting is constrained back to its color table. Symmetry renders distinct reflected passes and merges their coverage as one stroke, avoiding doubled opacity at intersections. [Profile, table and symmetry (265–335)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/brush_cmds.rs:265) · [Union compositor (747–785)](/Users/hung/work/aircraft/photocraft/crates/paint/src/render.rs:747)

#### Release commits one edit or reports failure

Release appends a changed final position with the last pressure, remembers the endpoint for Shift-click, and calls `commit_brush_stroke`. Missing/failed preview falls back to rendering all collected points. No document mutation is needed to select the tool or to build its provisional preview.

Original source: [Up (566–590)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/tool_input.rs:566) · [Endpoint (19–31)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/gesture.rs:19) · [Commit (622–680)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/strokes.rs:622) · [Reuse guard (568–588)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/brush_cmds.rs:568) · [Handler edit (273–335)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/brush_cmds.rs:273).

**Adapted Rust** — release and the engine edit are shown as consecutive fragments.

```rust
let mut p = stroke_params(app, d, &app.stylus.stroke_points(&d.points));
// Value: {points:[[120,80,0.7],…], erase:false, zoom:1.0, target:"pixels"}.
// If live exists: copy its seed into p, push remaining points, offer_live_stroke(live).
// offer_live_stroke: Result<(), EngineError>; a failed offer is cleared before fallback.
let result = app.run("paint.stroke", p); // Result<Value, String>; Err is reported by the app.
app.session.clear_live_stroke();

// In stroke_with: only a matching command/base revision/params/seed/point list reuses pixels.
let live = s.live_commit.take().filter(|l| l.matches(s, cmd, p, &pts));
let from_live = live.is_some();
let damage = s.edit(label, |doc, _| {
    if let Some(l) = live {
        let (surf, _) = channel_cmds::target_surface(doc, id, p)?;
        *surf = l.surface;
        return Ok(l.bounds); // Result<Rect, EngineError>.
    }
    let selection = doc.selection.clone();
    let (surf, lock) = channel_cmds::target_surface(doc, id, p)?;
    erase_locked(&mut brush, lock, background);
    let damage = render_stroke(surf, &brush, &pts, selection.as_ref(), lock, zoom);
    // Rect, e.g. Rect covering [110,70] to [140,100]; this fragment shows the no-symmetry case.
    // The full branch also merges symmetry passes and enforces an Indexed/Bitmap color table.
    Ok(damage)
})?;
// Result<Value, EngineError>: {"damage":[x,y,width,height], "fromLive":true} when reused.
```

`Session::edit` clones the document, applies the closure, restores untouched color planes if required, refreshes smart filters after a filter-mask edit, checks scratch health, and only then installs the result and records history/revision. Errors before installation leave the original document intact. Normal Brush history is “Brush Tool”; erasing-right strokes use “Eraser.” [Transactional edit (246–286)](/Users/hung/work/aircraft/photocraft/crates/engine/src/session/execute.rs:246) · [History (320–347)](/Users/hung/work/aircraft/photocraft/crates/engine/src/session/execute.rs:320) · [Label (427–435)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/brush_cmds.rs:427)

**Ownership/cancellation limit:** Brush has no dedicated Escape/context-change cancellation guard like Mixer Brush. Preview display and reuse check document/revision, but fallback builds parameters against the session at release; the Brush drag itself stores no document/layer owner. Switching targets during a synthetic gesture must not be treated as guaranteed cancellation. A surviving drag commits on Up, even for a click. Escape can instead reach normal deselection or screen-mode handling. [Preview/parameters (290–312)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/strokes.rs:290) · [Mixer-only cancellation (409–423)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/strokes.rs:409) · [Escape (860–870)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/input/shortcuts.rs:860)

### Paint a mask, Quick Mask, alpha channel, or selected color planes

The same press/move/preview/release pipeline writes a different surface. Destination is derived from Channels, mask-view state, and Layers targeting, rather than from whether the pointer is inside an existing selection. A selection multiplies coverage; it does not change Brush into a movement tool.

Original source: [UI target order (694–726)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/strokes.rs:694) · [Surface lookup (54–65)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/channel_cmds/targeting.rs:54) · [Paintability and locks (33–91)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/enabled.rs:33) · [Target/eraser conversion (248–263)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/brush_cmds.rs:248).

**Adapted Rust** — target choices and their consequences, with unrelated branches omitted.

```rust
match st.channel_view.target {
    ChannelTarget::Alpha(i) if i < st.doc.channels.len() => return json!({"channel":i}),
    ChannelTarget::Composite if st.doc.quick_mask.is_some() => return json!("quickMask"),
    ChannelTarget::LayerMask => return json!("mask"),
    ChannelTarget::Color(_) => {
        // A viewed mask wins. Otherwise a raster layer can return selected color-plane params.
    }
    _ => {}
}
// Next: targeted/shown smart-filter mask; then existing layer mask if targeted/viewed
// or on an adjustment layer; otherwise "pixels".

match target_of(p) {
    Target::Alpha(i) => doc.channels.get_mut(i)
        .map(|c| (&mut c.surface, false))
        .ok_or_else(|| EngineError::Other(format!("no alpha channel {i}"))),
    Target::QuickMask => doc.quick_mask.as_mut()
        .map(|c| (&mut c.surface, false))
        .ok_or_else(|| EngineError::Other("not in Quick Mask mode".into())),
    _ => {
        // Result<(&mut Surface, bool), EngineError> via paint_surface:
        // pixels: reject effective all/pixels locks; return effective transparency lock.
        // layer/filter mask: reject effective all-lock; require the requested mask.
    }
}
```

- Alpha/Quick Mask targets need no active pixel layer and bypass hidden-layer refusal. Layer and filter masks still belong to their layer and obey its visibility/all-lock rules. A missing layer mask is an error, not an implicit creation. [Visibility exception (41–55)](/Users/hung/work/aircraft/photocraft/crates/engine/src/hidden_target.rs:41)
- Grayscale destinations store tone. Erasing a mask/channel paints the background color instead of removing alpha; erasing transparency-locked pixels likewise paints background while retaining alpha. [Eraser conversion (248–263)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/brush_cmds.rs:248)
- Selected color planes edit the raster layer but restore unselected planes and alpha, both in preview and commit. Other targets use their actual surface format/depth. [Preview restriction (724–766)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/brush_cmds.rs:724) · [Commit restriction (292–300)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/channel_cmds/targeting.rs:292)
- Filter masks take the smart object's mask surface and trigger filter recomputation within the transactional edit. Commit admission accepts a paintable smart-filter mask through `mask_target_enabled`. Preview initialization separately calls `has_paintable`; a smart object without a layer mask can therefore have no live preview and render its filter-mask stroke at release. [Mask admission (121–135)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/channel_cmds/targeting.rs:121) · [Refresh (271–284)](/Users/hung/work/aircraft/photocraft/crates/engine/src/session/execute.rs:271)

### Shift-click or Shift-drag to paint a straight segment

Shift-click starts a new stroke with the previous endpoint and pressure only when that endpoint belongs to the active document. With no such endpoint, it starts at the click. The rubber-band line drawn while hovering is guidance; the subsequent stroke preview renders real dabs.

Original source: [Connection (500–522)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/tool_input.rs:500) · [Constraint and line preview (61–131)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/tools/stroke_constraint.rs:61).

**Adapted Rust** — preserve endpoint pressure and lock the direction after 3 screen points.

```rust
let from = app.last_stroke_end
    .filter(|(doc, _)| mods.shift && Some(*doc) == active_document)
    .map(|(_, point)| point); // Option<[f64;3]>; Some([100.0,80.0,0.4]).
if let Some(previous) = from { points.insert(0, previous); }
// Movement (also applied at release):
if !shift { *axis = None; return p; }
let a = axis.get_or_insert(Axis { origin: last, dir: None });
if a.dir.is_none() {
    let reach = 3.0 / f64::from(zoom.max(1e-3));
    if (p[0]-a.origin[0]).hypot(p[1]-a.origin[1]) < reach { return a.origin; }
    a.dir = snapped_dir(a.origin, p, std::f64::consts::FRAC_PI_4);
}
match a.dir { Some(dir) => project(a.origin, dir, p), None => a.origin }
// [f64;2], e.g. [125.0,80.0]; Shift pressed mid-drag uses the last point as origin.
```

Constrained points enter the same smoothing/dab preview and release command, with one stroke history step. Releasing Shift removes the axis lock; it does not cancel or undo prior points. The stored endpoint updates before commit, so it is not proof that the previous stroke succeeded. [Endpoint storage (19–31)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/gesture.rs:19)

### Confirm rasterization to paint a nonraster layer

A press targeting unlocked type, shape, Smart Object, or fill pixels opens a prompt and parks the click. Painting that layer's existing mask bypasses this prompt. Cancel leaves the document unchanged; it does not replay the click.

Original source: [Prompt contract (1–10)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/dialogs/rasterize_prompt.rs:1) · [Commands (45–53)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/dialogs/rasterize_prompt.rs:45) · [Admission/captured click (80–127)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/dialogs/rasterize_prompt.rs:80) · [Confirmation (202–224)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/dialogs/rasterize_prompt.rs:202).

**Adapted Rust** — confirmation rasterizes, then replays a stationary click if its layer is still active.

```rust
let r = app.run(kind.command(), json!({"layer": layer}))?;
// Result<Value,String>; layer.rasterize.type/shape/smartObject/fillContent, failure stops here.
app.ui.tool = tool;
let before = app.session.active().map(|st| st.history.past_len());
if app.session.active().and_then(|st| st.active_layer) == Some(LayerId(layer))
    && x.is_finite() && y.is_finite()
{
    tool_event(app, ToolEvent::Down {x,y,pressure:pressure.clamp(0.0,1.0)}, Modifiers::NONE);
    tool_event(app, ToolEvent::Up {x,y}, Modifiers::NONE);
}
let painted = app.session.active().map(|st| st.history.past_len()) != before;
Ok(json!({"rasterized":r,"painted":painted})) // Result<Value,String>; painted can be false.
```

Successful rasterization and painting create two history states. Only the original position/pressure is replayed, without the original modifiers or drag path. Missing captured data or a changed active layer can leave a successful rasterization with `painted:false`.

### Right-drag, Clear mode, or held tilde to erase with Brush

With the right-click Erase preference, secondary pointer events reach the Brush pipeline with `secondary_erase` armed. Clear mode or held `~` instead sets both `erase` and `mode:"clear"` at press. Alt-right resize and Shift-right mode menus intercept before ordinary erasing.

Original source: [Clear (81–90)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/paint_mouse.rs:81) · [Secondary routing (393–433)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/paint_mouse.rs:393) · [Params (295–312)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/strokes.rs:295) · [Mode/label (407–435)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/brush_cmds.rs:407).

**Adapted Rust** — erasing shares the Brush's tip, pressure, preview and release.

```rust
let clear = paint_mouse::clears(app, Tool::Brush); // bool; chosen Clear or held ~.
let erase = std::mem::take(&mut app.secondary_erase) || clear;
let mut p = json!({"points":points,"erase":erase,"zoom":zoom,"target":paint_target(app)});
if clear { p["mode"] = json!("clear"); }
// Engine resolution:
brush.erase |= clear_mode(p);
if brush.erase { brush.mode = BlendMode::Normal; brush.behind = false; }
let label = if brush.erase && !clear_mode(p) { "Eraser" } else { "Brush Tool" };
```

The flag is latched at press, so releasing `~` midway does not turn the existing drag back into paint. Release creates one history edit; alpha removal, locked-transparency fallback, and mask/channel background painting follow the branches above. Behind/Clear menu choices are disabled on the Background layer. [Menu restriction (115–127)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/brush_mode_menu.rs:115)

### Alt-click or Alt-drag to sample foreground color

Sampling is chosen at Down, after resize interception, and stays latched until Up even if Alt is released. It creates no Brush drag or pixel preview. A real secondary press and armed right erasing are excluded.

Original source: [Interception order (139–153)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/tool_input.rs:139) · [Sampling (188–241)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/cursors.rs:188).

**Adapted Rust** — sampled color is converted to gray for a non-pixel paint target.

```rust
if down { app.alt_sampling = !physical_right && !app.secondary_erase && alt_samples(app.active_tool(), mods); }
if !app.alt_sampling { return false; }
match event {
    ToolEvent::Down {x,y,..} | ToolEvent::Move {x,y,..} => {
        if let Some(mut rgb) = eyedropper_color(app,x,y) { // Option<[f32;3]>; None off-image/transparent/error.
            if paint_target(app) != json!("pixels") { rgb = [rgb_to_gray(rgb);3]; }
            let _ = app.run("tools.setColors", json!({"foreground":[rgb[0],rgb[1],rgb[2],1.0]}));
            // Result<Value,String>; Ok(Value::Null) on success; pixels are unchanged.
        }
    }
    ToolEvent::Up {..} => app.alt_sampling = false,
}
```

`eyedropper_color` calls `document.sampleColor` with the Eyedropper's Sample Size and Sample Layers settings. Fully transparent/off-image samples leave the foreground unchanged. There is no later `paint.stroke` commit for this press.

### Hold Command to move, or Space to navigate

Command/Ctrl makes the effective pointer tool Move without changing `ui.tool`; Command-Alt can duplicate before movement, and Command-arrows nudge. The resulting layer/selection movement, locks, preview, commit and undo are the [Move flow](cmd-tools.select.move.md#core-logic-flow). Space temporarily uses Hand; Command-Space and Alt-Space use Zoom. These routes do not generate Brush dabs.

Original source: [Effective tool (652–662)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/tool_input.rs:652) · [Held routes (1–30)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/input/hold_keys.rs:1) · [Nudge admission (107–121)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/input/hold_keys.rs:107).

**Adapted Rust** — resolve the dispatch boundary before creating a Brush drag.

```rust
match app.active_tool() {
    t if app.tool_override.is_none() && mods.command && hold_keys::cmd_moves(t)
        && app.ui.transform.is_none() && app.ui.text_edit.is_none() => Tool::Move,
    t => t,
} // Tool, e.g. Tool::Move while ui.tool is still Brush.
```

Native temporary tools latch through their drag until button release; releasing the key restores Brush afterward. View navigation changes view state without a painting history step. Command-Control-Alt quick-pick is checked before Control-Alt resize and selects the layer under the pointer. [Quick-pick precedence (139–145)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/tool_input.rs:139)

### Control-Alt or Alt-right drag to resize the brush

Down captures the anchor, initial size and hardness (or opacity if the HUD preference disables hardness). Move and Up update tool state; the anchor stays fixed while a tinted tip and numeric readout show the change. No stroke starts.

Original source: [Gesture, calculation and HUD (45–172)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/tools/brush_resize.rs:45).

**Adapted Rust** — deltas have already been transformed into screen-relative axes divided by zoom.

```rust
let size = (start.0 + 2.0 * dx).round().clamp(1.0, MAX_BRUSH_SIZE);
let v = (start.1 + dy * zoom / 200.0).clamp(0.0,1.0);
let v = (v * 100.0).round() / 100.0;
let brush = if vary_hardness { json!({"size":size,"hardness":v}) }
            else { json!({"size":size,"opacity":v}) };
if let Err(e) = app.run("tools.setBrush", json!({"brush":brush,"coalesce":format!("brush-resize:{}", r.key)})) {
    app.ui.status = e; // Result<Value,String>; failed update is visible.
}
// Up takes the Resize out of app state and makes the final update.
// A stale Alt-right gesture is also cleared once its button is no longer down.
```

Calls share one journal key, not a document history step. Settings updates are already applied during movement; finishing the gesture is not a delayed pixel commit.

### Right-click or Shift-right-click to choose brush settings

Plain right-click opens the preset picker by default; the options-bar brush chip opens the same picker. Shift-right opens a mode menu with Edit Brush, blend modes, Behind and Clear. Selection of a preset or setting changes the session brush for subsequent strokes.

Original source: [Routing/closing (414–444)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/paint_mouse.rs:414) · [Picker behavior (1–20)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/tools/brush_picker.rs:1) · [Modes (91–127)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/brush_mode_menu.rs:91) · [Settings handler (1078–1137)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/brush_cmds.rs:1078).

**Adapted Rust** — settings are prepared and validated before replacing the brush.

```rust
let mut b = s.tools.brush.clone();
if let Some(name) = p.get("preset").and_then(Value::as_str) {
    let preset = find_preset(s,name,"tools.setBrush")?; // Result<&BrushPreset,EngineError>.
    b = preset.picked_over(&s.tools.brush); // Honors preset captures/section locks.
}
// Merge the brush patch / scalar overrides (or reset to defaults when requested).
validate_brush(&b,"tools.setBrush")?;
s.load_brush_tips(&mut b).map_err(|e| bad("tools.setBrush",e))?;
// Only now update current_preset and any captured foreground color.
s.tools.brush = b;
s.tools.brush_changed();
Ok(brush_json(&s.tools.brush)) // Result<Value,EngineError>; serialized accepted brush.
```

A click picks a preset and keeps the picker open; double-click picks and closes. Outside click, Escape or Enter closes the picker without rolling back accepted settings. Presets may capture foreground color; missing tips/invalid patches fail before replacement. Normal/Behind modes change brush settings, while Clear also needs the tool's `brush_clear` flag so the next drag sends `mode:"clear"`. [Current mode (129–145)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/brush_mode_menu.rs:129) · [Mode application (187–215)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/input/blend_keys.rs:187)

### Shift-Alt-right to pick foreground color in the HUD

This opens a hue strip/wheel and saturation/brightness area, except when the right-click Erase preference takes priority. Picking changes foreground immediately; release or Escape closes the HUD and keeps the chosen color.

Original source: [Behavior (1–12)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/panels/color_hud.rs:1) · [Pick and apply (165–242)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/panels/color_hud.rs:165) · [Priority (398–412)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/paint_mouse.rs:398).

**Adapted Rust** — the picked HSV value becomes a coalesced tool-color command.

```rust
let [r,g,b] = hsb_to_rgb(new_hsv);
let rgba = [r,g,b,hud.start[3]]; // Preserve the foreground alpha captured at open.
if let Err(e) = app.run("tools.setColors", json!({"foreground":rgba,"coalesce":format!("color-hud:{}",hud.key)})) {
    app.ui.status = e;
}
// No Brush drag or paint.stroke; release closes the HUD, not a pixel transaction.
```

### Use keyboard and options controls to change the next stroke

The options bar and Brush Settings panel edit the same brush used by the preview and commit; size, opacity, flow, smoothing, pressure toggles and dynamics are not parameters of `tools.select.brush`. Painting symmetry is independent state consumed by both render paths. [Options (87–146)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/panels/options_bar.rs:87) · [Symmetry rendering (273–323)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/brush_cmds.rs:273)

- `[` / `]` dispatch `tools.decreaseBrushSize` / `tools.increaseBrushSize`; Shift adds the hardness variants. Layout mapping preserves these gestures on non-US keyboards. [Bindings (140–150)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/brush_key_cmds.rs:140) · [Layout mapping (1–25)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/input/layout_keys.rs:1)
- Number keys set opacity: `4` → 40%, rapid `4,5` → 45%, `0` → 100%, `0,0` → 0%. Shift sets flow (minimum 1%). Airbrush build-up swaps opacity/flow roles. [Number timing (1–15)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/input/opacity_keys.rs:1) · [Apply (89–118)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/input/opacity_keys.rs:89)

**Adapted Rust** — numeric input uses the common brush command; tool state has no pixel history edit.

```rust
let shifted = shift ^ brush.build_up; // With build-up, plain digits mean flow.
let value = percent / 100.0;
let patch = if shifted { json!({"flow":value.max(0.01)}) }
            else { json!({"opacity":value}) };
let _ = app.run("tools.setBrush", json!({"brush":patch})); // Result<Value,String>; validated before storage.
```


- `,` / `.` choose the previous/next preset in panel order, wrapping; Shift chooses first/last. An empty library returns an error. With no current preset, Next starts at the first and Previous at the last. [Preset stepping (95–113)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/brush_key_cmds.rs:95)
- Left/Right rotate the Brush tip by +1°/−1°; Shift uses 15°. Command-arrows take the Move nudge route instead. [Arrow routing (205–253)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/tools/move_mods.rs:205)
- Shift `+`/`−` cycle brush blend modes; Shift-Alt-letter selects a named mode. These change the Brush mode, not the active layer's mode. [Blend keys (83–122)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/input/blend_keys.rs:83)

**Adapted Rust** — size/hardness steps and angle wrapping feed `set_brush`, without changing document pixels.

```rust
let size = if up { (size * 1.25).round().max(size + 1.0) }
           else { (size / 1.25).round().min(size - 1.0) }.clamp(1.0, 5000.0);
let quarter = (hardness * 4.0).round();
let hardness = (quarter + if up { 1.0 } else { -1.0 }).clamp(0.0,4.0) / 4.0;
let a = (angle + degrees).rem_euclid(360.0);
let angle = if a > 180.0 { a - 360.0 } else { a };
// Each handler changes just its field via set_brush: Result<Value,EngineError>.
// A direct rotateBrush request rejects nonfinite degrees or values outside -360..360.
```

Original source: [Size/hardness steps (21–38)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/brush_key_cmds.rs:21) · [Rotation (121–135)](/Users/hung/work/aircraft/photocraft/crates/engine/src/cmd/brush_key_cmds.rs:121).

Shortcuts respect input focus and modal routing; typing in a field is not a Brush keyboard gesture. Accepted setting changes survive picker closure and are remembered per tool. [Dispatch (73–113)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/input/shortcut_dispatch.rs:73) · [Per-tool settings (14–54)](/Users/hung/work/aircraft/photocraft/crates/ui-egui/src/canvas/paint_mouse.rs:14)
