# Conservative FSD architecture defaults

Use this reference only when the repository does not already define a conflicting architecture contract.

## Layer decisions

| Layer | Owns | Must not own |
|---|---|---|
| `app` | routes, layouts, metadata, route handlers, thin composition | reusable UI, domain models, large forms and editors |
| `processes` | store assembly, providers, router, i18n, application-wide processes | entity-specific business rules, reusable visual blocks |
| `widgets` | independent screen blocks and substantial page composition | generic primitives, entity source-of-truth models |
| `features` | user actions and business scenarios | complete pages, application infrastructure |
| `entities` | domain concepts, types, rules, selectors, small entity UI | URL knowledge, page layout, cross-entity scenarios |
| `shared` | domain-agnostic UI, base API, utilities, hooks, config, assets | business words, routes, features, widgets |

The `processes` layer is an optional project extension. Do not introduce it into a repository that intentionally omits it.

## Placement questions

Ask in order:

1. Is it required by the routing framework? Use `app`.
2. Does it assemble or configure the entire running application? Use `processes` if available.
3. Is it a complete, independently meaningful UI block? Use `widgets`.
4. Is it a user action expressed as a verb? Use `features`.
5. Is it knowledge about a business noun? Use `entities`.
6. Is it useful without knowing the product domain? Use `shared`.
7. If none fits, keep it local and clarify its responsibility before creating a new slice.

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
