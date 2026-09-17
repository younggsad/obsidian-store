---
custom-width: 80
---
# Block 3 — TimeLog & учёт времени / Time Tracking

> [!summary] 
> RU Реализована собственная **Content Entity** `time_log` для учёта затраченного времени: CRUD через Entity API, выборки через EntityQuery, форма списания времени с client- и server-side валидацией.

> [!summary] 
> EN Implemented a custom **Content Entity** `time_log` for time tracking: CRUD via Entity API, queries via EntityQuery, and a time write-off form with client- and server-side validation.

## 🔧 Технологии и их роль / Technologies & their role

|Технология / Technology|За что отвечает (RU)|Responsible for (EN)|
|---|---|---|
|**Content Entity API**|`time_log` хранит данные (не конфигурацию) — реальные записи времени|Stores data (not config) — real time-log records|
|**Entity Attribute** (`revisionable: false`, `translatable: false`)|Явно отключает историю версий и переводы, т.к. они не нужны|Explicitly disables revisions/translations — not needed|
|**Entity Reference field**|Поля `task`, `uid` — связь с реальными node/user сущностями, а не просто числа|`task`, `uid` link to real entities, not just raw IDs|
|**Field types (decimal / date)**|`hours` → decimal (число), `log_date` → date без времени (нужен только день)|`hours` → decimal; `log_date` → date-only (day matters, not time)|
|**drush generate:entity:content**|Генерирует базовый каркас сущности (класс, routing, forms)|Generates entity boilerplate (class, routing, forms)|
|**Route upcasting** (`type: entity:node, bundle: task`)|Автоматически превращает `{task}` в объект `NodeInterface`, при отсутствии — 404|Auto-converts `{task}` param to `NodeInterface`; missing → 404|
|**`_permission: access administration pages`**|Ограничивает debug-маршруты только админам|Restricts debug routes to admins|
|**EntityQuery**|Выборка/агрегация без прямого SQL (сумма часов, сортировка по дате)|Query/aggregate without raw SQL (sum hours, sort by date)|
|**Render array** (`#markup`, `#theme`)|Стандартный способ Drupal выводить HTML вместо `echo/print`|Standard Drupal output instead of `echo/print`|
|**`create()` vs `save()`**|`create()` — объект в памяти; `save()` — запись в БД|`create()` = in-memory object; `save()` = DB write|
|**Form API — `FormBase`**|Обычная форма ввода данных (не confirm-форма)|Regular data-entry form (not a confirm form)|
|**States API** (`#states`)|Client-side: показывает/делает обязательным `over_estimate_reason` при чекбоксе|Client-side: shows/requires `over_estimate_reason` on checkbox|
|**`validateForm()`**|Server-side дублирующая проверка (дата не в будущем, `hours > 0`, reason обязателен) — States API можно обойти|Server-side re-check (no future date, `hours > 0`, required reason) — States API is bypassable|
|**`submitForm()`**|Здесь создаётся TimeLog и привязывается `uid` текущего пользователя из сессии|TimeLog created here; `uid` taken from current session|
|**`hook_form_BASE_FORM_ID_alter()`**|Точечно меняет только node-форму типа `task` (проверка `$node->bundle()`)|Targets only the `task` node form (`$node->bundle()` check)|
|**`Url::fromRoute()`**|Строит ссылку "Log time" через route API, а не конкатенацией строки|Builds "Log time" link via route API, not string concatenation|

---

# 📊 Block 4 — Статистика, Plugin API, DI / Stats, Plugin API, DI

> [!summary] RU 
> Добавлен сервис статистики, весь код переведён на Dependency Injection, создана Plugin-система отчётов и три кастомных plugin'а (Block, Widget, Formatter) для отображения времени.

> [!summary] EN Added a stats service, migrated code to Dependency Injection, built a report Plugin system, and three custom plugins (Block, Widget, Formatter) for displaying time data.

## 🔧 Технологии и их роль / Technologies & their role

|Технология / Technology|За что отвечает (RU)|Responsible for (EN)|
|---|---|---|
|**`TaskStatService`** (сервис)|Единая точка расчёта затраченного времени / остатка оценки — без дублирования логики|Single place for logged/remaining time calc — avoids duplication|
|**`*.services.yml`**|Регистрация сервиса в Service Container; порядок `arguments` должен совпадать с конструктором|Registers service in the container; `arguments` order must match constructor|
|**Dependency Injection (конструктор)**|Явные зависимости вместо `\Drupal::service()` — легче тестировать, нет скрытых связей|Explicit deps instead of `\Drupal::service()` — easier testing, no hidden coupling|
|**`ContainerInjectionInterface` + `create()`**|Для форм (`FormBase`) — Drupal сам создаёт объект и передаёт зависимости|For forms — Drupal builds the object and injects deps|
|**`ContainerFactoryPluginInterface` + `create()`**|То же самое, но для plugin'ов (Block/Widget/Formatter), с учётом plugin configuration|Same, but for plugins, also receiving plugin configuration|
|**Plugin API**|Позволяет добавлять новые типы отчётов без `switch/if` — каждый отчёт = отдельный класс|Lets you add report types without `switch/if` — each report is its own class|
|**`ReportGeneratorInterface`**|Общий контракт (`generate()`, `getLabel()`) для всех отчётов|Common contract (`generate()`, `getLabel()`) for all reports|
|**`ReportGeneratorManager`** (`DefaultPluginManager`)|Сканирует директорию plugins и собирает definitions|Scans plugin directory and builds definitions|
|**PHP Attributes** (`#[ReportGenerator(...)]`)|Современный способ discovery в Drupal 11 (вместо annotation `@ReportGenerator`)|Modern discovery mechanism in Drupal 11 (replaces `@ReportGenerator` docblocks)|
|**Разделение `generate()` / `getLabel()`**|Данные отчёта и его название — разные обязанности (Single Responsibility)|Report data vs. report name — separate responsibilities|
|**Custom Block plugin**|`Project Statistics` — получает текущий `node` через `CurrentRouteMatch`, не парсит URL|Gets current `node` via `CurrentRouteMatch`, no URL parsing|
|**Field Widget** (`Hours + Minutes`)|Конвертирует decimal ↔ часы/минуты в обе стороны при сохранении/открытии формы|Converts decimal ↔ hours/minutes both ways on save/load|
|**Field Formatter** (`Time Summary`)|Только форматирует уже готовые данные из `TaskStatService` (не считает сам через EntityQuery)|Only formats data already computed by `TaskStatService` (no own EntityQuery)|
|**`field_types` в Attribute plugin'а**|Определяет, для каких типов полей доступен Widget/Formatter (у нас — `decimal`)|Defines which field types the Widget/Formatter applies to (here — `decimal`)|
|**Cache invalidation**|Автоматический сброс кеша блока/formatter'а при изменении TimeLog|Auto cache invalidation of block/formatter when TimeLog changes|
|**PHPStan / PHPCS / GrumPHP**|Статический анализ и code style — проверка перед merge в `develop` → `main`|Static analysis & code style — checked before merging `develop` → `main`|

---

## ✅ Итог / Bottom line

- **RU:** Block 3 добавил доменную сущность `time_log` и полный жизненный цикл учёта времени (create → validate → save). Block 4 вынес логику в сервис, перевёл всё на DI, и построил расширяемую Plugin-архитектуру отчётов + UI-компоненты (block/widget/formatter) поверх неё.
- **EN:** Block 3 introduced the `time_log` domain entity and the full time-logging lifecycle (create → validate → save). Block 4 extracted logic into a service, moved everything to DI, and built an extensible report Plugin architecture plus UI components (block/widget/formatter) on top of it.