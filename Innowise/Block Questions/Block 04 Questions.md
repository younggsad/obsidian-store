---
custom-width: 80
---
# Summary по Block 4

## Task 4.1 — TaskStatService

Создан сервис `TaskStatService` для работы со статистикой задач и проектов. Реализованы расчёт затраченного времени, оставшейся оценки и общей статистики проекта. Сервис использует Entity API и Dependency Injection без прямого использования SQL.

Created the `TaskStatService` for task and project statistics. Implemented logged hours, remaining estimate, and project statistics calculations. The service uses the Entity API and Dependency Injection without direct SQL queries.

---

## Task 4.2 — Dependency Injection

Добавлен Dependency Injection для `TaskStatService` и `TimeLogWriteOffForm`. Зависимости передаются через конструктор и `create()` method. Убрано использование статического `\Drupal::` в соответствующем коде.

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

### English

All tasks **4.1–4.4** were completed and verified. Additional refactoring was performed according to the methodology. **PHPStan, PHPCS, and GrumPHP checks passed successfully.** All changes were merged through PRs into `develop` and then into `main`.

# Block 4 — Questions and Answers

## 1. Что такое сервис в терминологии Drupal и чем он принципиально отличается от обычного класса со статическими методами?

**Question:** What is a service in Drupal terminology and how is it fundamentally different from a regular class with static methods?

**Ответ:**  
Сервис — это объект, управляемый Drupal Service Container. Он предназначен для переиспользуемой бизнес-логики и может получать зависимости через Dependency Injection. Статические методы работают без объекта и скрывают зависимости внутри класса.

**Answer:**  
A service is an object managed by the Drupal Service Container. It is used for reusable business logic and can receive dependencies through Dependency Injection. Static methods work without an object and hide dependencies inside the class.

---

## 2. Почему было полезно вынести расчёт затраченного времени в `TaskStatService`, а не дублировать его?

**Question:** Why was it worth moving the logic for calculating the time spent from forms and handlers to a separate service `TaskStatService`, and not leaving it duplicated in each place of use?

**Ответ:**  
Чтобы иметь одну реализацию логики. Если расчёт изменится, мы изменим только сервис. Это уменьшает дублирование и риск ошибок.

**Answer:**  
It provides one implementation of the logic. If the calculation changes, we only need to update the service. This reduces duplication and the risk of errors.

---

## 3. Почему `TaskStatService` использует Entity API/EntityQuery, а не прямой SQL?

**Question:** Why do the `TaskStatService` methods work with TimeLog and Task through the Entity API/EntityQuery, and not through direct SQL queries to database tables?

**Ответ:**  
Потому что Drupal Entity API абстрагирует структуру базы данных и работает с сущностями Drupal. Это делает код более переносимым и соответствует архитектуре Drupal.

**Answer:**  
Because the Drupal Entity API abstracts the database structure and works with Drupal entities. This makes the code more portable and follows Drupal architecture.

---

## 4. Чем регистрация сервиса через `*.services.yml` отличается от `new TaskStatService()`?

**Question:** How is registering a service using `*.services.yml` different from simply creating an object using the `new TaskStatService()` statement inside a controller or form?

**Ответ:**  
При регистрации через `services.yml` объект создаёт Service Container и может автоматически передать ему зависимости. При `new` класс создаётся вручную, и зависимости нужно передавать самостоятельно.

**Answer:**  
With `services.yml`, the Service Container creates the object and can provide its dependencies automatically. With `new`, the class is created manually and dependencies must be provided manually.

---

## 5. Когда обнаружится ошибка, если в `*.services.yml` указан несуществующий класс?

**Question:** When will an error be detected if `*.services.yml` specifies a class `TaskStatService` that does not exist or contains a syntax error?

**Ответ:**  
Ошибка обычно обнаружится, когда Drupal загрузит или создаст этот сервис. Например, при очистке кеша `drush cr` или при первом обращении к сервису, в зависимости от конкретной ошибки.

**Answer:**  
The error is usually detected when Drupal loads or creates the service. For example, it can happen during `drush cr` or when the service is first requested, depending on the error.

---

