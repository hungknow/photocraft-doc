# tools.select.hand

Selects the Hand tool in PhotoCraft's UI; later pointer gestures navigate the active document view.

Verified against: `3d60ad01f941519307c755eed8e0896d6274cde6` · 2026-11-07

Source links use the local checkout at this revision because GitHub's exact blob views were unavailable to verify.

Navigation: [tools commands](../tools.md#tools) · [View/workspace actions](../../tools/view-workspace.md#view-workspace) · [command categories](../index.md#categories) · [tool categories](../../tools/index.md#categories)

## Context to use

The tool can be selected with or without a document. Panning and view navigation require an active document canvas. The shell considers a valid tool ID enabled even with no document open. [Availability](../../../../photocraft/crates/ui-egui/src/menus/state.rs#L5-L16)

- `ui.menu.invoke` dispatches this exact ID. A control-channel `engine.execute` request also reaches the shell route because this tool ID has no engine `CommandSpec`. [Control dispatch](../../../../photocraft/crates/ui-egui/src/control/dispatch.rs#L89-L122) · [Shell selection](../../../../photocraft/crates/ui-egui/src/menus/invoke.rs#L41-L56)
- The default `H` key selects Hand directly in the single-key tool handler. A user-assigned shortcut for `tools.select.hand` goes through the ID dispatcher. Holding `H` for 0.3 seconds or using the canvas while it is held makes that key selection spring-loaded; a quick tap stays selected. [Tool metadata](../../../../photocraft/crates/engine/src/shell_commands.rs#L102-L140) · [Shortcut dispatch order](../../../../photocraft/crates/ui-egui/src/input/shortcuts.rs#L772-L787) · [Default key](../../../../photocraft/crates/ui-egui/src/input/shortcuts.rs#L906-L936) · [Spring-loaded keys](../../../../photocraft/crates/ui-egui/src/input/spring_tools.rs#L1-L16)
- Clicking the Hand toolbar slot or choosing Hand from its Hand/Rotate View flyout selects the same tool directly through UI state, without dispatching this ID. Double-clicking the Hand slot selects it and runs Fit on Screen when a document is open. [Toolbar](../../../../photocraft/crates/ui-egui/src/panels/toolbar.rs#L25-L45) · [Toolbar selection](../../../../photocraft/crates/ui-egui/src/panels/toolbar.rs#L250-L305)
- `ui.set {"tool":"Hand"}` and a `ui.pointer` request with `tool: "Hand"` can also set the UI tool directly; neither request dispatches this ID. [UI state setter](../../../../photocraft/crates/ui-egui/src/control/dispatch.rs#L231-L285) · [Tool application](../../../../photocraft/crates/ui-egui/src/control/dispatch.rs#L357-L379) · [Pointer input](../../../../photocraft/crates/ui-egui/src/control/dispatch.rs#L1244-L1285)
- Holding Space temporarily activates the Hand over another tool. It leaves the selected tool unchanged. During a marquee, lasso, shape, or crop-frame drag, Space repositions that in-progress geometry instead of panning. [Temporary tool binding](../../../../photocraft/crates/engine/src/prefs/shortcuts.rs#L109-L120) · [Held tool and repositioning](../../../../photocraft/crates/ui-egui/src/input/hold_keys.rs#L173-L205) · [Reposition targets](../../../../photocraft/crates/ui-egui/src/input/hold_keys.rs#L222-L250)

The engine carries this ID as tool shortcut metadata, but it has no engine command implementation. Headless CLI sessions cannot execute the UI-state selection; the control protocol can because it dispatches through the egui shell. [Tool metadata](../../../../photocraft/crates/engine/src/shell_commands.rs#L102-L140) · [Control dispatch](../../../../photocraft/crates/ui-egui/src/control/dispatch.rs#L115-L122)

## Inputs and response

Send the exact ID through the control channel's menu-invocation method:

```json
{"method":"ui.menu.invoke","params":{"command":"tools.select.hand","params":{}}}
```

- Required: exact case-sensitive command ID `tools.select.hand`.
- Optional: the command `params` object defaults to `{}` and is ignored by the tool-selection handler; no fields are required.

The immediate shell value is `{"tool":"Hand"}`. This updates UI state only; panning and zooming are later view interactions and do not create document history. [Selection result](../../../../photocraft/crates/ui-egui/src/menus/invoke.rs#L76-L82)

The command can be selected without a document. A control request is refused while a blocking modal is open, and an installed automation authorizer may reject it. A different spelling or casing does not match the tool ID; the shell then treats it as an unknown command. [Control admission](../../../../photocraft/crates/ui-egui/src/control/dispatch.rs#L89-L122) · [Exact ID parser](../../../../photocraft/crates/ui-egui/src/input/tool_shortcuts.rs#L7-L18)

## Structures used

The shell maps the exact ID to `Tool::Hand`; the persistent selected tool, temporary key state, and per-document camera live separately. Original source: [tool enum](../../../../photocraft/crates/ui-egui/src/state/tool.rs#L3-L46) · [UI and view state](../../../../photocraft/crates/ui-egui/src/state.rs#L137-L169) · [selected tool fields](../../../../photocraft/crates/ui-egui/src/state.rs#L227-L238) · [per-document views](../../../../photocraft/crates/ui-egui/src/state.rs#L439-L449).

**Adapted Rust** — unrelated variants and state fields are omitted; samples show the state after selecting Hand and before panning.

```rust
enum Tool { /* other tools omitted */ Hand }

struct UiState {
    tool: Tool,                   // Tool::Hand
    spring: Option<Spring>,       // None after an explicit ID selection
    views: Vec<View>,             // views[0] belongs to the active document
}

struct View {
    zoom: f32,                    // 1.0 means 100%
    center: [f32; 2],             // [128.0, 96.0]: document point at canvas centre
    fit_pending: bool,            // false until Fit on Screen is requested
    fill_pending: bool,           // false until Fill Screen is requested
}
```

The default `H` key can create a spring-loaded selection, while Space is a different, temporary binding. Neither is the command ID itself. Original source: [spring state](../../../../photocraft/crates/ui-egui/src/input/spring_tools.rs#L28-L48) · [temporary tools](../../../../photocraft/crates/ui-egui/src/input/hold_keys.rs#L37-L66).

**Adapted Rust** — only fields that determine the Hand key's return behavior are shown.

```rust
struct Spring {
    previous: Tool,       // Tool::Brush: restored after a spring-loaded Hand drag
    tool: Tool,           // Tool::Hand
    keys: Vec<char>,      // ['H'] while the key remains down
    since: f64,           // 12.0: egui time when the key was pressed
    used: bool,           // true after canvas use or a second held tool letter
    released: bool,        // true while a canvas drag finishes after H was released
}

enum Temporary { Hand, ZoomIn, ZoomOut, Move }
// Space resolves to Temporary::Hand; it does not write UiState::tool.
```

Canvas right-click stores the selected view tool and the document point under the pointer. Original source: [canvas menu state](../../../../photocraft/crates/ui-egui/src/canvas/canvas_tool_menu.rs#L13-L44).

**Adapted Rust** — unrelated menu flags are omitted; the example is a right-click over the image.

```rust
struct CanvasToolMenu {
    pos: [f32; 2],          // [200.0, 100.0] in screen points
    tool: Tool,             // Tool::Hand
    anchor: Option<[f64; 2]>, // Some([64.0, 32.0]) in document pixels
}
```

The temporary Zoom gesture keeps its press point, document anchor, starting zoom, and optional rectangle preview. Original source: [Zoom drag state](../../../../photocraft/crates/ui-egui/src/tools/zoom_tool.rs#L18-L28).

**Adapted Rust** — unrelated Zoom implementation details are omitted; this is a rectangle drag before release.

```rust
struct ZoomDrag {
    anchor: Pos2,            // [300.0, 200.0] screen points where the button went down
    doc: [f64; 2],           // [64.0, 32.0] document point kept under the anchor
    zoom0: f32,              // 1.0: starting zoom
    rect_to: Option<Pos2>,   // Some([420.0, 300.0]) when Scrubby Zoom is off
}
```

## Core logic flow

### Invoke the ID to select Hand

The control method checks the modal gate and automation policy, then routes non-engine IDs through `menus::invoke`. The shell's early tool route recognizes only the canonical ID, calls the common explicit-selection helper, and returns before any document command or history path. [Control route](../../../../photocraft/crates/ui-egui/src/control/dispatch.rs#L89-L122) · [ID route](../../../../photocraft/crates/ui-egui/src/menus/invoke.rs#L41-L56) · [Tool lookup](../../../../photocraft/crates/ui-egui/src/input/tool_shortcuts.rs#L7-L18)

**Adapted Rust** — focused control gate and exact-ID route; the control branch returns `Outcome`, and the shell helper returns `Option<Result<Value, String>>`.

Purpose: pass shell tool IDs through the UI route and return the immediate selection value.

```rust
if !continuation && !crate::menus::modal_allows(app, id) {
    return err("command is unavailable while a modal dialog is open");
}
if let Some(authorize) = app.services.automation_command.as_ref() {
    if let Err(error) = authorize(id, &params) {
        return err(error);
    }
}
if req.method == "engine.execute" && photocraft_engine::commands::find(id).is_some() {
    return run_waiting(app, wait, |app| app.run(id, params));
}
run_waiting(app, wait, |app| crate::menus::invoke(app, ctx, id, params))
// Outcome; this shell ID returns Done({"tool":"Hand"}), not a document command result.
```

The menu invoker authorizes automation input, honors the discard prompt, then takes the tool route before ordinary command handlers.

The recognized tool is enabled without a document. Explicit selection clears temporary spring/eraser state, exits Artboard add mode when needed, and cancels an unfinished Frame gesture. The active document, layer, selection, mask target, pixel format, and layer locks remain untouched. [Enablement](../../../../photocraft/crates/ui-egui/src/menus/state.rs#L5-L16) · [Selection helper](../../../../photocraft/crates/ui-egui/src/tool_memory.rs#L24-L41)

**Adapted Rust** — the menu wrapper and early tool branch are shown as separate return layers.

Purpose: map the exact ID to the Hand state and its immediate value.

```rust
if app.automation_input
    && let Some(authorize) = app.services.automation_command.as_ref()
{
    authorize(id, &params)?;
}
if crate::discard_ui::intercept(app, id, &params) {
    return Ok(Value::Null);
}
if let Some(tool) = crate::tool_shortcuts::tool(id) {
    crate::tool_memory::select(app, tool);
    return Some(Ok(json!({"tool": tool})));
    // Option<Result<Value, String>>; exact ID gives Some(Ok({"tool":"Hand"}))
}
```

**Adapted Rust** — focused body of the selection helper.

Purpose: make an explicit tool choice persistent and clear stale tool overrides.

```rust
app.stylus.tool_before_eraser = None;
app.ui.spring = None;
if app.ui.tool == Tool::Frame && tool != Tool::Frame {
    frame_tool::cancel(app); // discard the unfinished frame gesture
}
app.ui.tool = tool; // Tool::Hand
ended_by_tool(app); // leave Artboard add mode unless Artboard remains selected
```

### Press the default H key or hold it as a temporary switch

With the default bindings, the plain `H` press reaches `select_tool_by_key` after registry shortcuts get first refusal. That branch directly assigns the selected tool; it does not dispatch `tools.select.hand`. A customized shortcut is instead owned by `shortcut_dispatch`, which calls `menus::invoke` with the assigned ID. [Key handler](../../../../photocraft/crates/ui-egui/src/input/shortcuts.rs#L772-L787) · [Tool selection](../../../../photocraft/crates/ui-egui/src/input/shortcuts.rs#L906-L936) · [Shortcut dispatcher](../../../../photocraft/crates/ui-egui/src/input/shortcut_dispatch.rs#L232-L284) · [Dispatch result](../../../../photocraft/crates/ui-egui/src/input/shortcut_dispatch.rs#L369-L419)

**Adapted Rust** — registry shortcuts get first refusal; default single-key selection then walks tools that still use their default keys.

Purpose: show when a key selection dispatches this ID and when it changes UI state directly.

```rust
if shortcut_dispatch::dispatch_pressed(app, ctx, focus, editing) {
    return true; // bool; an assigned tools.select.hand binding reaches menus::invoke here
}
let shift_switch = app.session.prefs().tools.use_shift_key_for_tool_switch;
let cycle_shift = shift_switch && ctx.input_mut(|i| i.consume_key(Modifiers::SHIFT, Key::H));
if cycle_shift || pressed(Key::H) { // default H path; Hand is its only default tool
    let before = app.ui.tool;
    app.ui.tool = Tool::Hand; // direct state selection; no ID dispatch
    tool_memory::ended_by_tool(app); // clear tool-specific modes that Hand cannot continue
    let (plain, now) = ctx.input(|i| (!cycle_shift && !i.modifiers.shift && !i.modifiers.alt, i.time));
    spring_tools::pressed(app, 'H', before, plain, now);
    if app.ui.spring.is_none() {
        app.stylus.tool_before_eraser = None; // permanent key selection clears the pen eraser marker
    }
}
```

After H is released, a quick tap with no canvas use stays selected. Holding H for 0.3 seconds or using the canvas while H is down makes the switch temporary; the previous tool returns after the pointer drag ends. Shift/Alt switches and typing/modal focus do not spring-load. [Spring transition](../../../../photocraft/crates/ui-egui/src/input/spring_tools.rs#L28-L124)

**Adapted Rust** — spring timing and release branches are retained; `dragging` includes a button still held.

Purpose: distinguish a permanent tap from a spring-loaded Hand gesture.

```rust
let Some(mut s) = app.ui.spring.take() else { return };
if app.ui.text_edit.is_some() || app.ui.tool != s.tool {
    return; // text editing or another explicit tool choice owns the current state
}
let (now, button_down) = ctx.input(|i| (i.time, i.pointer.any_down()));
s.keys.retain(|k| key_of(*k).is_some_and(|k| ctx.input(|i| i.key_down(k))));
if !s.keys.is_empty() {
    app.ui.spring = Some(s); // H remains held
    return;
}
if !s.used && now - s.since < SPRING_HOLD_SECS {
    return; // quick tap: current Hand selection stands
}
// Once the hold reaches SPRING_HOLD_SECS or canvas use marks s.used, release restores the prior tool.
s.released = true;
if app.drag.is_some() || button_down {
    app.ui.spring = Some(s); // finish the gesture with Hand, then return
} else {
    app.ui.tool = s.previous; // restore the tool from before H
}
```

### Click the toolbar slot to select Hand directly

The Hand and Rotate View tools share a toolbar slot. A plain click or flyout choice calls `tool_memory::select` directly, without dispatching this ID. Alt-click cycles the slot; right-click, long-press, or drag-open exposes its flyout. A plain double-click on Hand selects it and runs `view.fitOnScreen`; with no document, there is nothing to fit. [Toolbar slot](../../../../photocraft/crates/ui-egui/src/panels/toolbar.rs#L25-L45) · [Selection and double-click](../../../../photocraft/crates/ui-egui/src/panels/toolbar.rs#L250-L305)

**Adapted Rust** — the direct selection and Hand double-click use the toolbar's local slot variables; neither dispatches the tool ID.

Purpose: show the toolbar's direct state change and its separate fit command.

```rust
if slot_clicked {
    let pick = if slot.len() > 1 && alt {
        let from = if slot.contains(&app.ui.tool) { app.ui.tool } else { tool };
        next_slot_tool(slot, from)
    } else {
        tool
    };
    tool_memory::select(app, pick); // direct UI selection; no tools.select.hand dispatch
}
let plain_double = resp.double_clicked() && !modified && !prev_modified;
if plain_double && tool == Tool::Hand {
    tool_memory::select(app, tool);
    let _ = app.run("view.fitOnScreen", json!({})); // Result<Value, String>; no document means no fit
}
```

### Drag the Hand to pan the active view

The selected Hand yields a primary-button drag to an open Free Transform box in the primary canvas; Space-Hand and middle-button drags still pan. A modal dialog can allow a Hand or Space drag on the canvas behind it, unless a sampler, selection brush, pin, curve point, or shadow drag owns that target. Layer selection, pixel/mask targeting, layer locks, color model, and bit depth do not affect panning. [Transform ownership](../../../../photocraft/crates/ui-egui/src/canvas.rs#L2046-L2051) · [Dialog pan admission](../../../../photocraft/crates/ui-egui/src/canvas.rs#L679-L759) · [Dialog pan delta](../../../../photocraft/crates/ui-egui/src/dialogs.rs#L92-L103)

The canvas resolves temporary tools before selecting the gesture. A middle-button drag acts as Hand; a click without movement does not pan. During a drag, PhotoCraft unmaps screen delta through the current rotation/flip/pixel-aspect transform, divides by zoom, and subtracts the result from the active document's view centre. Release stops the drag; Enable Flick Panning can continue the camera glide, while the overscroll preference controls clamping at document bounds. [Canvas input and pan](../../../../photocraft/crates/ui-egui/src/canvas.rs#L2004-L2114) · [View state reconciliation](../../../../photocraft/crates/ui-egui/src/canvas.rs#L1806-L1830)

**Adapted Rust** — this group omits canvas painting and keeps the decisive target, threshold, transform, and release behavior.

Purpose: update only the active view camera while a Hand drag is active.

```rust
let temporary = hold_keys::for_frame(app, ctx, drawing || type_transform::active(app));
let tool = match temporary {
    Some(t) => t.tool(),
    None if middle_button_down => Tool::Hand,
    None => app.ui.tool,
};
let transform_owns_pointer = primary && app.ui.transform.is_some()
    && temporary.is_none() && !middle_button_down;
let hand_pan = tool == Tool::Hand && !transform_owns_pointer;
let right_click = right_button_alone && paint_mouse::right_click_pending(response);
let hand_drag = hand_pan && response.dragged()
    && !(app.ui.tool == Tool::Hand && !space_pan && right_click);

if hand_drag {
    let delta = xf.unmap_vec(response.drag_delta()) / (view.zoom / ppp);
    view.center[0] -= delta.x;
    view.center[1] -= delta.y;
    ctx.set_cursor_icon(tool_cursor::hand(ctx, true)); // closed-hand cursor while moving
} // View is written back at frame end; no document edit or history command runs.
```

### Hold Space to pan or reposition another tool's gesture

Space is the separate `tools.temporary.hand` binding. When no supported gesture is being drawn, the canvas uses Hand until Space is released; a started pointer gesture keeps its temporary tool through release. If a marquee, lasso, object selection, shape, or crop frame is already being drawn, the Hand key sets that gesture's reposition mode instead: dragging moves the in-progress outline/frame, and releasing Space resumes its original tool behavior. [Temporary tool frame](../../../../photocraft/crates/ui-egui/src/input/hold_keys.rs#L222-L250) · [Reposition targets](../../../../photocraft/crates/ui-egui/src/input/hold_keys.rs#L222-L250) · [Canvas dispatch](../../../../photocraft/crates/ui-egui/src/canvas.rs#L1982-L2014)

**Adapted Rust** — selection-drawing state wins over temporary Hand; otherwise the held binding supplies Hand.

Purpose: show the boundary between view panning and repositioning the active selection/frame gesture.

```rust
let reposition = hold_keys::reposition_held(app, ctx);
crop_ui::set_space(app, reposition);
if let Some(drag) = app.drag.as_mut().filter(|d| hold_keys::repositions(d.tool)) {
    drag.reposition = reposition; // keep the same tool; move its draft geometry
}
let temporary = hold_keys::for_frame(app, ctx, drawing || type_transform::active(app));
// Some(Temporary::Hand) selects the Hand for this frame only when no drawing owns Space.
```

### Hold Command or Alt to zoom while Hand is selected

When the Hand is idle, Command (Ctrl away from macOS) temporarily routes pointer input to Zoom In; Alt routes to Zoom Out, and Alt takes precedence when both are held. Modifier Keys panel latches count. A modifier pressed after the pointer press does not convert that drag, and middle-button panning stays a pan. Free Transform retains Command for its own handles. [Hand modifier selection](../../../../photocraft/crates/ui-egui/src/input/hold_keys.rs#L123-L142) · [Effective canvas tool](../../../../photocraft/crates/ui-egui/src/canvas.rs#L2004-L2024)

**Adapted Rust** — only the Hand-specific modifier branch and resulting Zoom gesture are shown.

Purpose: route a Hand modifier gesture to the Zoom view behavior without editing pixels.

```rust
let hand_press = ((pointer.primary_down() || pointer.primary_released()) && !pointer.primary_pressed())
    || pointer.middle_down();
if app.ui.tool != Tool::Hand || app.ui.transform.is_some() || hand_press {
    return None; // Option<Temporary>; this press remains a pan/transform gesture
}
let mods = workspace_ui::sticky_mods(app, input.modifiers);
if mods.alt {
    Some(Temporary::ZoomOut) // Option<Temporary>; Alt wins when both modifiers are held
} else if mods.command {
    Some(Temporary::ZoomIn)
} else {
    None
}
```

#### Zoom with a click, drag, or animated hold

The canvas maps the temporary modifier choice to `Tool::Zoom`. A click steps the view in or out. With Scrubby Zoom on, a horizontal drag zooms continuously around the press point (right zooms in, left zooms out). With it off, dragging builds a rectangle preview; release fits that rectangle to the canvas. A sub-4-point rectangle steps instead, and Alt makes that step zoom out. These gestures only change the view. [Zoom routing](../../../../photocraft/crates/ui-egui/src/canvas.rs#L2004-L2024) · [Zoom drag and rectangle commit](../../../../photocraft/crates/ui-egui/src/tools/zoom_tool.rs#L53-L110)

**Adapted Rust** — press starts the anchored drag; movement either updates the scrub zoom or the rectangle preview; release removes the drag and applies the rectangle.

Purpose: show when the zoom changes continuously and when a preview becomes a view change.

```rust
if buttons.started {
    if let Some(anchor) = press_origin.filter(|p| xf.rect.contains(*p)).or(pointer) {
        ctx.data_mut(|d| d.insert_temp(zoom_id, ZoomDrag {
            anchor,
            doc: xf.to_doc(anchor),
            zoom0: view.zoom,
            rect_to: (!scrubby).then_some(anchor),
        })); // temporary ZoomDrag; no view change until movement
    }
}
let Some(mut drag) = ctx.data(|d| d.get_temp::<ZoomDrag>(zoom_id)) else {
    return buttons.started || buttons.dragged || buttons.stopped; // bool; canvas keeps ownership
};
if let Some(p) = pointer.filter(|_| buttons.dragged || buttons.stopped) {
    if let Some(to) = drag.rect_to.as_mut() {
        *to = p; // preview endpoint; zoom_tool::draw renders the rectangle as marching ants
    } else {
        let zoom = scrub_zoom(drag.zoom0, p.x - drag.anchor.x, view.doc_size);
        if zoom.to_bits() != view.zoom.to_bits() {
            anchor_view(view, xf, drag.anchor, drag.doc, zoom, app.ppp);
        }
    }
    ctx.data_mut(|d| d.insert_temp(zoom_id, drag));
}
if buttons.stopped {
    ctx.data_mut(|d| d.remove::<ZoomDrag>(zoom_id));
    if let Some(to) = drag.rect_to {
        zoom_to_rect(view, xf, Rect::from_two_pos(drag.anchor, to), drag.anchor, sticky_alt, app.ppp);
        // Scrubby Zoom off: release commits the preview rectangle into View.zoom/View.center.
    }
}
```

#### Hold still to start Animated Zoom

Animated Zoom starts only when its preference is enabled, the canvas owns the press, and the pointer stays within 6 points for 0.3 seconds. Moving first turns the press into the drag above. While held, it advances zoom around the same document anchor; releasing after it starts does not add a click step. [Animated hold state](../../../../photocraft/crates/ui-egui/src/canvas/zoom_hold.rs#L16-L23) · [Hold update](../../../../photocraft/crates/ui-egui/src/canvas/zoom_hold.rs#L59-L120)

**Adapted Rust** — a still press becomes a hold after the delay; a drag suppresses hold behavior.

Purpose: show the Animated Zoom threshold and its anchored view update.

```rust
if !enabled || !down {
    if held.is_some() {
        ctx.data_mut(|d| d.remove::<Hold>(id));
    }
    return enabled && held.is_some_and(|h| h.last.is_some()); // bool; an active hold owns its release
}
if h.last.is_none() {
    if pos.is_some_and(|p| p.distance(h.anchor) > 6.0) {
        h.dragged = true;
        return false; // bool; let zoom_tool::drag own this press instead
    }
    if now < h.t0 + 0.3 {
        return false; // bool; wait for the still-press delay
    }
    h.last = Some(h.t0 + 0.3);
}
let dt = now - h.last.unwrap_or(now);
let zoom = hold_zoom(view.zoom, dt, out, view.doc_size);
if zoom.to_bits() != view.zoom.to_bits() {
    anchor_view(view, xf, h.anchor, h.doc, zoom, ppp);
}
h.last = Some(now);
ctx.data_mut(|d| d.insert_temp(id, h));
ctx.request_repaint();
true // bool; canvas skips scrub and click on this hold, including release
```

### Right-click the image to open the view menu

A right-click with the selected Hand opens the Zoom/Hand view menu at the pointer's document position. A right-click on the pasteboard opens its color menu instead. A right drag that stays within the right-click threshold opens the view menu; once it moves beyond that threshold it pans. Space makes a right drag a pan and suppresses the temporary Hand's view menu. [Menu dispatch](../../../../photocraft/crates/ui-egui/src/canvas.rs#L902-L940) · [View menu state](../../../../photocraft/crates/ui-egui/src/canvas/canvas_tool_menu.rs#L882-L905)

**Adapted Rust** — the Hand-specific branch and view-menu target are kept together.

Purpose: open the view menu on a right-click without moving the view.

```rust
if right_click && !pasteboard_at(app, tool, xf.to_doc(pointer)) && is_view_tool(tool) {
    if tool == app.ui.tool {
        let opened = canvas_tool_menu::open_view(app, ctx, tool, screen_pos, xf.to_doc(pointer));
        // bool; true stores CanvasToolMenu.anchor at the clicked document point
    }
}
```

#### Choose a view menu row

The menu offers Fit on Screen, Fill Screen, 100%, 200%, Print Size, Zoom In, and Zoom Out. Fill Screen sets `fill_pending`; the other rows invoke their View command. Zoom In/Out re-anchor the camera so the clicked document point stays under the pointer. The options bar's 100%, Fit Screen, and Fill Screen buttons also change only view state. These operations do not change document history. [Menu rows](../../../../photocraft/crates/ui-egui/src/canvas/canvas_tool_menu.rs#L357-L372) · [Menu choice](../../../../photocraft/crates/ui-egui/src/canvas/canvas_tool_menu.rs#L909-L935) · [Hand options bar](../../../../photocraft/crates/ui-egui/src/panels/options_bar.rs#L672-L698)

**Adapted Rust** — the menu choice handles Fill Screen directly and delegates the other rows.

Purpose: update view state from the menu and keep the clicked point anchored for Zoom In/Out.

```rust
let i = app.session.active_index().ok_or("no document")?;
if command == FILL_SCREEN {
    let v = app.ui.views.get_mut(i).ok_or("no document view")?;
    v.fit_pending = false;
    v.fill_pending = true;
    v.request_zoom_animation(app.session.prefs().general.animated_zoom);
    v.zoom_anchor = None;
    return Ok(()); // Result<(), String>; Fill Screen changes the view directly
}
let before = app.ui.views.get(i).map(|v| (v.zoom, v.center));
crate::menus::invoke(app, ctx, command, json!({}))?; // Result<Value, String>; View command result is discarded
if matches!(command, "view.zoomIn" | "view.zoomOut") {
    let (Some((zoom0, center0)), Some(doc)) = (before, menu.anchor) else { return Ok(()) };
    let center_on_point = app.session.prefs().tools.zoom_clicked_point_to_center;
    let Some(v) = app.ui.views.get_mut(i) else { return Ok(()) };
    let doc = [doc[0] as f32, doc[1] as f32];
    let ratio = zoom0 / v.zoom;
    v.center = if center_on_point || !ratio.is_finite() {
        doc
    } else {
        [doc[0] - (doc[0] - center0[0]) * ratio, doc[1] - (doc[1] - center0[1]) * ratio]
    };
    v.zoom_anchor = (v.zoom_animating && !center_on_point).then_some(([doc[0] as f64, doc[1] as f64], menu.pos));
} // All branches update the view; none edits document history.
```
