# {{exact command ID}}

- {{One short sentence describing the result.}}

Verified against: {{source revision and date}}
Navigation: {{command category heading link}} · {{tool category heading links}}

## Context to use

{{State the prerequisite and relevant document/tool state in one or two short sentences.}}

{{If there are UI entry points, introduce them briefly and use one short bullet per distinct action or gesture. Place evidence links beside the action they support.}}

- {{UI action and its effect. See [concise source label](pinned source URL).}}

{{State headless behavior in its own sentence when it differs from the UI. Omit irrelevant blocks.}}

## Inputs and response

{{State the accepted parameter shape and prerequisite. Show a small valid example when useful:}}

```json
{{small valid JSON invocation parameters}}
```

{{Group optional fields and defaults into short bullets. Name required fields separately when present.}}

- `{{field}}`: {{default or requirement, with a brief explanation.}}

{{State the actual return value and resulting document/UI change in a short sentence.}}

{{Introduce important errors, then list distinct cases in short bullets if needed. State clamping or coercion separately. Put evidence links near the claims they support.}}

## Structures used

{{Show the Rust definitions or focused adapted fragments for the structures that carry this command's state. Include fields and variants needed by the flow. Keep the source link for each structure and label any adapted code. In Rust comments, give concrete sample values and explain what each value means for this command. Do not replace the code with a list of type names.}}

Original source: [{{type and path}}](https://github.com/storytold/photocraft/blob/{{verified full commit SHA}}/{{repo-relative Rust path}}#L{{start}}-L{{end}}).

**Adapted Rust** — {{state any omitted fields or variants; use “Verbatim Rust” only for unchanged code.}}

```rust
{{struct, enum, or focused Rust fragment with concrete sample values and non-obvious meanings commented beside the relevant fields}}
```

## Core logic flow

{{Use short phase headings for admission, parameter handling, algorithm branches, state update and response as relevant. Include preview/commit or async phases when present. Show enough simplified code to explain material branches. Put explanations beside the relevant Rust lines as comments; avoid follow-up prose that repeats the code. Show return types and sample results only where non-obvious.}}

### {{Phase with meaningful logic}}

Original source: [{{symbol and path}}](https://github.com/storytold/photocraft/blob/{{verified full commit SHA}}/{{repo-relative Rust path}}#L{{start}}-L{{end}}).

**Adapted Rust** — {{identify what was shortened or renamed; use “Verbatim Rust” only for unchanged contiguous source. If helper names or control flow are invented for explanation, call the snippet pseudocode.}}

Purpose: {{one short sentence describing what this code group does and why it matters to the next phase.}}

```rust
{{show essential branches, loops, conditions, and value transformations in Rust-like code; comment behavior beside relevant lines and give actual types plus samples for non-obvious results}}
```

{{Repeat for each important phase. Use short bullet lists for phase context or steps where possible. Keep explanations of Rust behavior in comments beside the code they describe. Omit incidental setup, logging and boilerplate. Avoid pasting whole handlers when a focused adaptation explains the same behavior. Keep GitHub line links pinned and verify their ranges.}}