## 6. Почему использование `\Drupal::` в конструкторе в Problem 4.1 было временным компромиссом?

**Question:** In Problem 4.1, the use of `\Drupal::` in the service constructor is acceptable, but in business methods it is not. Why is this a temporary compromise and not good practice in general?

**Ответ:**  
Это временное решение для получения зависимостей, когда DI ещё не был полностью реализован. Но такой подход создаёт скрытые зависимости. Правильный вариант — передавать зависимости через конструктор.

**Answer:**  
It is a temporary solution for getting dependencies before full DI was implemented. However, it creates hidden dependencies. The better approach is to pass dependencies through the constructor.

---

## 7. Service Locator и Dependency Injection: в чём разница и почему DI предпочтительнее?

**Question:** Compare Service Locator (`\Drupal::service()`) and Dependency Injection: What is the difference between the approaches and why is DI preferred in Drupal?

**Ответ:**  
Service Locator получает зависимость внутри класса в момент использования. DI передаёт её в класс через конструктор. DI лучше показывает зависимости, упрощает тестирование и делает код более предсказуемым.

**Answer:**  
A Service Locator gets a dependency inside the class when it is needed. DI passes the dependency to the class through the constructor. DI makes dependencies clear, testing easier, and code more predictable.

---

## 8. Почему `\Drupal::` внутри класса усложняет unit testing?

**Question:** Why does using `\Drupal::` inside a class make unit testing more difficult?

**Ответ:**  
Потому что класс сам обращается к глобальному контейнеру. В тесте сложнее заменить эту зависимость mock-объектом. При DI mock можно просто передать через конструктор.

**Answer:**  
Because the class directly accesses the global container. It is harder to replace this dependency with a mock in a test. With DI, a mock can simply be passed to the constructor.

---

## 9. Чем DI обычного сервиса отличается от DI формы на основе `FormBase`?

**Question:** How does the dependency injection mechanism of a regular service (`TaskStatService`) differ from the mechanism of a form based on `FormBase`? Why are different approaches used?

**Ответ:**  
Для обычного сервиса зависимости передаются через конструктор и регистрируются в `services.yml`. Для формы Drupal создаёт объект через `create(ContainerInterface $container)`, который получает зависимости из контейнера.

**Answer:**  
For a regular service, dependencies are passed through the constructor and defined in `services.yml`. For a form, Drupal creates the object through `create(ContainerInterface $container)`, which gets dependencies from the container.

---

## 10. Что такое `ContainerInjectionInterface` и зачем нужен `create()`?

**Question:** What is `ContainerInjectionInterface` and what is the role of the `create()` method?

**Ответ:**  
`ContainerInjectionInterface` показывает, что объект может получать зависимости из Service Container. Метод `create()` получает контейнер и создаёт объект, передавая ему необходимые зависимости.

**Answer:**  
`ContainerInjectionInterface` indicates that an object can receive dependencies from the Service Container. The `create()` method receives the container and creates the object with the required dependencies.

---

## 11. Что произойдёт, если порядок `arguments` в `services.yml` не совпадает с конструктором?

**Question:** What happens if in `*.services.yml` we list `arguments` for `TaskStatService` in an order that does not match the order of the constructor parameters?

**Ответ:**  
Зависимости попадут не в те параметры конструктора. Это может привести к ошибке типов или неправильной работе сервиса.

**Answer:**  
The dependencies will be passed to the wrong constructor parameters. This can cause a type error or incorrect service behavior.

---

## 12. Почему стандарт DrupalJira требует получать container services только через конструктор?

**Question:** Why does the DrupalJira team standard require that any class that works with container services must obtain them only through the constructor - what practical problems does this solve?

**Ответ:**  
Это делает зависимости явными, упрощает тестирование и уменьшает скрытые зависимости. Также такой код легче поддерживать и изменять.

**Answer:**  
It makes dependencies explicit, simplifies testing, and reduces hidden dependencies. This also makes the code easier to maintain and change.

---

## 13. Что такое Plugin API в Drupal и какую проблему он решает?

**Question:** What is the Plugin API in Drupal and what problem does it solve? Why did you choose your own plugin system for the reporting system, and not an array of classes with switch/if by report type?

