---
custom-width: 80
---
### 1. Что Figma MCP предоставляет AI coding agent, чего не даёт скриншот или обычная ссылка на Figma?

**What does Figma MCP provide to an AI coding agent that a screenshot or ordinary Figma link does not?**

**Ответ:** Figma MCP предоставляет структурированный доступ к дизайну: компонентам, стилям, размерам, цветам и layout-информации. Скриншот показывает только внешний вид, а обычная ссылка не даёт AI такой структурированной информации для генерации кода.

**Answer:** Figma MCP provides structured access to the design, including components, styles, dimensions, colors, and layout information. A screenshot shows only the visual result, while a normal link does not provide this structured information for code generation.

---

### 2. Почему credentials и локальная конфигурация Figma MCP должны находиться вне репозитория?

**Why must Figma MCP credentials and local client configuration remain outside the repository?**

**Ответ:** Credentials могут содержать токены и другие секреты. Их нельзя хранить в Git, потому что они могут попасть в публичную историю и дать другим людям доступ к Figma.

**Answer:** Credentials may contain tokens and other secrets. They must not be stored in Git because they could become available in the repository history and give other people access to Figma.

---

### 3. Почему Figma MCP является development dependency, а не runtime dependency Drupal theme?

**Why is Figma MCP a development dependency rather than a runtime dependency of the Drupal theme?**

**Ответ:** Figma MCP нужен разработчику и AI во время работы с дизайном и кодом. Drupal на production не должен подключаться к Figma для отображения сайта.

**Answer:** Figma MCP is needed by developers and AI during design and development. Drupal production does not need to connect to Figma to render the website.

---

### 4. Как компонент dashboard из Figma должен быть сопоставлен с Drupal View field, template или render array?

**How should a Figma dashboard component be mapped to a Drupal View field, template, or render array?**

**Ответ:** Сначала нужно определить, какие данные компонента приходят из Drupal View. Затем HTML-структуру компонента реализовать в Twig template или render array, а значения заполнить реальными View fields.

**Answer:** First, identify which component data comes from the Drupal View. Then implement the component's HTML structure in a Twig template or render array and populate it with real View fields.

---

### 5. Почему существующий `/board/%` View должен оставаться источником Task data вместо копирования Figma mock content в Twig?

**Why must the existing `/board/%` View remain the source of Task data instead of copying Figma mock content into Twig?**

**Ответ:** View получает реальные Tasks из Drupal и применяет фильтры и доступ. Если скопировать mock data в Twig, дизайн будет статичным и перестанет отражать реальные данные приложения.

**Answer:** The View gets real Tasks from Drupal and applies filters and access rules. If mock data is copied into Twig, the design becomes static and no longer represents the real application data.

---

### 6. Когда нужно использовать View-specific Twig template вместо широкого page или node template override?

**When should a View-specific Twig template be used instead of a broad page or node template override?**

**Ответ:** View-specific template следует использовать, когда изменение относится только к конкретному View или display. Это безопаснее, чем менять общий page или node template, который может повлиять на другие страницы.

**Answer:** A View-specific template should be used when the change applies only to a specific View or display. It is safer than changing a general page or node template that may affect other pages.

---

### 7. Почему повторяющиеся значения из Figma лучше представить как CSS custom properties?

**Why are repeated Figma values better represented as CSS custom properties than copied into every component selector?**

**Ответ:** CSS custom properties позволяют хранить общие цвета, spacing и другие значения в одном месте. Это уменьшает дублирование и упрощает изменение design system.

**Answer:** CSS custom properties allow shared colors, spacing, and other values to be defined in one place. This reduces duplication and makes the design system easier to maintain.

---

### 8. Какую функциональность Drupal можно сломать, удалив template attributes, render children или contextual-link placeholders?

**What Drupal functionality can be broken by removing template attributes, render children, or contextual-link placeholders?**

**Ответ:** Можно сломать accessibility, field rendering, contextual links, cache metadata и другие механизмы Drupal. Поэтому существующие `attributes`, `children` и специальные placeholders нельзя удалять без понимания их назначения.

**Answer:** This can break accessibility, field rendering, contextual links, cache metadata, and other Drupal mechanisms. Therefore, existing `attributes`, `children`, and special placeholders should not be removed without understanding their purpose.

