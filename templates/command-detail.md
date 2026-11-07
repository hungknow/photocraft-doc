# {{exact command ID}}

- {{One short sentence describing the result.}}

Use bullet lists for explanations, routes, branches, prerequisites, defaults, results, and errors wherever practical. Keep paragraphs to brief orientation or transitions; split long explanations into bullets. Keep each bullet focused on one point.

Verified against: {{source revision and date}}
Navigation: {{command category heading link}} · {{tool category heading links}}

## Context to use

{{State the prerequisite and relevant document/tool state in one or two short sentences.}}

{{Identify every verified way to reach this behavior: menu, toolbar or panel action, shortcut, pointer gesture, control channel, CLI, automation, and direct engine dispatch where applicable. Use one short bullet per distinct action or gesture, with nearby evidence. Distinguish actions that dispatch this ID from actions that directly select the same tool or state.}}

- {{UI action and its effect. See [concise source label](pinned source URL).}}

{{State headless behavior in its own sentence when it differs from the UI. Omit irrelevant blocks.}}

## Inputs and response

{{State the accepted parameter shape and prerequisite. Show a small valid example when useful:}}

```json
{{small valid JSON invocation parameters}}
```

{{Group optional fields and defaults into short bullets. Name required fields separately when present.}}

- `{{field}}`: {{default or requirement, with a brief explanation.}}

{{State the immediate return value and resulting document/UI change in a short sentence. If later gestures or asynchronous work have different results, name them here and explain each under Core logic flow. Do not present tool selection as the document edit performed by a later gesture.}}

{{Introduce important errors, then list distinct cases in short bullets if needed. State clamping or coercion separately. Put evidence links near the claims they support.}}

## Structures used

{{Show the Rust definitions or focused adapted fragments for the structures that carry state across every relevant flow: command parameters, document state, UI/tool state, pointer or keyboard gesture state, preview/job state, and history where applicable. Include only fields and variants used in the walkthrough. Keep the source link for each structure and label any adapted code. In Rust comments, give concrete sample values and explain what each value means. Do not replace the code with a list of type names. Repeat this source-and-fragment group for additional structures as needed.}}

Original source: [{{type and path}}](https://github.com/storytold/photocraft/blob/{{verified full commit SHA}}/{{repo-relative Rust path}}#L{{start}}-L{{end}}).

**Adapted Rust** — {{state any omitted fields or variants; use “Verbatim Rust” only for unchanged code.}}

```rust
{{struct, enum, or focused Rust fragment with concrete sample values and non-obvious meanings commented beside the relevant fields}}
```

## Algorithm

{{Inspect the full downstream behavior to determine whether a mathematical, geometric, image-processing, rendering, or other numerical algorithm materially defines the command. If one is present, this section is required—even when the command only selects a tool or stores state and another handler or render pass later applies the algorithm. Trace the actual computation through that consuming/rendering path, include units, and give a worked example with realistic sample inputs, intermediate values, and output. Explain what changes on screen or in the document. Cite the pinned source and label adapted code. Omit this section only when source tracing confirms that no such algorithm is available or relevant.}}

## Core logic flow

{{First give one representative action in actual execution order. Name each real caller and callee and its file: entry/selection → event → guards/interceptors → algorithm/state → render/preview → commit/result/history. Explain when the next call occurs in a later gesture or render pass. For each non-obvious method, say what it accepts, what its return type/value means, what state it changes, and how failure is represented. Use a short numbered route or a compact call map with source links. Say that the following branch subsections are alternatives, not one continuous execution sequence.}}

### Read the execution route before the individual flows

1. {{Actual entry and resulting state; cite source.}}
2. {{Actual dispatch and interception order; cite source.}}
3. {{Actual algorithm and preview/render boundary; cite source.}}
4. {{Actual commit, response, and history; cite source.}}

{{Briefly explain the meaning and return/failure semantics of non-obvious methods at their first use, either in this route or the relevant flow.}}

{{Then create a separate level-three subsection for each verified, materially distinct logic flow. Name the flow by its trigger and result, such as “Invoke the ID to select the tool,” “Drag selected pixels,” “Drag layers,” or “Nudge with arrow keys.” Do not combine independent flows under a generic phase heading. Within each flow, trace its dispatch point, admission and interception, target choice, parameter or modifier handling, lower-layer algorithm, preview, commit or cancel, result, and history as applicable. Use level-four headings inside a flow only when its phases need separation. Include no-document, selection versus layer or mask, lock, error, async completion, and ownership branches where they change the result. For a tool-selection ID, separately explain selecting the tool and using it; follow later gestures to the commands they actually dispatch. Do not imply a direct UI action dispatched the ID unless the source proves it. Omit flows that do not exist. Show enough simplified code to explain decisive branches, with return types and sample results where non-obvious. Put explanations beside the relevant Rust lines as comments; avoid prose that repeats the code.}}

### {{Distinct logic flow: trigger and result}}

{{State which entry path starts this flow and how it differs from the other flows. Keep the whole flow in this subsection. Add `####` phase headings only if needed.}}

Original source: [{{symbol and path}}](https://github.com/storytold/photocraft/blob/{{verified full commit SHA}}/{{repo-relative Rust path}}#L{{start}}-L{{end}}).

**Adapted Rust** — {{identify what was shortened or renamed; use “Verbatim Rust” only for unchanged contiguous source. If helper names or control flow are invented for explanation, call the snippet pseudocode.}}

Purpose: {{one short sentence describing what this code group does and which flow or next phase it leads to.}}

```rust
{{show essential branches, loops, conditions, and value transformations in Rust-like code; comment behavior beside relevant lines and give actual types plus samples for non-obvious results}}
```

{{Repeat the `###` subsection for every distinct logic flow, including separate pointer targets and keyboard paths when they produce different results. Keep press, movement, release, preview, commit, and cancellation for one flow together under its subsection. If a flow branches to another command, name that command and trace the result far enough to explain the user-visible effect and history. For pointer tools, check click without movement and modifier changes. For asynchronous work, check start, completion, failure, and cancellation. Show where a preview is constructed and where the edit is committed; a pointer-position update alone is not a preview. Use short bullets for context or steps where possible. Keep Rust explanations in comments beside the code. Omit incidental setup, logging and boilerplate. Avoid pasting whole handlers when a focused adaptation explains the same behavior. Keep GitHub line links pinned and verify their ranges.}}
