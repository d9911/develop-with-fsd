# Architecture and code decomposition

Architecture is the set of modules, components, and relationships between them. Responsibilities, public contracts, state ownership, and interaction rules make that structure reviewable. An architectural component can contain several files or React components.

## Map the affected area

| Element | Record |
|---|---|
| Module/component | Name/path, responsibility, public API, authoritative state owner, server/client/tooling runtime, source location |
| Relationship | Source, target, type, direction, exchanged contract, error/cancellation rules where relevant, source location |

Relationship types include imports, calls, props/callbacks, events, state reads/writes, and network/storage I/O. An import graph establishes source dependencies; it does not prove a runtime call occurred. Trace real call sites and mark unverified edges. Keep a compact working map, update it after changes, and persist it only in an approved documentation location.

## Decompose by responsibility

1. Identify behavior and invariants that must remain stable.
2. Name independent reasons to change: presentation, workflow policy, domain rules, transport contract, or resource lifecycle. Keep code that changes together cohesive.
3. Find the existing owner and actual consumers. Extract a boundary for a concrete independent responsibility or invariant even with one consumer; similar markup alone does not establish shared semantics.
4. Define the smallest public inputs, outputs, errors, and permitted side effects. Hide irrelevant implementation details and internal mutable state.
5. Trace callers and verify ownership, dependencies, and server/client boundaries. Moving the same coupling into more files is insufficient.
6. When authorized to change code, extract incrementally and check public behavior with existing checks allowed by the user's test policy.

## Choose the construct

| Responsibility | Construct |
|---|---|
| Deterministic calculation, validation, selection, mapping | Pure function in its owning model/lib; explicit inputs and return value, no hidden I/O |
| React state, effects, subscriptions, lifecycle coordination | Focused custom hook with dependencies and cleanup |
| Rendering and interaction surface | Component with coherent props; children/slots for caller-supplied composition |
| Network, storage, clock, browser, third-party behavior | Adapter translating external values and failures into the consumer's contract |
| Stateful invariant or resource lifecycle | Encapsulated object, closure, or class with supported operations |
| Application assembly | Existing composition owner binds concrete adapters and installs providers |

FSD determines ownership; the construct expresses behavior. A calculation does not need a hook. A hook shares logic, not a state instance: separate calls have separate local state unless they read an explicitly shared store/context. Avoid replacing one overloaded component with one overloaded hook.

## State and side effects

- Name the authority for each value: server/query cache, URL, shared client state, or local editing state. Unsaved drafts can deliberately differ from server data.
- Derive views instead of synchronizing redundant copies unless editing or another concrete requirement needs a snapshot.
- Keep subscriptions, resource disposal, cancellation, and stale-response handling with their owner.
- Reuse existing session, error, retry, and validation policy across transport adapters. TypeScript assertions do not validate external JSON.
- Keep domain calculations usable without DOM/network when that boundary has value. Avoid pass-through wrappers created solely to simulate layering.

## OOP at a boundary

- Encapsulation: public operations preserve invariants and protect internal mutable state.
- Abstraction: expose the capability and behavior the consumer needs.
- Polymorphism: replacements preserve the same observable contract; see [SOLID audit](solid-audit.md).
- Inheritance: require an actual substitutable relationship. Prefer composition when sharing behavior without that relationship.

Modules and functions can provide useful encapsulation and abstraction. Choose classes for concrete invariants or lifecycles, not to satisfy a class count.

## Sources

Primary sources inspected during the 2026-10-31–2026-11-01 audit:

- [CMU SEI: Software Architecture](https://www.sei.cmu.edu/software-architecture/) — structure, behavior, relationships, qualities, tradeoffs.
- [FSD: Slices and segments](https://feature-sliced.design/docs/reference/slices-segments) — cohesion, independence, public APIs.
- [React: Reusing logic with custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks) — hook boundaries and independent local state per call.
- [Robert C. Martin: Single Responsibility](https://blog.cleancoder.com/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html) — reasons for change.

The construct table is practical synthesis for this skill. Repository contracts and verified behavior take precedence over generic preferences.
