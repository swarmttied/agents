---
name: clean-coder
description: Creates, reviews, and refactors code using pragmatic Clean Code and Clean Architecture principles while preserving correctness and separating design findings from coding defects.
user-invocable: true
---

# Clean Coder

Use these instructions when creating, reviewing, or refactoring software. Optimize for code that is correct, easy to understand, safe to change, testable, observable, and appropriately efficient.

Clean Code and Clean Architecture are guidelines, not line-count rules or a mandatory folder template. Apply them in the context of the repository, its domain, runtime constraints, and existing conventions. Do not introduce abstractions without a demonstrated boundary, variation point, or testing need.

## Core principles

### Clean Code

1. **Preserve correctness first.** Readability does not compensate for incorrect behavior.
2. **Reveal intent.** Names should describe domain meaning and expected outcomes, not implementation mechanics.
3. **Keep responsibilities cohesive.** A method or class should have one primary reason to change. Do not split code merely to make methods shorter.
4. **Make control flow explicit.** Prefer guard clauses, bounded nesting, and clear success/failure paths.
5. **Make dependencies and side effects visible.** Inject external collaborators. Avoid hidden global state, unexpected mutation, and work concealed in property accessors.
6. **Handle failure deliberately.** Validate data at boundaries, catch only failures the current layer can handle, preserve diagnostic context, and never create a false success.
7. **Remove accidental complexity.** Eliminate duplication, magic values, unnecessary conversions, repeated enumeration, and speculative abstractions.
8. **Use comments for context and rationale.** Prefer expressive code for what happens; use comments for why, constraints, non-obvious tradeoffs, or external requirements.
9. **Design for verification.** Important behavior and edge cases should be testable without real databases, networks, clocks, or storage.
10. **Measure performance-sensitive decisions.** Do not sacrifice clarity for imagined performance or add layers blindly where profiling shows they are harmful.

### Clean Architecture

The central rule is that **source-code dependencies point toward higher-level policy**:

- **Domain:** enterprise and domain rules and concepts; no dependency on databases, web APIs, UI, cloud SDKs, or frameworks.
- **Application/use cases:** application-specific workflows; depends on the domain and defines the ports it needs.
- **Adapters/infrastructure:** implements application ports for databases, external APIs, storage, messaging, and telemetry.
- **Entry points/frameworks:** functions, controllers, hosts, and configuration; compose implementations at the application boundary.

Control flow can call outward through an interface, but the interface should be owned by the inner layer that needs the capability. Data crossing a boundary should be shaped for the inner layer rather than exposing framework, transport, or persistence types.

Do not force four projects onto every codebase. The invariant is separation of policy from volatile details and inward-pointing compile-time dependencies, not a particular directory diagram.

## Creating code

Before writing:

1. Identify the use case, business invariants, inputs, outputs, failure modes, and external side effects.
2. Locate existing domain vocabulary, conventions, abstractions, and similar implementations.
3. Decide which layer owns each responsibility. Put interfaces at the layer that consumes the capability.
4. Define observable acceptance criteria and edge cases.

While writing:

- Use domain-specific names and consistent terminology.
- Keep orchestration readable as a sequence of use-case steps.
- Separate pure decisions and transformations from I/O where practical.
- Prefer immutable inputs and results; make necessary mutation obvious and local.
- Use dependency injection for external resources.
- For .NET I/O, use `async`/`await` end to end and accept a `CancellationToken` at application boundaries.
- Validate external identifiers and nullable data before conversion or dereference.
- Use explicit comparison semantics, such as `StringComparison.OrdinalIgnoreCase` for protocol or status values.
- Return a meaningful result when callers need to know complete, partial, or failed outcomes.
- Add focused tests for success, boundary values, invalid external data, dependency failures, and partial progress.

Before finishing:

- Verify behavior with the narrowest relevant build and tests.
- Check that dependencies still point in the intended direction.
- Remove dead code, stale comments, accidental public or protected members, and temporary diagnostics.
- Confirm logs contain useful context without secrets or duplicated noise.

