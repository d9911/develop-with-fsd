# Шаблон конкретного проектного варианта

Это предоставленный проектный шаблон с расширением `processes`, а не универсальный стандарт FSD. Номер версии FSD в исходном шаблоне не установлен. Применять только при явном выборе этого варианта; существующие проекты автоматически не перестраивать.

`src/app/**/page.tsx` здесь — маршруты Next.js App Router. Отсутствие отдельного слоя FSD Pages — локальное решение. App Router допускает отдельный слой экранов с безопасным именем, например `_pages`; логические App и Pages в FSD не являются альтернативными роутерами. `processes` не синоним Pages.

Источники: [FSD и Next.js](https://feature-sliced.design/docs/guides/tech/with-nextjs), [слои FSD](https://feature-sliced.design/docs/reference/layers). Ниже сохранён выбранный проектный вариант, а не инструкция смешивать разные схемы.

Правило типизации Дениса: именованные `type`/`interface`, включая props, хранить в `types.ts` или `types/` внутри модуля/компонента-владельца, рядом с его остальными файлами. Общего хранилища бизнес-типов проекта нет. Файлы маршрутов получают контракты через публичный API владельца; исключения для вывода типов и служебных деклараций — в [SKILL.md](../SKILL.md#keep-typescript-contracts-inside-their-owner).

Обязательное правило констант Дениса: общие технические значения хранятся в `src/shared/config/constants.ts`, связанные таблицы/параметры — в отдельных файлах `shared/config/`. Предметные константы принадлежат `config/constants.ts` своего entity/feature/widget. Константы и именованные типы не смешиваются: для типов конфигурации при необходимости есть локальный `config/types.ts`, для props — `ui/types.ts`. Пустые файлы не создаются. Вычисления внутри функций, изменяемое состояние и обязательные literal-экспорты Next.js остаются на месте. См. [полное правило](../SKILL.md#keep-constants-and-types-separate).

next-project/ # Проектный вариант: Next.js App Router + processes + TypeScript + Redux Toolkit + i18n.
│   # app → processes → widgets → features → entities → shared
│   #
│   # ВАЖНО:
│   # ├── src/app      — только маршрутизация и специальные файлы Next.js;
│   # ├── store        — в src/processes/store, а не в src/app;
│   # ├── pages        — отдельный FSD-слой отсутствует по решению этого варианта, не из-за запрета App Router;
│   # ├── shared       — не знает о бизнес-логике;
│   # └── папки создаём только тогда, когда они реально нужны проекту.
│
├── .github/                                  # GitHub-инфраструктура и автоматизация
│   ├── workflows/
│   │   ├── ci.yml                            # lint → typecheck → test → build
│   │   ├── pull-request.yml                  # Проверки Pull Request
│   │   ├── deploy-preview.yml                # Развёртывание preview-окружения
│   │   ├── deploy-production.yml             # Развёртывание production
│   │   └── security.yml                      # Аудит зависимостей и безопасности
│   ├── ISSUE_TEMPLATE/                       # Шаблоны задач и баг-репортов
│   ├── PULL_REQUEST_TEMPLATE.md              # Чек-лист Pull Request
│   ├── CODEOWNERS                            # Ответственные за части проекта
│   └── dependabot.yml                        # Автоматическое обновление зависимостей
│
├── .husky/                                   # Git hooks; создавать только при Husky
│   ├── pre-commit                            # lint-staged перед коммитом
│   ├── pre-push                              # Быстрые тесты перед push
│   └── commit-msg                            # Проверка формата commit message
│
├── .storybook/                               # Конфигурация Storybook; опционально
│   ├── main.ts                               # Stories, addons, aliases
│   ├── preview.ts                            # Глобальные decorators и стили
│   └── manager.ts                            # Настройка интерфейса Storybook
│
├── docker/                                   # Расширенная Docker-инфраструктура
│   ├── development/
│   │   └── Dockerfile                        # Контейнер для локальной разработки
│   ├── production/
│   │   └── Dockerfile                        # Минимальный production image
│   └── nginx/
│       └── nginx.conf                        # Reverse proxy и кеширование статики
│
├── docs/                                     # Документация проекта
│   ├── architecture/
│   │   ├── README.md                         # Обзор архитектуры и слоёв
│   │   ├── dependency-rules.md               # Разрешённые направления импортов
│   │   ├── folder-structure.md               # Правила выбора папки для нового кода
│   │   └── decisions/                        # Architecture Decision Records
│   ├── api/                                  # Описание backend API и контрактов
│   ├── deployment/                           # Инструкции по развёртыванию
│   ├── testing/                              # Стратегия unit/integration/e2e
│   └── i18n/                                 # Правила работы с переводами
│
├── e2e/                                      # Сквозные браузерные тесты
│   ├── auth/
│   │   ├── login.spec.ts
│   │   ├── registration.spec.ts
│   │   └── logout.spec.ts
│   ├── projects/
│   ├── dashboard/
│   ├── fixtures/                             # Тестовые данные и mock-файлы
│   ├── pages/                                # Page Objects; только если оправданы
│   └── global.setup.ts                       # Авторизация и подготовка окружения
│
├── public/                                   # Статика с прямым публичным URL
│   │                                         # Например public/logo.svg → /logo.svg
│   ├── fonts/                                # Публичные шрифты
│   ├── icons/                                # Favicon и PWA-иконки
│   ├── images/                               # Публичные изображения
│   ├── downloads/                            # Документы для скачивания
│   ├── locales/                              # Только при HTTP-загрузке переводов
│   ├── favicon.ico
│   ├── manifest.webmanifest
│   └── robots.txt                            # Не нужен, если используется app/robots.ts
│
├── scripts/                                  # Инфраструктурные скрипты вне runtime
│   ├── build/
│   │   ├── analyze-bundle.mjs                # Анализ размера bundle
│   │   └── verify-build.mjs                  # Проверка production-сборки
│   ├── deployment/                           # Подготовка release/deploy
│   ├── generators/
│   │   ├── create-entity.mjs                 # Генератор entities/<slice>
│   │   ├── create-feature.mjs                # Генератор features/<slice>
│   │   └── create-widget.mjs                 # Генератор widgets/<slice>
│   ├── i18n/
│   │   ├── check-dictionaries.mjs            # Проверка одинаковых ключей языков
│   │   └── find-unused-keys.mjs              # Поиск неиспользуемых переводов
│   ├── migrations/                           # Одноразовые миграции кода/данных
│   └── visual-regression/                    # Создание и сравнение screenshots
│
├── src/ # Слои внутри src:
│   │   # app       — URL, layouts и специальные файлы Next.js;
│   │   # processes — store, providers, i18n и глобальные процессы;
│   │   # widgets   — крупные самостоятельные блоки интерфейса;
│   │   # features  — действия пользователя;
│   │   # entities  — бизнес-сущности;
│   │   # shared    — универсальная техническая база.
│   │
│   ├── app/     # Next.js App Router
│   │   │   # В app разрешено:
│   │   │   # ├── page.tsx, layout.tsx, loading.tsx;
│   │   │   # ├── error.tsx, not-found.tsx, route.ts;
│   │   │   # ├── metadata, sitemap, robots, manifest;
│   │   │   # ├── тонкая композиция готовых widgets;
│   │   │   # └── route-specific Server Actions.
│   │   │   # В app не размещаем:
│   │   │   # ├── Redux store и бизнесовые slices;
│   │   │   # ├── огромные формы и таблицы;
│   │   │   # ├── общие API-клиенты;
│   │   │   # ├── универсальные компоненты;
│   │   │   # └── сложную бизнес-логику.
│   │   │
│   │   ├── (marketing)/                      # Route Group; имени нет в URL
│   │   │   │                                 # Публичная маркетинговая часть
│   │   │   ├── layout.tsx                    # Header/Footer маркетинга
│   │   │   ├── page.tsx                      # URL: /
│   │   │   ├── about/
│   │   │   │   └── page.tsx                  # URL: /about
│   │   │   ├── pricing/
│   │   │   │   ├── loading.tsx               # Skeleton только для /pricing
│   │   │   │   └── page.tsx                  # URL: /pricing
│   │   │   └── blog/
│   │   │       ├── page.tsx                  # URL: /blog
│   │   │       └── [slug]/                   # Динамический сегмент
│   │   │           ├── page.tsx              # URL: /blog/:slug
│   │   │           ├── loading.tsx
│   │   │           ├── error.tsx
│   │   │           └── not-found.tsx
│   │   │
│   │   ├── (auth)/                           # Route Group авторизации
│   │   │   ├── layout.tsx                    # Общая оболочка auth-страниц
│   │   │   ├── login/
│   │   │   │   └── page.tsx                  # Подключает feature login
│   │   │   ├── registration/
│   │   │   │   └── page.tsx
│   │   │   ├── forgot-password/
│   │   │   │   └── page.tsx
│   │   │   └── reset-password/
│   │   │       └── page.tsx
│   │   │
│   │   ├── (application)/                    # Закрытая часть приложения
│   │   │   ├── layout.tsx                    # Auth check + App Shell
│   │   │   │
│   │   │   ├── dashboard/
│   │   │   │   ├── layout.tsx                # Layout только dashboard
│   │   │   │   ├── loading.tsx
│   │   │   │   ├── error.tsx
│   │   │   │   └── page.tsx
│   │   │   │
│   │   │   ├── projects/
│   │   │   │   ├── page.tsx                  # URL: /projects
│   │   │   │   ├── create/
│   │   │   │   │   └── page.tsx              # URL: /projects/create
│   │   │   │   └── [projectId]/
│   │   │   │       ├── layout.tsx
│   │   │   │       ├── page.tsx              # URL: /projects/:projectId
│   │   │   │       ├── settings/
│   │   │   │       │   └── page.tsx
│   │   │   │       └── publications/
│   │   │   │           ├── page.tsx
│   │   │   │           └── [publicationId]/
│   │   │   │               └── page.tsx
│   │   │   │
│   │   │   ├── profile/
│   │   │   │   └── page.tsx
│   │   │   └── settings/
│   │   │       ├── layout.tsx
│   │   │       ├── page.tsx
│   │   │       ├── account/
│   │   │       │   └── page.tsx
│   │   │       ├── notifications/
│   │   │       │   └── page.tsx
│   │   │       └── billing/
│   │   │           └── page.tsx
│   │   │
│   │   ├── [locale]/                         # ВАРИАНТ i18n №1: locale в URL
│   │   │   │                                 # Примеры: /ru/projects, /en/projects
│   │   │   │                                 # Использовать для SEO и публичных сайтов
│   │   │   ├── layout.tsx
│   │   │   └── ...
│   │   │
│   │   │   # ВАРИАНТ i18n №2:
│   │   │   # Не создавать [locale], а хранить язык в cookie.
│   │   │   # Подходит для кабинетов, CRM и внутренних систем.
│   │   │   #
│   │   │   # ВАРИАНТ i18n №3:
│   │   │   # Публичные страницы используют [locale],
│   │   │   # закрытый кабинет использует cookie.
│   │   │
│   │   ├── @modal/                            # Parallel Route для модальных страниц
│   │   │   ├── default.tsx                   # Fallback параллельного маршрута
│   │   │   └── (.)projects/[projectId]/
│   │   │       └── page.tsx                  # Intercepting Route для модалки
│   │   │
│   │   ├── api/                               # Route Handlers / BFF
│   │   │   │   # Использовать для:
│   │   │   │   # ├── proxy к backend;
│   │   │   │   # ├── скрытия server-only токенов;
│   │   │   │   # ├── webhook endpoints;
│   │   │   │   # ├── OAuth callback;
│   │   │   │   # ├── upload/download;
│   │   │   │   # └── преобразования backend response.
│   │   │   │
│   │   │   ├── auth/
│   │   │   │   ├── login/
│   │   │   │   │   └── route.ts              # POST /api/auth/login
│   │   │   │   ├── logout/
│   │   │   │   │   └── route.ts
│   │   │   │   └── refresh/
│   │   │   │       └── route.ts
│   │   │   ├── health/
│   │   │   │   └── route.ts                  # Healthcheck
│   │   │   ├── upload/
│   │   │   │   └── route.ts                  # Загрузка файлов
│   │   │   ├── webhooks/
│   │   │   │   └── payment/
│   │   │   │       └── route.ts              # Webhook платёжной системы
│   │   │   └── proxy/
│   │   │       └── [[...path]]/
│   │   │           └── route.ts              # Optional catch-all proxy
│   │   │
│   │   ├── _components/                      # Компоненты только этого route tree
│   │   │                                     # Не заменяют widgets/features/shared
│   │   ├── _lib/                             # Локальные route-level helpers
│   │   ├── _actions/                         # Общие Server Actions app-уровня
│   │   │
│   │   ├── layout.tsx                        # Обязательный корневой layout
│   │   │                                     # Подключает AppProviders
│   │   ├── template.tsx                      # Пересоздаётся при навигации
│   │   ├── loading.tsx                       # Глобальный Suspense fallback
│   │   ├── error.tsx                         # Ошибка текущего route tree
│   │   ├── global-error.tsx                  # Ошибка корневого layout
│   │   ├── not-found.tsx                     # Глобальная 404
│   │   ├── forbidden.tsx                     # UI для 403; зависит от версии Next
│   │   ├── unauthorized.tsx                  # UI для 401; зависит от версии Next
│   │   ├── sitemap.ts                        # Генерация sitemap.xml
│   │   ├── robots.ts                         # Генерация robots.txt
│   │   ├── manifest.ts                       # Web App Manifest
│   │   ├── icon.png                          # Иконка приложения
│   │   ├── opengraph-image.tsx               # Динамическое OG-изображение
│   │   └── globals.css                       # Только глобальные стили
│   │
│   ├── processes/                            # Проектное расширение FSD
│   │   │                                     # Сквозная инфраструктура приложения
│   │   │   # Здесь находится то, что:
│   │   │   # ├── связывает всё приложение;
│   │   │   # ├── живёт дольше одного экрана;
│   │   │   # ├── не принадлежит одной entity/feature;
│   │   │   # └── подключается на уровне root layout.
│   │   │
│   │   ├── store/                            # Redux Toolkit находится ЗДЕСЬ
│   │   │   │
│   │   │   │   # Redux используем для:
│   │   │   │   # ├── глобального изменяемого client-state;
│   │   │   │   # ├── сложных редакторов и черновиков;
│   │   │   │   # ├── optimistic updates;
│   │   │   │   # ├── offline queue;
│   │   │   │   # └── состояния, общего для удалённых частей UI.
│   │   │   │
│   │   │   │   # Не кладём в Redux:
│   │   │   │   # ├── server-only данные;
│   │   │   │   # ├── секреты;
│   │   │   │   # ├── params/searchParams;
│   │   │   │   # ├── простое локальное состояние;
│   │   │   │   # └── статические конфиги и словари.
│   │   │   │
│   │   │   ├── store.ts                      # makeStore(); фабрика на запрос
│   │   │   │                                 # Не экспортировать общий server singleton
│   │   │   ├── rootReducer.ts                # Собирает reducers всех slices
│   │   │   ├── middleware.ts                 # Общие Redux middleware
│   │   │   ├── listenerMiddleware.ts         # Реакция на Redux actions
│   │   │   ├── hooks.ts                      # useAppDispatch/useAppSelector/useAppStore
│   │   │   ├── types.ts                      # AppStore/RootState/AppDispatch
│   │   │   ├── StoreProvider.tsx             # Client Component + Redux Provider
│   │   │   ├── StoreBootstrap.tsx            # Начальная инициализация client-state
│   │   │   ├── persistence/
│   │   │   │   ├── config.ts                 # redux-persist config
│   │   │   │   ├── storage.ts                # Безопасный доступ к browser storage
│   │   │   │   ├── migrations.ts             # Миграции старого persisted state
│   │   │   │   └── transforms.ts             # Сериализация Date/Set/особых полей
│   │   │   ├── slices/
│   │   │   │   │
│   │   │   │   │   # Только глобальные или legacy slices.
│   │   │   │   │   # Бизнесовые slices должны жить у владельца:
│   │   │   │   │   # entities/user/model/slice.ts
│   │   │   │   │   # features/filter-projects/model/slice.ts
│   │   │   │   │
│   │   │   │   ├── app-session/              # Глобальная client-сессия
│   │   │   │   ├── network/                  # online/offline
│   │   │   │   └── global-ui/                # Глобальные dialogs/toasts/sidebar
│   │   │   └── index.ts                      # Публичный API store
│   │   │
│   │   ├── providers/
│   │   │   ├── AppProviders.tsx              # Композиция всех providers
│   │   │   ├── StoreProvider.tsx             # Реэкспорт Redux Provider; опционально
│   │   │   ├── I18nProvider.tsx              # Контекст переводов
│   │   │   ├── ThemeProvider.tsx             # Тема приложения
│   │   │   ├── QueryProvider.tsx             # TanStack Query; если используется
│   │   │   ├── AnalyticsProvider.tsx         # Аналитика
│   │   │   ├── ErrorTrackingProvider.tsx     # Sentry/аналог
│   │   │   ├── FeatureFlagsProvider.tsx      # Feature flags
│   │   │   └── index.ts
│   │   │
│   │   ├── i18n/
│   │   │   ├── config.ts                     # Настройка i18n; locale импортируются из shared/config/constants.ts
│   │   │   ├── types.ts                      # Locale/Dictionary/TranslationKey
│   │   │   ├── server.ts                     # Получение словаря в Server Component
│   │   │   ├── client.ts                     # Client translation adapter
│   │   │   ├── navigation.ts                 # Locale-aware Link/router
│   │   │   ├── locale-resolver.ts            # URL → cookie → header → default
│   │   │   ├── storage.ts                    # Сохранение языка в cookie/storage
│   │   │   ├── dictionaries/
│   │   │   │   │
│   │   │   │   │   # Делим словарь по namespace,
│   │   │   │   │   # а не создаём один огромный ru.ts.
│   │   │   │   │
│   │   │   │   ├── ru/
│   │   │   │   │   ├── common.ts             # Общие тексты
│   │   │   │   │   ├── validation.ts         # Ошибки валидации
│   │   │   │   │   ├── auth.ts
│   │   │   │   │   ├── dashboard.ts
│   │   │   │   │   ├── projects.ts
│   │   │   │   │   ├── publications.ts
│   │   │   │   │   ├── settings.ts
│   │   │   │   │   └── errors.ts
│   │   │   │   ├── en/
│   │   │   │   │   ├── common.ts
│   │   │   │   │   ├── validation.ts
│   │   │   │   │   ├── auth.ts
│   │   │   │   │   ├── dashboard.ts
│   │   │   │   │   ├── projects.ts
│   │   │   │   │   ├── publications.ts
│   │   │   │   │   ├── settings.ts
│   │   │   │   │   └── errors.ts
│   │   │   │   └── index.ts                  # Сборка словарей
│   │   │   └── index.ts                      # Публичный API i18n
│   │   │
│   │   ├── auth/
│   │   │   ├── server/
│   │   │   │   ├── get-session.ts            # Получение server-сессии
│   │   │   │   ├── require-session.ts        # Проверка авторизации
│   │   │   │   └── permissions.ts            # Проверка ролей и прав
│   │   │   ├── client/
│   │   │   │   └── AuthProvider.tsx          # Client auth-context при необходимости
│   │   │   └── index.ts
│   │   │
│   │   ├── router/
│   │   │   ├── routes.ts                     # Типизированные URL builders
│   │   │   ├── config.ts                     # Настройки маршрутов
│   │   │   ├── guards.ts                     # Правила доступа
│   │   │   └── index.ts
│   │   │
│   │   ├── monitoring/
│   │   │   ├── analytics.ts                  # Отправка событий аналитики
│   │   │   ├── error-tracking.ts             # Регистрация ошибок
│   │   │   ├── logger.ts                     # Общий logger
│   │   │   └── web-vitals.ts                 # Метрики производительности
│   │   │
│   │   └── feature-flags/
│   │       ├── config.ts                     # Описание флагов
│   │       ├── server.ts                     # Получение flags на сервере
│   │       ├── client.ts                     # Client adapter
│   │       └── index.ts
│   │
│   ├── widgets/                              # Крупные самостоятельные UI-блоки
│   │   │
│   │   │   # Widget отвечает на вопрос:
│   │   │   # «Какой законченный блок видит пользователь?»
│   │   │   #
│   │   │   # Widget может:
│   │   │   # ├── объединять несколько features;
│   │   │   # ├── показывать несколько entities;
│   │   │   # └── использовать shared.
│   │   │   #
│   │   │   # Widget не должен импортировать app/processes.
│   │   │
│   │   ├── app-header/
│   │   │   ├── ui/
│   │   │   │   ├── AppHeader.tsx
│   │   │   │   └── AppHeader.module.scss
│   │   │   ├── model/                        # Локальная модель header
│   │   │   ├── lib/
│   │   │   └── index.ts                      # Публичный API widget
│   │   │
│   │   ├── app-sidebar/
│   │   ├── dashboard-overview/
│   │   ├── project-list/
│   │   ├── publication-composer/
│   │   ├── notification-center/
│   │   │
│   │   └── page-blocks/                      # Композиция сложных страниц
│   │       │
│   │       │   # Создавать page-block, если экран:
│   │       │   # ├── объединяет несколько widgets;
│   │       │   # ├── имеет сложную композиционную логику;
│   │       │   # ├── используется похожими маршрутами;
│   │       │   # └── делает page.tsx слишком большим.
│   │       │
│   │       │   # Не создавать page-block для:
│   │       │   # page.tsx → return <ProjectList />
│   │       │
│   │       ├── dashboard-page/
│   │       │   ├── ui/
│   │       │   │   └── DashboardPage.tsx
│   │       │   └── index.ts
│   │       ├── project-details-page/
│   │       └── profile-settings-page/
│   │
│   ├── features/                             # Пользовательские действия и сценарии
│   │   │
│   │   │   # Feature отвечает на вопрос:
│   │   │   # «Что пользователь делает?»
│   │   │   #
│   │   │   # Примеры:
│   │   │   # ├── войти;
│   │   │   # ├── создать проект;
│   │   │   # ├── удалить публикацию;
│   │   │   # ├── изменить язык;
│   │   │   # └── отфильтровать список.
│   │   │   #
│   │   │   # Feature может импортировать entities/shared.
│   │   │   # Feature не импортирует widgets/app/processes.
│   │   │
│   │   ├── auth/
│   │   │   ├── login/
│   │   │   │   ├── ui/
│   │   │   │   │   ├── LoginForm.tsx
│   │   │   │   │   └── LoginForm.module.scss
│   │   │   │   ├── model/
│   │   │   │   │   ├── schema.ts             # Схема валидации
│   │   │   │   │   ├── types.ts
│   │   │   │   │   ├── slice.ts              # Только если feature нужен Redux
│   │   │   │   │   └── selectors.ts
│   │   │   │   ├── api/
│   │   │   │   │   └── login.ts              # Запрос авторизации
│   │   │   │   ├── server/
│   │   │   │   │   └── login-action.ts       # Server Action
│   │   │   │   ├── lib/
│   │   │   │   └── index.ts
│   │   │   └── logout/
│   │   │
│   │   ├── create-project/
│   │   │   ├── ui/
│   │   │   ├── model/
│   │   │   ├── api/
│   │   │   ├── server/
│   │   │   ├── lib/
│   │   │   └── index.ts
│   │   ├── update-project/
│   │   ├── delete-project/
│   │   ├── filter-projects/
│   │   ├── create-publication/
│   │   ├── upload-publication-media/
│   │   ├── schedule-publication/
│   │   ├── change-language/
│   │   ├── change-theme/
│   │   └── manage-notifications/
│   │
│   ├── entities/                             # Бизнес-сущности приложения
│   │   │   # Entity отвечает на вопрос: «Что это за объект предметной области?»
│   │   │   #
│   │   │   # Здесь находятся:
│   │   │   # ├── типы сущности;
│   │   │   # ├── API сущности;
│   │   │   # ├── Redux slice сущности;
│   │   │   # ├── selectors;
│   │   │   # ├── бизнесовые преобразования;
│   │   │   # └── небольшой UI сущности.
│   │   │   #
│   │   │   # Entity может импортировать только shared.
│   │   │
│   │   ├── user/
│   │   │   ├── api/
│   │   │   │   ├── get-user.ts
│   │   │   │   ├── update-user.ts
│   │   │   │   └── user-api.ts               # RTK Query endpoints; если нужен
│   │   │   ├── model/
│   │   │   │   ├── types.ts                  # User/UserId/UserRole
│   │   │   │   ├── schema.ts                 # Runtime validation
│   │   │   │   ├── slice.ts                  # Redux slice владельца User
│   │   │   │   └── selectors.ts
│   │   │   ├── lib/
│   │   │   │   ├── get-user-name.ts
│   │   │   │   └── check-user-permission.ts
│   │   │   ├── ui/
│   │   │   │   ├── UserAvatar.tsx
│   │   │   │   ├── UserCard.tsx
│   │   │   │   └── UserBadge.tsx
│   │   │   └── index.ts                      # Единственная публичная точка входа
│   │   │
│   │   ├── project/
│   │   ├── publication/
│   │   ├── notification/
│   │   ├── subscription/
│   │   └── payment/
│   ├── proxy.ts  # Next.js 16 Proxy или middlewares.ts # или такое название можно использовать для логики i18n и редиректов
│   │            # Redirect/rewrite/locale/simple access check  Не помещать сюда тяжёлую бизнес-логику
│   │
│   └── shared/                               # Универсальная техническая база
│       │
│       │   # Shared не знает:
│       │   # ├── что такое Project/User/Publication;
│       │   # ├── какие features существуют;
│       │   # ├── какие widgets существуют;
│       │   # └── какие URL есть в app.
│       │   #
│       │   # Если компонент содержит бизнесовые слова,
│       │   # скорее всего, он не должен находиться в shared.
│       │
│       ├── api/
│       │   ├── http/
│       │   │   ├── client.ts                  # Базовый browser HTTP client
│       │   │   ├── server-client.ts           # Server-only HTTP client
│       │   │   ├── errors.ts                  # Общие API errors
│       │   │   ├── interceptors.ts            # Общие interceptors
│       │   │   └── types.ts
│       │   ├── base-api/
│       │   │   ├── api.ts                     # Базовый RTK Query createApi
│       │   │   └── base-query.ts
│       │   ├── graphql/                       # GraphQL client; если используется
│       │   ├── websocket/                     # WebSocket client; если используется
│       │   └── index.ts
│       │
│       ├── ui/                                # UI-kit без бизнес-логики
│       │   ├── button/
│       │   │   ├── Button.tsx
│       │   │   ├── Button.module.scss
│       │   │   ├── Button.test.tsx
│       │   │   ├── Button.stories.tsx
│       │   │   └── index.ts
│       │   ├── input/
│       │   ├── modal/
│       │   ├── select/
│       │   ├── table/
│       │   ├── tabs/
│       │   ├── tooltip/
│       │   ├── typography/
│       │   ├── spinner/
│       │   └── index.ts
│       │
│       ├── lib/                               # Универсальные функции
│       │   ├── browser/
│       │   │   ├── storage.ts                 # localStorage/sessionStorage
│       │   │   └── clipboard.ts
│       │   ├── server/
│       │   │   ├── cookies.ts                 # Server cookie helpers
│       │   │   └── headers.ts
│       │   ├── date/                          # Работа с датами
│       │   ├── number/                        # Форматирование чисел
│       │   ├── validation/                    # Общие validation helpers
│       │   ├── class-names/                   # cn()/classnames helpers
│       │   └── testing/                       # Общие test utilities
│       │
│       ├── hooks/                             # Только универсальные client hooks
│       │   ├── use-debounce.ts
│       │   ├── use-media-query.ts
│       │   ├── use-mounted.ts
│       │   └── use-event-listener.ts
│       │
│       ├── config/                           # Общие значения отдельно от типов
│       │   ├── constants.ts                   # Общие технические константы
│       │   ├── types.ts                       # Типы конфигурации, только если есть
│       │   ├── env.client.ts                  # Только NEXT_PUBLIC_*
│       │   ├── env.server.ts                  # Server-only env и secrets
│       │   ├── application.ts                 # Универсальная конфигурация
│       │   └── index.ts
│       │
│       ├── types/                             # Только domain-agnostic utility types; бизнес-типы у владельцев
│       ├── styles/
│       │   ├── reset.scss
│       │   ├── variables.scss
│       │   ├── mixins.scss
│       │   ├── themes.scss
│       │   └── tokens.css
│       │
│       ├── assets/
│       │   │
│       │   │   # Использовать shared/assets, когда файл:
│       │   │   # ├── импортируется из TypeScript/SCSS;
│       │   │   # ├── принадлежит компоненту;
│       │   │   # └── не требует постоянного публичного URL.
│       │   │
│       │   ├── icons/
│       │   ├── images/
│       │   └── fonts/
│       │
│       └── index.ts                           # Использовать осторожно:
│                                             # большой общий barrel может создавать cycles
│
├── tests/                                    # Общие тесты вне конкретных slices
│   ├── architecture/
│   │   ├── fsd-imports.test.ts               # Проверка направления зависимостей
│   │   ├── public-api.test.ts                 # Проверка index.ts
│   │   └── forbidden-imports.test.ts          # Запрещённые cross-imports
│   ├── integration/                          # Межмодульные тесты
│   ├── contracts/                            # API/schema contracts
│   ├── fixtures/                             # Общие тестовые данные
│   └── setup.ts                              # Общая настройка test runner
│
├── instrumentation.ts                       # Server instrumentation/OpenTelemetry
├── instrumentation-client.ts                # Client monitoring; если нужен
│
├── .dockerignore                            # Исключения Docker context
├── .editorconfig                            # Общие правила редакторов
├── .env.example                             # Документация env без реальных секретов
├── .env.local                               # Локальные секреты; не коммитить
├── .env.test                                # Тестовое окружение
├── .gitignore
├── .lintstagedrc.mjs                        # Команды для staged-файлов
├── .npmrc                                   # Настройки package manager
├── .prettierignore
├── .prettierrc
├── Dockerfile                               # Основная production-сборка
├── docker-compose.yml                       # Локальные сервисы
├── Makefile                                 # Короткие команды проекта
│
├── eslint.config.mjs                        # ESLint + FSD import boundaries
├── next.config.ts                           # Конфигурация Next.js
├── playwright.config.ts                     # E2E и visual tests
├── postcss.config.mjs                       # PostCSS/Tailwind
├── prettier.config.mjs                      # Форматирование
├── tsconfig.json                            # TypeScript и aliases
├── vitest.config.ts                         # Unit/integration tests
│
├── package.json                             # Зависимости и npm/yarn scripts
├── yarn.lock                                # Или package-lock/pnpm-lock; только один
├── README.md                                # Запуск и краткий обзор
└── PROJECT_ARCHITECTURE.md                  # Главные архитектурные правила


# ─────────────────────────────────────────────────────────────────────
# ШАБЛОН ВНУТРЕННЕЙ СТРУКТУРЫ FSD SLICE
# ─────────────────────────────────────────────────────────────────────

├── src/features/example-feature/
│   ├── ui/                                  # React-компоненты feature
│   ├── model/                               # State/schema/selectors/types
│   ├── api/                                 # Запросы feature
│   ├── server/                              # Server Actions/server-only код
│   ├── lib/                                 # Внутренние функции feature
│   ├── config/                              # Локальная конфигурация
│   │   ├── constants.ts                     # Константы владельца, если есть
│   │   └── types.ts                         # Типы config, если есть
│   └── index.ts                             # Публичный API
│
│   # При именованных props нужен ui/types.ts; при константах — config/constants.ts.
│   # Необязательно создавать все segments сразу.
│   # Если feature содержит только один UI-компонент:
│   #
│   # example-feature/
│   # ├── ui/ExampleFeature.tsx
│   # └── index.ts
│
│
│   # ─────────────────────────────────────────────────────────────────────
│   # ПРАВИЛА ПУБЛИЧНОГО API
│   # ─────────────────────────────────────────────────────────────────────
│
├── src/entities/project/index.ts            # Экспортирует только публичный контракт
│                                             # export { ProjectCard } from './ui/...'
│                                             # export { projectReducer } from './model/...'
│                                             # export type { Project } from './model/...'
│
│   # Правильно:
│   # import { ProjectCard } from '@/entities/project'
│   #
│   # Нежелательно:
│   # import { ProjectCard } from '@/entities/project/ui/ProjectCard'
│   #
│   # Исключение:
│   # точечный dynamic import тяжёлого компонента,
│   # если это явно разрешено правилами проекта.
│
│
│   # ─────────────────────────────────────────────────────────────────────
│   # РАСПРЕДЕЛЕНИЕ СОСТОЯНИЯ
│   # ─────────────────────────────────────────────────────────────────────
│
├── state-placement/
│   ├── params-searchParams                   # URL и фильтры, которыми делятся ссылкой
│   ├── server-component-fetch               # Серверные данные для SSR/RSC
│   ├── redux-toolkit                        # Глобальный изменяемый client-state
│   ├── rtk-query                            # Client-side server-state
│   ├── react-context                        # Нечасто меняемая scoped-конфигурация
│   ├── useState-useReducer                  # Локальное состояние компонента
│   ├── form-state                           # Состояние конкретной формы
│   └── cookie                               # Locale/theme/session preference
│
│
│   # ─────────────────────────────────────────────────────────────────────
│   # SERVER И CLIENT COMPONENTS
│   # ─────────────────────────────────────────────────────────────────────
│
├── rendering-rules/
│   ├── server-component                     # Используется по умолчанию
│   │                                         # Может читать БД/API и server env
│   │                                         # Не использует hooks/Redux/browser API
│   │
│   └── client-component                     # Имеет директиву 'use client'
│                                             # Нужен для:
│                                             # ├── useState/useEffect;
│                                             # ├── Redux hooks/React Context;
│                                             # ├── browser API;
│                                             # ├── event handlers;
│                                             # └── client-only UI libraries.
│
│
│   # ─────────────────────────────────────────────────────────────────────
│   # ИМЕНОВАНИЕ
│   # ─────────────────────────────────────────────────────────────────────
│
└── naming-conventions/
    ├── layers-and-slices                     # kebab-case:
    │                                         # features/create-project
    │                                         # entities/social-account
    │                                         # widgets/publication-composer
    │
    ├── react-components                      # PascalCase:
    │                                         # CreateProjectForm.tsx
    │                                         # PublicationComposer.tsx
    │
    ├── helpers-and-actions                   # kebab-case:
    │                                         # create-project-action.ts
    │                                         # format-publication-date.ts
    │
    ├── tests                                 # *.test.ts / *.test.tsx
    ├── stories                               # *.stories.tsx
    └── styles                                # Component.module.scss
