---
custom-width: 80
---
## Task 5.1 — Render Arrays & Theming

### 1. Что такое Render Array и почему в Drupal принято строить вывод с его помощью, а не объединять HTML-строки в PHP?

**What is a Render Array and why is it customary in Drupal to build output using it instead of concatenating HTML strings in PHP?**

### Answer

Render Array — структурированный массив для описания вывода Drupal. Он поддерживает **Render API, Twig, Theme API, cache metadata и overrides**.

A Render Array is a structured array describing Drupal output. It supports **Render API, Twig, Theme API, cache metadata, and overrides**.

---

### 2. В чём разница между `#markup` и `#theme` в Render Array и когда следует использовать каждый из этих ключей?

**What is the difference between `#markup` and `#theme` in a Render Array, and when is each key appropriate?**

### Answer

`#markup` — готовый небольшой текст/markup. `#theme` — тематизированный компонент через **theme hook и Twig**.

`#markup` is for small ready-to-render text/markup. `#theme` is for a **themed component using a theme hook and Twig**.

---

### 3. Почему в Task 5.1 нужно зарегистрировать новый theme hook через `hook_theme()`, а не напрямую отрендерить Twig-шаблон?

**Why should Task 5.1 register a new theme hook using `hook_theme()` instead of directly rendering the Twig template?**

### Answer

`hook_theme()` подключает шаблон к **Theme API**, давая preprocess, overrides и theme registry.

`hook_theme()` integrates the template with the **Theme API**, providing preprocess, overrides, and the theme registry.

---

### 4. Как Drupal находит шаблон `drupaljira-project-stats.html.twig`, если он есть и в модуле, и в активной теме?

**How does Drupal find the `drupaljira-project-stats.html.twig` template if it exists in both the module and the active theme?**

### Answer

Модуль регистрирует шаблон через `hook_theme()`, а активная тема может его **override**.

The module registers the template through `hook_theme()`, and the active theme can **override** it.

---

### 5. Почему подготовка переменных должна происходить в `hook_preprocess_HOOK()`, а не внутри Twig?

**Why should variable preparation happen in `hook_preprocess_HOOK()` instead of inside Twig?**

### Answer

Preprocess готовит данные**, Twig только отображает их. Расчёты и сервисы должны оставаться в PHP.

Preprocess prepares data**, while Twig only displays it. Calculations and services should stay in PHP.

---

### 6. Почему preprocess должен передавать в шаблон уже отформатированные значения?

**Why does preprocess need to pass already formatted values to the template?**

### Answer

Это разделяет ответственность: **Service → расчёт, preprocess → подготовка, Twig → markup**.

It separates responsibilities: **Service → calculation, preprocess → preparation, Twig → markup**.

---

### 7. Что произойдёт с Project Statistics при отключении модуля?

**What happens to Project Statistics if its module is disabled?**

### Answer

Компонент исчезнет, потому что его код недоступен, но **fatal error быть не должно**.

The component disappears because its code is unavailable, but there should be **no fatal error**.

---

### 8. В чём архитектурная разница между theme hook с preprocess и возвратом строки из `build()`?

**What is the architectural difference between a theme hook with preprocess and returning a string from `build()`?**

### Answer

Theme hook разделяет **данные, подготовку и presentation**. HTML-строка смешивает application и presentation logic.

A theme hook separates **data, preparation, and presentation**. An HTML string mixes application and presentation logic.

---

# Task 5.2 — Cache API

### 9. Объясните разницу между Cache Tags, Cache Contexts и `max-age`.

**Explain the difference between Cache Tags, Cache Contexts, and `max-age`.**

### Answer

**Tags** — что инвалидировать. **Contexts** — для каких условий нужны разные варианты. **`max-age`** — как долго кеш действителен.

**Tags** define what to invalidate. **Contexts** define when different variants are needed. **`max-age`** defines how long the cache is valid.

---

### 10. Почему выбран `drupaljira_project_stats:{nid}`, а не `node:{nid}`?

**Why was `drupaljira_project_stats:{nid}` chosen instead of `node:{nid}`?**

### Answer

Custom tag инвалидирует только **кеш статистики конкретного проекта**, а не весь node-related cache.

The custom tag invalidates only the **statistics cache of one project**, not all node-related cache.

---

### 11. Почему используется `Cache::invalidateTags()`, а не `drush cr`?

**Why use `Cache::invalidateTags()` instead of `drush cr`?**

### Answer

`invalidateTags()` очищает только связанные cache entries. `drush cr` очищает кеш **намного шире**.

`invalidateTags()` invalidates only related cache entries. `drush cr` clears cache **much more broadly**.

---

### 12. Как проверить, что cache tags не пересекаются между A и B?