## Reviewing code

Review in this order so cosmetic observations do not hide real defects:

1. **Correctness:** Does the implementation satisfy the use case? Check nulls, conversions, ordering, equality, status handling, deferred execution, concurrency, retries, and partial failure.
2. **Boundary safety:** Are external responses validated? Are exceptions handled at a layer that can make a meaningful decision?
3. **Architecture:** Do core policies depend on infrastructure? Are ports owned by the use-case layer? Is framework-specific data leaking inward?
4. **Cohesion and coupling:** Does each class have a focused responsibility? Are collaborators and side effects explicit?
5. **Readability:** Are names, control flow, and abstractions understandable without explanatory narration?
6. **Testability:** Can business decisions be tested independently? Are important negative and edge cases covered?
7. **Operations:** Are cancellation, telemetry, correlation, idempotency, and failure reporting appropriate for the workload?
8. **Performance:** Is I/O bounded and asynchronous? Are collections repeatedly queried or materialized unnecessarily?
9. **Style:** Apply repository formatting and naming conventions last.

### Classify findings

Keep these categories separate:

- **Clean Code and Clean Architecture findings:** maintainability, readability, cohesion, coupling, testability, dependency direction, and clarity of intent. Do not claim that these necessarily cause incorrect runtime behavior.
- **Coding issues and pitfalls:** bugs, unsafe edge cases, runtime failures, concurrency hazards, incorrect results, data-loss risks, and measurable performance problems.

For every finding, provide:

- category;
- severity: critical, high, medium, or low;
- exact file and line range;
- violated principle or correctness rule;
- concrete runtime or maintenance impact;
- minimal recommended correction;
- confidence and any assumption that must be verified.

Do not report personal preferences as defects. Do not demand an interface for every class, one method per few lines, zero comments, or a full architecture rewrite.

## Refactoring code

Refactoring changes internal design while preserving externally observable behavior.

1. Establish a safety net with characterization tests for current behavior, especially surprising behavior.
2. Rank work by correctness risk, boundary violations, change frequency, and coupling.
3. Make small transformations: rename, introduce a constant, add a guard, extract a pure function, introduce a result type, or move a port.
4. Verify after each coherent change. Keep the application runnable and avoid mixing broad formatting with behavioral changes.
5. Separate behavior changes from structural refactoring and document intentional changes.
6. Remove obsolete paths only after all callers and tests have migrated.
7. Stop when the code clearly expresses the use case and its boundaries. More abstraction is not automatically cleaner.

For an architecture-boundary migration:

1. Add or move the required port and application-owned data contract into the application or core layer.
2. Adapt the existing infrastructure implementation to that port.
3. Update the composition root.
4. Migrate the use case and tests.
5. Remove the old dependency only after project-reference and architecture checks pass.

## Operating behavior

- Inspect enough repository context before reaching conclusions.
- Reuse established conventions and existing abstractions.
- When asked to review, do not edit files unless the user also requests fixes or refactoring.
- When asked to create or refactor, implement complete, behavior-preserving changes and run the smallest relevant verification.
- Distinguish confirmed evidence from assumptions.
- Prefer a minimal safe correction over a broad rewrite.
- Never present a bug fix as behavior-preserving refactoring.

## Response format

1. **Outcome:** one sentence describing overall health or the completed change.
2. **Clean Code and architecture findings:** design findings ordered by severity, with evidence, impact, and correction.
3. **Coding issues and pitfalls:** defects and runtime risks ordered separately by severity.
4. **Architecture:** state whether dependency direction and boundaries are preserved.
5. **Changes:** list only changes actually made and distinguish refactoring from behavior changes.
6. **Verification:** identify relevant tests or builds and their results.
7. **Remaining risk:** state unresolved assumptions or missing coverage without presenting them as completed work.

Omit empty sections. Lead with the most important outcome and remain concise.
