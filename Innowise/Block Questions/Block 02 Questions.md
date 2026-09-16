---
custom-width: 80
---
## 1

**Вопрос:** Почему связь Task → Project реализована через поле Entity reference на существующий узел `project`, а не через текстовое/автокомплит поле со свободным вводом названия или ID проекта?

**Question:** Why is the Task → Project connection implemented by the Entity reference field to the existing `project` node, and not by a text/autocomplete field with free entry of the project name or ID?

**Ответ:** Entity reference гарантирует, что можно выбрать только реальный проект — без опечаток и несуществующих названий. Также это позволяет Views строить списки и фильтры на основе связи. Если проект переименуют, связь не сломается.

**Answer:** Entity reference only lets you pick a real, existing project — no typos, no fake names. It also lets Views build lists and filters based on the real connection. If someone renames the project, the link still works.

> 📍 **Где/для чего:** поле `field_project` на content type Task (`field.field.node.task.field_project.yml`), заполняется на форме создания задачи; используется как контекстный аргумент View `task_board` для фильтрации задач по проекту.

---

## 2

**Вопрос:** Почему `field_status` реализовано как List (text) с фиксированными машинными значениями, а не как ссылка на термин таксономии или поле Boolean?

**Question:** Why is `field_status` implemented as a List (text) with fixed machine values, and not as a taxonomy term reference or Boolean field?

**Ответ:** Список статусов маленький и фиксированный — его меняют только разработчики, не редакторы. Поэтому достаточно простого List (text); таксономия нужна для списков, которые редакторы могут пополнять сами. Boolean хранит только два значения (да/нет), а статусов больше двух.

**Answer:** The list of statuses is small and fixed — only developers change it, not editors. So a simple List (text) field is enough; taxonomy is made for lists that editors need to add to. Boolean only has two values (true/false), but we need more than two statuses.

> 📍 **Где/для чего:** поле `field_status` на Task; значения (Backlog, In Progress, Done и т.д.) — machine names колонок Kanban-доски `task_board`; используется для группировки карточек по колонкам и для AJAX-обновления при drag&drop.

---

## 3

**Вопрос:** Что такое кардинальность поля в Field API и почему у `field_project` она равна 1, а не «неограниченно»?

**Question:** What is the cardinality of a field in the Field API and why is `field_project` equal to 1 and not unlimited?

**Ответ:** Кардинальность — это сколько значений может хранить поле: одно, определённое число или неограниченно. Задача принадлежит только одному проекту, поэтому кардинальность равна 1. Если было бы «неограниченно», задача могла бы неправильно принадлежать нескольким проектам сразу.

**Answer:** Cardinality means how many values a field can hold — one, a set number, or unlimited. A task belongs to only one project, so cardinality is 1. If it were unlimited, a task could wrongly belong to many projects at once.

> 📍 **Где/для чего:** настройка `cardinality: 1` в `field.storage.node.field_project.yml`; обеспечивает, что каждая задача попадает ровно на одну доску проекта.

---

## 4

**Вопрос:** Почему `field_estimate` — это Number (decimal), а не Integer или текстовое поле?

**Question:** Why is `field_estimate` a Number (decimal) and not an Integer or a text field?

**Ответ:** Оценка времени может включать дробные числа, например 0.5 или 1.5 часа. Integer хранит только целые числа. Текстовое поле не проверяет, что это число, не даёт сортировать его или считать.

**Answer:** Time estimates can include half numbers, like 0.5 or 1.5 hours. Integer can only store whole numbers. A text field can't check that the value is really a number, sort it, or do math with it.

> 📍 **Где/для чего:** поле `field_estimate` на Task; заполняется на форме задачи; для отображения оценки на карточке и подсчёта суммарной нагрузки по проекту.

---

## 5

**Вопрос:** В чём разница между `field.storage.*.yml` и `field.field.*.yml` в конфигурации, и почему `field_project` требует оба файла?

**Question:** What is the difference between `field.storage.*.yml` and `field.field.*.yml` in configuration, and why does `field_project` need both files?

**Ответ:** `field.storage.*.yml` настраивает поле на уровне базы данных — его тип и сколько значений оно может хранить. `field.field.*.yml` привязывает это поле к конкретному типу контента, со своей меткой и настройками. Хранилище создаётся один раз, но для каждого типа контента, использующего поле, нужен отдельный файл `field.field`.