**How can we check that cache tags do not overlap between projects A and B?**

### Answer

Прогреть A и B, изменить TimeLog в A. **A пересчитывается, B остаётся закешированным.**

Warm A and B, then change a TimeLog in A. **A is recalculated, while B remains cached.**

---

### 13. Что такое cache bubbling?

**What is cache bubbling?**

### Answer

Cache metadata дочернего Render Array **передаётся родительскому**.

Cache metadata from a child Render Array is **propagated to its parent**.

---

### 14. Почему для Kanban используется `user` или `user.roles`, а не `max-age: 0`?

**Why use `user` or `user.roles` for Kanban instead of `max-age: 0`?**

### Answer

Context сохраняет кеширование, но создаёт разные варианты для пользователей/ролей. `max-age: 0` **отключает кеширование**.

A context keeps caching but creates different variants for users/roles. `max-age: 0` **disables caching**.

---

### 15. Как выбрать между `user` и `user.roles`?

**How do we choose between `user` and `user.roles`?**

### Answer

Если output зависит от конкретного пользователя — `user`; только от роли — `user.roles`. Более узкий context создаёт **меньше вариантов**.

If output depends on the individual user, use `user`; if only on the role, use `user.roles`. A narrower context creates **fewer variants**.

---

### 16. Как проверить, что cache context реально влияет на output?

**How can we check that the cache context actually changes the output?**

### Answer

Открыть один Kanban двумя пользователями. Каждый должен видеть **свои `my task`**.

Open the same Kanban as two users. Each user should see **their own `my task`** markers.

---

### 17. Почему отдельно проверяется anonymous user?

**Why is anonymous user tested separately?**

### Answer

Anonymous не имеет обычной authenticated identity, поэтому нужно проверить **отсутствие ошибок и `my task`**.

Anonymous users do not have a normal authenticated identity, so we must check for **no errors and no `my task` marker**.

---

### 18. В чём разница между Entity Hooks и Event Subscriber?

**What is the difference between Entity Hooks and an Event Subscriber?**

### Answer

Entity hooks — простой Drupal-specific подход. Event Subscriber — **более OO-подход**, удобный для отдельного класса и нескольких событий.

Entity hooks are a simple Drupal-specific approach. An Event Subscriber is a **more object-oriented approach**, useful for a separate class and multiple events.

---

# Task 5.3 — DurationFormatter

### 19. Что именно было продублировано в Task 5.3 и что НЕ было продублировано?

**What exactly was duplicated in Task 5.3 and what was NOT duplicated?**

### Answer

Дублировалось **форматирование**: rounding, unit suffix и сборка строки. Расчёт часов через `TaskStatService` не дублировался.

**Formatting** was duplicated: rounding, unit suffixes, and string assembly. Hour calculation through `TaskStatService` was not duplicated.

---

### 20. Почему `DurationFormatter` не должен обращаться к `TaskStatService`?

**Why should `DurationFormatter` not access `TaskStatService`?**

### Answer

`TaskStatService` отвечает за **calculation**, `DurationFormatter` — только за **formatting**.

`TaskStatService` handles **calculation**, while `DurationFormatter` handles only **formatting**.

---

### 21. Почему Task 5.3 — DRY refactoring, а не Strategy/Plugin?

**Why is Task 5.3 a DRY refactoring rather than Strategy/Plugin?**

### Answer

Мы объединяем **одинаковую логику**, а не создаём несколько взаимозаменяемых алгоритмов.

We centralize **the same logic** instead of creating several interchangeable algorithms.

---

### 22. Как `formatSummary()` должен использовать `format()` и зачем?

**How should `formatSummary()` use `format()`, and why?**

### Answer

`formatSummary()` вызывает `format()` для каждого значения. Это оставляет правила форматирования **в одном месте**.

`formatSummary()` calls `format()` for each value. This keeps formatting rules **in one place**.

---

### 23. Какие признаки в коде подтверждают правильность Task 5.3?

**What code-level signs confirm that Task 5.3 was implemented correctly?**

### Answer

`DurationFormatter` зарегистрирован; `format()` содержит formatting logic; `formatSummary()` использует `format()`; Field Formatter и preprocess используют сервис.

`DurationFormatter` is registered; `format()` contains the formatting logic; `formatSummary()` uses `format()`; the Field Formatter and preprocess use the service.

---

### 24. Почему output не должен измениться после Task 5.3 и как это проверить?

**Why should the output not change after Task 5.3, and how can we verify it?**

### Answer

Это **refactoring, а не изменение функциональности**. Проверяем одинаковые входные данные и сравниваем полученные строки.

It is a **refactoring, not a functional change**. We use the same inputs and compare the resulting strings.