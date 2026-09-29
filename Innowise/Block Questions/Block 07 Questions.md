---
custom-width: 80
---
## 1. States vs transitions

**Вопрос:** Чем состояния отличаются от переходов в модели Workflow и почему это разделение важнее, чем просто хранить одно текстовое поле со статусом?

**Question:** How are states different from transitions in the Workflow model, and why is this separation more important than simply storing a single text field for an entity with a status value?

**Ответ:** **State** — это текущее состояние сущности, например `review`. **Transition** — это разрешённое действие, которое переводит сущность из одного состояния в другое, например `approve: review → done`. Такое разделение позволяет управлять допустимыми переходами и правами доступа, а не просто хранить произвольный текст.

**Answer:** A **state** represents the current condition of an entity, for example `review`. A **transition** is an allowed action that moves the entity from one state to another, for example `approve: review → done`. This separation allows Drupal to control valid transitions and permissions instead of just storing arbitrary text.

---

## 2. List field vs `moderation_state`

**Вопрос:** В чём фундаментальная разница между List field со статусами и полем `moderation_state` с точки зрения защиты от неправильного значения?

**Question:** What is the fundamental difference between a List field with statuses and the `moderation_state` field that Content Moderation adds, in terms of what prevents the user from setting an invalid value?

**Ответ:** List field проверяет только, что значение существует среди разрешённых значений списка. `moderation_state` дополнительно связан с **Workflow**, поэтому Drupal проверяет, разрешён ли конкретный переход из текущего состояния. Пользователь не может просто установить любое следующее состояние, если такой transition не разрешён workflow.

**Answer:** A List field checks that the value exists in the configured list. `moderation_state` is connected to a **Workflow**, so Drupal also checks whether the transition is allowed from the current state. A user cannot simply set any next state if the workflow does not allow that transition.

---

## 3. Permissions for transitions

**Вопрос:** Где Content Moderation хранит и проверяет permissions для конкретного transition, например `approve`, и чем это отличается от проверки роли напрямую в controller или form?

**Question:** Where exactly does Content Moderation store and check permissions for a specific transition, such as `approve`, and how does this differ from checking the role directly in the controller or form code?

**Ответ:** Переходы и их настройки определяются в **конфигурации Workflow**, а доступ проверяется через Drupal Access API и permissions Content Moderation, связанные с переходами. Это лучше прямой проверки роли, потому что логика не привязана к конкретной роли и может использоваться одинаково в формах, маршрутах и других местах.

**Answer:** Transitions and their configuration are defined in the **Workflow configuration**, and access is checked through Drupal's Access API and Content Moderation permissions. This is better than checking a specific role directly because the logic is not hard-coded to one role and can be reused by forms, routes, and other parts of Drupal.

---

## 4. Content Moderation/Workflows vs State Machine

**Вопрос:** Почему в этой задаче нужно использовать core Content Moderation и Workflows, а не `drupal/state_machine`?

**Question:** Why do you need to use the core Content Moderation and Workflows modules in this task instead of `drupal/state_machine`?

**Ответ:** Потому что задача требует именно **core Content Moderation** и его интеграции с revisionable content. `state_machine` — полноценное альтернативное решение для конечных автоматов, но это contrib-модуль и он не является частью требуемой архитектуры Drupal Content Moderation.

**Answer:** Because the task specifically requires **core Content Moderation** and its integration with revisionable content. `state_machine` is a valid alternative for implementing state machines, but it is a contrib module and is not part of the required Drupal Content Moderation architecture.

---

## 5. Revisionable Task

**Вопрос:** Почему `task` должен быть revisionable до подключения Content Moderation?

**Question:** Why does the `task` content type have to be configured as revisionable before it can be connected to Content Moderation?

**Ответ:** Content Moderation работает с **revision**, чтобы разные версии контента могли находиться в разных moderation states. Если Task не поддерживает revisions, Content Moderation не сможет корректно хранить и управлять moderated versions.

**Answer:** Content Moderation works with **revisions**, allowing different versions of content to have different moderation states. If a Task is not revisionable, Content Moderation cannot correctly store and manage moderated versions.

---

## 6. Why all four states are published

**Вопрос:** Почему в этой задаче `backlog`, `in_progress`, `review` и `done` отмечены как published/default revision, а не только `done`?