**Answer:** `field.storage.*.yml` sets up the field at the database level — its type and how many values it can hold. `field.field.*.yml` attaches that field to one specific content type, with its own label and settings. You create the storage once, but you need a separate `field.field` file for each content type that uses it.

> 📍 **Где/для чего:** `field.storage.node.field_project.yml` + `field.field.node.task.field_project.yml`; оба хранятся в `config/sync`, экспортируются вместе с модулем/темой при деплое.

---

## 6

**Вопрос:** По задаче требуется, чтобы `field_status` предзаполнялось значением Backlog на форме создания. Где именно настраивается это значение, и что произойдёт с этим полем у узла Task, созданного программно через API, а не через форму?

**Question:** The task requires that `field_status` be pre-populated with the Backlog value on the creation form. Where exactly is this value configured, and what will happen to this field for a Task node created programmatically through the API, and not through a form?

**Ответ:** Значение по умолчанию задаётся в файле `field.field.*.yml`, в настройке `default_value`. Этот дефолт работает только через **веб-форму** — он автоматически заполняет поле, когда кто-то открывает страницу создания задачи. Если задача создаётся кодом через API (не через форму), этот дефолт не применяется. Поле останется пустым, если код сам не установит статус.

**Answer:** The default value is set in the `field.field.*.yml` file, in the `default_value` setting. This default only works through the **web form** — it fills the field automatically when someone opens the "create task" page. If a task is created by code through the API (not the form), this default is skipped. The field will stay empty unless the code sets the status itself.

> 📍 **Где/для чего:** `default_value: backlog` в `field.field.node.task.field_status.yml`; для того чтобы новая задача сразу попадала в колонку Backlog на доске без ручного выбора статуса.

---

## 7

**Вопрос:** Почему `field_attachments` реализовано как ссылка на Media (Media reference), а не как обычное File-поле?

**Question:** Why is `field_attachments` implemented as a Media reference (link to the Media entity), and not as a regular File field?

**Ответ:** С Media один файл можно использовать в разных местах, и к нему можно добавить дополнительную информацию (например, alt-текст или тип файла). Media также даёт единую библиотеку для управления всеми файлами, и поддерживает то, что не является файлом, например удалённые видео. Обычное File-поле просто хранит один файл без переиспользования и доп. информации.

**Answer:** With Media, one file can be reused in many places, and you can add extra info to it (like alt text or file type). Media also gives a central library to manage all files in one place, and it supports things that aren't files, like remote videos. A plain File field just stores one file with no reuse and no extra info.

> 📍 **Где/для чего:** поле `field_attachments` на Task, тип `entity_reference` → Media; используется на форме задачи и в Full view mode для показа прикреплённых файлов/изображений.

---

## 8

**Вопрос:** Чем сущность Media принципиально отличается от сущности File?

**Question:** How is the Media entity fundamentally different from the File entity?

**Ответ:** File — это простая запись о загруженном файле: путь, тип, размер. У неё нет дополнительных полей. Media — более богатая сущность: у неё есть свои типы (бандлы), свои дополнительные поля, история версий, и её можно использовать в разных местах одновременно.

**Answer:** File is a basic record of an uploaded file — just its path, type, and size. It has no extra fields. Media is a richer entity: it has its own types (bundles), its own extra fields, version history, and can be linked from many places at once.

> 📍 **Где/для чего:** используется модуль Media (ядро Drupal); бандлы Image и Document настроены под задачи проекта; для единого хранилища вложений всего сайта, а не только Task.

---

## 9

**Вопрос:** Почему в задаче создаётся отдельный media-бандл для документов, а не переиспользуется существующий бандл Image?

**Question:** Why does the task create a separate media bundle for documents, and not reuse the existing Image bundle?

**Ответ:** У каждого бандла свои поля и правила. Бандл Image ожидает поля только для изображений (например, alt-текст и размер) и разрешает только форматы изображений. Это не подходит для PDF или Word-файлов. Отдельный бандл Document позволяет настроить правильные поля и правильные разрешённые форматы для документов.

**Answer:** Each bundle has its own fields and rules. The Image bundle expects image-only fields (like alt text and size) and only allows image file types. That doesn't work for PDFs or Word files. A separate Document bundle lets us set the right fields and the right allowed file types for documents.

