---
custom-width: 80
---
Во втором блоке мы расширили DrupalJira и реализовали основную функциональность для работы с проектами, задачами, документацией и Kanban-доской.

В задании 2.1 мы создали модель контента для проектов и задач. Для этого добавили два типа материалов: Project и Task. Проект содержит заголовок и Body. Задача связана с конкретным проектом через Entity Reference. Также у задачи есть статус, исполнитель и оценка времени. Для статуса мы создали четыре значения: Backlog, In Progress, Review и Done. Поле исполнителя связано с пользователями Drupal, а поле оценки позволяет хранить числовое значение. Таким образом, задача имеет всю необходимую информацию для дальнейшей работы на Kanban-доске.

В задании 2.2 мы добавили работу с файлами через Media и Media Library. В Drupal включили необходимые модули и создали два типа медиа: Image и Document. Для Document используется File field, а для Image — Image field. После этого в типе Task добавили поле Attachments, которое позволяет прикреплять к задаче несколько файлов. Для выбора файлов используется Media Library, поэтому пользователь может выбирать уже существующие медиа или загружать новые. Мы также проверили, что загруженные изображения корректно отображаются на странице задачи.

В задании 2.3 мы реализовали раздел Documentation для проектов. Здесь использовали Paragraphs, чтобы сделать документацию гибкой и состоящей из переиспользуемых блоков. Создали тип Documentation, который связан с конкретным Project. Внутри документации можно добавлять любое количество Paragraphs и менять их порядок. Мы создали четыре типа блоков: Text, Code, Callout и Image. Text используется для обычного текста, Code — для примеров кода с возможностью указать язык, Callout — для информационных, предупреждающих и успешных сообщений, а Image — для изображений из Media Library. Для Code предусмотрены языки PHP, Twig, YAML и JavaScript. Для Callout есть типы Info, Warning и Success. Таким образом, получилась небольшая гибкая система базы знаний внутри DrupalJira.

В задании 2.4 мы реализовали Kanban-доску для задач. Для этого создали View с machine name task_board и отдельный маршрут `/project/%/board`. Доска получает проект из контекстного фильтра и показывает только задачи выбранного проекта. Задачи распределяются по четырём колонкам в зависимости от значения field_status: Backlog, In Progress, Review и Done.

Для отображения задач используется Teaser view mode. Мы специально скрыли в Teaser ненужные поля, например Project, Status, Attachments и стандартные links, чтобы карточка содержала только необходимую информацию.

Также мы включили AJAX для View и использовали Frontend Editing. Пользователь может открыть задачу непосредственно с Kanban-доски. При клике на карточку задача открывается в модальном окне в Full view, поэтому пользователь не переходит на отдельную страницу. Внутри Full view можно редактировать описание задачи через Frontend Editing, без перехода на стандартную страницу редактирования.

Для перемещения задач между колонками мы реализовали native HTML5 drag-and-drop. У каждой карточки есть `draggable=true`. JavaScript обрабатывает события `dragstart`, `dragover`, `dragleave`, `drop` и визуально перемещает карточку между колонками. После drop выполняется AJAX-запрос на наш custom Drupal route. Backend проверяет права пользователя, проверяет, что это именно Task, валидирует новый статус и сохраняет его в `field_status`. Поэтому изменение не только отображается визуально, но и сохраняется в Drupal. После перезагрузки страницы задача остаётся в новой колонке.

Для этой функциональности мы создали собственный custom module `drupaljira_board`. В нём находятся routes и controller для обновления статуса и загрузки задачи в модальное окно. JavaScript и CSS находятся в custom theme `drupaljira_theme`.

Кроме самой функциональности мы уделили внимание Drupal-подходу к конфигурации. Настройки типов материалов, полей, Views, Media, Paragraphs и Frontend Editing хранятся в `config/sync`, поэтому конфигурацию можно переносить между окружениями через Git. После изменений мы проверяли `config:status`, чтобы убедиться, что активная конфигурация и репозиторий синхронизированы.

Также мы настроили автоматический запуск PHPCBF при сохранении PHP-файла в VS Code. Для этого добавили небольшой shell script, который запускает PHPCBF внутри DDEV, и настроили Run On Save. В результате при сохранении PHP-файла автоматически исправляются нарушения Drupal Coding Standards, которые PHPCBF умеет исправлять.