**Question:** Why are all four states marked as published and default revision instead of only `done`?

**Ответ:** Потому что эти состояния описывают **рабочий процесс задачи**, а не её публичность. Task должна оставаться доступной пользователям проекта на всех рабочих этапах. Здесь moderation state используется для workflow, а не как классическая схема `draft → published`.

**Answer:** Because these states describe the **task workflow**, not whether the task should be publicly visible. A Task should remain available to project users in all working stages. Here, moderation states are used for workflow management rather than the traditional `draft → published` model.

---

## 7. Why List text for project type

**Вопрос:** Почему `field_project_type` должен быть List (text) с `kanban`/`scrum`, а не Boolean?

**Question:** Why does `field_project_type` require a List (text) with `kanban`/`scrum` instead of a Boolean field?

**Ответ:** Потому что это **дискретный тип**, который потенциально может расширяться. List field явно хранит machine value `kanban` или `scrum`, поэтому код работает с понятными значениями и легко может быть расширен, например `waterfall`.

**Answer:** Because this is a **discrete type** that may be extended in the future. A List field explicitly stores the machine values `kanban` and `scrum`, making the code clear and allowing additional types such as `waterfall` later.

---

## 8. Why not taxonomy/entity reference

**Вопрос:** Почему запрещено использовать taxonomy field или Entity Reference для project type?

**Question:** Why is it forbidden to use a taxonomy field or Entity Reference for the project type?

**Ответ:** Потому что `kanban` и `scrum` — это **фиксированные технические варианты поведения**, а не самостоятельные сущности или управляемые редакторами категории. Entity Reference добавил бы ненужную зависимость от отдельных entities.

**Answer:** Because `kanban` and `scrum` are **fixed technical behavior types**, not independent entities or editor-managed categories. An Entity Reference would introduce an unnecessary dependency on separate entities.

---

## 9. Why default `kanban`

**Вопрос:** Почему default value должен быть `kanban`, а не пустым?

**Question:** Why should the default value be `kanban` instead of leaving the field empty?

**Ответ:** Потому что существующая система уже основана на Kanban, поэтому `kanban` сохраняет **backward-compatible behavior**. Новые Project nodes автоматически получают корректный тип без дополнительного действия редактора.

**Answer:** Because the existing system is already based on Kanban, so `kanban` preserves **backward-compatible behavior**. New Project nodes automatically receive a valid type without requiring an extra action from the editor.

---

## 10. Required field and existing nodes

**Вопрос:** Почему добавление required field само по себе не ломает существующие Project nodes?

**Question:** Why does adding a new required field not by itself break existing Project nodes?

**Ответ:** Потому что `required` прежде всего применяется при **создании и редактировании формы**. Уже существующие entities не получают значение автоматически только из-за изменения field definition. Поэтому для старых данных требуется отдельная migration/update hook.

**Answer:** Because `required` mainly applies when **creating or editing** an entity. Existing entities do not automatically receive a value just because the field definition changed. Therefore, old data requires a separate migration or update hook.

---

## 11. `hook_update_N()` vs `hook_post_update_NAME()`

**Вопрос:** В чём разница между `hook_update_N()` и `hook_post_update_NAME()` и почему здесь используется `hook_update_N()`?

**Question:** What is the difference between `hook_update_N()` and `hook_post_update_NAME()`, and why is `hook_update_N()` used here?

**Ответ:** `hook_update_N()` является **versioned database update** с числовым идентификатором и предназначен для последовательных schema/data updates. `hook_post_update_NAME()` используется для post-update операций, которые выполняются после обычных numbered updates. Здесь `hook_update_N()` подходит, потому что мы выполняем конкретную версионируемую миграцию существующих данных.

**Answer:** `hook_update_N()` is a **versioned database update** with a numeric identifier and is intended for ordered schema and data updates. `hook_post_update_NAME()` is used for post-update operations that run after numbered updates. Here, `hook_update_N()` is appropriate because we are performing a specific versioned migration of existing data.

---

## 12. Why not `drush scr`

**Вопрос:** Почему нельзя просто написать одноразовый script и запустить его через `drush scr` при deployment?

**Question:** Why can't we simply write a one-time script and run it with `drush scr` during deployment?