> 📍 **Где/для чего:** новый media type `document` (`media.type.document.yml`) с полем File, разрешены расширения pdf/docx/xlsx; для прикрепления к задачам и страницам документации.

---

## 10

**Вопрос:** Как технически ограничивается список допустимых типов медиа (Image и Document) для поля `field_attachments`, если это entity reference на Media?

**Question:** How is the list of allowed media types (Image and Document) technically limited for the `field_attachments` field if it is an entity reference on Media?

**Ответ:** В настройках поля есть опция "Target bundles" (часть настроек обработчика ссылок). Там выбираются Image и Document, и только эти два типа будут разрешены для поля.

**Answer:** In the field's settings, there's an option called "Target bundles" (part of the reference handler settings). You choose Image and Document there, and only those two types will be allowed for this field.

> 📍 **Где/для чего:** `handler_settings.target_bundles: [image, document]` в `field.field.node.task.field_attachments.yml`; чтобы в Media Library редактор не видел лишние типы медиа (видео, аудио и т.п.).

---

## 11

**Вопрос:** Чем виджет Media Library отличается от стандартного виджета автокомплита entity reference, и почему `field_attachments` требует именно его?

**Question:** How does the Media Library widget differ from the standard entity reference autocomplete widget, and why does `field_attachments` require it?

**Ответ:** Media Library показывает визуальную сетку файлов с миниатюрами, поиском, фильтрами и позволяет загружать новые файлы прямо там. Виджет автокомплита просто просит ввести название в текстовое поле. Для выбора файлов визуальная библиотека намного удобнее и понятнее, особенно для нетехнических редакторов.

**Answer:** The Media Library shows a visual grid of files with thumbnails, search, filters, and lets you upload new files right there. The autocomplete widget just asks you to type a name into a text box. For choosing files, the visual library is much easier and clearer, especially for editors who aren't technical.

> 📍 **Где/для чего:** widget `media_library_widget` в form display Task для `field_attachments`; для удобного прикрепления файлов при заполнении задачи через форму.

---

## 12

**Вопрос:** Почему для конструктора страниц документации выбраны Paragraphs, а не одно большое текстовое поле body с WYSIWYG-редактором?

**Question:** Why did you choose Paragraphs for the documentation page builder, rather than one large body text field with a WYSIWYG editor?

**Ответ:** Paragraphs позволяют строить страницу из отдельных переиспользуемых блоков — у каждого блока (код, callout, изображение) свои поля, правила и оформление. Блоки легко переставлять местами. Одно поле body с WYSIWYG — это просто один большой кусок HTML без структуры, без отдельных правил для каждого блока и без переиспользования.

**Answer:** Paragraphs let you build a page from separate, reusable blocks — each block (like a code block, a callout, or an image) has its own fields, rules, and design. You can reorder blocks easily. A single body field with WYSIWYG editor is just one big blob of HTML, with no structure, no separate rules per block, and no reuse.

> 📍 **Где/для чего:** content type Documentation, поле `field_content` (Paragraphs); типы параграфов: `code`, `callout`, `image` и др.; для конструктора страниц документации с гибкой структурой блоков.

---

## 13

**Вопрос:** Чем Entity reference revisions отличается от обычного Entity reference, и почему `field_content` использует именно его?

**Question:** How does Entity reference revisions differ from regular Entity reference, and why does `field_content` use it?

**Ответ:** Entity reference revisions ссылается на конкретную сохранённую версию параграфа, а не просто на его текущую версию. Поэтому, когда родительская страница сохраняется как новая версия, она запоминает, как именно выглядел каждый параграф в этот момент. Обычный entity reference всегда указывает на самую новую версию, поэтому старые версии страницы теряли бы историю параграфов.

**Answer:** Entity reference revisions points to one specific saved version of a paragraph, not just its current version. So when the parent page is saved as a new version, it remembers exactly how each paragraph looked at that moment. Regular entity reference always points to the newest version, so old versions of the page would lose their paragraph history.

> 📍 **Где/для чего:** тип поля `entity_reference_revisions` в `field.storage.node.field_content.yml`; для корректной истории версий страницы Documentation вместе с изменениями параграфов.

---

## 14

**Вопрос:** Зачем параграфам собственные ревизии, если родительский узел Documentation и так версионируется?

**Question:** Why do paragraphs even need their own revisions if the parent Documentation node is also revised?

