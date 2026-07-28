# Develop with FSD

[Русская версия](README.ru.md)

`develop-with-fsd` is a Codex skill for architecture-aware development with Feature-Sliced Design. It helps Codex place code in the correct owner, preserve dependency direction and public APIs, avoid speculative abstractions, and verify the repository's architectural contracts.

The skill is designed primarily for React, Next.js App Router, TypeScript, Redux Toolkit, and other modular frontend projects. Repository-specific rules always take precedence over its generic defaults.

## What it does

- reads the repository's `AGENTS.md` and architecture documentation first;
- inspects existing slices, aliases, lint rules, tests, and knowledge graphs;
- classifies code as `app`, `processes`, `widgets`, `features`, `entities`, or `shared`;
- keeps route files focused on routing and composition;
- preserves downward dependency flow and slice public APIs;
- avoids empty folders and abstractions created for hypothetical reuse;
- checks whether Server or Client Components are appropriate;
- runs relevant tests, type checks, lint, architecture checks, and builds;
- reports which verification commands actually ran.

Default extended dependency direction:

```text
app → processes → widgets → features → entities → shared
```

The `processes` layer is optional. The skill does not introduce it when a repository intentionally uses a different FSD variant.

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
    └── fsd-architecture.md
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

The skill may also be selected implicitly when Codex recognizes a matching architecture or FSD task.

## Decision model

The skill asks what owns each responsibility and places it in the lowest valid layer:

| Layer       | Responsibility                                                           |
| ----------- | ------------------------------------------------------------------------ |
| `app`       | Routes, layouts, metadata, route handlers, and thin composition          |
| `processes` | Store assembly, providers, router, i18n, and app-wide processes          |
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
- [`agents/openai.yaml`](agents/openai.yaml) — Codex interface metadata.
