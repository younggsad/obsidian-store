---
custom-width: 80
---
## 1. Render Array

### Question

**Что такое Render Array и почему в Drupal принято строить вывод с его помощью, а не объединять HTML-строки в PHP?**

**What is a Render Array and why is it customary in Drupal to build output using it instead of concatenating HTML strings in PHP?**

### Answer

Render Array — это структурированный PHP-массив, который описывает, что Drupal должен отрендерить. Он позволяет использовать **Render API, Twig, Theme API, cache metadata и theme overrides**, поэтому presentation logic не смешивается с PHP-кодом.

A Render Array is a structured PHP array that describes what Drupal should render. It allows Drupal to use the **Render API, Twig, Theme API, cache metadata, and theme overrides**, keeping presentation logic separate from PHP code.

---

## 2. `#markup` и `#theme`

### Question

**В чём разница между `#markup` и `#theme` в Render Array и когда следует использовать каждый из этих ключей?**

**What is the difference between `#markup` and `#theme` in a Render Array, and when is each key appropriate?**

### Answer

`#markup` используется для небольшого готового текста или markup. `#theme` используется для **тематизированного компонента**, который должен быть отрендерен через theme hook и обычно Twig-шаблон.

`#markup` is used for a small piece of ready-to-render text or markup. `#theme` is used for a **themed component** that should be rendered through a theme hook and usually a Twig template.

---

## 3. `hook_theme()`

### Question

**Почему в Task 5.1 нужно зарегистрировать новый theme hook через `hook_theme()`, а не просто напрямую отрендерить Twig-шаблон с помощью `twig_render_template()` или чтения файла?**

**Why should Task 5.1 register a new theme hook using `hook_theme()` instead of directly rendering the Twig template with `twig_render_template()` or reading the file?**

### Answer

`hook_theme()` подключает шаблон к **Drupal Theme API**. Это даёт поддержку preprocess, theme overrides, theme registry и стандартного процесса рендеринга. Прямой вызов Twig обходит эту архитектуру.

`hook_theme()` integrates the template with Drupal's **Theme API**. It provides preprocess functions, theme overrides, the theme registry, and the normal rendering process. A direct Twig call bypasses this architecture.

---

## 4. Поиск Twig-шаблона

### Question

**Как Drupal находит шаблон `drupaljira-project-stats.html.twig`, если он объявлен в модуле, но файл с таким же именем также находится в `templates/` активной темы?**

**How does Drupal find the `drupaljira-project-stats.html.twig` template if it is declared in a module, but a file with the same name also exists in the `templates/` directory of the active theme?**

### Answer

Модуль регистрирует theme hook и исходный шаблон через `hook_theme()`. После построения theme registry активная тема может **переопределить шаблон**, поэтому версия из активной темы используется вместо версии из модуля.

The module registers the theme hook and the original template through `hook_theme()`. After the theme registry is built, the active theme can **override the template**, so the theme version is used instead of the module version.

---

## 5. `hook_preprocess_HOOK()`

### Question

**Почему в этом блоке подготовка переменных должна происходить строго в `hook_preprocess_HOOK()`, а не внутри Twig-шаблона?**

**Why should variable preparation in this block happen strictly in `hook_preprocess_HOOK()` instead of inside the Twig template?**

### Answer

Preprocess отвечает за **подготовку данных для presentation layer**, а Twig — за их отображение. Поэтому расчёты, форматирование и получение данных из сервисов должны находиться в PHP, а Twig должен оставаться простым.

Preprocess is responsible for **preparing data for the presentation layer**, while Twig is responsible for displaying it. Therefore, calculations, formatting, and service calls should stay in PHP, while Twig should remain simple.

---

## 6. Уже отформатированные значения

### Question

**Почему в Task 5.1 preprocess должен передавать в шаблон уже отформатированные значения, а не исходные числа из сервиса? Почему это важно с точки зрения границ компонентов?**

**Why does Task 5.1 require preprocess to pass already formatted values to the template instead of raw numbers from the service? Why is this important in terms of component boundaries?**

### Answer

Это сохраняет чёткую **границу ответственности**. `TaskStatService` рассчитывает числа, preprocess подготавливает данные для отображения, а Twig отвечает только за markup.

This creates a clear **separation of responsibilities**. `TaskStatService` calculates numbers, preprocess prepares display values, and Twig is responsible only for markup.