---

### 9. Почему entity loading, workflow decisions и access checks нельзя реализовывать в Twig?

**Why should entity loading, workflow decisions, and access checks not be implemented in Twig?**

**Ответ:** Twig предназначен в первую очередь для presentation layer. Entity loading, workflow logic и access control должны выполняться в PHP, сервисах или Drupal API, чтобы код был безопасным, тестируемым и поддерживаемым.

**Answer:** Twig is mainly intended for the presentation layer. Entity loading, workflow logic, and access control should be handled by PHP, services, or Drupal APIs so the code remains secure, testable, and maintainable.

---

### 10. Почему JavaScript, подключённый к View, должен использовать Drupal behaviors и `once()`?

**Why must JavaScript attached to a View use Drupal behaviors and `once()`?**

**Ответ:** Drupal behaviors позволяют корректно выполнять JavaScript при первоначальной загрузке и при AJAX-обновлениях. `once()` предотвращает повторное добавление обработчиков к одному и тому же элементу.

**Answer:** Drupal behaviors allow JavaScript to work correctly on the initial page load and after AJAX updates. `once()` prevents the same event handlers from being attached to an element multiple times.

---

### 11. Как изменение Task data может доказать, что redesigned board всё ещё использует live Drupal rendering?

**How can changing Task data prove that the redesigned board still uses live Drupal rendering?**

**Ответ:** Нужно изменить реальные данные Task, например название, статус или assignee, и обновить board. Если UI показывает новые значения без изменения Twig или mock data, значит board использует live Drupal rendering.

**Answer:** Change real Task data, such as its title, status, or assignee, and reload the board. If the UI shows the new values without changing Twig or mock data, the board is using live Drupal rendering.

---

### 12. Почему визуальное скрытие запрещённого действия не равно Drupal access control?

**Why is hiding a forbidden action visually not equivalent to enforcing Drupal access control?**

**Ответ:** CSS или JavaScript только скрывают элемент интерфейса, но не запрещают сам запрос. Реальный access check должен выполняться на сервере Drupal, чтобы пользователь не мог обойти ограничение напрямую.

**Answer:** CSS or JavaScript only hides the interface element but does not block the request itself. A real access check must be performed by Drupal on the server so the restriction cannot be bypassed directly.

---

### 13. Какие cache contexts и tags могут быть важны для board, который зависит от project, permissions, filters и Task updates?

**What cache contexts and tags may matter when a board varies by project, user permissions, filters, and Task updates?**

**Ответ:** Могут понадобиться contexts, например `user.permissions` и параметры запроса, а также cache tags для конкретного project и Task entities. Это позволяет Drupal правильно разделять варианты кеша и инвалидировать их после изменений.

**Answer:** Contexts such as `user.permissions` and query parameters may be needed, as well as cache tags for the relevant project and Task entities. This allows Drupal to vary cached output correctly and invalidate it after changes.

---

### 14. Когда visual fidelity к Figma должна уступить semantic HTML или accessibility requirements?

**When should visual fidelity to Figma yield to semantic HTML or accessibility requirements?**

**Ответ:** Если точное повторение дизайна ухудшает accessibility, нужно предпочесть semantic HTML и доступность. Например, интерактивный элемент должен быть настоящей `<button>`, даже если Figma визуально показывает обычный `<div>`.

**Answer:** If exactly copying the design harms accessibility, semantic HTML and accessibility should take priority. For example, an interactive element should be a real `<button>`, even if Figma visually represents it as a `<div>`.

---

### 15. Почему AI-generated Drupal code нужно проверять вручную, даже если он проходит coding standards?

**Why must AI-generated Drupal code be manually reviewed even when it passes coding-standard checks?**

**Ответ:** Coding standards проверяют в основном стиль и некоторые технические проблемы, но не гарантируют правильную архитектуру, security, access control или соответствие требованиям Drupal. AI-код всё равно должен пройти manual review и тестирование.

**Answer:** Coding standards mainly check style and some technical issues, but they do not guarantee correct architecture, security, access control, or compliance with Drupal requirements. AI-generated code still needs manual review and testing.