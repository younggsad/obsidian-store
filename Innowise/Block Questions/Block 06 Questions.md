---
custom-width: 80
---
### 1. Hook в Drupal / Drupal hooks

**RU:** Что такое hook и как Drupal находит его реализации?  
**EN:** What is a hook and how does Drupal find its implementations?

**RU:** Hook — точка расширения Drupal. Core через Module Handler находит соответствующие функции во включённых модулях и вызывает их.  
**EN:** A hook is a Drupal extension point. Core uses the Module Handler to find matching functions in enabled modules and calls them.

---

### 2. Entity insert hooks / Entity insert hooks

**RU:** Чем `hook_ENTITY_TYPE_insert()` отличается от `hook_entity_insert()` и когда использовать каждый вариант?  
**EN:** How is `hook_ENTITY_TYPE_insert()` different from `hook_entity_insert()`, and when should each be used?

**RU:** `hook_ENTITY_TYPE_insert()` работает с конкретным типом entity, например `hook_node_insert()`. `hook_entity_insert()` работает со всеми entity.  
**EN:** `hook_ENTITY_TYPE_insert()` targets one entity type, such as `hook_node_insert()`. `hook_entity_insert()` targets all entity types.

---

### 3. Hook vs Event Subscriber / Procedural vs Object-oriented

**RU:** В чём фундаментальная разница между hook и Event Subscriber?  
**EN:** What is the fundamental difference between a hook and an Event Subscriber?

**RU:** Hook — процедурный вызов функции. Event Subscriber — объектный подход, где dispatcher передаёт объект события subscriber'у.  
**EN:** A hook is a procedural function call. An Event Subscriber is object-oriented and receives an event object from the dispatcher.

---

### 4. Subscriber priority / Hook order

**RU:** Как определяется порядок вызова subscribers и hooks?  
**EN:** How is the execution order of subscribers and hooks determined?

**RU:** Subscribers вызываются по `priority`: большее значение выполняется раньше. Hooks обычно выполняются в порядке модулей, а не по priority subscribers.  
**EN:** Subscribers are called by `priority`: higher values run first. Hooks generally follow module execution order rather than subscriber priority.

---

### 5. Events for logging / Events vs hooks

**RU:** Почему Event лучше hook для логирования TimeLog при большом количестве subscribers?  
**EN:** Why is an Event preferable to a hook for TimeLog logging with many subscribers?

**RU:** Несколько независимых subscribers могут реагировать на одно событие без изменения основного кода создания TimeLog.  
**EN:** Multiple independent subscribers can react to the same event without changing the main TimeLog creation code.

---

### 6. Hook + Event duplication / Avoiding duplicate processing

**RU:** Почему hook и custom event не должны одновременно работать в production?  
**EN:** Why should a hook and custom event not run together in production?

**RU:** Один TimeLog может быть обработан дважды. Проверяем это созданием одного TimeLog и убеждаемся, что лог содержит только одну запись.  
**EN:** One TimeLog could be processed twice. We check this by creating one TimeLog and verifying that only one log entry is created.

---

### 7. Dependency Injection / Event Dispatcher

**RU:** Что нужно внедрить через DI для dispatch custom event и почему не использовать `\Drupal::service()`?  
**EN:** What should be injected via DI to dispatch a custom event, and why not use `\Drupal::service()`?

**RU:** Нужно внедрить `EventDispatcherInterface`. Это делает зависимость явной, улучшает тестируемость и соответствует DI-подходу Drupal.  
**EN:** Inject `EventDispatcherInterface`. This makes the dependency explicit, improves testability, and follows Drupal's DI approach.

---

### 8. Custom Event class / TimeLog event

**RU:** Что должен содержать custom event для создания TimeLog и зачем отдельный PHP-класс?  
**EN:** What should a custom TimeLog creation event contain, and why use a separate PHP class?

**RU:** Event должен содержать созданный TimeLog и необходимые данные. Отдельный класс задаёт собственный тип события и чёткий контракт для subscribers.  
**EN:** The event should contain the created TimeLog and required data. A separate class provides a dedicated event type and clear subscriber contract.

---

### 9. Event Subscriber registration / Service tag

**RU:** Как зарегистрировать Event Subscriber и какой tag нужен?  
**EN:** How is an Event Subscriber registered and which tag is required?

**RU:** Subscriber регистрируется как service в `services.yml` с tag `event_subscriber`.  
**EN:** The subscriber is registered as a service in `services.yml` with the `event_subscriber` tag.

---

# Migrate API

### 10. Migrate API vs PHP script / Migration benefits

