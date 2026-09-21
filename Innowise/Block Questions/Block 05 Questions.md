---
custom-width: 80
---
## 1. Render Array

### Question

**Что такое Render Array и почему в Drupal принято строить вывод через него, а не объединять HTML-строки в PHP?**

**What is a Render Array and why does Drupal use it instead of concatenating HTML strings in PHP?**

### Answer

Render Array — это структурированный PHP-массив, который описывает, что и как Drupal должен отрендерить. Он позволяет использовать **Render API, Twig, Theme API, cache metadata и theme overrides**, поэтому presentation logic не смешивается с PHP-кодом.

A Render Array is a structured PHP array that describes what Drupal should render. It allows Drupal to use the **Render API, Twig, Theme API, cache metadata, and theme overrides**, keeping presentation logic separate from PHP code.

---

## 2. `#markup` vs `#theme`

### Question

**В чём разница между `#markup` и `#theme` и когда используется каждый из них?**

**What is the difference between `#markup` and `#theme`, and when should each one be used?**

### Answer

`#markup` используется для вывода небольшого готового текста или markup. `#theme` используется для **тематизированного компонента**, который должен быть отрендерен через theme hook и обычно Twig-шаблон.

`#markup` is used for a small piece of ready-to-render text or markup. `#theme` is used for a **themed component** that should be rendered through a theme hook and usually a Twig template.

---

## 3. `hook_theme()`

### Question

**Почему в Task 5.1 нужно зарегистрировать theme hook через `hook_theme()`, а не использовать `twig_render_template()` напрямую?**

**Why should Task 5.1 register a theme hook with `hook_theme()` instead of using `twig_render_template()` directly?**

### Answer

`hook_theme()` подключает шаблон к **Drupal Theme API**. Это даёт поддержку preprocess, theme overrides, theme registry и стандартного процесса рендеринга. Прямой вызов Twig обходит эту архитектуру.

`hook_theme()` integrates the template with Drupal's **Theme API**. It provides preprocess functions, theme overrides, the theme registry, and the normal rendering process. A direct Twig call bypasses this architecture.

---

## 4. Поиск Twig-шаблона

### Question

**Как Drupal находит `drupaljira-project-stats.html.twig`, если он есть и в модуле, и в активной теме?**

**How does Drupal find `drupaljira-project-stats.html.twig` if it exists both in the module and in the active theme?**

### Answer

Модуль регистрирует theme hook через `hook_theme()` и указывает исходный шаблон. После построения theme registry Drupal позволяет активной теме **переопределить шаблон**. Поэтому шаблон из активной темы используется вместо шаблона модуля.

The module registers the theme hook with `hook_theme()` and defines the original template. After building the theme registry, Drupal allows the active theme to **override the template**. Therefore, the theme version is used instead of the module version.

---

## 5. Preprocess

### Question

**Почему подготовка переменных должна происходить в `hook_preprocess_HOOK()`, а не внутри Twig?**

**Why should variables be prepared in `hook_preprocess_HOOK()` instead of inside Twig?**

### Answer

Preprocess отвечает за **подготовку данных для presentation layer**, а Twig — за их отображение. Поэтому расчёты, форматирование и получение данных из сервисов должны находиться в PHP, а Twig должен оставаться простым.

Preprocess is responsible for **preparing data for the presentation layer**, while Twig is responsible for displaying it. Calculations, formatting, and service calls should therefore stay in PHP, keeping Twig simple.

---

## 6. Уже отформатированные значения

### Question

**Почему preprocess должен передавать в Twig уже отформатированные значения, а не raw numbers?**

**Why should preprocess pass already formatted values to Twig instead of raw numbers?**

### Answer

Это сохраняет чёткую **границу ответственности**. `TaskStatService` рассчитывает числа, preprocess подготавливает данные для отображения, а Twig отвечает только за markup.