---

## 7. Отключение модуля

### Question

**Что произойдёт с блоком “Project Statistics”, если отключить модуль, который регистрирует его theme hook, и почему это должно происходить без fatal error?**

**What happens to the “Project Statistics” block if the module that registers its theme hook is disabled, and why should this happen without a fatal error?**

### Answer

Компонент, предоставляемый этим модулем, должен исчезнуть, потому что его код больше недоступен. При этом не должно быть **fatal error**, потому что Drupal должен корректно обработать отсутствие компонента.

The component provided by this module should disappear because its implementation is no longer available. However, there should be **no fatal error**, because Drupal should handle the missing component gracefully.

---

## 8. Архитектурная разница

### Question

**В чём архитектурная разница между theme hook с preprocess и простым вызовом сервиса и возвратом строки внутри `build()` блока?**

**What is the architectural difference between a theme hook with preprocess and simply calling a service and returning a string inside a block's `build()` method?**

### Answer

Theme hook разделяет **данные, подготовку и presentation**: service → preprocess → Twig. При возврате HTML-строки module code одновременно содержит application logic и presentation logic, поэтому его сложнее переопределять и поддерживать.

A theme hook separates **data, preparation, and presentation**: service → preprocess → Twig. When an HTML string is returned, the module contains both application logic and presentation logic, making it harder to override and maintain.

---

# Task 5.2 — Cache API

## 9. Cache Tags, Cache Contexts и `max-age`

### Question

**Объясните разницу между Cache Tags, Cache Contexts и `max-age`: за что отвечает каждый из этих трёх типов cache metadata?**

**Explain the difference between Cache Tags, Cache Contexts, and `max-age` — what is each of these three types of cache metadata responsible for?**

### Answer

**Cache Tags** определяют, какие cache entries нужно инвалидировать при изменении данных. **Cache Contexts** определяют, для каких условий создаются разные cache variants. **`max-age`** определяет, как долго cache считается действительным.

**Cache Tags** define which cache entries should be invalidated when data changes. **Cache Contexts** define when different cache variants are needed. **`max-age`** defines how long the cached data remains valid.

---

## 10. Custom Cache Tag

### Question

**Почему для инвалидирования Project Statistics был выбран custom cache tag вида `drupaljira_project_stats:{nid}`, а не, например, общий tag `node:{nid}`?**

**Why was a custom cache tag such as `drupaljira_project_stats:{nid}` chosen to invalidate Project Statistics instead of the general `node:{nid}` tag?**

### Answer

Потому что нам нужно инвалидировать именно **кеш статистики конкретного проекта**, а не весь кеш, связанный с node. Это делает инвалидирование более точным.

Because we need to invalidate only the **statistics cache of a specific project**, not all cached data related to the node. This makes cache invalidation more precise.

---

## 11. `Cache::invalidateTags()`

### Question

**Почему инвалидирование кеша при сохранении или удалении TimeLog должно выполняться через `\Drupal\Core\Cache\Cache::invalidateTags()`, а не через `drush cr` или полную очистку кеша сайта?**

**Why should cache invalidation when saving or deleting a TimeLog use `\Drupal\Core\Cache\Cache::invalidateTags()` instead of `drush cr` or clearing the entire site cache?**

### Answer

`invalidateTags()` инвалидирует только связанные cache entries. `drush cr` очищает кеш значительно шире и является административной операцией, а не частью обычной application logic.

`invalidateTags()` invalidates only the related cache entries. `drush cr` clears much more cache and is an administrative operation, not part of normal application logic.

---

## 12. Изоляция проектов

### Question

**Как на практике проверить, что cache tags не пересекаются между проектами A и B, и почему это важно?**

**How can we check in practice that cache tags do not overlap between projects A and B, and why is this important?**

### Answer

Нужно прогреть кеш для проектов A и B, затем изменить TimeLog проекта A. Статистика A должна пересчитаться, а кеш B должен остаться действительным.

We should warm the cache for projects A and B and then change a TimeLog in project A. Project A should be recalculated, while the cache for project B should remain valid.

Это доказывает, что invalidation происходит только для нужного проекта.

This proves that invalidation affects only the required project.

---

## 13. Cache Bubbling

### Question