В итоге во втором блоке мы построили полноценную функциональность вокруг проектов и задач: создали модель данных, добавили вложения, сделали гибкую систему документации и реализовали интерактивную Kanban-доску с модальными окнами, Frontend Editing и сохранением статусов через AJAX.

---

# English version

In the second block, we extended DrupalJira and implemented the main functionality for working with projects, tasks, documentation, and a Kanban board.

In Task 2.1, we created the content model for projects and tasks. We added two content types: Project and Task. A Project contains a title and a Body field. A Task is connected to a specific project using an Entity Reference field. A task also has a status, an assignee, and an estimate. We created four possible statuses: Backlog, In Progress, Review, and Done. The assignee field references Drupal users, and the estimate field stores a numeric value. This gives us all the basic information needed to manage tasks on the Kanban board.

In Task 2.2, we added file management using Media and Media Library. We enabled the required Drupal modules and created two media types: Image and Document. The Document media type uses a File field, while the Image media type uses an Image field. Then we added an Attachments field to the Task content type. This field allows users to attach multiple files to a task. The Media Library widget is used to select or upload files. We also tested the functionality and verified that uploaded images are displayed correctly on the task page.

In Task 2.3, we implemented a Documentation system for projects. We used the Paragraphs module to make the documentation flexible and based on reusable content blocks. We created a Documentation content type connected to a specific Project. Each documentation page can contain any number of Paragraphs, and their order can be changed.

We created four paragraph types: Text, Code, Callout, and Image. Text is used for regular documentation text. Code is used for code examples and supports languages such as PHP, Twig, YAML, and JavaScript. Callout is used for informational, warning, and success messages. Image is used for images from the Media Library. This gives the project a flexible knowledge base system.

In Task 2.4, we implemented a Kanban board for tasks. We created a View with the machine name `task_board` and a route `/project/%/board`. The selected project is passed through a contextual filter, so the board displays only tasks belonging to that project. Tasks are divided into four columns according to the `field_status` value: Backlog, In Progress, Review, and Done.

The tasks are displayed using the Teaser view mode. We configured the Teaser display so that unnecessary fields, such as Project, Status, Attachments, and standard links, are hidden. This keeps the task cards simple and focused.

We also enabled AJAX for the View and configured Frontend Editing. A user can open a task directly from the Kanban board. When the user clicks a task card, the task is loaded into a modal window using the Full view mode. The user does not have to navigate to a separate task page. Inside the Full view, the task description can be edited using Frontend Editing without opening the standard Drupal edit page.

For moving tasks between columns, we implemented native HTML5 drag and drop. Each task card has `draggable=true`. JavaScript handles events such as `dragstart`, `dragover`, `dragleave`, and `drop`. When a card is dropped into another column, it is moved visually and an AJAX request is sent to a custom Drupal route.

On the backend, the controller checks the user's permissions, verifies that the entity is a Task, validates the new status, and saves the value to `field_status`. This means that the change is not only visual — it is actually saved in Drupal. After refreshing the page, the task remains in the new column.

For this functionality, we created a custom Drupal module called `drupaljira_board`. The module contains the routes and controller responsible for updating task statuses and loading tasks into the modal window. The JavaScript and CSS for the board are included in our custom theme, `drupaljira_theme`.

We also followed Drupal's configuration management approach. The configuration for content types, fields, Views, Media, Paragraphs, and Frontend Editing is stored in `config/sync`. This allows us to transfer configuration between environments using Git. After making changes, we checked the configuration status to make sure that the active Drupal configuration and the repository were synchronized.

Finally, we added automatic PHPCBF execution when saving PHP files in VS Code. We created a small shell script that runs PHPCBF inside the DDEV environment and configured the Run On Save extension. Now, when a PHP file is saved, PHPCBF automatically fixes coding standard violations that can be fixed automatically.

Overall, during the second block we built the main project management functionality for DrupalJira. We created the data model, added file attachments, implemented a flexible documentation system, and built an interactive Kanban board with modal task views, Frontend Editing, drag and drop, and AJAX-based status persistence.