---
custom-width: 80
---
## 1 · Task → Project

> [!question] Вопрос **RU:** Entity reference, а не текст/автокомплит? **EN:** Why Entity reference, not free text?

> [!success] Ответ **RU:** Гарантирует связь только с реальным проектом, без опечаток; работает с Views и не ломается при переименовании. **EN:** Guarantees a link to a real project only, no typos; works with Views and survives renaming.

> [!tip] Реализация **RU:** `field_project` на Task, контекстный аргумент View `task_board`. **EN:** `field_project` on Task, contextual argument for View `task_board`.

---

## 2 · field_status

> [!question] Вопрос **RU:** List (text), а не taxonomy/Boolean? **EN:** Why List (text), not taxonomy/Boolean?

> [!success] Ответ **RU:** Статусы — маленький фиксированный набор, меняет только разработчик; Boolean хватает только на 2 значения. **EN:** Statuses are a small fixed set controlled by devs; Boolean only fits 2 values.

> [!tip] Реализация **RU:** Значения = колонки доски (Backlog, In Progress, Done). **EN:** Values = board columns (Backlog, In Progress, Done).

---

## 3 · Кардинальность field_project

> [!question] Вопрос **RU:** Почему = 1, а не unlimited? **EN:** Why = 1, not unlimited?

> [!success] Ответ **RU:** Кардинальность — сколько значений хранит поле; задача принадлежит одному проекту. **EN:** Cardinality = how many values a field holds; a task belongs to one project only.

> [!tip] Реализация **RU:** `cardinality: 1` в field.storage. **EN:** `cardinality: 1` in field.storage.

---

## 4 · field_estimate

> [!question] Вопрос **RU:** decimal, а не Integer/текст? **EN:** decimal, not Integer/text?

> [!success] Ответ **RU:** Оценка бывает дробной (0.5, 1.5); Integer и текст этого не поддерживают. **EN:** Estimates can be fractional (0.5, 1.5); Integer and text don't support that.

> [!tip] Реализация **RU:** Для подсчёта нагрузки на карточке/проекте. **EN:** Used to calculate workload on the card/project.

---

## 5 · field.storage vs field.field

> [!question] Вопрос **RU:** В чём разница? **EN:** What's the difference?

> [!success] Ответ **RU:** storage — тип и кардинальность на уровне БД (один раз); field — привязка к бандлу (для каждого типа контента). **EN:** storage — type/cardinality at DB level (once); field — attachment to a bundle (per content type).

> [!tip] Реализация **RU:** Оба файла нужны для `field_project` на Task. **EN:** Both files are needed for `field_project` on Task.

---

## 6 · Default "Backlog"

> [!question] Вопрос **RU:** Где настраивается, и что с API-созданием? **EN:** Where is it set, and what about API creation?

> [!success] Ответ **RU:** Задаётся в `default_value` field.field.yml; работает только через форму — при создании через API поле останется пустым. **EN:** Set in `default_value` in field.field.yml; only works via the form — API-created nodes leave it empty.

> [!tip] Реализация **RU:** Нужно явно указывать статус в коде при программном создании. **EN:** Status must be set explicitly in code when creating programmatically.

---

## 7 · field_attachments

> [!question] Вопрос **RU:** Media reference, а не File? **EN:** Media reference, not File?

> [!success] Ответ **RU:** Media даёт переиспользование, метаданные и библиотеку; File — просто файл без этого. **EN:** Media gives reuse, metadata, and a library; File is just a raw file without that.

> [!tip] Реализация **RU:** Прикрепление файлов к задаче через форму. **EN:** Used to attach files to a task via the form.

---

## 8 · Media vs File

> [!question] Вопрос **RU:** В чём принципиальная разница? **EN:** What's the fundamental difference?

> [!success] Ответ **RU:** File — простая запись (путь, тип, размер); Media — сущность с бандлами, полями и ревизиями. **EN:** File is a basic record; Media is a full entity with bundles, fields, revisions.

