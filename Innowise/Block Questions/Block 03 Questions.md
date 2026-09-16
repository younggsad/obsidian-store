---
custom-width: 80
---
## Summary — Block 3

**Block 3 реализует полноценную систему учёта времени в DrupalJira** на базе собственной Content Entity `time_log`. Блок включает создание сущности, программную работу с ней через Entity API и EntityQuery, а также пользовательскую форму списания времени с серверной валидацией и динамическим отображением причины перерасхода.

### Реализация по заданиям

|Задание|Что реализовано|
|---|---|
|**Task 3.1 — Custom TimeLog entity**|Создана собственная Content Entity `time_log` в кастомном модуле. Добавлены поля `task`, `uid`, `hours`, `log_date`, `notes`, `over_estimate_reason`. Отключены **revisions** и **translatability**. Реализованы административная форма создания и административный список TimeLog-записей.|
|**Task 3.2 — Entity API / CRUD / EntityQuery**|Добавлен `TimeLogDebugController` с тремя административными маршрутами. Реализован полный CRUD-цикл через Entity API: `create() → save() → load() → update → delete()`. Через `EntityQuery` реализованы выборка TimeLog по задаче с сортировкой по `log_date` и получение общей суммы списанных часов.|
|**Task 3.3 — Form API**|Создана отдельная форма списания времени `/task/{task}/log-time`. Реализованы поля `hours`, `log_date`, `notes`, checkbox перерасхода и динамическое поле `over_estimate_reason`. Для динамики использован **Drupal States API**, для безопасности — серверная `validateForm()`.|
|**Task 3.4**|Реализована функциональность следующего этапа блока, связанная с использованием TimeLog в проекте и обработкой данных учёта времени. Код интегрирован с существующей сущностью `time_log` и предыдущими механизмами блока.|

### Архитектура

Основная логика блока расположена в **кастомном модуле DrupalJira**, поэтому функциональность не зависит от стандартного Node API для хранения записей времени.

### Что было проверено

- **TimeLog является Content Entity**, а не Config Entity.
- Записи времени сохраняются в базе данных как отдельные сущности.
- `task` и `uid` являются обязательными полями.
- CRUD работает через стандартный **Entity API**, без прямого SQL.
- Выборки выполняются через **EntityQuery**.
- Для несуществующего task используется route entity upcasting, поэтому возвращается **404**.
- Административные debug-маршруты защищены permission **`access administration pages`**.
- Форма списания времени доступна только авторизованному пользователю с необходимым правом.
- Будущая дата списания запрещена.
- `hours <= 0` запрещены.
- `over_estimate_reason` становится обязательным при включённом checkbox.
- Клиентская динамика реализована через **States API без дополнительного JavaScript**.
- После успешного сохранения создаётся `time_log`, связанный с текущей задачей и пользователем.
- Код блока проходит предусмотренные **PHPCS и PHPStan** проверки без новых ошибок.

### Итог блока 3

**Block 3 перевёл DrupalJira от простого хранения задач к полноценному учёту фактически затраченного времени.** В результате появилась отдельная доменная сущность `time_log`, программный API для работы с ней и удобный пользовательский сценарий списания времени с необходимыми проверками.

**Основные Drupal-механизмы, продемонстрированные в блоке:** Content Entity API, Entity Storage, Entity API CRUD, EntityQuery, Routing с entity upcasting, Form API, States API, access control и серверная валидация.


## Summary — Block 3

**Block 3 implements a complete time-tracking system for DrupalJira** based on a custom `time_log` Content Entity. The block covers entity creation, programmatic CRUD operations, EntityQuery-based data retrieval, and a dedicated time write-off form with client-side and server-side validation.

### Implementation by task

|Task|Implementation|
|---|---|
|**Task 3.1 — Custom TimeLog entity**|Created a custom `time_log` Content Entity inside a custom Drupal module. Added the required fields: `task`, `uid`, `hours`, `log_date`, `notes`, and `over_estimate_reason`. Revisions and translatability were explicitly disabled. Generated administrative create and list functionality for TimeLog records.|
|**Task 3.2 — Entity API / CRUD / EntityQuery**|Implemented `TimeLogDebugController` with three administrative routes. Demonstrated the complete CRUD lifecycle using Entity API: `create() → save() → load() → update → delete()`. Added EntityQuery operations for retrieving TimeLogs belonging to a specific task and calculating the total number of logged hours.|
|**Task 3.3 — Form API**|Implemented a dedicated time write-off form at `/task/{task}/log-time`. Added `hours`, `log_date`, `notes`, an over-estimate checkbox, and the `over_estimate_reason` field. The dynamic field visibility is handled by Drupal's **States API**, while server-side validation prevents invalid dates, zero/negative hours, and missing over-estimate reasons.|
|**Task 3.4**|Implemented the functionality required for the next stage of the time-tracking block, integrating it with the existing `time_log` entity and the functionality implemented in the previous tasks.|

