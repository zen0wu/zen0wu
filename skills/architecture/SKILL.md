---
name: architecture
description: "Apply Zeno's architecture reasoning when designing or evaluating systems, integrations, infrastructure, ownership boundaries, or capability constraints. Use for system-level explanations and trade-offs, not routine code-only edits."
---

# Architecture

Start with the concrete problem and its constraints. Prefer the smallest design that solves it; name what each component owns and make the trade-offs explicit.

## Reason from the responsible component

- Define every system component, integration, event, or overloaded technical term before using it, then use one canonical name for it throughout the explanation.
- Name the exact component responsible for each behavior. Do not hide responsibility behind vague subjects such as "it", "the platform", or "the current path".
- Separate what the underlying product supports, what an integration supports, what its API or infrastructure adapter exposes, and what the current repository configuration enables.
- Qualify and classify limitations. Distinguish physically impossible, unsupported by the underlying product or API, unsupported by the current implementation or configuration, possible but unvalidated, and possible but rejected because of operational trade-offs.
- Resolve ambiguous terminology before reasoning. Analyze interpretations separately when they materially change the answer.
- Before making an impossibility claim, check the nearest alternative mechanism that relaxes one assumption and verify its capabilities using current code or primary documentation.
- Use a capability matrix when two or more independent dimensions affect the conclusion.

## Examples, only when useful

Read the relevant section of [references/examples.md](references/examples.md) when a conclusion depends on layers or on what a system can support:

- [Layered integrations](references/examples.md#layered-integrations): distinguish product behavior, native integration, API or adapter exposure, and current configuration.
- [Infrastructure layers](references/examples.md#infrastructure-layers): scope a limitation to the actual provisioner and configuration, then check the nearest alternative.

These are reasoning examples, not current product documentation. Verify present-day capabilities before relying on their specific GitHub, Buildkite, or AWS details. Do not load the examples for routine design work when the rules above are enough.