> [!tip] Реализация **RU:** Media = центральная библиотека вложений сайта. **EN:** Media = central attachment library for the whole site.

---

## 9 · Отдельный бандл Document

> [!question] Вопрос **RU:** Почему не переиспользовать Image? **EN:** Why not reuse the Image bundle?

> [!success] Ответ **RU:** У каждого бандла свои поля/правила; Image не подходит под PDF/Word. **EN:** Each bundle has its own fields/rules; Image doesn't fit PDF/Word files.

> [!tip] Реализация **RU:** Новый media type `document` с нужными расширениями. **EN:** New media type `document` with the required file extensions.

---

## 10 · Ограничение типов медиа

> [!question] Вопрос **RU:** Как технически ограничить Image+Document? **EN:** How to technically restrict to Image+Document?

> [!success] Ответ **RU:** Через "Target bundles" в настройках handler'а поля. **EN:** Via "Target bundles" in the field's reference handler settings.

> [!tip] Реализация **RU:** `target_bundles: [image, document]`. **EN:** `target_bundles: [image, document]`.

---

## 11 · Media Library vs автокомплит

> [!question] Вопрос **RU:** В чём разница виджетов? **EN:** How do the widgets differ?

> [!success] Ответ **RU:** Library — визуальный браузер с превью/поиском/загрузкой; автокомплит — просто текстовое поле. **EN:** Library is a visual browser with preview/search/upload; autocomplete is just a text field.

> [!tip] Реализация **RU:** Widget `media_library_widget` для удобства редакторов. **EN:** `media_library_widget` used for editor convenience.

---

## 12 · Paragraphs

> [!question] Вопрос **RU:** Почему не body + WYSIWYG? **EN:** Why not body + WYSIWYG?

> [!success] Ответ **RU:** Paragraphs = переиспользуемые структурированные блоки со своими полями/шаблонами; body — просто HTML-"каша". **EN:** Paragraphs = reusable structured blocks with own fields/templates; body is just an HTML blob.

> [!tip] Реализация **RU:** `field_content` на Documentation, типы: code, callout, image. **EN:** `field_content` on Documentation, types: code, callout, image.

---

## 13 · Entity reference revisions

> [!question] Вопрос **RU:** Зачем для field_content? **EN:** Why for field_content?

> [!success] Ответ **RU:** Ссылается на конкретную ревизию параграфа, а не на последнюю версию — сохраняет историю страницы. **EN:** Points to a specific paragraph revision, not the latest — preserves page history.

> [!tip] Реализация **RU:** Тип поля `entity_reference_revisions`. **EN:** Field type `entity_reference_revisions`.

---

## 14 · Ревизии параграфов

> [!question] Вопрос **RU:** Зачем свои, если узел уже версионируется? **EN:** Why own revisions if the node is revisioned too?

> [!success] Ответ **RU:** Параграфы хранятся отдельно; без своих ревизий правка ломала бы старые версии страницы. **EN:** Paragraphs are stored separately; without own revisions, edits would break old page versions.

> [!tip] Реализация **RU:** Для корректного отката версий Documentation. **EN:** Needed for correctly rolling back Documentation versions.

---

## 15 · field_language / field_callout_type

> [!question] Вопрос **RU:** Почему List, а не taxonomy? **EN:** Why List, not taxonomy?

> [!success] Ответ **RU:** Маленький фиксированный набор, завязан на код (CSS-классы); расширяемость taxonomy не нужна. **EN:** Small fixed set tied to code (CSS classes); taxonomy's extensibility isn't needed.

> [!tip] Реализация **RU:** Используются в `hook_preprocess_paragraph()`. **EN:** Used inside `hook_preprocess_paragraph()`.

---

## 16 · Template suggestions

> [!question] Вопрос **RU:** Как Drupal находит paragraph--code.html.twig? **EN:** How does Drupal find paragraph--code.html.twig?

> [!success] Ответ **RU:** Через template suggestions: имя строится как `paragraph--{bundle}`, выбирается самый специфичный файл. **EN:** Via template suggestions: name is built as `paragraph--{bundle}`, most specific file wins.

