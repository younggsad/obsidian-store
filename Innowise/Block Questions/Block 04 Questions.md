---
custom-width: 80
---
# Summary по Block 4

## Task 4.1 — TaskStatService

### Русский

Создан сервис `TaskStatService` для работы со статистикой задач и проектов. Реализованы расчёт затраченного времени, оставшейся оценки и общей статистики проекта. Сервис использует Entity API и Dependency Injection без прямого использования SQL.

### English

Created the `TaskStatService` for task and project statistics. Implemented logged hours, remaining estimate, and project statistics calculations. The service uses the Entity API and Dependency Injection without direct SQL queries.

---

## Task 4.2 — Dependency Injection

### Русский

Добавлен Dependency Injection для `TaskStatService` и `TimeLogWriteOffForm`. Зависимости передаются через конструктор и `create()` method. Убрано использование статического `\Drupal::` в соответствующем коде.

### English

Added Dependency Injection to `TaskStatService` and `TimeLogWriteOffForm`. Dependencies are passed through the constructor and `create()` method. Static `\Drupal::` usage was removed from the related code.

---

## Task 4.3 — ReportGenerator Plugin API

### Русский

Создан Plugin API для генерации отчётов. Добавлены `ReportGeneratorInterface`, `ReportGeneratorManager` и два плагина: `ProjectSummaryReport` и `OverdueTasksReport`. Используется Attribute-based plugin discovery и Dependency Injection. Работа discovery была проверена — обнаруживаются ровно два плагина.

### English

Created a Plugin API for report generation. Added `ReportGeneratorInterface`, `ReportGeneratorManager`, and two plugins: `ProjectSummaryReport` and `OverdueTasksReport`. Attribute-based plugin discovery and Dependency Injection are used. Plugin discovery was verified and exactly two plugins are detected.

---

## Task 4.4 — Custom Block, Field Widget and Formatter

### Русский

Создан `Project Statistics` Block для отображения статистики проекта, `Hours + Minutes` Widget для удобного ввода оценки времени и `Time Summary` Formatter для отображения затраченного и оставшегося времени. Добавлено автоматическое инвалидирование кеша при изменении TimeLog. Обновлена конфигурация `field_estimate` для поддержки двух знаков после запятой и экспортировано размещение блока.

### English

Created the `Project Statistics` Block for displaying project statistics, the `Hours + Minutes` Widget for entering time estimates, and the `Time Summary` Formatter for displaying logged and remaining time. Automatic cache invalidation was added for TimeLog changes. The `field_estimate` configuration was updated to support two decimal places, and the block placement was exported.

---

## Final Block 4

### Русский

Все задачи **4.1–4.4** выполнены и проверены. Проведён дополнительный рефакторинг согласно методичке. **PHPStan, PHPCS и GrumPHP прошли успешно.** Все изменения были объединены через PR сначала в `develop`, затем в `main`.

### 🇬🇧 English

All tasks **4.1–4.4** were completed and verified. Additional refactoring was performed according to the methodology. **PHPStan, PHPCS, and GrumPHP checks passed successfully.** All changes were merged through PRs into `develop` and then into `main`.