**Что такое cache bubbling в Render Array и зачем он нужен?**

**What is cache bubbling in a Render Array and why is it needed?**

### Answer

Cache bubbling — это передача cache metadata от дочернего Render Array к родительскому Render Array. Благодаря этому Drupal знает, от каких данных зависит общий результат и как его кешировать.

Cache bubbling means that cache metadata from a child Render Array is propagated to its parent Render Array. This allows Drupal to know what the complete output depends on and how it should be cached.

---

## 14. Cache Context `user`

### Question

**Почему для Kanban Board выбран cache context `user` или `user.roles`, а не полное отключение кеша через `#cache => ['max-age' => 0]`?**

**Why is the `user` or `user.roles` cache context selected for the Kanban Board instead of completely disabling caching with `#cache => ['max-age' => 0]`?**

### Answer

Kanban нужно **кешировать**, но отдельно для разных пользователей или ролей. Cache context создаёт разные cache variants. `max-age: 0` полностью отключил бы полезное кеширование.

The Kanban board should still be **cached**, but separately for different users or roles. A cache context creates different cache variants. `max-age: 0` would completely disable useful caching.

---

## 15. `user` vs `user.roles`

### Question

**По какому критерию нужно выбирать между context `user` и более узким `user.roles`, и почему более узкий context обычно предпочтительнее?**

**By what criterion should we choose between the `user` context and the narrower `user.roles` context, and why is the narrower context generally preferable?**

### Answer

Нужно определить, **от чего реально зависит output**. Если он зависит от конкретного пользователя, нужен `user`. Если только от его роли — достаточно `user.roles`. Более узкий context создаёт меньше cache variants.

We should determine **what the output actually depends on**. If it depends on the individual user, use `user`. If it depends only on the user's role, `user.roles` is enough. A narrower context creates fewer cache variants.

---

## 16. Проверка cache context

### Question

**Как на практике проверить, что cache context действительно изменяет output, а не просто указан в `#cache['contexts']`?**

**How can we check in practice that the cache context actually changes the output and is not only registered in `#cache['contexts']`?**

### Answer

Нужно открыть один и тот же Kanban от имени двух пользователей с разными assigned tasks. Если каждый пользователь видит только свои задачи как `my task`, значит Drupal использует разные cache variants.

We should open the same Kanban board as two users with different assigned tasks. If each user sees only their own tasks marked as `my task`, Drupal is using different cache variants.

---

## 17. Anonymous User

### Question

**Почему acceptance criteria отдельно проверяет поведение anonymous user без session на Kanban Board?**

**Why does the acceptance criteria separately check the behavior of an anonymous user without a session on the Kanban Board?**

### Answer

Anonymous user не имеет обычной authenticated identity. Поэтому нужно убедиться, что board работает без PHP errors и не показывает персональную отметку `my task`.

An anonymous user does not have a normal authenticated identity. Therefore, we need to verify that the board works without PHP errors and does not show a personal `my task` marker.

---

## 18. Entity Hooks и Event Subscriber

### Question

**В чём разница между двумя способами инвалидирования из Task 5.2 — `hook_ENTITY_TYPE_insert/update/delete` и Event Subscriber для entity events? Когда лучше использовать каждый из них?**

**What is the difference between the two invalidation methods mentioned in Task 5.2 — `hook_ENTITY_TYPE_insert/update/delete` and an Event Subscriber for entity events? When is each approach better?**

### Answer

Entity hooks — простой Drupal-specific способ реагировать на изменения конкретного entity type. Event Subscriber — более объектно-ориентированный подход, удобный для отдельного класса и нескольких связанных событий.

Entity hooks are a simple Drupal-specific way to react to changes of a specific entity type. An Event Subscriber is a more object-oriented approach, useful when logic belongs in a separate class or handles several related events.

---

# Task 5.3 — DurationFormatter

## 19. Что было продублировано?

### Question

**Что именно было продублировано в Task 5.3 и, что самое важное, что НЕ было продублировано?**

**What exactly was duplicated in Task 5.3 and, most importantly, what was NOT duplicated?**

### Answer

Было продублировано **форматирование времени**: rounding, unit suffix и сборка human-readable строки. Сам расчёт часов не дублировался — все компоненты продолжали получать числа через `TaskStatService`.

