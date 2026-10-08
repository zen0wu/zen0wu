---
name: coding
description: "Apply Zeno's coding preferences when implementing, debugging, refactoring, testing, explaining, or reviewing concrete code. Focus on readable workflows, domain boundaries, data shapes, abstractions, and failures; not system-level architecture alone."
---

# Coding

Make the concrete workflow easy to follow. Apply these preferences within the requested scope and the repository's constraints; a diagnosis or review is not permission to edit.

## Code should read like a workflow

- Use concrete domain nouns and verbs. Avoid placeholder words such as `data`, `value`, `item`, `block`, or `state`, and process jargon such as `prepare`, `preflight`, or `handle`, unless they are genuinely precise.
- Declare a variable as close as possible to its first use.
- Keep control flow streamlined. Avoid nested conditionals and inner functions unless the structure genuinely requires them.
- Order a file as the main entry point or workflow, mid-layer business functions in call order, then leaf and shared utilities. Put only the types and constants needed to understand the workflow before it.
- Use prominent section headers for distinct workflows and short intent comments for major phases of a long function.
- Keep cleverness local and sparse. If it cannot be removed, explain it at the beginning of the relevant code block.
- Prefer stateless, pure functions when practical.

## Boundaries should own one concept

- A mid-layer function must own a recognizable domain step. If its purpose is not obvious from its name and signature, redesign the boundary before documenting it.
- Keep discovery and iteration, parsing, policy, validation, and mutation separate. Lower-level operations handle one explicit unit; orchestration owns scanning and repetition.
- Treat function-valued parameters, fields, and return values as a design smell. Use a callback only when varying behavior is itself the domain concept.
- Validate external and versioned data at the boundary, then convert it once into typed, immutable, canonical domain objects. Do not leak raw maps, version checks, or legacy-only fields into core validation or execution.
- Make illegal states unrepresentable without forcing the design into functional, object-oriented, or another form of purism.

## Abstractions must earn their place

- Create a reusable or parameterized abstraction only when it generalizes at least three real examples. A one-call-site function is justified only when it names a real domain phase or invariant and makes its caller easier to scan.
- Before finishing, inventory every new function, type, field, option, and layer. Remove or inline anything that does not own a clear domain concept or invariant.

## Fail explicitly

- Let failures surface. Add `try`/`catch` only when the code can handle a specific failure honestly.
- Throw specific error classes or typed error values. Never throw a generic `new Error`.

## Plan before mutation

For multi-target or destructive workflows, build the complete plan, validate the whole plan, then apply mutations. Execution should consume the plan without rediscovering or reinterpreting the source data.

## Examples, only when useful

The rules above are sufficient for routine work. Read the relevant section of [references/examples.md](references/examples.md) when deciding a trade-off or explaining why a pattern helps:

- [Boolean logic](references/examples.md#boolean-logic): condition names, nesting, and avoiding shared-state mutation.
- [Data shape](references/examples.md#data-shape): scattered flags, impossible states, and typed actions.
- [Abstraction timing](references/examples.md#abstraction-timing): callback-heavy helpers and premature reuse.
- [API design, defaults, and error handling](references/examples.md#api-design-defaults-and-error-handling): separating operations and exposing honest failures.

Examples illustrate the reasoning, not templates to copy mechanically. Do not load every example for every task.