### Architecture

The functionality is implemented inside a **custom DrupalJira module** rather than using standard nodes for time records.

### Validation and testing

The implementation covers the main acceptance criteria from the tasks:

- `TimeLog` is implemented as a **Content Entity**, not a Config Entity.
- `task` and `uid` are required fields.
- TimeLog records are stored as separate entities in the database.
- Entity operations use the standard **Entity API**, without direct SQL.
- Data retrieval uses **EntityQuery**.
- Route entity upcasting provides proper **404 responses** for non-existent or invalid task IDs.
- Administrative debug routes are protected by the `access administration pages` permission.
- The time write-off form is restricted to authorized users.
- Future `log_date` values are rejected.
- `hours <= 0` values are rejected.
- `over_estimate_reason` becomes required when the over-estimate checkbox is enabled.
- Dynamic form behavior is implemented with **Drupal States API**, without custom JavaScript.
- Successfully submitted forms create a `time_log` entity associated with the current task and user.
- The implementation passes the configured **PHPCS and PHPStan** checks without new errors.

### Block 3 result

**Block 3 extends DrupalJira from a task management system into a system capable of tracking actual time spent on tasks.** It introduces a dedicated domain entity, programmatic APIs for working with time records, and a user-friendly time write-off workflow with appropriate validation.

The main Drupal concepts demonstrated in this block are **Content Entity API, Entity Storage, Entity API CRUD, EntityQuery, route parameter entity upcasting, Form API, States API, access control, and server-side validation**.

## QUESTIONS

## 1. Как Content Entity принципиально отличается от Config Entity и почему для TimeLog была выбрана Content Entity?

**1. How is a content entity fundamentally different from a config entity, and why was the content entity chosen for TimeLog?**

**Ответ на русском:**  
Content Entity хранит **данные сайта**, например задачи и записи времени. Config Entity хранит **конфигурацию сайта**, например настройки и типы сущностей. `TimeLog` хранит реальные записи о затраченном времени пользователей, поэтому это **Content Entity**.

**Answer in English:**  
A Content Entity stores **site data**, such as tasks and time records. A Config Entity stores **site configuration**, such as settings and entity types. `TimeLog` stores real time records created by users, so it is a **Content Entity**.

---

## 2. Чем Content Entity отличается от node (стандартного контентного типа)? Почему обычный content type не подошёл для TimeLog?

**2. How is a content entity different from a node (standard content type)? Why wasn't the usual content type suitable for TimeLog?**

**Ответ на русском:**  
Node — это один из видов Content Entity, предназначенный для обычного контента Drupal. `TimeLog` — это специальная техническая сущность для учёта времени. Ей не нужны revisions, translations и обычные списки контента, поэтому отдельная Content Entity подходит лучше.

**Answer in English:**  
A node is one type of Content Entity used for normal Drupal content. `TimeLog` is a special entity for tracking work time. It does not need revisions, translations, or normal content lists, so a separate Content Entity is more suitable.

---

## 3. Что означают `revisionable = FALSE` и `translatable = FALSE` в annotation/attribute сущности и какие практические последствия это имеет?

**3. What do `revisionable = FALSE` and `translatable = FALSE` mean in an entity annotation/attribute, and what practical implications does this have?**

**Ответ на русском:**  
`revisionable = FALSE` означает, что у записи **нет истории версий**. `translatable = FALSE` означает, что запись **нельзя переводить на другие языки**. Поэтому у TimeLog нет revision history и translation UI.

**Answer in English:**  
`revisionable = FALSE` means that the entity has **no revision history**. `translatable = FALSE` means that the entity **cannot have translations**. Therefore, TimeLog has no revision history or translation UI.

---

## 4. Почему в задании явно указано отключить эти флаги, а не оставить значения по умолчанию от генератора?