The duplicated part was **time formatting**: rounding, unit suffixes, and building human-readable strings. The actual hour calculation was not duplicated because all components continued to get numbers from `TaskStatService`.

---

## 20. Ответственность `DurationFormatter`

### Question

**Почему `DurationFormatter` не должен обращаться к `TaskStatService` и сам выполнять aggregation или calculation? Какую границу ответственности это сохраняет?**

**Why should `DurationFormatter` not access `TaskStatService` or perform aggregation or calculations itself? What responsibility boundary does this preserve?**

### Answer

`TaskStatService` отвечает за **calculation**, а `DurationFormatter` — только за **formatting** готовых чисел. Это сохраняет разделение ответственности и не смешивает calculation logic с presentation logic.

`TaskStatService` is responsible for **calculation**, while `DurationFormatter` is responsible only for **formatting** ready numbers. This preserves separation of responsibilities and keeps calculation logic separate from presentation logic.

---

## 21. DRY vs Strategy/Plugin

### Question

**Почему Task 5.3 является примером DRY refactoring, а не Strategy или Plugin, в отличие от подхода с `ReportGenerator` в Block 4?**

**Why is Task 5.3 an example of DRY refactoring rather than Strategy or Plugin, unlike the approach used with `ReportGenerator` in Block 4?**

### Answer

В Task 5.3 мы объединяем **одинаковую существующую логику** форматирования в одном месте. Мы не создаём несколько взаимозаменяемых алгоритмов. Strategy/Plugin нужен, когда приложение должно выбирать между разными реализациями.

In Task 5.3, we move **the same existing formatting logic** into one place. We are not creating several interchangeable algorithms. Strategy/Plugin is useful when the application needs to choose between different implementations.

---

## 22. `formatSummary()` и `format()`

### Question

**Как метод `formatSummary()` должен повторно использовать метод `format()` внутри `DurationFormatter` и почему это требование явно указано в задании?**

**How should the `formatSummary()` method reuse the `format()` method inside `DurationFormatter`, and why is this requirement explicitly stated in the task?**

### Answer

`formatSummary()` должен вызывать `format()` для каждого значения вместо повторения rounding и unit suffix logic. Это гарантирует, что правило форматирования находится **только в одном месте**.

`formatSummary()` should call `format()` for each value instead of repeating the rounding and unit suffix logic. This guarantees that the formatting rule exists **in only one place**.

---

## 23. Проверка рефакторинга по коду

### Question

**Какие наблюдаемые признаки в коде, а не только совпадение итогового текста на экране, можно использовать для проверки правильности рефакторинга Task 5.3?**

**What observable signs in the code, not just the coincidence of the final text on the screen, can be used to verify that the Task 5.3 refactoring was done correctly?**

### Answer

Нужно проверить, что:

- `DurationFormatter` зарегистрирован как service;
- `format()` содержит rounding и unit suffix;
- `formatSummary()` вызывает `format()`;
- Field Formatter использует `DurationFormatter`;
- preprocess использует `DurationFormatter`;
- consumers больше не содержат собственные `round()` и unit suffix logic.

We should check that:

- `DurationFormatter` is registered as a service;
- `format()` contains the rounding and unit suffix logic;
- `formatSummary()` calls `format()`;
- the Field Formatter uses `DurationFormatter`;
- preprocess uses `DurationFormatter`;
- consumers no longer contain their own `round()` or unit suffix logic.

---

## 24. Сохранение внешнего поведения

### Question

**Почему Task 5.3 требует, чтобы внешне видимое поведение и output на экране не изменились после рефакторинга, и как это можно проверить без визуального сравнения “на глаз”?**

**Why does Task 5.3 require externally visible behavior and screen output to remain unchanged after the refactoring, and how can this be checked without visual comparison?**

### Answer

Потому что это **refactoring**, а не изменение функциональности. Мы меняем внутреннюю реализацию, но результат должен остаться прежним. Это можно проверить одинаковыми входными данными и сравнением возвращаемых строк, например для `2.5`, `1.0` и `1.5`, а также повторным запуском тестов Field Formatter и Project Statistics.

Because this is a **refactoring**, not a functional change. We change the internal implementation, but the result must remain the same. We can check this by using the same input values and comparing the returned strings, for example for `2.5`, `1.0`, and `1.5`, and by running the existing Field Formatter and Project Statistics tests again.