**Ответ:**  
Plugin API позволяет автоматически находить и использовать разные реализации одной задачи. Для отчётов это позволяет добавлять новые типы отчётов без большого `switch` или `if`. Каждый отчёт становится отдельным plugin.

**Answer:**  
The Plugin API allows Drupal to discover and use different implementations of the same task. For reports, this allows us to add new report types without a large `switch` or `if`. Each report is a separate plugin.

---

## 14. Из каких трёх частей состоит наша система ReportGenerator?

**Question:** What three parts does the ReportGenerator plugin system consist of and what is the role of each?

**Ответ:**

1. `ReportGeneratorInterface` — определяет общий контракт.
2. `ReportGeneratorManager` — ищет и управляет plugins.
3. Конкретные plugins — `ProjectSummaryReport` и `OverdueTasksReport`, которые создают отчёты.

**Answer:**

1. `ReportGeneratorInterface` — defines the common contract.
2. `ReportGeneratorManager` — discovers and manages plugins.
3. Concrete plugins — `ProjectSummaryReport` and `OverdueTasksReport`, which generate reports.

---

## 15. Зачем нужен `ReportGeneratorInterface`?

**Question:** Why do ReportGenerator plugins need a formal `ReportGeneratorInterface` interface when they could simply require a `generate()` method without a declared contract?

**Ответ:**  
Интерфейс гарантирует, что каждый plugin имеет одинаковые методы и сигнатуры. Это делает систему предсказуемой и позволяет менеджеру работать с разными plugins одинаково.

**Answer:**  
The interface guarantees that every plugin has the same methods and signatures. This makes the system predictable and allows the manager to work with different plugins in the same way.

---

## 16. Зачем разделять `generate()` и `getLabel()`?

**Question:** What does dividing into separate methods `generate()` and `getLabel()` give instead of one method that combines calculating report data and getting its name?

**Ответ:**  
`generate()` отвечает только за данные отчёта, а `getLabel()` — только за его название. Это разделяет разные обязанности и делает код проще.

**Answer:**  
`generate()` is responsible only for report data, while `getLabel()` is responsible only for its name. This separates responsibilities and makes the code simpler.

---

## 17. Как `DefaultPluginManager` узнаёт о `ProjectSummaryReport`?

**Question:** How does discovery of plugins work through `DefaultPluginManager` and the PHP attribute `#[ReportGenerator]` - how does the manager know about the existence of `ProjectSummaryReport` without explicit registration?

**Ответ:**  
`DefaultPluginManager` сканирует определённую директорию plugins и ищет классы с нужным Attribute. `#[ReportGenerator]` содержит ID и label, поэтому plugin автоматически попадает в definitions после очистки кеша.

**Answer:**  
`DefaultPluginManager` scans the plugin directory and looks for classes with the required Attribute. `#[ReportGenerator]` contains the ID and label, so the plugin is automatically added to the definitions after cache clearing.

---

## 18. PHP Attributes и Annotation DocBlocks: почему выбрали Attributes?

**Question:** Compare PHP attributes (`#[ReportGenerator(...)]`) and annotation docblocks (`@ReportGenerator`) for discovery plugins. Why was the first option chosen for Drupal 11?

**Ответ:**  
PHP Attributes являются частью современного PHP и имеют структурированный синтаксис. Drupal 11 активно использует Attributes для plugin discovery, поэтому этот подход соответствует современной архитектуре Drupal.

**Answer:**  
PHP Attributes are part of modern PHP and provide a structured syntax. Drupal 11 uses Attributes for plugin discovery, so this approach fits the modern Drupal architecture.

---

## 19. Почему plugins получают `TaskStatService` через `create()`?

**Question:** Why do the `ProjectSummaryReport` and `OverdueTasksReport` plugins get `TaskStatService` through `create()` and not through `\Drupal::service()` inside `generate()`?

**Ответ:**  
Потому что это Dependency Injection. Зависимость передаётся при создании plugin, а не получается скрыто внутри `generate()`. Это улучшает тестируемость и структуру кода.

**Answer:**  
Because this is Dependency Injection. The dependency is provided when the plugin is created instead of being obtained inside `generate()`. This improves testing and code structure.