**Ответ:** Параграфы хранятся отдельно от страницы. Без собственной истории версий, редактирование параграфа меняло бы его везде, где он используется, и старые версии страницы не могли бы показать старое содержимое параграфа. Ревизии параграфов позволяют каждой версии страницы иметь свой точный снимок параграфов — это нужно для правильной истории и безопасного отката.

**Answer:** Paragraphs are stored separately from the page. Without their own version history, editing a paragraph would change it everywhere it's used, and old page versions couldn't show the old paragraph content. Paragraph revisions let each version of the page keep its own exact snapshot of the paragraphs, so you can view history correctly and roll back safely.

> 📍 **Где/для чего:** ревизии включены по умолчанию модулем Paragraphs; используется вместе с ревизиями Documentation для функции "откатить версию страницы" в админке.

---

## 15

**Вопрос:** Почему `field_language` и `field_callout_type` реализованы как List (text), а не как ссылка на термин таксономии, хотя по сути это тоже управляемые словари значений?

**Question:** Why are `field_language` and `field_callout_type` implemented as List (text) and not as taxonomy term reference, although in essence they are also managed dictionaries of values?

**Ответ:** Эти списки маленькие, фиксированные и управляются разработчиками — они тесно связаны с кодом (например, какой CSS-класс или шаблон использовать). Изменение всё равно требует правок кода, поэтому главное преимущество таксономии (редакторы могут добавлять новые термины через интерфейс) здесь бесполезно. Таксономия только добавила бы лишнюю сложность без причины.

**Answer:** These lists are small, fixed, and controlled by developers — they're closely tied to code (like which CSS class or template to use). Changing them always needs a code change anyway, so taxonomy's main benefit (editors can add new terms through the UI) isn't useful here. Taxonomy would just add extra complexity for no reason.

> 📍 **Где/для чего:** `field_language` на параграфе `code` (значения: php, js, twig и т.д.), `field_callout_type` на параграфе `callout` (info, warning, danger); используются в `hook_preprocess_paragraph()` для подбора CSS-класса.

---

## 16

**Вопрос:** Как Drupal определяет, что для параграфа типа `code` шаблоном должен быть `paragraph--code.html.twig`, а не общий `paragraph.html.twig`?

**Question:** How does Drupal determine that for a paragraph of type `code` the template should be `paragraph--code.html.twig` and not the generic `paragraph.html.twig`?

**Ответ:** У Drupal есть система "template suggestions" (предложения шаблонов). При рендеринге параграфа она строит список возможных имён шаблонов, добавляя тип параграфа к базовому имени — например, `paragraph--code`. Затем Drupal ищет самый конкретный файл шаблона, который реально существует, и использует общий `paragraph.html.twig` только если конкретного не найдено.

**Answer:** Drupal has a system called "template suggestions." When it renders a paragraph, it builds a list of possible template names, adding the paragraph's type (bundle) to the base name — like `paragraph--code`. Drupal then looks for the most specific template file that actually exists, and uses the general `paragraph.html.twig` only if no specific one is found.

> 📍 **Где/для чего:** файл `templates/paragraph--code.html.twig` в теме; создан для отдельного оформления блока кода (подсветка синтаксиса, кнопка "копировать").

---

## 17

**Вопрос:** Почему `hook_preprocess_paragraph()` должен вычислять CSS-класс на основе `field_language`, а не просто выводить значение поля напрямую в twig-шаблоне, и почему важно проверять бандл параграфа прямо в самом хуке?

**Question:** Why should `hook_preprocess_paragraph()` calculate a CSS class based on `field_language`, and not just print the field value directly in the twig template (`{{ content.field_language }}`), and why is it important to check the paragraph's bundle in the hook itself?

**Ответ:** Хорошая практика — держать логику (PHP-код) отдельно от отображения (Twig-шаблоны). Twig должен просто показывать данные, а не преобразовывать их — например, превращать сырое значение в имя CSS-класса. Проверка бандла внутри хука важна, потому что `hook_preprocess_paragraph()` срабатывает для **каждого** типа параграфа. Без проверки бандла код может попытаться прочитать `field_language` у параграфа, у которого этого поля вообще нет, что вызовет ошибку.

