# OOP and SOLID design, implementation, and audit

Use for OOP, SOLID, encapsulation, maintainability, and architecture work: design, implementation, refactoring, and review. Use these as design guidance, not packages to install. Start with the [module and relationship map](architecture-composition.md).

## Evidence

Read instructions, Git status, relevant diff, actual contracts, callers, and checks. Query existing graphs without rebuilding automatically, then verify edges in current source. Trace a representative user action through state and transport, including error, cancellation, session recovery, and validation boundaries.

Check upward imports, same-layer cross-slice imports, deep imports, and cycles separately. Read baseline exceptions and scanner coverage. A passing baseline check establishes its stated policy; it may allow existing debt. Erased type cycles do not establish a runtime cycle. Green TypeScript does not establish runtime validation or behavioral substitutability.

Audit-only work produces findings and proportionate existing checks. Refactors, new tests, new enforcement, or architecture migration require corresponding scope. Honor the user's test policy and budget.

## Review matrix

| Principle | Inspect | Insufficient evidence |
|---|---|---|
| SRP | Independent reasons to change mixed in one owner, such as workflow policy, transport, and rendering | Line count, number of hooks or functions alone |
| OCP | A real extension forces changes in unrelated stable owners; identify a useful extension seam | Every switch or union is not automatically a defect; fixed cases may need a simple switch |
| LSP | Implementations of one public contract preserve accepted inputs, outputs, errors, side effects, cancellation, and state invariants | Typecheck, `implements`, or inheritance presence/absence alone |
| ISP | Actual consumers depend on capabilities they do not use; inspect callers, no-op callbacks, and incompatible prop modes | A large props type alone |
| DIP | Domain policy depends on a stable capability, with concrete I/O bound by an adapter/composition owner | FSD direction, a constructor, or an interface alone; an imported function can already be a suitable seam |

These checks apply to function and module contracts as well as classes. React recommends function components for new code. Do not demand class migration to demonstrate OOP. If no substitution contract is inspected, mark LSP not applicable to that abstraction or insufficient evidence instead of claiming compliance.

## OOP checks

Assess these separately from the five SOLID principles; a functional/module design need not use every OOP mechanism.

| Mechanism | Inspect | Do not infer |
|---|---|---|
| Encapsulation | Identify the invariant and every mutation path, including setters, exported stores and returned references; public operations must preserve the invariant | A private field, getter or `readonly` annotation alone proves protection |
| Abstraction | Consumers see a coherent capability with explicit inputs, outcomes and errors; implementation details remain owned by the provider | Every function needs an interface, class or repository wrapper |
| Polymorphism | All actual replacements preserve the same behavioral contract, including failure, cancellation and side effects | A matching signature or successful mock proves production substitutability |
| Inheritance/composition | A subtype fulfills the parent contract; shared behavior without that relationship uses composition when useful | Inheritance is required for reuse, or composition requires another abstraction layer |

For encapsulation, inspect whether a getter exposes the original mutable collection or nested object. TypeScript `private` is enforced by the type checker; `readonly` does not freeze runtime values or guarantee deep immutability. Choose restricted operations, an appropriate snapshot, a closure or runtime private fields only when the invariant requires it; a shallow copy still shares nested values. These are design boundaries, not authorization controls. See [TypeScript classes](https://www.typescriptlang.org/docs/handbook/2/classes.html#caveats) and [readonly properties](https://www.typescriptlang.org/docs/handbook/2/objects.html#readonly-properties).

For example, an adapter that always throws “unsupported” for a save operation promised by its contract is not a valid replacement for that contract. Inspect actual promises and consumers before flagging it: a documented read-only capability is a different contract. Prefer separating capabilities over adding no-op methods to satisfy a broad interface.

## Implementation and refactoring workflow

1. Record the concrete behavior and invariants to preserve: accepted input, outputs, errors, validation, state transitions, side effects and cancellation where relevant.
2. Trace affected callers, exports, adapters, existing tests and fixtures. Identify responsibilities with independent reasons to change and the owner of each dependency.
3. Choose the smallest useful construct from [architecture-composition.md](architecture-composition.md). Keep a direct function when sufficient; introduce a capability boundary only for a concrete policy/detail separation or real variation.
4. Place named contracts in the owner's `types.ts`/`types/` under the [local typing rule](../SKILL.md#keep-typescript-contracts-inside-their-owner). Bind concrete I/O in the established composition owner. Keep existing public APIs and dependency direction intact; apply FSD slice rules where the repository uses FSD.
5. Change one coherent responsibility at a time and update every affected caller/export. Moving an oversized component into an equally overloaded hook does not establish SRP; adding an interface alone does not establish DIP.
6. Compare the resulting behavior with step 1 using the allowed existing checks; update existing tests where appropriate. Follow the user's new-test restrictions. Report unverified invariants instead of claiming equivalence from typecheck alone.

## Related checks

| Principle | Practical check | Guard against overengineering |
|---|---|---|
| Separation of concerns (SoC) | Rendering, workflow policy, domain rules and I/O have identifiable owners | Do not add a layer for every function |
| High cohesion / low coupling | Rules that change together stay together; dependencies expose supported contracts | File count and folder names do not prove independence |
| DRY | Reuse one source of a shared rule after comparing semantics and failure behavior | Similar markup or data shape alone does not justify consolidation |
| KISS / YAGNI | Choose the simplest design satisfying today's verified requirements | No speculative registries, DI containers, repositories or extension systems |
| Composition over inheritance | Combine capabilities without claiming a false subtype relationship | Do not replace a simple function with a hierarchy or wrapper chain |
| Single source of truth | Each value has an explicit authority; derive views where possible | Do not duplicate state by default or confuse an intentional unsaved draft with accidental duplication |

For React state ownership, see [Sharing State Between Components](https://react.dev/learn/sharing-state-between-components#a-single-source-of-truth-for-each-state).

For example, a scenario can accept `save: (draft: Draft, signal: AbortSignal) => Promise<SavedDraft>` and a concrete adapter can be selected by the caller. Use this boundary when it contains a real changing detail; do not introduce it solely for an extra abstraction.

## Report

For a full review, include separate results for the five SOLID principles, the four OOP rows, and the related checks above. For a narrow task, limit the report to its scope. Record each requested principle as supported by inspected evidence, debt found, not applicable to the inspected abstraction, or insufficient evidence. Each material finding needs severity, confidence, file/line, evidence, consequence, and smallest remediation direction. Use a compact summary: `principle | status | evidence/affected consumer | minimal action or verification gap`; do not mark unrelated code compliant. Separate design debt from a reproduced functional defect.

Report exact checks and exception policies, failures with established causes, and missing runtime/browser/service coverage. Distinguish stale fixtures from product defects using the real contract. Do not invent global compliance percentages.

## Sources

Primary sources inspected during the 2026-10-31–2026-11-01 audit:

- [React Component](https://react.dev/reference/react/Component) — function-component recommendation.
- [Robert C. Martin: SOLID Relevance](https://blog.cleancoder.com/uncle-bob/2020/10/18/Solid-Relevance.html) — principles beyond inheritance.
- [Robert C. Martin: Single Responsibility](https://blog.cleancoder.com/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html) — reasons to change.
- [FSD layers](https://feature-sliced.design/docs/reference/layers), [public APIs](https://feature-sliced.design/docs/reference/public-api) — isolation, cross-import APIs, cycles.

TypeScript and React references added for this expansion were checked on 2026-11-01. The frontend mapping and workflow are practical guidance, not a new FSD specification.