This creates a clear **separation of responsibilities**. `TaskStatService` calculates numbers, preprocess prepares display values, and Twig is responsible only for markup.

---

## 7. Отключение модуля

### Question

**Что произойдёт с Project Statistics при отключении модуля, который регистрирует theme hook?**

**What happens to Project Statistics if the module that registers the theme hook is disabled?**

### Answer

Компонент, предоставляемый этим модулем, должен исчезнуть, потому что его код больше недоступен. При этом не должно возникать **fatal error**, потому что Drupal должен корректно обработать отсутствие компонента.

The component provided by this module should disappear because its implementation is no longer available. However, there should be **no fatal error**, because Drupal should handle the missing component gracefully.

---

## 8. Theme Hook vs строка

### Question

**В чём архитектурная разница между theme hook с preprocess и простым вызовом сервиса с возвратом HTML-строки из `build()`?**

**What is the architectural difference between a theme hook with preprocess and simply calling a service and returning an HTML string from `build()`?**

### Answer

Theme hook разделяет **данные, подготовку и presentation**: service → preprocess → Twig. При возврате HTML-строки module code одновременно содержит application logic и presentation logic, поэтому его сложнее переопределять и поддерживать.

A theme hook separates **data, preparation, and presentation**: service → preprocess → Twig. With an HTML string, the module contains both application logic and presentation logic, making it harder to override and maintain.

---

# Task 5.2 — Cache API

## 9. Cache Tags, Contexts и `max-age`

### Question

**В чём разница между Cache Tags, Cache Contexts и `max-age`?**

**What is the difference between Cache Tags, Cache Contexts, and `max-age`?**

### Answer

**Cache Tags** определяют, что нужно инвалидировать при изменении данных. **Cache Contexts** определяют, для каких условий нужен отдельный cache variant. **`max-age`** определяет, как долго cache считается действительным.

**Cache Tags** define what should be invalidated when data changes. **Cache Contexts** define when different cache variants are required. **`max-age`** defines how long the cached data remains valid.

---

## 10. Custom cache tag

### Question

**Почему используется `drupaljira_project_stats:{nid}`, а не общий `node:{nid}`?**

**Why is `drupaljira_project_stats:{nid}` used instead of the general `node:{nid}` tag?**

### Answer

Потому что нам нужно инвалидировать именно **кеш статистики проекта**, а не весь кеш, связанный с node. Например, изменение TimeLog проекта A должно инвалидировать только `drupaljira_project_stats:A`.

Because we need to invalidate only the **project statistics cache**, not all cached data related to the node. For example, changing a TimeLog for project A should invalidate only `drupaljira_project_stats:A`.

---

## 11. `invalidateTags()`

### Question

**Почему нужно использовать `Cache::invalidateTags()`, а не `drush cr`?**

**Why should we use `Cache::invalidateTags()` instead of `drush cr`?**

### Answer

`invalidateTags()` инвалидирует только связанные cache entries. `drush cr` очищает кеш значительно шире и является административной операцией, а не частью обычной бизнес-логики приложения.

`invalidateTags()` invalidates only the related cache entries. `drush cr` clears much more cache and is an administrative operation, not something that should be part of normal application logic.

---

## 12. Проверка изоляции cache tags

### Question

**Как проверить, что cache tags двух проектов не пересекаются?**

**How can we check that cache tags do not overlap between two projects?**

### Answer

Нужно прогреть кеш для проектов A и B, затем изменить TimeLog проекта A. После этого статистика A должна пересчитаться, а кеш B должен остаться действительным и вернуть старое cache entry.

We should warm the cache for projects A and B and then change a TimeLog in project A. Project A should be recalculated, while project B should keep its cached entry.

---

## 13. Cache bubbling

### Question

**Что такое cache bubbling в Render Array и зачем он нужен?**

**What is cache bubbling in a Render Array and why is it needed?**

### Answer

