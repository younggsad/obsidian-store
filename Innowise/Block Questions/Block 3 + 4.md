---
custom-width: 80
---
# Block 3 — TimeLog & учёт времени / Time Tracking

> [!summary] RU 
> Реализована собственная **Content Entity** `time_log` для учёта затраченного времени: CRUD через Entity API, выборки через EntityQuery, форма списания времени с client- и server-side валидацией.

> [!summary] EN 
> Implemented a custom **Content Entity** `time_log` for time tracking: CRUD via Entity API, queries via EntityQuery, and a time write-off form with client- and server-side validation.

## Технологии и их роль / Technologies & their role

- **Content Entity API**
    - RU: `time_log` хранит данные (не конфигурацию) — реальные записи времени
    - EN: Stores data (not config) — real time-log records
- **Entity Attribute** (`revisionable: false`, `translatable: false`)
    - RU: Явно отключает историю версий и переводы, т.к. они не нужны
    - EN: Explicitly disables revisions/translations — not needed
- **Entity Reference field**
    - RU: Поля `task`, `uid` — связь с реальными node/user сущностями, а не просто числа
    - EN: `task`, `uid` link to real entities, not just raw IDs
- **Field types (decimal / date)**
    - RU: `hours` → decimal (число), `log_date` → date без времени (нужен только день)
    - EN: `hours` → decimal; `log_date` → date-only (day matters, not time)
- **drush generate:entity:content**
    - RU: Генерирует базовый каркас сущности (класс, routing, forms)
    - EN: Generates entity boilerplate (class, routing, forms)
- **Route upcasting** (`type: entity:node, bundle: task`)
    - RU: Автоматически превращает `{task}` в объект `NodeInterface`, при отсутствии — 404
    - EN: Auto-converts `{task}` param to `NodeInterface`; missing → 404
- **`_permission: access administration pages`**
    - RU: Ограничивает debug-маршруты только админам
    - EN: Restricts debug routes to admins
- **EntityQuery**
    - RU: Выборка/агрегация без прямого SQL (сумма часов, сортировка по дате)
    - EN: Query/aggregate without raw SQL (sum hours, sort by date)
- **Render array** (`#markup`, `#theme`)
    - RU: Стандартный способ Drupal выводить HTML вместо `echo/print`
    - EN: Standard Drupal output instead of `echo/print`
- **`create()` vs `save()`**
    - RU: `create()` — объект в памяти; `save()` — запись в БД
    - EN: `create()` = in-memory object; `save()` = DB write
- **Form API — `FormBase`**
    - RU: Обычная форма ввода данных (не confirm-форма)
    - EN: Regular data-entry form (not a confirm form)
- **States API** (`#states`)
    - RU: Client-side — показывает/делает обязательным `over_estimate_reason` при чекбоксе
    - EN: Client-side — shows/requires `over_estimate_reason` on checkbox
- **`validateForm()`**
    - RU: Server-side дублирующая проверка (дата не в будущем, `hours > 0`, reason обязателен) — States API можно обойти
    - EN: Server-side re-check (no future date, `hours > 0`, required reason) — States API is bypassable
- **`submitForm()`**
    - RU: Здесь создаётся TimeLog и привязывается `uid` текущего пользователя из сессии
    - EN: TimeLog created here; `uid` taken from current session
- **`hook_form_BASE_FORM_ID_alter()`**
    - RU: Точечно меняет только node-форму типа `task` (проверка `$node->bundle()`)
    - EN: Targets only the `task` node form (`$node->bundle()` check)
- **`Url::fromRoute()`**
    - RU: Строит ссылку "Log time" через route API, а не конкатенацией строки
    - EN: Builds "Log time" link via route API, not string concatenation

---

# Block 4 — Статистика, Plugin API, DI / Stats, Plugin API, DI

> [!summary] RU 
> Добавлен сервис статистики, весь код переведён на Dependency Injection, создана Plugin-система отчётов и три кастомных plugin'а (Block, Widget, Formatter) для отображения времени.

> [!summary] EN 
> Added a stats service, migrated code to Dependency Injection, built a report Plugin system, and three custom plugins (Block, Widget, Formatter) for displaying time data.

