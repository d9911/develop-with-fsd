---
name: develop-with-fsd
description: Guide architecture-aware development using Feature-Sliced Design. Use for planning, creating, changing, moving, or reviewing application code in repositories that use or are adopting FSD, especially React, Next.js App Router, TypeScript, Redux Toolkit, and modular frontend projects. Determine ownership, preserve dependency direction and public APIs, avoid speculative layers, and verify architectural contracts after changes.
---

# Develop with FSD

Develop within the repository's actual architecture. Treat FSD as ownership and dependency rules, not as a requirement to create every possible folder.

## Establish the local contract

1. Read the nearest `AGENTS.md` and repository instructions.
2. Inspect architecture documentation, aliases, lint rules, tests, and existing neighboring slices.
3. If a code knowledge graph exists, query it before broad source browsing.
4. Prefer the repository's documented FSD variant over generic conventions.
5. If the repository has no FSD contract, read [references/fsd-architecture.md](references/fsd-architecture.md) and use its conservative defaults.
6. When creating or planning a new Next.js App Router project with TypeScript, FSD, Redux Toolkit, and i18n, read the complete user-supplied [Next.js project structure template](references/next-project-template.md). Treat optional folders as optional and create them only when the project needs them.
7. State any unresolved ownership choice before making a change that depends on it.

Never claim that a convention is enforced unless a lint rule, test, or other executable check verifies it.

## Classify ownership before editing

For every new responsibility, answer:

- What business concept or user action owns it?
- Which existing slice already owns related state, behavior, or UI?
- What is the highest layer that needs to know about it?
- Is it reusable now, or merely similar to something else?

Place code in the lowest valid owner:

- route, layout, metadata, route handler, thin screen composition → `app`
- application-wide provider, store assembly, router, i18n, long-lived process → `processes` when the repository uses it
- independently meaningful screen block or page composition → `widgets`
- user action with business value → `features`
- domain data, rules, types, selectors, entity UI → `entities`
- domain-agnostic UI, API base, utilities, hooks, config → `shared`

Keep code local to its current slice until a second real consumer justifies promotion. Do not create empty layers, placeholder segments, registries, factories, or wrappers for hypothetical reuse.

## Preserve dependency direction

Use the repository's declared dependency graph. With the extended hierarchy, default to:

```text
app → processes → widgets → features → entities → shared
```

Allow higher layers to depend on lower layers. Do not introduce reverse imports. Do not use aliases, barrels, dynamic imports, or type-only imports to conceal a forbidden dependency.

Import another slice through its public API when the repository uses public APIs. Keep private files private. Inside one slice, prefer direct relative imports when that matches the local convention and avoids self-import cycles.

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

Do not create all segments by default. Keep route files thin. Keep domain language out of `shared`. Keep URL-specific behavior out of `entities` and `shared`.

Prefer Server Components in Next.js App Router unless hooks, event handlers, browser APIs, context, Redux hooks, or a client-only library require `'use client'`. Place the client boundary as low as practical.

Keep production TSX focused. Follow repository limits when present; otherwise treat 400 lines as a review signal and avoid exceeding 500 lines without a documented generated-file, test, or report exception. Split by ownership or behavior, not by arbitrary line ranges.

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

If a knowledge graph is part of the repository workflow, update it after code changes. Report exactly which checks ran and which did not run.

## Review checklist

Before completion, confirm:

- every changed responsibility has one clear owner;
- dependency direction remains valid;
- cross-slice imports use the intended public API;
- `shared` contains no project business knowledge;
- route files contain composition rather than feature implementation;
- client boundaries and global state are justified;
- no speculative folders or abstractions were added;
- tests and documentation match the actual architecture.