**4. Why does the task explicitly indicate to disable these flags, rather than leaving the default values from the generator?**

**Ответ на русском:**  
Чтобы явно показать, что эти возможности **не нужны TimeLog**. Это также подтверждает требования задания на уровне определения сущности, а не просто случайное поведение генератора.

**Answer in English:**  
It explicitly shows that these features are **not needed for TimeLog**. It also makes the requirement clear at the entity definition level instead of depending on the generator's defaults.

---

## 5. Почему поля `task` и `uid` реализованы как entity reference, а не как обычные text/numeric поля с ID?

**5. Why are the `task` and `uid` fields in TimeLog implemented as an entity reference, and not as a regular text/numeric field with an ID?**

**Ответ на русском:**  
Entity Reference связывает TimeLog непосредственно с **Drupal-сущностью** — задачей или пользователем. Drupal понимает эту связь и может использовать её для загрузки связанных сущностей и EntityQuery. Простое число хранило бы только ID без такой связи.

**Answer in English:**  
An Entity Reference connects TimeLog directly to a **Drupal entity**, such as a task or user. Drupal understands this relationship and can use it to load related entities and build EntityQuery operations. A simple number would only store an ID.

---

## 6. Какая разница между полем `hours` типа decimal и полем `log_date` типа date without time с точки зрения выбора типа поля? Почему дате списания не нужна временная часть?

**6. What is the difference between a `hours` decimal field and a `log_date` date without time field in terms of field type selection? Why doesn't the write-off date need a time component?**

**Ответ на русском:**  
`hours` хранит **числовое значение**, например `2.5`, поэтому используется decimal. `log_date` хранит только **календарную дату**, например `2026-09-14`. Для задания важно, в какой день было списано время, а точное время не требуется.

**Answer in English:**  
`hours` stores a **numeric value**, for example `2.5`, so a decimal field is used. `log_date` stores only a **calendar date**, such as `2026-09-14`. The task needs the day of the write-off, not the exact time.

---

## 7. Почему используется CLI entity generator (`drush generate:entity:content`), а не написание Content Entity вручную с нуля?

**7. Why use the CLI entity generator (`drush generate:entity:content`) rather than manually write the content entity code from scratch?**

**Ответ на русском:**  
Генератор создаёт **правильную базовую структуру** сущности: класс, routing, forms, lists и другие необходимые файлы. После этого ненужный сгенерированный код можно удалить и адаптировать сущность под TimeLog.

**Answer in English:**  
The generator creates the **correct basic structure** for the entity, including the class, routing, forms, lists, and other required files. Then unnecessary generated code can be removed and the entity can be adapted for TimeLog.

---

## 8. Почему route `drupaljira.timelog_debug.crud` использует upcasting `{task}` в `NodeInterface` через `options.parameters`, а не получает ID и не загружает node вручную?

**8. Why does the `drupaljira.timelog_debug.crud` route use upcasting of the `{task}` parameter to `NodeInterface` via `options.parameters` (`type: entity:node`, `bundle: task`), rather than receiving and manually loading the node ID inside the controller?**

**Ответ на русском:**  
Entity upcasting позволяет Drupal **автоматически превратить ID в объект Node**. Кроме того, Drupal проверяет существование node и её bundle. Поэтому контроллер получает сразу готовый `NodeInterface`, а лишний код `load()` не нужен.

**Answer in English:**  
Entity upcasting allows Drupal to **automatically convert the ID into a Node object**. Drupal also checks that the node exists and has the correct bundle. The controller therefore receives a ready `NodeInterface` object and does not need manual loading.

---

## 9. Какая связь между upcasting route parameter и требованием, что несуществующий `{task}` должен возвращать 404, а не PHP fatal error?

**9. What's the connection between upcasting a route parameter and the task's requirement that opening a non-existent `{task}` should return a 404 rather than a PHP fatal error?**

**Ответ на русском:**  
При upcasting Drupal пытается найти сущность. Если node не существует, Drupal **останавливает обработку маршрута и возвращает 404**. Поэтому контроллер не получает неправильный объект и не вызывает ошибку при работе с ним.

**Answer in English:**  
During upcasting, Drupal tries to find the entity. If the node does not exist, Drupal **stops route processing and returns 404**. The controller therefore does not receive an invalid object and does not produce a PHP error.

---