**Answer:** It's good practice to keep logic (PHP code) separate from display (Twig templates). Twig should just show data, not transform it — like turning a raw value into a CSS class name. The bundle check inside the hook matters because `hook_preprocess_paragraph()` runs for **every** paragraph type. Without checking the bundle first, the code might try to read `field_language` on a paragraph type that doesn't even have that field, causing an error.

> 📍 **Где/для чего:** `.theme` файл темы, функция `hook_preprocess_paragraph()` с проверкой `$variables['paragraph']->bundle() === 'code'`; для добавления класса `language-{lang}` к блоку кода.

---

## 18

**Вопрос:** Почему View `task_board` использует контекстный фильтр (contextual filter) по `field_project`, а не обычный фильтр с фиксированным значением или exposed-фильтр?

**Question:** Why does View `task_board` use the contextual filter on `field_project` rather than a regular fixed-value or exposed filter?

**Ответ:** Контекстный фильтр автоматически берёт значение из URL страницы, например `/board/{project_id}`. Это значит, что один View может показывать доску для **любого** проекта, без жёсткой привязки к одному. Фиксированный фильтр навсегда привязал бы View к одному проекту. Exposed-фильтр требовал бы, чтобы пользователь вводил или выбирал значение вручную каждый раз, вместо автоматического получения из страницы.

**Answer:** A contextual filter gets its value automatically from the page URL, like `/board/{project_id}`. This means one View can show the board for **any** project, without hardcoding a specific one. A fixed filter would lock the View to just one project forever. An exposed filter would need the user to type or select the value by hand each time, instead of it coming automatically from the page.

> 📍 **Где/для чего:** View `task_board`, страница с путём `/board/%project_id` и contextual filter `field_project_target_id`; чтобы одна доска работала для всех проектов сайта.

---

## 19

**Вопрос:** Что произойдёт с View, если контекстный фильтр не получит значение аргумента (например, при обращении к странице доски без указания проекта), и какие настройки это контролируют?

**Question:** What happens to the View if the context filter does not receive an argument value (for example, when accessing a board page without specifying a project), and what settings control this?

**Ответ:** Это контролируется настройкой контекстного фильтра "When the filter value is NOT available" (когда значение фильтра недоступно). Можно выбрать: показать все результаты, ничего не показывать, показать ошибку "страница не найдена", показать сводку, или использовать значение по умолчанию из другого источника. Если эта настройка неправильная, доска может показать задачи со всех проектов вперемешку или пустую страницу/ошибку.

**Answer:** This is controlled by the contextual filter's setting called "When the filter value is NOT available." You can choose: show all results, show nothing, show a "page not found" error, show a summary, or use a default value from somewhere else. If this setting isn't configured correctly, the board might show tasks from all projects mixed together, or show an empty/error page.

> 📍 **Где/для чего:** в `task_board` выбрано "Display 'Page not found'"; чтобы доска без указания проекта не показывала смешанные данные всех проектов.

---

## 20

**Вопрос:** Почему View доски включает режим AJAX, и что технически меняется по сравнению с обычным (не-AJAX) View при взаимодействии со страницей?

**Question:** Why does the Board View enable AJAX mode, and what does it technically change compared to a regular (non-AJAX) View when interacting with the page?

**Ответ:** Режим AJAX позволяет View обновляться самостоятельно — фильтры, сортировка, пагинация — без перезагрузки всей страницы. Только маленькая изменённая часть страницы загружается и обновляется в фоне. Это очень важно для Kanban-доски, где пользователи постоянно фильтруют и перетаскивают карточки. Без AJAX каждое маленькое действие перезагружало бы всю страницу, что кажется медленным и ломает плавную работу доски.

**Answer:** AJAX mode lets the View update itself — filters, sorting, pagination — without reloading the whole page. Only the small changed part of the page is fetched and updated in the background. This matters a lot for a Kanban board, where users filter and drag cards around constantly. Without AJAX, every small action would reload the entire page, which feels slow and breaks the smooth board experience.

> 📍 **Где/для чего:** настройка "Use AJAX" включена в display Page у View `task_board`; для плавного обновления карточек при фильтрации и после drag&drop.

---

## 21

**Вопрос:** Почему карточка задачи в колонке рендерится в режиме отображения Teaser, а при клике открывается режим Full в модальном окне, а не один и тот же режим отображения в обоих местах?