## Технологии и их роль / Technologies & their role

- **`TaskStatService`** (сервис)
    - RU: Единая точка расчёта затраченного времени / остатка оценки — без дублирования логики
    - EN: Single place for logged/remaining time calc — avoids duplication
- **`*.services.yml`**
    - RU: Регистрация сервиса в Service Container; порядок `arguments` должен совпадать с конструктором
    - EN: Registers service in the container; `arguments` order must match constructor
- **Dependency Injection (конструктор)**
    - RU: Явные зависимости вместо `\Drupal::service()` — легче тестировать, нет скрытых связей
    - EN: Explicit deps instead of `\Drupal::service()` — easier testing, no hidden coupling
- **`ContainerInjectionInterface` + `create()`**
    - RU: Для форм (`FormBase`) — Drupal сам создаёт объект и передаёт зависимости
    - EN: For forms — Drupal builds the object and injects deps
- **`ContainerFactoryPluginInterface` + `create()`**
    - RU: То же самое, но для plugin'ов (Block/Widget/Formatter), с учётом plugin configuration
    - EN: Same, but for plugins, also receiving plugin configuration
- **Plugin API**
    - RU: Позволяет добавлять новые типы отчётов без `switch/if` — каждый отчёт = отдельный класс
    - EN: Lets you add report types without `switch/if` — each report is its own class
- **`ReportGeneratorInterface`**
    - RU: Общий контракт (`generate()`, `getLabel()`) для всех отчётов
    - EN: Common contract (`generate()`, `getLabel()`) for all reports
- **`ReportGeneratorManager`** (`DefaultPluginManager`)
    - RU: Сканирует директорию plugins и собирает definitions
    - EN: Scans plugin directory and builds definitions
- **PHP Attributes** (`#[ReportGenerator(...)]`)
    - RU: Современный способ discovery в Drupal 11 (вместо annotation `@ReportGenerator`)
    - EN: Modern discovery mechanism in Drupal 11 (replaces `@ReportGenerator` docblocks)
- **Разделение `generate()` / `getLabel()`**
    - RU: Данные отчёта и его название — разные обязанности (Single Responsibility)
    - EN: Report data vs. report name — separate responsibilities
- **Custom Block plugin**
    - RU: `Project Statistics` — получает текущий `node` через `CurrentRouteMatch`, не парсит URL
    - EN: Gets current `node` via `CurrentRouteMatch`, no URL parsing
- **Field Widget** (`Hours + Minutes`)
    - RU: Конвертирует decimal ↔ часы/минуты в обе стороны при сохранении/открытии формы
    - EN: Converts decimal ↔ hours/minutes both ways on save/load
- **Field Formatter** (`Time Summary`)
    - RU: Только форматирует уже готовые данные из `TaskStatService` (не считает сам через EntityQuery)
    - EN: Only formats data already computed by `TaskStatService` (no own EntityQuery)
- **`field_types` в Attribute plugin'а**
    - RU: Определяет, для каких типов полей доступен Widget/Formatter (у нас — `decimal`)
    - EN: Defines which field types the Widget/Formatter applies to (here — `decimal`)
- **Cache invalidation**
    - RU: Автоматический сброс кеша блока/formatter'а при изменении TimeLog
    - EN: Auto cache invalidation of block/formatter when TimeLog changes
- **PHPStan / PHPCS / GrumPHP**
    - RU: Статический анализ и code style — проверка перед merge в `develop` → `main`
    - EN: Static analysis & code style — checked before merging `develop` → `main`

---

## Итог / Bottom line

- **RU:** Block 3 добавил доменную сущность `time_log` и полный жизненный цикл учёта времени (create → validate → save). Block 4 вынес логику в сервис, перевёл всё на DI, и построил расширяемую Plugin-архитектуру отчётов + UI-компоненты (block/widget/formatter) поверх неё.
- **EN:** Block 3 introduced the `time_log` domain entity and the full time-logging lifecycle (create → validate → save). Block 4 extracted logic into a service, moved everything to DI, and built an extensible report Plugin architecture plus UI components (block/widget/formatter) on top of it.