## 10. Почему маршруты `timelog-debug/*` защищены требованием `access administration pages`, а не доступны всем авторизованным пользователям?

**10. Why are `timelog-debug/*` routes protected by the `access administration pages` requirement rather than being left available to all authenticated users?**

**Ответ на русском:**  
Это **debug/demo-инструменты администратора**, а не обычные пользовательские страницы. Поэтому доступ к ним ограничен административным permission, чтобы обычные пользователи не могли запускать CRUD-тесты и просматривать служебную информацию.

**Answer in English:**  
These are **debug/demo tools for administrators**, not normal user pages. Therefore, access is limited by an administrative permission so regular users cannot run CRUD tests or see internal information.

---

## 11. Чем EntityQuery отличается от прямого SQL-запроса (`Database::getConnection()`) к таблице entity и почему прямой доступ к базе данных запрещён?

**11. How does EntityQuery differ from a direct SQL query (`Database::getConnection()`) to an entity table, and why is it explicitly prohibited in the task to access the database directly?**

**Ответ на русском:**  
EntityQuery работает через **Drupal Entity API** и знает структуру сущностей и их полей. Прямой SQL работает непосредственно с таблицами базы данных. В задании EntityQuery требуется, чтобы код использовал стандартный Drupal API и не зависел напрямую от структуры таблиц.

**Answer in English:**  
EntityQuery works through the **Drupal Entity API** and understands entities and their fields. Direct SQL works directly with database tables. The task requires EntityQuery so the code uses the standard Drupal API instead of depending directly on database tables.

---

## 12. Почему EntityQuery или aggregation query достаточно для подсчёта суммы часов, а не обязательно делать `loadMultiple()` и складывать значения в PHP?

**12. Why is EntityQuery (or an aggregation query based on it) sufficient to calculate the sum of hours, and not necessarily `entityTypeManager()->getStorage('time_log')->loadMultiple()` followed by summation in PHP?**

**Ответ на русском:**  
Для суммы не нужно загружать все объекты. Запрос может сразу получить **одно итоговое значение** из базы данных. Это уменьшает количество данных, передаваемых в PHP, и не требует создавать много объектов сущностей.

**Answer in English:**  
There is no need to load every entity just to calculate a sum. A query can return **one total value** from the database. This reduces the data passed to PHP and avoids creating many entity objects.

---

## 13. Почему контроллер возвращает render array (`#markup`/`#theme => 'table'`), а не выводит HTML через `echo`/`print`?

**13. Why does the controller return a render array (`#markup`/`#theme => 'table'`) rather than output HTML directly via `echo`/`print`?**

**Ответ на русском:**  
Render array соответствует **стандартному способу Drupal формировать страницы**. Drupal сам обрабатывает и отображает результат. `echo` и `print` смешивали бы бизнес-логику контроллера с непосредственным выводом HTML.

**Answer in English:**  
A render array follows the **standard Drupal way of building pages**. Drupal processes and renders the result itself. Using `echo` or `print` would mix controller logic with direct HTML output.

---

## 14. Почему метод `create()` storage entity не сохраняет запись сразу, а требует отдельного вызова `save()`?

**14. Why does the `create()` method of a storage entity not save the record immediately, but requires a separate call to `save()`?**

**Ответ на русском:**  
`create()` только **создаёт объект сущности в памяти** и устанавливает его поля. `save()` отдельно отвечает за запись объекта в базу данных. Это позволяет сначала изменить данные, а затем сохранить готовую сущность.

**Answer in English:**  
`create()` only **creates the entity object in memory** and sets its fields. `save()` performs the actual database operation. This allows the entity to be prepared or changed before it is saved.

---

## 15. В чём разница между access logic через `hook_form_alter()` и явной проверкой permission в controller/route, и почему в Task 3.2 доступ ограничен через route requirement (`_permission`)?

**15. What's the difference between `hook_form_alter()`-like access logic and explicit permission checks in a controller/route - why in Problem 3.2 is access restricted via a route requirement (`_permission`) rather than a check inside the controller method body?**

**Ответ на русском:**  
`hook_form_alter()` изменяет **форму**, а route requirement определяет, **может ли пользователь вообще открыть маршрут**. В Task 3.2 это правильнее делать на уровне маршрута: Drupal проверяет permission до запуска controller.

