# Разработка с FSD

[English version](README.md)

`develop-with-fsd` — skill для Codex, который помогает разрабатывать проект с учётом архитектуры Feature-Sliced Design. Он определяет владельца кода, сохраняет направление зависимостей и публичные API, не создаёт преждевременные абстракции и проверяет архитектурные контракты репозитория.

Skill рассчитан прежде всего на React, Next.js App Router, TypeScript, Redux Toolkit и другие модульные frontend-проекты. Правила конкретного репозитория всегда имеют приоритет над универсальными рекомендациями.

## Что делает skill=-0
- применяет SOLID и ООП при проектировании, реализации, рефакторинге и аудите; проверяет DRY, KISS/YAGNI, связность, зависимости и владельцев состояния без обязательного перехода на классы;
- сначала читает `AGENTS.md` и архитектурную документацию репозитория;
- проверяет существующие slices, aliases, lint-правила, тесты и граф знаний;
- распределяет код между `app`, `pages`, `widgets`, `features`, `entities` и `shared`, сохраняя существующее расширение `processes`;
- оставляет route-файлы ответственными за маршрутизацию и композицию;
- сохраняет нисходящее направление зависимостей и публичные API slices;
- не создаёт пустые папки и абстракции для гипотетического переиспользования;
- определяет, где нужны Server Components и Client Components;
- запускает подходящие тесты, typecheck, lint, архитектурные проверки и сборку;
- сообщает, какие проверки действительно были выполнены.

Логические слои FSD v2.1: `App → Pages → Widgets → Features → Entities → Shared`. App и Pages имеют разные роли. Каталоги роутеров и route-файлы Next.js сопоставляются со слоями отдельно.

Вложенный шаблон — проектный вариант App Router/processes, а не универсальный скелет FSD. Сначала фиксируются заявленная версия и источник; при их отсутствии — «версия не установлена». См. [выбор варианта](references/fsd-architecture.md).

## Установка

Разместите каталог skill в глобальной папке Codex:

```text
~/.codex/skills/develop-with-fsd/
```

Обязательные и поставляемые файлы:

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

`README.md` и `README.ru.md` документируют проект, но не требуются для выполнения skill.

## Использование

Вызовите skill явно:

```text
$develop-with-fsd реализуй создание проекта
```

Другие примеры:

```text
$develop-with-fsd проверь эти изменения на нарушения FSD
```

```text
$develop-with-fsd определи, где должно храниться это Redux-состояние
```

```text
$develop-with-fsd раздели этот большой TSX-компонент без изменения поведения
```

```text
$develop-with-fsd проверь SOLID, ООП, DRY и владельцев состояния; приведи доказательства по каждому принципу
```

```text
$develop-with-fsd выполни рефакторинг этого сценария по SOLID с сохранением поведения
```

Skill также может выбираться автоматически, если Codex распознает подходящую архитектурную или FSD-задачу.

## Модель принятия решений

Skill определяет владельца каждой ответственности и размещает код в самом нижнем допустимом слое:

| Слой        | Ответственность                                                       |
| ----------- | --------------------------------------------------------------------- |
| `app` | Логическая сборка приложения; пути файлов роутера определяются выбранным сопоставлением |
| `pages` | UI и поведение экрана, если слой используется |
| `processes` | Существующее документированное расширение проекта |
| `widgets`   | Самостоятельные блоки экрана и значительная композиция страницы       |
| `features`  | Действия пользователя с бизнес-ценностью                              |
| `entities`  | Данные, правила, типы, selectors и UI бизнес-сущностей                |
| `shared`    | Независимые от бизнеса UI, базовый API, утилиты, hooks и конфигурация |

Код остаётся локальным, пока реальный второй потребитель не обоснует перенос к более общему владельцу.

## Важное ограничение

Skill направляет решения Codex, но сам по себе не обеспечивает принудительное соблюдение архитектуры. Для надёжного контроля нужны механизмы репозитория: ESLint-правила границ импортов, архитектурные тесты, проверки публичных API, поиск циклических зависимостей и CI.

Skill не утверждает, что правило контролируется автоматически, если это не подтверждено исполняемой проверкой репозитория.

## Файлы

- [`SKILL.md`](SKILL.md) — исполняемые инструкции для Codex.
- [`references/fsd-architecture.md`](references/fsd-architecture.md) — консервативные правила для репозиториев без собственного FSD-контракта.
- [`references/next-project-template.md`](references/next-project-template.md) — полная предоставленная пользователем структура проекта на Next.js App Router, TypeScript, FSD, Redux Toolkit и i18n.
- [`agents/openai.yaml`](agents/openai.yaml) — метаданные интерфейса Codex.

## Architecture / Архитектура

Архитектура — набор модулей, компонентов и связей между ними. Architecture is the set of modules, components, and relationships between them.

- [Architecture and decomposition](references/architecture-composition.md): responsibilities, contracts, state ownership, functions/hooks/components/adapters/classes.
- [OOP and SOLID audit](references/solid-audit.md): criteria, evidence, baseline exceptions, and limits of verification.

The existing project contract takes precedence; these criteria do not require class migration.
