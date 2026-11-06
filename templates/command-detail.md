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

## Core logic flow

{{Use a short numbered list to name every important phase: admission, parameter handling, sampling/data preparation, algorithm branches, state update and response. Include preview/commit or async phases when present. Then show enough simplified code to explain each phase and its material branches. Do not reduce the flow to function names or hide the algorithm behind invented helper calls. Explain source-supported behavior in bullets and short sentences.}}

### {{Phase with meaningful logic}}

Original source: [{{symbol and path}}](https://github.com/storytold/photocraft/blob/{{verified full commit SHA}}/{{repo-relative Rust path}}#L{{start}}-L{{end}}).

**Adapted Rust** — {{identify what was shortened or renamed; use “Verbatim Rust” only for unchanged contiguous source. If helper names or control flow are invented for explanation, call the snippet pseudocode.}}

```rust
// Input: {{actual inputs and preconditions}}.
// Response: {{returned value or state change}}.
// Explanation: {{how this phase produces the response}}.
{{show the essential branches, loops, conditions and value transformations in Rust-like code; keep enough lines to make the algorithm understandable}}
```

- {{One short sentence describing a subtle branch or boundary.}}

{{Repeat for each important phase. Omit incidental setup, logging and boilerplate. Avoid pasting whole handlers when a focused adaptation explains the same behavior. Keep GitHub line links pinned and verify their ranges.}}

## Structures used

- `{{type}}`: {{fields and role in one short sentence}}. [{{original definition}}](https://github.com/storytold/photocraft/blob/{{verified full commit SHA}}/{{repo-relative Rust path}}#L{{start}}-L{{end}}).
- {{List only structures needed to follow the flow.}}