**Answer in English:**  
`hook_form_alter()` changes a **form**, while a route requirement controls **whether the user can access the route at all**. In Task 3.2, the route requirement is better because Drupal checks the permission before calling the controller.

---

## 16. Почему форма списания времени реализована через `FormBase`, а не через `ConfirmFormBase`?

**16. Why is the time write-off form implemented through `FormBase`, and not through `ConfirmFormBase`?**

**Ответ на русском:**  
`FormBase` предназначен для **обычных пользовательских форм с вводом данных**. `ConfirmFormBase` используется для подтверждения действия, например удаления. Списание времени — это форма с несколькими полями, а не confirmation form.

**Answer in English:**  
`FormBase` is designed for **normal forms where users enter data**. `ConfirmFormBase` is used for confirmation actions, such as deletion. Time write-off is a data-entry form, not a confirmation form.

---

## 17. Как работает States API (`#states`) для элемента “Excess Reason” и почему это client-side решение?

**17. How does the States API (`#states`) work on the “Excess Reason” element, and why is this a client-side solution and not a server-side one?**

**Ответ на русском:**  
`#states` связывает состояние поля с другим элементом. Если checkbox отмечен, поле `Excess Reason` становится **видимым и обязательным** прямо в браузере. Это client-side поведение, потому что оно происходит без отправки формы на сервер.

**Answer in English:**  
`#states` connects the state of one form element to another. When the checkbox is checked, the `Excess Reason` field becomes **visible and required** in the browser. It is client-side behavior because it happens without submitting the form to the server.

---

## 18. Почему States API недостаточно и зачем нужна повторная server-side validation в `validateForm()`?

**18. Why is the States API not enough, and why does the task explicitly require duplicate server-side validation in `validateForm()` for the Excess Reason field and other restrictions?**

**Ответ на русском:**  
States API управляет интерфейсом, но **не является защитой данных на сервере**. Пользователь может отправить изменённый HTTP-запрос. Поэтому сервер должен самостоятельно проверить `reason`, дату и часы перед сохранением.

**Answer in English:**  
The States API controls the interface, but it is **not server-side data protection**. A user can send a modified HTTP request. Therefore, the server must validate the reason, date, and hours before saving the record.

---

## 19. Почему проверки `log_date` не в будущем и `hours > 0` находятся в `validateForm()`, а не, например, в constraints самого TimeLog entity?

**19. Why are the `log_date` not in the future and `hours > 0` checks implemented in `validateForm()` rather than, for example, TimeLog entity field level constraints?**

**Ответ на русском:**  
В рамках этого задания эти проверки относятся именно к **сценарию формы списания времени**. `validateForm()` позволяет проверить данные непосредственно перед созданием TimeLog и показать понятную ошибку пользователю.

**Answer in English:**  
In this task, these checks are part of the **time write-off form workflow**. `validateForm()` checks the submitted values before creating the TimeLog and allows the form to show a clear validation error to the user.

---

## 20. В чём разница между этапами `validateForm()` и `submitForm()` в Form API и почему создание TimeLog происходит в `submitForm()`?

**20. What is the difference between the `validateForm()` and `submitForm()` stages in the Form API lifecycle, and why does the TimeLog entry occur at `submitForm()`?**

**Ответ на русском:**  
`validateForm()` **проверяет данные** и может остановить отправку при ошибке. `submitForm()` выполняется после успешной проверки и выполняет основное действие. Поэтому TimeLog создаётся именно там.

**Answer in English:**  
`validateForm()` **checks the submitted data** and can stop the submission if there is an error. `submitForm()` runs after successful validation and performs the main action. Therefore, the TimeLog is created there.

---

## 21. Почему при отправке формы TimeLog привязывается к текущему пользователю программно в `submitForm()`, а не через отдельное поле выбора пользователя?

**21. Why is the TimeLog entry when submitting a form bound to the currently logged in user programmatically (in `submitForm()`) rather than via a separate user select field on the form?**

**Ответ на русском:**  
TimeLog должен показывать, **кто реально списал время**. Пользователь не должен выбирать другого автора вручную. Поэтому ID текущего пользователя берётся из текущей Drupal-сессии и сохраняется в `uid`.

**Answer in English:**  
TimeLog should record **who actually logged the time**. The user should not manually select another author. Therefore, the current user's ID is taken from the Drupal session and stored in `uid`.

---

