# Develop with FSD

[Русская версия](README.ru.md)

`develop-with-fsd` is a Codex skill for architecture-aware development with Feature-Sliced Design. It helps Codex place code in the correct owner, preserve dependency direction and public APIs, avoid speculative abstractions, and verify the repository's architectural contracts.

The skill is designed primarily for React, Next.js App Router, TypeScript, Redux Toolkit, and other modular frontend projects. Repository-specific rules always take precedence over its generic defaults.

## What it does

- applies SOLID and OOP during design, implementation, refactoring, and review; checks DRY, KISS/YAGNI, cohesion, coupling, and state ownership without requiring classes;
- reads the repository's `AGENTS.md` and architecture documentation first;
- inspects existing slices, aliases, lint rules, tests, and knowledge graphs;
- classifies code as `app`, `pages`, `widgets`, `features`, `entities`, or `shared`, preserving an existing `processes` extension;
- requires constants in the owning `config` (`shared/config` for shared technical values) and named contracts in separate local `types.ts`; creates these files only when needed;
- keeps route files focused on routing and composition;
- preserves downward dependency flow and slice public APIs;
- avoids empty folders and abstractions created for hypothetical reuse;
- checks whether Server or Client Components are appropriate;
- runs relevant tests, type checks, lint, architecture checks, and builds;
- reports which verification commands actually ran.

FSD v2.1 logical layers: `App → Pages → Widgets → Features → Entities → Shared`. App and Pages have distinct roles. Next.js router folders and route files are mapped separately.

The bundled template is a custom App Router/processes variant, not a universal FSD skeleton. Record the declared version and source first; when absent, report “version not established”. See [variant selection](references/fsd-architecture.md).

## Installation

Place the skill directory in your global Codex skills folder:

```text
~/.codex/skills/develop-with-fsd/
```

Required and bundled files:

```text
develop-with-fsd/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── architecture-composition.md
    ├── solid-audit.md
    ├── fsd-architecture.md
    └── next-project-template.md
```

`README.md` and `README.ru.md` document the project but are not required for skill execution.

## Usage

Invoke the skill explicitly:

```text
$develop-with-fsd implement project creation
```

Other examples:

```text
$develop-with-fsd review this change for FSD violations
```

```text
$develop-with-fsd decide where this Redux state should live
```

```text
$develop-with-fsd split this large TSX component without changing behavior
```

```text
$develop-with-fsd audit SOLID, OOP, DRY and state ownership; report evidence per principle
```

```text
$develop-with-fsd refactor this workflow using SOLID while preserving behavior
```

The skill may also be selected implicitly when Codex recognizes a matching architecture or FSD task.

## Decision model

The skill asks what owns each responsibility and places it in the lowest valid layer:

| Layer       | Responsibility                                                           |
| ----------- | ------------------------------------------------------------------------ |
| `app` | Logical application assembly; framework router paths follow the selected mapping |
| `pages` | Screen-owned UI and behavior when used |
| `processes` | Existing documented project extension for app-wide processes |
| `widgets`   | Independent screen blocks and substantial page composition               |
| `features`  | User actions with business value                                         |
| `entities`  | Domain data, rules, types, selectors, and entity UI                      |
| `shared`    | Domain-agnostic UI, API foundations, utilities, hooks, and configuration |

Code remains local until a real second consumer justifies promotion to a broader owner.

## Important limitation

A skill guides Codex's decisions; it does not enforce architecture by itself. Durable enforcement requires repository-owned mechanisms such as ESLint import-boundary rules, architecture tests, public API tests, circular-dependency checks, and CI.

The skill never claims that a convention is enforced unless an executable repository check verifies it.

## Files

- [`SKILL.md`](SKILL.md) — executable instructions for Codex.
- [`references/fsd-architecture.md`](references/fsd-architecture.md) — conservative defaults for repositories without their own FSD contract.
- [`references/next-project-template.md`](references/next-project-template.md) — complete user-supplied Next.js App Router, TypeScript, FSD, Redux Toolkit, and i18n project structure.
- [`agents/openai.yaml`](agents/openai.yaml) — Codex interface metadata.

## Architecture / Архитектура

Архитектура — набор модулей, компонентов и связей между ними. Architecture is the set of modules, components, and relationships between them.

- [Architecture and decomposition](references/architecture-composition.md): responsibilities, contracts, state ownership, functions/hooks/components/adapters/classes.
- [OOP and SOLID audit](references/solid-audit.md): criteria, evidence, baseline exceptions, and limits of verification.

The existing project contract takes precedence; these criteria do not require class migration.