> [!tip] Реализация **RU:** Файл лежит в `templates/` темы. **EN:** File is located in the theme's `templates/` folder.

---

## 17 · hook_preprocess_paragraph()

> [!question] Вопрос **RU:** Почему класс в хуке, а не в Twig, и зачем проверка бандла? **EN:** Why calculate the class in the hook, not Twig, and why check the bundle?

> [!success] Ответ **RU:** Логика — в PHP, Twig только отображает; хук срабатывает для всех параграфов, без проверки бандла — ошибка на полях, которых нет. **EN:** Logic belongs in PHP, Twig only displays; the hook fires for all paragraphs — without a bundle check, it errors on missing fields.

> [!tip] Реализация **RU:** Класс `language-{lang}` для блока кода. **EN:** Adds a `language-{lang}` class for the code block.

---

## 18 · task_board: contextual filter

> [!question] Вопрос **RU:** Почему не fixed/exposed? **EN:** Why not fixed/exposed?

> [!success] Ответ **RU:** Берёт значение из URL — один View работает для любого проекта. **EN:** Takes the value from the URL — one View works for any project.

> [!tip] Реализация **RU:** Путь `/board/%project_id`. **EN:** Path `/board/%project_id`.

---

## 19 · Отсутствие аргумента фильтра

> [!question] Вопрос **RU:** Что произойдёт с View? **EN:** What happens to the View?

> [!success] Ответ **RU:** Управляется настройкой "When filter value is NOT available" — от "показать всё" до 404. **EN:** Controlled by "When filter value is NOT available" — from "show all" to 404.

> [!tip] Реализация **RU:** В `task_board` выбрано "Page not found". **EN:** `task_board` is set to "Page not found".

---

## 20 · AJAX-режим доски

> [!question] Вопрос **RU:** Зачем он нужен? **EN:** Why is it needed?

> [!success] Ответ **RU:** Обновляет только часть страницы без полной перезагрузки — важно для интерактивности. **EN:** Updates only part of the page without a full reload — key for interactivity.

> [!tip] Реализация **RU:** Опция "Use AJAX" в display доски. **EN:** "Use AJAX" option enabled in the board's display.

---

## 21 · Teaser / Full

> [!question] Вопрос **RU:** Почему разные режимы для карточки и модалки? **EN:** Why different modes for card and modal?

> [!success] Ответ **RU:** Teaser — компактно для доски, Full — все детали для просмотра. **EN:** Teaser is compact for the board, Full shows all details for viewing.

> [!tip] Реализация **RU:** Клик по карточке открывает Full через Ajax-модалку. **EN:** Clicking the card opens Full mode via an Ajax modal.

---

## 22 · Frontend Editing

> [!question] Вопрос **RU:** Чем отличается от /node/{id}/edit? **EN:** How does it differ from /node/{id}/edit?

> [!success] Ответ **RU:** Пользователь не покидает доску; форма грузится и сохраняется через AJAX в модалке. **EN:** User stays on the board; form loads and saves via AJAX in a modal.

> [!tip] Реализация **RU:** Drupal Ajax API + `OpenModalDialogCommand`. **EN:** Drupal Ajax API + `OpenModalDialogCommand`.

---

## 23 · Drag&drop

> [!question] Вопрос **RU:** Зачем AJAX и Drupal.behaviors? **EN:** Why AJAX and Drupal.behaviors?

> [!success] Ответ **RU:** Без AJAX статус не сохраняется в БД; `Drupal.behaviors` (в отличие от `ready()`) перезапускается для контента, добавленного через AJAX. **EN:** Without AJAX the status isn't saved to the DB; `Drupal.behaviors` (unlike `ready()`) reruns for AJAX-added content.

> [!tip] Реализация **RU:** `Drupal.behaviors.taskBoard` шлёт POST на смену `field_status`. **EN:** `Drupal.behaviors.taskBoard` sends a POST request to update `field_status`.