**Ответ:** Потому что `hook_update_N()` является частью **version-controlled update system** Drupal. Drupal знает, какие updates уже были выполнены, и может автоматически выполнить нужный update на другом окружении. `drush scr` не предоставляет такой tracking и может быть случайно пропущен или выполнен повторно.

**Answer:** Because `hook_update_N()` is part of Drupal's **version-controlled update system**. Drupal tracks which updates have already been executed and can automatically run the required update on another environment. `drush scr` does not provide this tracking and can be skipped or accidentally executed again.

---

## 13. Why `$sandbox` and batches

**Вопрос:** Почему в `hook_update_N()` нужен `$sandbox` и batch processing?

**Question:** Why is `$sandbox` necessary in `hook_update_N()` and why should entities be processed in batches?

**Ответ:** `$sandbox` позволяет сохранять **progress между запусками одного update**. Batch processing уменьшает потребление памяти и снижает риск timeout на большом количестве entities. Нельзя рассчитывать, что все Project и Task безопасно загрузятся одним `loadMultiple()`.

**Answer:** `$sandbox` stores the **progress of the update between executions**. Batch processing reduces memory usage and the risk of timeouts when many entities exist. We should not assume that all Projects and Tasks can safely be loaded with one `loadMultiple()` call.

---

## 14. Entity API vs direct SQL

**Вопрос:** Почему данные нужно обновлять через Entity API, а не прямым SQL UPDATE?

**Question:** Why should data be updated through the Entity API instead of using direct SQL UPDATE statements?

**Ответ:** Entity API учитывает **field storage, revisions, hooks/events, validation и cache invalidation**. Прямой SQL может изменить значение в таблице, но обойти важную Drupal-логику и оставить систему в неконсистентном состоянии.

**Answer:** The Entity API handles **field storage, revisions, hooks/events, validation, and cache invalidation**. Direct SQL may change the database value but can bypass important Drupal logic and leave the system inconsistent.

---

## 15. Idempotency

**Вопрос:** Как обеспечивается idempotency update?

**Question:** How is update idempotency ensured?

**Ответ:** Update обрабатывает только entities, которым действительно нужно установить новое значение, и проверяет существующие данные перед изменением. Кроме того, Drupal сохраняет выполненный update в системе schema updates, поэтому после успешного выполнения `drush updatedb -y` повторно его не запускает.

**Answer:** The update changes only entities that actually need the new value and checks their existing data before modifying them. Drupal also records completed updates in its schema update system, so `drush updatedb -y` does not run the same successful update again.

---

## 16. Migration based on previous status

**Вопрос:** Почему состояние Task нужно определять по предыдущему `field_status`, а не всем дать `backlog`?

**Question:** Why should the Task state migration be based on the previous `field_status` value instead of assigning `backlog` to every non-migrated Task?

**Ответ:** Потому что migration должна **сохранить существующую бизнес-логику**. Например, старый `working` должен стать `in_progress`, а `completed` — `done`. Если всем установить `backlog`, мы потеряем историческое состояние задач.

**Answer:** Because the migration must **preserve the existing business logic**. For example, old `working` should become `in_progress`, while `completed` should become `done`. Assigning `backlog` to everything would lose the existing task states.

---

## 17. `/admin/reports/status`

**Вопрос:** Что покажет `/admin/reports/status`, если разработчик добавил новый `hook_update_N()`, но не запустил `drush updatedb`?

**Question:** What will `/admin/reports/status` show if a developer added a new `hook_update_N()` but forgot to run `drush updatedb`?

**Ответ:** Drupal покажет, что **database updates are pending** и требуется выполнить `update.php` или `drush updatedb`. Это важный сигнал, потому что код уже ожидает новую структуру или данные, а база данных ещё не синхронизирована с версией кода.

**Answer:** Drupal will show that **database updates are pending** and that `update.php` or `drush updatedb` should be run. This is important because the code expects the new database structure or data, while the database is still at an older version.

---

## 18. Sprints route and Access API

**Вопрос:** Почему доступ к Sprints route реализован через Access API самого route, а не через проверку типа проекта внутри controller?

**Question:** Why is access to the Sprints route implemented through the route's Access API instead of checking the project type inside the controller?

