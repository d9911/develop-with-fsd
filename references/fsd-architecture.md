# FSD version, framework mapping, and local variants

Use this reference only when the repository does not already define a conflicting architecture contract.

## Identify the variant first

Record `FSD version + source | framework/router | logical layer → physical path | local extensions/checks`. If no version is declared, say it is not established; do not guess from the tree.

FSD v2.1 uses a pages-first decomposition model; v2.0 projects remain valid according to the migration guide. App and Pages are distinct logical layers. Next.js `app/**/page.tsx` is a route file, and Next.js `pages/` is a router folder; neither name alone determines an FSD layer. The official Next.js integration guide maps logical layers to `_app` and `_pages` to avoid collisions. Keep a verified existing mapping during ordinary work.

A custom `app → processes → widgets → features → entities → shared` contract must be labelled as a project variant, not a numbered standard version. Processes is deprecated in standard FSD and does not replace Pages by definition. The table below lists possible owners across variants, not a skeleton to instantiate together.

Sources: [v2.1 migration](https://feature-sliced.design/docs/guides/migration/from-v2-0), [Next.js mapping](https://feature-sliced.design/docs/guides/tech/with-nextjs).

## Layer decisions

| Layer | Owns | Must not own |
|---|---|---|
| `app` | application assembly, providers, router configuration, global styles; physical router files follow the explicit framework mapping | page-local implementation and reusable domain models |
| `pages` | screen-owned UI, composition, data access and behavior when used | unrelated cross-page behavior |
| `processes` (only a verified local extension) | responsibilities explicitly assigned by the local contract | entity-specific business rules, reusable visual blocks |
| `widgets` | independent screen blocks and substantial page composition | generic primitives, entity source-of-truth models |
| `features` | user actions and business scenarios | complete pages, application infrastructure |
| `entities` | domain concepts, types, rules, selectors, small entity UI | URL knowledge, page layout, cross-entity scenarios |
| `shared` | domain-agnostic UI, base API, utilities, hooks, config, assets | domain business rules and higher-layer dependencies |

Standard FSD uses `app → pages → widgets → features → entities → shared`. The standard deprecates `processes`; keep an established project extension during ordinary work and do not migrate automatically. In Next.js App Router, follow the existing screen owner without accidentally introducing a Pages Router.

## Placement questions

Ask in order:

1. Is it required by the routing framework? Use the selected router's required path; do not infer its FSD owner from the directory name.
2. Does it assemble the application? Use `app`, or the existing documented `processes` owner.
3. Is it screen-owned composition? Use the existing page owner; reusable independent UI blocks may belong in `widgets`.
4. Is it a user action expressed as a verb? Use `features`.
5. Is it knowledge about a business noun? Use `entities`.
6. Is it useful without knowing the product domain? Use `shared`.
7. If none fits, keep it local and clarify its responsibility before creating a new slice.

## Type placement

Denis's mandatory TypeScript/Next.js project convention is defined in [the skill](../SKILL.md#keep-typescript-contracts-inside-their-owner); it is not a requirement of an FSD version. Named types/interfaces belong in `types.ts` or `types/` within their owner, alongside that owner's other files:

| Contract | Example location |
|---|---|
| Domain entity | `entities/project/model/types.ts` |
| User scenario | `features/create-project/model/types.ts` |
| Component props | `features/create-project/ui/types.ts` or `ui/CreateProjectForm/types.ts` |
| Several distinct local contracts | `ui/CreateProjectForm/types/props.ts`, `types/state.ts` within the same component directory |
| Domain-independent utility type | Existing shared library's `types.ts`, or `shared/types/` if the project uses it |

Choose one appropriate local form; do not scaffold every example. Import shared contracts from their owner using `import type`, and expose supported cross-slice contracts with `export type` in that owner's public API. Do not create a global collection of business types. Timepost route entrypoints obtain contracts through their component/scenario API, without separate files in `app`/`pages`. Inference, inline annotations and generic parameters remain inline; runtime schemas and required/generated framework declarations retain their proper locations.

## State placement

- Shareable filters and navigation state → URL/search params.
- Server-rendered data → Server Component or repository server-data mechanism.
- Remote client cache → existing query/cache solution.
- Global mutable client state → application store.
- Entity-owned state and selectors → entity model.
- One scenario's state → feature model.
- One component's transient state → local state.
- One form's input state → form state.
- Stable scoped configuration → context.
- Locale, theme, or session preference → repository-defined cookie/storage mechanism.

Do not move state into Redux merely because several components exist in the same file.

## Public APIs

A slice public API should export only its supported contract:

```ts
export { ProjectCard } from './ui/ProjectCard'
export { selectProjectById } from './model/selectors'
export type { Project, ProjectId } from './model/types'
```

Prefer:

```ts
import { ProjectCard } from '@/entities/project'
```

Avoid cross-slice deep imports:

```ts
import { ProjectCard } from '@/entities/project/ui/ProjectCard'
```

Do not create a repository-wide barrel that re-exports every slice. It obscures ownership and can create cycles.

## Slice shape

Create segments only when used:

```text
features/create-project/
├── ui/CreateProjectForm.tsx
├── model/schema.ts
├── api/create-project.ts
└── index.ts
```

A one-component slice may contain only `ui/Component.tsx` and `index.ts`. Do not scaffold empty `api`, `model`, `lib`, or `config` directories.

## Next.js App Router

- Keep `page.tsx` and `layout.tsx` focused on routing and composition.
- Use route-private `_components`, `_lib`, or `_actions` only for code owned exclusively by that route tree.
- Promote route-private code only when another real consumer appears.
- Keep server-only secrets and server APIs outside Client Components.
- Add `'use client'` only where interactivity requires it; do not propagate it upward without need.
- Keep route-specific Server Actions near their route or owning feature according to repository convention.

## Naming

- entities: business nouns, usually kebab-case slice names;
- features: action phrases such as `create-project`;
- widgets: meaningful UI blocks;
- React components: PascalCase;
- helpers/actions: follow the repository's established convention;
- tests: follow the configured runner's naming convention.

Avoid vague owners such as `common`, `misc`, `helpers`, `utils`, `data`, or `Block` when a precise responsibility can be named.

## Enforcement

A skill guides decisions but cannot enforce architecture by itself. For durable enforcement, add repository-owned checks such as:

- ESLint import-boundary rules;
- architecture tests for forbidden dependencies;
- public-API tests;
- circular-dependency checks;
- production TSX size contracts with explicit exceptions;
- CI steps that run these checks.

Introduce enforcement only when requested or already part of the repository contract.

## Review boundaries

Check same-layer cross-slice dependencies separately. Explicit entity relationships may use `@x`; App and Shared contain segments that may import each other. Business-themed UI/assets and generic route constants alone are not business logic. Read the exact baseline policy: passing may establish equality or only prevent debt growth. Neither import direction nor a barrel proves SOLID. See [solid-audit.md](solid-audit.md) and [architecture-composition.md](architecture-composition.md).

Primary references checked during this audit: [FSD layers](https://feature-sliced.design/docs/reference/layers), [public APIs](https://feature-sliced.design/docs/reference/public-api).