## 22. В чём разница между `hook_form_alter()` и `hook_form_BASE_FORM_ID_alter()`/`hook_form_FORM_ID_alter()`, и почему для node edit form типа `task` предпочтительнее более конкретный hook?

**22. What is the difference between `hook_form_alter()` and `hook_form_BASE_FORM_ID_alter()`/`hook_form_FORM_ID_alter()`, and why is it preferable to the generic `hook_form_alter()` for a node edit form like `task`?**

**Ответ на русском:**  
`hook_form_alter()` применяется **ко многим формам**, а более конкретные hooks работают только с определённой базовой или конкретной формой. Для формы редактирования task это уменьшает риск случайно изменить другие формы.

**Answer in English:**  
`hook_form_alter()` can affect **many forms**, while the more specific hooks target a particular base form or form ID. For the task node edit form, this reduces the risk of changing unrelated forms.

---

## 23. Почему alter-hook должен добавлять ссылку “Remove time” только для nodes типа `task`, а не для всех content types — и как это технически гарантируется?

**23. Why should the alter-hook add the “Remove time” link only for nodes of type `task`, and not for all content types - how is this technically guaranteed?**

**Ответ на русском:**  
Проверяется **bundle node**, например `$node->bundle() === 'task'`. Только после этой проверки элемент формы добавляется. Поэтому для `project` и других типов контента ссылка не появляется.

**Answer in English:**  
The node **bundle** is checked, for example with `$node->bundle() === 'task'`. The form element is added only after this check, so it does not appear for `project` or other content types.

---

## 24. Почему task ID для создания `href` берётся из node entity в контексте формы (`$form_state`/entity form), а не из global request state, например текущего URL?

**24. Why is the task ID for generating a `href` link taken from the node entity in the form context (`$form_state`/entity form), and not from the global request state (for example, the current URL)?**

**Ответ на русском:**  
Форма редактирования уже содержит **конкретную node entity**. Поэтому ID нужно брать из самой сущности. Это надёжнее, чем анализировать URL, потому что URL может иметь другой формат или дополнительные параметры.

**Answer in English:**  
The edit form already contains the **specific node entity** being edited. Therefore, its ID should be taken from the entity itself. This is more reliable than parsing the current URL, which can have a different format or extra parameters.

---

## 25. Почему при создании нового task, который ещё не сохранён, ссылка не должна отображаться или не должна приводить к ошибке?

**25. Why on the form for creating a new task (the node has not yet been saved) the link should either not be displayed or not lead to an error - what is the technical reason behind this requirement?**

**Ответ на русском:**  
Новая node ещё **не имеет ID в базе данных**. Ссылка `/task/{id}/log-time` требует существующую task. Поэтому нельзя строить такую ссылку с пустым или несуществующим ID.

**Answer in English:**  
A new node **does not have a database ID yet**. The `/task/{id}/log-time` route requires an existing task. Therefore, the link should not be created with an empty or invalid ID.

---

## 26. Почему ссылка создаётся через `Url::fromRoute()` с route parameters, а не через конкатенацию строки `/task/` . `$nid` . `/log-time`?

**26. Why is the link formed through `Url::fromRoute()` with route parameters, and not through concatenation of a string like `'/task/' . $nid . '/log-time'`?**

**Ответ на русском:**  
`Url::fromRoute()` использует **имя Drupal route** и передаёт параметры отдельно. Drupal сам строит правильный URL. Это безопаснее и лучше соответствует Drupal Routing API.

**Answer in English:**  
`Url::fromRoute()` uses the **Drupal route name** and passes route parameters separately. Drupal builds the correct URL. This is safer and follows the Drupal Routing API.

---

## 27. Как элемент, добавленный alter hook в node edit form, связан со всей формой (`node_form`) и почему для этого не нужно переопределять controller или route редактирования?

**27. How does an element added via an alter hook to a node's edit form relate to the overall form (`node_form`) - why doesn't this require overriding the edit form's controller or route?**

**Ответ на русском:**  
Alter hook получает **готовый render/form array** node form и добавляет в него новый элемент. Основной controller и route продолжают работать как раньше. Поэтому не нужно создавать собственную форму редактирования или переопределять стандартный route.

**Answer in English:**  
The alter hook receives the **existing node form array** and adds a new element to it. The original controller and route continue to work normally. Therefore, there is no need to replace the standard edit form or route.