**Ответ:** Потому что access control должен выполняться **до выполнения controller**. Route access callback является стандартным Drupal Access API и позволяет одинаково защищать route независимо от того, кто и каким способом её вызывает.

**Answer:** Because access control should happen **before the controller is executed**. A route access callback is the standard Drupal Access API and protects the route consistently regardless of how it is accessed.

---

## 19. Same condition for tab and route

**Вопрос:** Почему скрытие Sprints tab и проверка route access должны использовать одно условие?

**Question:** Why should hiding the Sprints tab and checking access to the route use the same condition?

**Ответ:** Чтобы **UI и security logic не расходились**. Если tab скрыта, но route доступен, пользователь всё равно может открыть URL напрямую. Если условия одинаковые, интерфейс точно отражает реальную доступность функциональности.

**Answer:** So that **UI and access logic do not diverge**. If the tab is hidden but the route is accessible, a user can still open the URL directly. Using the same condition keeps the interface consistent with actual access.

---

## 20. Why not CSS/JS hiding

**Вопрос:** Почему нельзя использовать `display: none` или `hidden` для Sprints link?

**Question:** Why is it forbidden to use `display: none` or `hidden` in CSS/JS to hide the Sprints link?

**Ответ:** Потому что CSS/JS влияет только на **presentation**, а не на access control. Пользователь всё ещё может знать URL и открыть route напрямую. Кроме того, сервер должен сам решать, имеет ли пользователь доступ.

**Answer:** Because CSS/JS only affects **presentation**, not access control. A user can still know the URL and open the route directly. The server must enforce access independently of the UI.

---

## 21. Project type + node access

**Вопрос:** Почему Sprints access должен учитывать и `field_project_type`, и обычный access к Project node?

**Question:** Why should Sprints access check both `field_project_type` and normal access to the Project node?

**Ответ:** Потому что одного Scrum type недостаточно. Пользователь должен иметь право **просматривать сам Project**, и проект одновременно должен быть Scrum. Это принцип **defense in depth**: оба условия должны быть выполнены.

**Answer:** Because being a Scrum project is not enough. The user must also have permission to **view the Project itself**, and the project must be a Scrum project. Both conditions must be satisfied as **defense in depth**.

---

## 22. Why prepare Sprints extension point

**Вопрос:** Зачем создавать tab и stub route для Scrum заранее, если реальные sprint data будут в следующем блоке?

**Question:** Why prepare a tab and stub route for Scrum functionality in advance if real sprint data will only appear in the next block?

**Ответ:** Это создаёт **extension point** и заранее определяет архитектурную границу для Scrum functionality. Следующий блок сможет добавить sprint logic без изменения базовой структуры Project UI и routing.

**Answer:** It creates an **extension point** and defines the architectural boundary for Scrum functionality in advance. The next block can add sprint logic without redesigning the basic Project UI and routing.

---

## 23. Why conditional project UI

**Вопрос:** Почему недостаточно одного универсального интерфейса Project со всеми элементами?

**Question:** Why is one universal Project interface with all elements not enough?

**Ответ:** Потому что Kanban и Scrum имеют **разные workflows и функциональные элементы**. Например, Sprints относятся к Scrum, но не к Kanban. Интерфейс должен отображать только релевантные функции для конкретного project type.

**Answer:** Because Kanban and Scrum have **different workflows and functional elements**. For example, Sprints belong to Scrum but not to Kanban. The interface should show only functionality relevant to the selected project type.

---

## 24. Changing Kanban → Scrum

**Вопрос:** Почему после изменения `field_project_type` с Kanban на Scrum Sprints должны появиться сразу?

**Question:** Why should changing `field_project_type` from Kanban to Scrum immediately make the Sprints section available?

**Ответ:** Потому что доступность Sprints должна определяться **динамически по текущему значению field**, а не быть захардкожена при создании Project. После сохранения Project новое значение используется access и UI logic автоматически. Rebuild routing cache нужен только при изменении **самой routing configuration**, а не при изменении значения поля.

**Answer:** Because Sprints availability should be determined **dynamically from the current field value**, not hard-coded when the Project is created. After saving the Project, the new value is automatically used by the access and UI logic. A routing cache rebuild is needed only when the **route configuration itself** changes, not when a field value changes.