**Question:** Why is the task card in the column rendered in the Teaser view mode, and when you click on it, it opens the Full mode in the modal window, and not the same view mode in both places?

**Ответ:** Режимы отображения позволяют одному контенту показывать разное количество информации в разных местах. Teaser показывает только ключевую информацию (заголовок, исполнитель, оценка) — идеально для маленькой карточки на загруженной доске. Full показывает всё (описание, вложения, все поля) — подходит для детального просмотра. Использование Full на доске сделало бы её слишком загруженной; использование Teaser в модальном окне скрыло бы важные детали.

**Answer:** View modes let the same content show different amounts of information in different places. Teaser mode shows just the key info (title, assignee, estimate) — perfect for a small card on a busy board. Full mode shows everything (description, attachments, all fields) — right for a detailed view. Using Full mode on the board would make it too crowded; using Teaser mode in the modal would hide important details.

> 📍 **Где/для чего:** row style View настроен на "Content: Task (Teaser)"; ссылка на карточке открывает `/node/{id}` в модалке через Ajax (Full mode); для баланса компактности доски и полноты информации.

---

## 22

**Вопрос:** Чем подход Frontend Editing (модальное окно + inline-попап для редактирования поля) отличается от стандартного перехода на `/node/{id}/edit` с точки зрения UX и что происходит технически?

**Question:** How does the Frontend Editing approach (modal window + inline popup for editing a field) differ from the standard transition to `/node/{id}/edit`, in terms of UX and what happens technically?

**Ответ:** С точки зрения UX пользователь остаётся там же, где был — на доске — вместо перехода на отдельную страницу редактирования и потери контекста. Редактирование происходит прямо там, в попапе или модальном окне. Технически форма редактирования загружается через AJAX-запрос в модальное окно, и при отправке ответ обновляет только эту часть страницы — вместо загрузки целой новой страницы с последующим возвратом пользователя обратно.

**Answer:** In terms of UX, the user stays right where they are — on the board — instead of jumping to a different edit page and losing their place. Editing happens right there, in a popup or modal window. Technically, the edit form loads through an AJAX request into the modal, and when submitted, the response updates just that part of the page — instead of loading a whole new page and then sending the user back afterward.

> 📍 **Где/для чего:** кастомная ссылка/кнопка на карточке использует Drupal Ajax API (`use-ajax` + `OpenModalDialogCommand`); для быстрого редактирования полей задачи без ухода с доски.

---

## 23

**Вопрос:** Почему изменение статуса при drag&drop должно сопровождаться AJAX-запросом к серверу, а не ограничиваться визуальным переносом карточки в DOM, и почему обработчик этого переноса должен быть оформлен как `Drupal.behaviors`, а не как обычный `$(document).ready()`/скрипт, выполняемый один раз при загрузке страницы?

**Question:** Why should a status change during drag&drop be accompanied by an AJAX request to the server, and not limited to the visual transfer of the card into the DOM, and why should the handler for this transfer be designed as `Drupal.behaviors`, and not as a regular `$(document).ready()`/script, executed once when the page is loaded?

**Ответ:** Если просто переместить карточку визуально (в браузере), изменение нигде не сохраняется. Оно исчезает при перезагрузке страницы, и другие пользователи его не увидят — база данных не обновляется. Поэтому нужен AJAX-запрос, чтобы реально сохранить новый статус на сервере. Что касается `Drupal.behaviors`: Drupal заново запускает эти поведения каждый раз, когда новый контент добавляется на страницу через AJAX. Обычный `$(document).ready()` выполняется только один раз, при самой первой загрузке страницы — он никогда не привяжет drag&drop к карточкам, добавленным или заменённым позже через AJAX.

**Answer:** If you only move the card visually (in the browser), the change is not saved anywhere. It disappears when the page reloads, and other users won't see it — the database never gets updated. That's why an AJAX request is needed, to actually save the new status on the server. As for `Drupal.behaviors`: Drupal reruns these behaviors every time new content is added to the page through AJAX. A plain `$(document).ready()` only runs once, at the very first page load — it would never attach drag&drop to cards that get added or replaced later through AJAX.

> 📍 **Где/для чего:** кастомная JS-библиотека `js/task-board.js` (`Drupal.behaviors.taskBoard`), подключена через `*.libraries.yml`; при drop отправляет AJAX POST на кастомный route, обновляющий `field_status` задачи в БД.