Cache bubbling — это передача cache metadata от дочернего Render Array к его родительскому Render Array. Это позволяет Drupal знать, от каких данных зависит весь результат и когда его нужно изменить или инвалидировать.

Cache bubbling means that cache metadata from a child Render Array is propagated to its parent. This allows Drupal to know what the complete output depends on and when it needs to be invalidated or varied.

---

## 14. Context `user`

### Question

**Почему для Kanban выбран context `user`, а не `max-age: 0`?**

**Why is the `user` cache context used for Kanban instead of `max-age: 0`?**

### Answer

Потому что Kanban нужно **кешировать**, но отдельно для каждого пользователя. Context `user` создаёт разные cache variants. `max-age: 0` полностью отключил бы полезное кеширование этого Render Array.

Because the Kanban board should still be **cached**, but separately for each user. The `user` context creates different cache variants. `max-age: 0` would disable useful caching for this Render Array.

---

## 15. `user` vs `user.roles`

### Question

**Как выбрать между `user` и `user.roles` и почему более узкий context обычно лучше?**

**How should we choose between `user` and `user.roles`, and why is a narrower context usually better?**

### Answer

Нужно смотреть на то, **от чего реально зависит output**. Если вывод зависит от конкретного пользователя — нужен `user`. Если он зависит только от ролей — достаточно `user.roles`. Более узкий context создаёт меньше cache variants.

We should look at **what the output actually depends on**. If it depends on the individual user, use `user`. If it depends only on roles, `user.roles` is enough. A narrower context creates fewer cache variants.

---

## 16. Проверка cache context

### Question

**Как проверить, что cache context действительно меняет output, а не просто указан в `#cache`?**

**How can we verify that the cache context actually changes the output?**

### Answer

Нужно открыть один и тот же Kanban от имени двух пользователей с разными assigned tasks. Если пользователь A видит свои задачи как `my task`, а B — свои, значит Drupal использует разные cache variants.

We should open the same Kanban board as two users with different assigned tasks. If user A sees their own tasks marked as `my task` and user B sees their own tasks, Drupal is using different cache variants.

---

## 17. Anonymous user

### Question

**Почему отдельно проверяется anonymous user без session?**

**Why is an anonymous user without a session tested separately?**

### Answer

Anonymous users не имеют обычного user identity, как authenticated users. Поэтому нужно убедиться, что Kanban корректно работает без персонализации, не вызывает PHP errors и не показывает чужие `my task` отметки.

Anonymous users do not have a normal authenticated user identity. Therefore, we must verify that the Kanban works correctly without personalization, causes no PHP errors, and does not show another user's `my task` markers.

---

## 18. Entity hooks vs Event Subscriber

### Question

**В чём разница между `hook_ENTITY_TYPE_insert/update/delete` и Event Subscriber?**

**What is the difference between `hook_ENTITY_TYPE_insert/update/delete` and an Event Subscriber?**

### Answer

Entity hooks — это более простой Drupal-specific способ реагировать на изменения конкретного entity type. Event Subscriber использует **event system** и лучше подходит, когда нужна отдельная объектно-ориентированная логика или обработка нескольких событий.

Entity hooks are a simpler Drupal-specific way to react to changes of a specific entity type. An Event Subscriber uses the **event system** and is useful when the logic should be organized as a separate object or handle several events.

---

# Task 5.3 — DurationFormatter

## 19. Что именно было устранено?

### Question

**Что именно было продублировано в Task 5.3 и что НЕ было продублировано?**

**What exactly was duplicated in Task 5.3, and what was NOT duplicated?**

### Answer

Было продублировано **форматирование чисел**: rounding, unit suffix и сборка human-readable строки. Сам расчёт часов не дублировался — все компоненты продолжали использовать `TaskStatService`.

The duplicated part was **number formatting**: rounding, unit suffixes, and building human-readable strings. The actual hour calculations were not duplicated because all components continued to use `TaskStatService`.

---

## 20. Почему `DurationFormatter` не использует `TaskStatService`?

