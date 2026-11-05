---
name: develop-with-fsd
description: Use when planning, changing, or reviewing frontend code with Feature-Sliced Design, constants/type placement, or applying OOP, SOLID, DRY, KISS, YAGNI, responsibility, coupling, public APIs, and dependency boundaries in React, Next.js, TypeScript, or Redux Toolkit projects.
---

# Develop with FSD

Develop within the repository's actual architecture. Treat FSD as ownership and dependency rules, not as a requirement to create every possible folder.

Architecture is the set of modules, components, and relationships between them. Describe responsibilities, public contracts, state ownership, and interaction rules; folder names alone do not establish boundaries.

For decomposition and architecture work, read [architecture-composition.md](references/architecture-composition.md). For OOP/SOLID design, implementation, refactoring, or review, also read [solid-audit.md](references/solid-audit.md). These principles apply to existing modular code even without FSD; do not impose FSD folders as a prerequisite.

## Select the work mode

- **Review:** inspect the requested scope and produce evidence per principle; do not silently refactor.
- **Design/implementation:** identify owners, invariants, actual callers, and the smallest contracts before coding; apply the SOLID/OOP matrices to the affected boundaries.
- **Refactoring:** record observable behavior to preserve, trace every affected caller/export, change one coherent responsibility, and verify the contract with permitted checks. Preserve errors, cancellation, validation, and state ownership.

For a full architecture review, assess SRP, OCP, LSP, ISP, DIP; OOP encapsulation, abstraction, polymorphism, inheritance/composition; and related separation of concerns, cohesion/coupling, DRY, KISS/YAGNI, and state authority. For a narrow task, check only affected principles. Use explicit evidence or mark a principle not applicable/insufficiently inspected. SOLID is a set of design principles, OOP a programming paradigm, and FSD an architectural methodology; they are not interchangeable compliance formats.

## Establish the local contract

1. Read the nearest `AGENTS.md` and repository instructions.
2. Inspect architecture documentation, aliases, lint rules, tests, and existing neighboring slices.
3. Query a relevant fragment of an existing graph and verify its edges in current code. Do not install tools or build a full graph automatically.
4. Record the declared FSD version and its source, the framework router, physical paths for logical layers, and project-specific extensions before suggesting a structure. If the version is not declared, report 'version not established'; do not infer it from folder names or the presence of processes. Prefer the verified local contract.
5. For FSD placement work, if the repository has no FSD contract, read [references/fsd-architecture.md](references/fsd-architecture.md) and use its conservative defaults.
6. Use the [Next.js project structure template](references/next-project-template.md) only for a project that explicitly adopts that custom App Router/processes variant. It is not a universal FSD skeleton or evidence of a standard version. Read it completely when that variant is selected; optional folders remain optional.
7. State any unresolved ownership choice before making a change that depends on it.

Inspect scanner coverage, baselines, allowlists, and suppressions. A passing check proves its particular policy, not the absence of debt. Audit-only requests authorize inspection and proportionate existing checks; fixes require a request to change code.

## Classify ownership before editing

Keep a compact map of the affected area: module/component → responsibility, public API, state owner, runtime, source; relationship → type, direction, contract, source. Distinguish imports from runtime calls, props, events, state access, and I/O. Update the map after changes and save it only in an approved documentation location.

For every new responsibility, answer:

- What business concept or user action owns it?
- Which existing slice already owns related state, behavior, or UI?
- What is the highest layer that needs to know about it?
- Is it reusable now, or merely similar to something else?

Place code in the lowest valid owner:

- framework route files → paths required by the selected router; application assembly → logical FSD App, unless explicitly assigned to a local extension
- screen-owned UI and behavior → `pages` when used by the repository, otherwise its existing screen owner
- application-wide provider, store assembly, router, i18n, long-lived process → `processes` when the repository uses it
- independently meaningful screen block or page composition → `widgets`
- user action with business value → `features`
- domain data, rules, types, selectors, entity UI → `entities`
- domain-agnostic UI, API base, utilities, hooks, config → `shared`

Keep code local to its owner. Extract a boundary for a concrete independent responsibility, invariant, or real reuse; similar-looking code alone does not justify a shared abstraction. Do not create empty layers, placeholder segments, registries, factories, or wrappers for hypothetical reuse.

## Preserve dependency direction

First identify the architectural variant; never present a combined `app/page → processes` skeleton.

