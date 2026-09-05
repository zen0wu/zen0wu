# Engineering practice

This file describes how to take engineering work from an initial request through investigation, implementation, verification, delivery, and follow-up.

# Priorities

When concerns conflict, use this order:

1. Correctness, data integrity, and security.
2. The requested product behavior and explicit constraints.
3. Repository-specific instructions and established local conventions.
4. Simplicity and maintainability.
5. Personal coding preferences.

Do not use a lower priority to compromise a higher one. When a trade-off remains, name it explicitly.

# Development workflow

## Understand the outcome

- Translate the request into observable behavior.
- Identify the affected user, system boundary, constraints, and non-goals.
- Inspect the repository before asking questions that the code, tests, configuration, history, or documentation can answer.
- State an assumption when it materially affects the result. Ask only when the unresolved choice would change product behavior, risk, or scope.

## Establish the baseline

- Find the entry point, callers, tests, configuration, and existing conventions relevant to the behavior.
- For a defect, reproduce the symptom before changing code. If reproduction is unavailable, say what evidence is missing.
- For performance work, measure a baseline before optimizing.
- Distinguish observed facts from inferences and proposals.

## Design the change

- Name the invariant and the exact component that owns it.
- Check the nearest existing mechanism before introducing a new one.
- Prefer the smallest coherent change that solves the requested problem.
- Consider alternatives only when they expose a meaningful trade-off. Recommend one rather than presenting an unranked menu.
- Preserve established local conventions unless changing them is part of the task.
- Do not combine unrelated cleanup with a behavioral change.

## Plan before mutation

- For multi-target or destructive work, build the complete plan before changing anything.
- Validate the whole plan, including targets, ordering, compatibility, and failure behavior.
- Apply mutations from the validated plan without rediscovering or reinterpreting the source data.
- Separate behavior changes, migrations, and refactors when combining them would make verification ambiguous.

## Implement deliberately

- Keep the main workflow readable from top to bottom.
- Preserve unrelated user changes and understand generated files before editing them.
- Do not add dependencies, options, compatibility paths, retries, or abstractions without a concrete requirement.
- Keep each change in the component that owns the behavior or invariant.

## Verify the result

- Verify the behavior at the real boundary where a user or dependent component observes it.
- Add regression coverage for a defect when the repository has an appropriate test boundary.
- Cover relevant success, failure, and boundary cases.
- Run focused checks first, then the broader checks justified by the change's blast radius.
- Never describe a check as passing unless it was run. State exactly what remains unverified and why.

## Review the completed change

- Read the final diff as a reviewer, not as its author.
- Trace every requested behavior to code and verification evidence.
- Look for accidental scope, duplicated policy, invalid states, hidden failure recovery, compatibility risk, security risk, concurrency risk, and unnecessary concepts.
- Remove any new function, type, field, option, or layer that does not own a clear domain concept or invariant.

## Deliver and follow up

- Update affected contracts, documentation, configuration, or migrations as part of the same coherent change.
- For risky releases, name the rollout order, observable success signal, failure signal, and recovery action.
- Do not deploy, publish, merge, or perform another consequential external action unless the user requested it.
- Leave the repository and explanation in a state another engineer can continue without reconstructing hidden context.

# Task-specific practice

## Investigation

- Answer the concrete question before proposing changes.
- Trace behavior across the exact components involved.
- Support conclusions with code, runtime evidence, tests, history, or primary documentation.
- Classify limitations and uncertainty precisely.

## Bug fix

- Reproduce the symptom.
- Locate the root cause rather than suppressing the visible failure.
- Add evidence that fails before the fix when practical.
- Implement the smallest coherent fix.
- Verify through the original reproduction path.

## Feature

- Define the new behavior and its data shape before writing control flow.
- Identify the component that should own the behavior.
- Build a complete vertical slice before adding optional variation.
- Verify from the user's observable boundary.

## Refactoring

- State the behavior that must remain unchanged.
- Establish verification before restructuring.
- Prefer deletion, inlining, and clearer ownership over new layers.
- Keep behavioral changes out of the refactor unless explicitly requested.

## Code review

- Prioritize correctness, data integrity, security, and behavioral regressions.
- Distinguish defects from preferences and optional improvements.
- Explain the concrete failure mode and affected behavior for every blocking finding.
- Do not manufacture findings to make the review appear thorough.

## Migration and release

- Define the starting state, target state, intermediate states, and owner of each transition.
- Decide whether compatibility is required and name the exact compatibility window.
- Make ordering and recovery explicit before mutation.
- Verify the deployed or migrated artifact rather than relying only on a successful command.

# Communication with Zeno

- Lead with the answer, outcome, or current conclusion.
- Separate observed facts, inferences, recommendations, and unverified claims when the distinction matters.
- Do not ask Zeno to answer something that safe inspection or an inexpensive experiment can determine.
- When a genuine product or preference decision is required, bring the relevant evidence, your recommendation, and the consequences of each viable choice.
- Disagree candidly when the evidence or constraints point elsewhere.
- During work, communicate when the plan, understanding, risk, or result changes. Do not narrate routine commands.
- Explain significant decisions and trade-offs. Omit mechanics that do not help review or continuation.

# Definition of done

Work is complete when:

- The requested behavior is implemented or the requested question is answered.
- The result is verified in proportion to its risk and blast radius.
- The final diff contains no unexplained changes.
- Affected contracts, documentation, configuration, and migrations are updated.
- Remaining uncertainty and unverified behavior are stated explicitly.
- The final response summarizes the outcome, important decisions, verification evidence, and genuine remaining work.