### Question

**Почему `DurationFormatter` не должен обращаться к `TaskStatService` и сам выполнять расчёты?**

**Why should `DurationFormatter` not access `TaskStatService` or perform calculations itself?**

### Answer

Потому что у сервисов разные ответственности. `TaskStatService` отвечает за **calculation**, а `DurationFormatter` — только за **presentation formatting**. Это сохраняет чёткую границу между расчётом и отображением.

Because the services have different responsibilities. `TaskStatService` is responsible for **calculation**, while `DurationFormatter` is responsible only for **presentation formatting**. This keeps calculation and presentation clearly separated.

---

## 21. Почему это DRY, а не Strategy/Plugin?

### Question

**Почему Task 5.3 — это DRY refactoring, а не Strategy или Plugin, как в Block 4 с `ReportGenerator`?**

**Why is Task 5.3 a DRY refactoring rather than a Strategy or Plugin pattern like `ReportGenerator` in Block 4?**

### Answer

Здесь мы просто выносим **одинаковую логику форматирования** в одно место и переиспользуем её. Мы не создаём несколько взаимозаменяемых алгоритмов. Strategy/Plugin нужен, когда приложение должно выбирать между разными реализациями.

Here we simply move **duplicated formatting logic** into one reusable service. We are not creating multiple interchangeable algorithms. Strategy or Plugin is useful when the application needs to choose between different implementations.

---

## 22. `formatSummary()` и `format()`

### Question

**Как `formatSummary()` должен использовать `format()` и почему это важно?**

**How should `formatSummary()` reuse `format()`, and why is this important?**

### Answer

`formatSummary()` должен вызывать `format()` для каждого значения, а не повторять `round()` и unit suffix самостоятельно. Это гарантирует, что правило форматирования существует **в одном месте**.

`formatSummary()` should call `format()` for each value instead of repeating rounding and unit suffix logic. This guarantees that the formatting rule exists in **one place**.

Например:

```
return sprintf(
    '%s (%s written off, %s left)',
    $this->format($estimate),
    $this->format($logged),
    $this->format($remaining),
);
```

---

## 23. Как проверить правильность рефакторинга по коду?

### Question

**Какие признаки в коде показывают, что Task 5.3 действительно выполнен правильно?**

**What code-level signs show that the Task 5.3 refactoring was done correctly?**

### Answer

Нужно проверить, что:

- существует зарегистрированный `DurationFormatter`;
- `format()` содержит единственную логику rounding/unit;
- `formatSummary()` вызывает `format()`;
- Formatter использует `DurationFormatter`;
- preprocess использует `DurationFormatter`;
- в этих consumers больше нет собственного `round()` и unit suffix.

We should check that:

- `DurationFormatter` is registered;
- `format()` contains the only rounding/unit logic;
- `formatSummary()` calls `format()`;
- the Field Formatter uses `DurationFormatter`;
- preprocess uses `DurationFormatter`;
- consumers no longer contain their own rounding or unit suffix logic.

---

## 24. Почему нельзя менять output?

### Question

**Почему при рефакторинге Task 5.3 внешний output не должен измениться и как это проверить?**

**Why must the visible output stay unchanged after Task 5.3, and how can we verify it?**

### Answer

Потому что это **refactoring**, а не изменение функциональности. Мы меняем внутреннюю архитектуру, но поведение системы должно остаться прежним.

Проверять можно не только визуально: использовать те же входные значения и сравнить строки, которые возвращают старый и новый код, например для `2.5`, `1.0` и `1.5`. Также нужно проверить Field Formatter и Project Statistics.

Because this is a **refactoring**, not a functional change. We change the internal architecture, but the system behavior must remain the same.

We can test this without visual comparison by using the same input values and comparing the strings produced by the old and new logic, for example `2.5`, `1.0`, and `1.5`. We should also test the Field Formatter and Project Statistics output.