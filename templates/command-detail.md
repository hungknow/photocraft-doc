# `{{exact command ID}}` — {{label}}

Lifecycle: {{ACTIVE or DELETED}} · Documentation: {{CURRENT, STALE, or INCOMPLETE}}
Verified against: {{revision, dirty state, date, source root}}
Scope/report: {{scan report link}}
Navigation: {{command index anchor link}} · {{tool index link}}

## Purpose

{{Explain the concrete outcome and why a user would invoke it. State scope and limitations supported by the implementation.}}

## Context to use

{{Typical task, required document/selection/layer or mask state, supported modes/depths, and availability. Distinguish enabled checks from runtime validation.}}

| Entry point | Trigger | Parameters/conditions | Implementation route |
|---|---|---|---|
| {{specific tool anchor or non-UI entrypoint}} | {{action/phase}} | {{values/derivation}} | {{shell/dialog/engine/helper links}} |

## Inputs and results

| Parameter | Type | Required/default | Validation | Meaning |
|---|---|---|---|---|
| {{name}} | {{actual accepted type}} | {{requirement}} | {{bounds/errors}} | {{purpose}} |

{{Result shape, state changes, and a valid invocation example with prerequisites and its actual dispatch surface. For zero parameters say so. Distinguish example data from source excerpts.}}

## Core logic flow

{{Numbered explanation from invocation through admission, dispatch, core algorithm, mutation, result and UI update. Include meaningful branches, failure paths, and async completion. A short Mermaid diagram is optional.}}

### {{Step and responsibility}}

Source: {{relative source link, Rust symbol, line span, verified revision}}

```rust
{{verbatim contiguous Rust excerpt}}
```

{{Explain the excerpt's inputs, branches, outputs and connection to the next step. Repeat this block for the core handler and important lower-layer implementation, not just its call site. Identify omissions outside code blocks.}}

## Structures used

| Type | Role | Important fields/variants | Ownership and mutation | Defined in |
|---|---|---|---|---|
| {{Rust type}} | {{why needed here}} | {{fields and semantics}} | {{borrow/clone/shared data and mutations}} | {{source link}} |

Source: {{relative source link, type symbol, line span, verified revision}}

```rust
{{verbatim relevant struct, enum, or type alias excerpt}}
```

{{Explain how these values move through the flow. Repeat for core types; split noncontiguous field excerpts and explain omissions outside code.}}

## Effects, history, and failure behavior

{{Document mutation boundaries, undo/redo/coalescing, journal behavior, preview/cancel rollback, I/O, asynchronous jobs, and error propagation as applicable. Do not infer atomicity or undo from naming alone.}}

## Tests and evidence

| Test/source | Behavior established | Verification |
|---|---|---|
| {{test link and symbol}} | {{case and limits}} | {{inspected only, or actual command/result}} |

## Related commands and tools

{{Specific reciprocal tool links, related command index links, and existing detail pages. Explain relationships briefly.}}

## Limitations and open questions

{{Unresolved tracing, conditional branches not verified, stale evidence or missing tests. Say None identified only when justified.}}

## Notes

{{Preserve human-authored notes here.}}

## Revision history

| Date/scan | Source revision | Change |
|---|---|---|
| {{date and scan link}} | {{revision/dirty}} | {{creation, refresh, stale, deletion, or restoration}} |