- **FSD v2.1 guidance:** logical layers `App → Pages → Widgets → Features → Entities → Shared`. App and Pages have different responsibilities and may coexist; layers are optional. v2.1 emphasizes page-owned UI/logic. The official migration guide says v2.0 projects remain valid; folder names do not establish a version.
- **Next.js mapping:** `app/**/page.tsx` is an App Router route file, not the FSD Pages layer. Next.js `pages/` is a router directory. Identify the router used for the affected route and map FSD layers separately. The official integration guide uses `_app`/`_pages` to avoid naming collisions; preserve existing mappings unless migration is requested.
- **Existing custom processes variant:** use `app → processes → widgets → features → entities → shared` only when repository documentation/checks establish that exact contract. Label it as a project extension; do not assign it a historical FSD version. In the standard, Processes is deprecated, not a synonym for Pages.

Sources: [v2.0 to v2.1](https://feature-sliced.design/docs/guides/migration/from-v2-0), [Next.js integration](https://feature-sliced.design/docs/guides/tech/with-nextjs), [layers](https://feature-sliced.design/docs/reference/layers).

Allow higher layers to depend on lower layers. Do not introduce reverse imports. Do not use aliases, barrels, dynamic imports, or type-only imports to conceal a forbidden dependency.

Import another slice through its public API when the repository uses public APIs. Keep private files private. Inside one slice, prefer direct relative imports when that matches the local convention and avoids self-import cycles.

Check same-layer cross-slice imports independently of upward imports. Slices are isolated by default; explicit entity relationships may use an `@x` API or a documented repository exception. App and Shared are segment-based exceptions. A public barrel does not authorize an otherwise forbidden dependency. Separate type-only cycles from value-import cycles and state scanner limits. Report differences from standard FSD; migrate an existing architecture only when requested.

## Implement the smallest coherent slice

Create only segments required by the current task:

```text
<slice>/
├── ui/
├── model/
├── api/
├── lib/
├── config/
└── index.ts
```

Do not create all segments by default. Keep route files thin. Keep domain business rules and higher-layer dependencies out of `shared`; business-themed assets/UI and generic route constants may be valid under the local contract. Keep URL-specific behavior out of `entities` and `shared`.

Prefer Server Components in Next.js App Router unless hooks, event handlers, browser APIs, context, Redux hooks, or a client-only library require `'use client'`. Place the client boundary as low as practical.

Use pure functions for deterministic rules, hooks for React state/effect coordination, components for UI, adapters for I/O, and encapsulated objects or classes for a concrete invariant or lifecycle. A hook shares logic, not a state instance. Avoid moving an overloaded component into an equally overloaded hook.

Keep production TSX focused. Follow repository limits when present; otherwise treat 400 lines as a review signal and avoid exceeding 500 lines without a documented generated-file, test, or report exception. A line count alone does not establish an SRP violation: identify independent reasons to change. Split by ownership or behavior, not by arbitrary line ranges.

## Keep constants and types separate

Mandatory Denis convention for frontend projects, including React/Next.js: **named static constants and named TypeScript contracts MUST have separate files**. Apply it when creating or changing a module. A requested project-wide migration includes its callers, public exports and existing checks.

| Definition | Required owner and location |
|---|---|
| Domain-independent shared values: locale, time zone, units of time, refresh settings | `shared/config/constants.ts` |
| Cohesive shared lookup/options: icons, date formats | `shared/config/icons.ts`, `shared/config/formatting.ts` when a separate file is useful |
| Entity tags, metric definitions, plans, domain thresholds | `entities/<slice>/config/constants.ts` |
| Feature/widget presets, tabs, navigation, validation limits | `<layer>/<slice>/config/constants.ts` |
| Types for that configuration, if named contracts exist | Its local `config/types.ts` |
| Props, domain and scenario contracts | Their owning `ui/types.ts` or `model/types.ts`, as specified below |
| Secret or environment-backed server configuration | Separate server-only config module; never export secrets through client/shared constant barrels |

Before implementation, classify each named module-level value as static configuration, behavior, derived service or mutable state. Move static tables, presets, defaults and options out of implementation `.ts`/`.tsx` and out of `types.ts`. Import one authoritative definition; update consumers and public exports. Keep business constants inside their slice: `shared/config` is not a global business-constants bucket.

The rule is about ownership, not the `const` keyword. Function-local calculations, closures, initial-state factories, mutable caches/stores and derived service instances remain with their behavior and lifecycle. Do not replace every literal or inferred value with an artificial alias. Do not create empty `config` or `types` files.

Framework/tooling entrypoints keep required exports and native file locations. For example, Next.js `dynamic`, `runtime` and `revalidate` retain literal values where static analysis requires them; metadata payloads may come from config. Preserve server/client boundaries and observable behavior.

```text
src/
├── shared/config/
│   ├── constants.ts        # shared static values
│   └── types.ts            # only when config owns named types
├── entities/gtu/
│   ├── config/constants.ts # GTU tags and metrics
│   └── model/types.ts      # GTU contracts
└── widgets/dashboard/
    ├── config/constants.ts # dashboard presets
    ├── ui/Dashboard.tsx
    └── ui/types.ts         # named props, if present
```

For a review, report misplaced definitions. For authorized edits, move them and verify imports/types; do not declare the convention enforced without an executable check. See [placement examples](references/fsd-architecture.md#constant-placement).

## Keep TypeScript contracts inside their owner

Mandatory project convention requested by Denis for TypeScript/Next.js: place named `type`/`interface` declarations, including component props, in `types.ts` or a `types/` directory **inside the owning component/module**, alongside its other implementation files. Do not leave these declarations scattered through implementation `.ts`/`.tsx` files or in loose files beside the project. This is a project convention, not a universal FSD or Next.js requirement.

- Use one local `types.ts` per folder when sufficient; use `types/` only when meaningful separation is needed. Follow the repository's file naming convention; do not create empty type containers.
- Domain contracts belong to their entity's `model/types.ts`; scenario contracts to their feature's `model/types.ts`; component-only props to local `ui/types.ts` or the component's own `types/`. Reuse the original contract instead of copying it between owners.
- Do not collect business types in a project-wide `types/` or `shared/types`. Only genuinely domain-independent utility types may have a shared owner.
- Respect the local route contract: in Timepost, `app`/`pages` entrypoints do not get separate type files; they consume contracts through the entry component/scenario's public API. Keep URL `params`/`searchParams` distinct from display props and preserve their framework-required async shape.
- Use `import type` and `export type`; cross-slice consumers use the owner's public API without bypassing layer rules.
- Inferred types, inline annotations, and generic parameters do not require artificial named declarations. Runtime schemas and behavior stay with their owner; static values follow the config rule above; named types inferred from schemas belong in the owner's type file, referencing the schema through a type-only import. Keep required/generated framework declaration files in their required locations.

Apply this rule to touched code; do not perform a project-wide type migration without a request. See [type placement examples](references/fsd-architecture.md#type-placement).

## Work safely

1. Inspect the working tree before editing.
2. Preserve unrelated and user-authored changes.
3. Make the narrowest change that satisfies the request.
4. Do not move existing code merely to make the tree look more canonical.
5. When migration is necessary, do it incrementally and keep behavior stable.
6. Do not add a new abstraction layer without an actual owner and consumer.

## Verify

Run the narrowest relevant checks first, then broader repository checks when proportionate:

- focused tests for changed behavior;
- TypeScript typecheck;
- lint and import-boundary checks;
- architecture/public-API tests;
- production build when routing, rendering boundaries, or configuration changed.

Honor the user's test policy and execution budget. Reuse existing focused checks for reviews; do not repeat successful checks without changes or new evidence. Update a repository graph after application code changes when required. Report results, exceptions, failures, and unverified behavior. Type checking alone does not prove runtime validation or substitutability.

## Review checklist

Before completion, confirm:

- every changed responsibility has one clear owner;
- dependency direction remains valid;
- cross-slice imports use the intended public API;
- named static values live in the owning `config` and stay separate from type declarations; shared values use `shared/config`;
- named TypeScript contracts live in the owner's `types.ts`/`types/`, with route/tooling exceptions respected;
- `shared` contains no domain business rules or higher-layer dependencies;
- route files contain composition rather than feature implementation;
- client boundaries and global state are justified;
- no speculative folders or abstractions were added;
- tests and documentation match the actual architecture.

For each audit finding provide severity, confidence, file/line, evidence, consequence, and minimal remediation. Evaluate each requested principle as supported, debt found, not applicable to the inspected abstraction, or insufficient evidence. Do not invent compliance percentages or classify every switch, large props type, or function component as a defect.