**RU:** Зачем использовать Migrate API вместо PHP-скрипта с `Node::create()`?  
**EN:** Why use Migrate API instead of a PHP script with `Node::create()`?

**RU:** Migrate API предоставляет source/process/destination, tracking, повторный запуск и rollback. Это делает импорт воспроизводимым и управляемым.  
**EN:** Migrate API provides source/process/destination, tracking, re-runs, and rollback. This makes the import reproducible and manageable.

---

### 11. Source / Process / Destination

**RU:** Что означают source, process и destination в migration?  
**EN:** What do source, process, and destination mean in a migration?

**RU:** **Source** — откуда берём данные. **Process** — как преобразуем данные. **Destination** — куда сохраняем результат.  
**EN:** **Source** — where data comes from. **Process** — how data is transformed. **Destination** — where the result is stored.

---

### 12. Declarative migration / YAML vs PHP

**RU:** Почему migration должна быть declarative YAML, а не custom PHP class?  
**EN:** Why should a migration be declarative YAML instead of a custom PHP class?

**RU:** YAML проще читать, поддерживать и воспроизводить. Custom class нужна только для сложной или нестандартной логики.  
**EN:** YAML is easier to read, maintain, and reproduce. A custom class is needed only for complex or non-standard logic.

---

### 13. Entity lookup / Project by name

**RU:** Как lookup связывает task с существующим project по имени?  
**EN:** How does lookup link a task to an existing project by name?

**RU:** `entity_lookup` ищет `project` node, у которого `title` совпадает с `project_name`, и возвращает его ID. Это лучше hard-coded ID, потому что не зависит от конкретного NID.  
**EN:** `entity_lookup` finds a `project` node whose `title` matches `project_name` and returns its ID. This is better than a hard-coded ID because it does not depend on a specific NID.

---

### 14. Migration idempotency / Re-import

**RU:** Что происходит при повторном `migrate:import` без `--update`?  
**EN:** What happens when `migrate:import` is run again without `--update`?

**RU:** Уже импортированные source IDs отмечены в migration map, поэтому Drupal не создаёт дубликаты.  
**EN:** Previously imported source IDs are stored in the migration map, so Drupal does not create duplicates.

---

### 15. Migration rollback / Rollback scope

**RU:** Что делает `migrate:rollback` и почему он не удаляет другие tasks?  
**EN:** What does `migrate:rollback` do and why does it not remove other tasks?

**RU:** Rollback удаляет entities, созданные данной migration, используя её migration map. Другие tasks не затрагиваются.  
**EN:** Rollback removes entities created by that migration using its migration map. Other tasks are not affected.

---

### 16. Status normalization / Process pipeline

**RU:** Почему status нужно нормализовать в process pipeline?  
**EN:** Why should status normalization be part of the process pipeline?

**RU:** Source может использовать другие значения. `static_map` преобразует их в допустимые Drupal values до сохранения entity.  
**EN:** The source may use different values. `static_map` converts them to valid Drupal values before the entity is saved.

---

### 17. Idempotency vs Rollback / Migration safety

**RU:** В чём разница между idempotency и rollback и зачем нужны оба?  
**EN:** What is the difference between idempotency and rollback, and why are both needed?

**RU:** **Idempotency** позволяет повторять import без дубликатов. **Rollback** удаляет результаты migration. Для исторического backlog нужны оба механизма.  
**EN:** **Idempotency** allows re-running the import without duplicates. **Rollback** removes migration results. Both are needed for the historical backlog.

---

### 18. migrate_plus / CSV source

**RU:** Какую роль играют `migrate_plus` и `migrate_source_csv` относительно Drupal core?  
**EN:** What roles do `migrate_plus` and `migrate_source_csv` play compared with Drupal core?

**RU:** Core предоставляет базовый Migrate API. `migrate_plus` расширяет migration plugins/configuration, а `migrate_source_csv` добавляет CSV source plugin.  
**EN:** Core provides the basic Migrate API. `migrate_plus` extends migration plugins/configuration, while `migrate_source_csv` adds the CSV source plugin.

---

### 19. Drush migration status / Migration discovery

**RU:** Почему наличие YAML недостаточно и нужно проверять migration через Drush?  
**EN:** Why is the YAML file alone not enough, and why check the migration through Drush?

**RU:** YAML сам по себе не доказывает, что Drupal обнаружил и зарегистрировал migration. `drush migrate:status` подтверждает, что migration доступна для запуска.  
**EN:** A YAML file alone does not prove that Drupal discovered and registered the migration. `drush migrate:status` confirms that the migration is available to run.