---

## 20. Для чего нужны Block, Field Widget и Field Formatter?

**Question:** What is each of the three types of kernel plugins used for - Block, Field Widget, Field Formatter - and what is the fundamental difference in their purpose?

**Ответ:**

- **Block** — выводит отдельный блок контента.
- **Field Widget** — отвечает за ввод значения поля в форме.
- **Field Formatter** — отвечает за отображение значения поля.

Главное отличие — они работают на разных этапах: вывод блока, ввод данных и отображение данных.

**Answer:**

- **Block** — displays a separate block of content.
- **Field Widget** — controls how a field value is entered in a form.
- **Field Formatter** — controls how a field value is displayed.

The main difference is their purpose: block output, data input, and data display.

---

## 21. Как Project Statistics Block определяет текущий проект?

**Question:** How does the Project Statistics Block Plugin determine which project to show statistics for based on the current page, and why shouldn't this be done by manually parsing the URL?

**Ответ:**  
Block получает текущий `node` через `CurrentRouteMatch`. Если это project — используется он. Если это task — берётся проект из `field_project`. Парсить URL не нужно, потому что Drupal уже предоставляет объект текущего route parameter.

**Answer:**  
The block gets the current `node` through `CurrentRouteMatch`. If it is a project, it uses that project. If it is a task, it gets the project from `field_project`. URL parsing is unnecessary because Drupal already provides the current route parameter.

---

## 22. Что такое `ContainerFactoryPluginInterface` и чем он отличается от `ContainerInjectionInterface`?

**Question:** What is `ContainerFactoryPluginInterface` and how is it different from the `ContainerInjectionInterface` that forms use?

**Ответ:**  
`ContainerFactoryPluginInterface` используется для plugins, которым нужны зависимости из контейнера. У них есть `create()` с параметрами plugin configuration. `ContainerInjectionInterface` используется для обычных объектов, например форм, и также предоставляет `create()` для получения зависимостей.

**Answer:**  
`ContainerFactoryPluginInterface` is used for plugins that need container dependencies. Their `create()` method also receives plugin configuration. `ContainerInjectionInterface` is used for regular container-injectable objects, such as forms, and also provides `create()` for getting dependencies.

---

## 23. Зачем Hours + Minutes Widget конвертирует значение в обе стороны?

**Question:** Why does the Field Widget "Hours + Minutes" implement a value conversion in both directions - assembling the decimal number when saving and parsing it back into hours and minutes when opening the form?

**Ответ:**  
Потому что база хранит время как decimal, например `2.5`, а пользователю удобнее работать с `2 часа 30 минут`. Поэтому при сохранении мы объединяем значения, а при редактировании разделяем их обратно.

**Answer:**  
Because the database stores the time as a decimal value, such as `2.5`, while users can work more easily with `2 hours 30 minutes`. On save, the values are combined, and when editing, they are split back.

---

## 24. Почему Time Summary Formatter получает `TaskStatService` через DI?

**Question:** Why should the "Time Summary" Field Formatter receive `TaskStatService` via DI rather than calculate the written time directly inside `viewElements()` via its own EntityQuery?

**Ответ:**  
Чтобы не дублировать бизнес-логику. `TaskStatService` уже отвечает за расчёт logged hours и remaining estimate. Formatter должен только форматировать готовые данные.

**Answer:**  
To avoid duplicating business logic. `TaskStatService` already calculates logged hours and remaining estimate. The formatter should only format the resulting data.

---

## 25. От чего зависит доступность Field Widget/Formatter для определённых типов полей?

**Question:** What determines which field types a particular Field Widget/Field Formatter is available for, and how is this expressed in the plugin's PHP attribute?

**Ответ:**  
Это определяется параметром `field_types` в PHP Attribute. В нашем случае указано:

```
field_types: ['decimal']
```

Поэтому `HoursMinutesWidget` и `TimeSummaryFormatter` доступны для decimal-полей.

**Answer:**  
It is determined by the `field_types` parameter in the PHP Attribute. In our case, it is:

```
field_types: ['decimal']
```

Therefore, `HoursMinutesWidget` and `TimeSummaryFormatter` are available for decimal fields.