# {{exact command ID}}

{{short user-facing label}}

Verified against: {{source revision and date}}
Navigation: {{command category heading link}} · {{tool category heading links}}

## Purpose

- {{One short sentence describing the result.}}

## Context to use

- {{When to run it and required document/tool state.}}
- {{Important UI/headless distinction, if any.}}

## Inputs and response

- Input: {{required fields and meaningful defaults; show a small example if useful.}}
- Response: {{actual return value and document/UI change.}}
- Failure: {{important validation or admission error, if relevant.}}

## Structures used

{{Show the Rust definitions or focused adapted fragments for the structures that carry this command's state. Include fields and variants needed by the flow. Keep the source link for each structure and label any adapted code. In Rust comments, give concrete sample values and explain what each value means for this command. Do not replace the code with a list of type names.}}

Original source: [{{type and path}}](https://github.com/storytold/photocraft/blob/{{verified full commit SHA}}/{{repo-relative Rust path}}#L{{start}}-L{{end}}).

**Adapted Rust** — {{state any omitted fields or variants; use “Verbatim Rust” only for unchanged code.}}

```rust
// Sample: {{a concrete, plausible value for this command, with field meanings}}.
{{struct, enum, or focused Rust fragment showing its important fields and types}}
```

- {{One short sentence connecting the structure to the flow. Repeat for other essential structures.}}

## Core logic flow

{{Use a short numbered list to name every important phase: admission, parameter handling, sampling/data preparation, algorithm branches, state update and response. Include preview/commit or async phases when present. Then show enough simplified code to explain each phase and its material branches. Show the actual return type and a plausible sample result only where a call or branch result is not obvious. For long values, keep decisive fields or give the type name. Do not reduce the flow to function names or hide the algorithm behind invented helper calls. Explain source-supported behavior in bullets and short sentences.}}

### {{Phase with meaningful logic}}

Original source: [{{symbol and path}}](https://github.com/storytold/photocraft/blob/{{verified full commit SHA}}/{{repo-relative Rust path}}#L{{start}}-L{{end}}).

**Adapted Rust** — {{identify what was shortened or renamed; use “Verbatim Rust” only for unchanged contiguous source. If helper names or control flow are invented for explanation, call the snippet pseudocode.}}

Purpose: {{one short sentence describing what this code group does and why it matters to the next phase.}}

```rust
{{show essential branches, loops, conditions, and value transformations in Rust-like code}}
{{at a non-obvious result only: // Returns Option<Region>, e.g. Some(Region { /* fields omitted */ })}}
```

- {{One short sentence describing a subtle branch or boundary.}}

{{Repeat for each important phase. Omit incidental setup, logging and boilerplate. Avoid pasting whole handlers when a focused adaptation explains the same behavior. Keep GitHub line links pinned and verify their ranges.}}
