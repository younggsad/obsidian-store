---
custom-width: 80
---
---

> [!info] Как пользоваться / How to use
> 
> **RU:** Конспект собран из всех блоков проекта **DrupalJira** (окружение → конфигурация → контент-модель → сущности → сервисы/плагины → render/cache → hooks/events/migrate → workflow/update → фронтенд/Figma → Playwright). Каждый вопрос: сначала RU, затем EN. Учите по блокам; в конце — шпаргалка «X vs Y» и «фразы для собеседования».
> 
> **EN:** Built from all **DrupalJira** blocks. Each item: RU first, then EN. Study block by block; the end has an "X vs Y" cheat sheet and interview soundbites.

## Карта блоков / Block map

|#|Блок / Block|За что отвечает / What it covers|
|---|---|---|
|**A**|[[#A. Окружение и инструменты / Environment & Tooling]]|DDEV, Composer, git, Xdebug, PHPCS/PHPStan/GrumPHP|
|**B**|[[#B. Управление конфигурацией / Configuration Management]]|`cex`/`cim`, config vs content, Config Split|
|**C**|[[#C. Контент-модель, Views, тема, AJAX / Content Modeling, Views, Theming, AJAX]]|Fields, Media, Paragraphs, Views, view modes, template suggestions, `Drupal.behaviors`|
|**D**|[[#D. Content Entity, Entity API, Routing, Form API / Content Entity, Entity API, Routing, Form API]]|TimeLog, CRUD, EntityQuery, upcasting, Form API, States API, form alter|
|**E**|[[#E. Сервисы, DI, Plugin API / Services, DI, Plugin API]]|Services, Dependency Injection, ReportGenerator plugin, Block/Widget/Formatter|
|**F**|[[#F. Render API, Theme, Cache API / Render API, Theme, Cache API]]|Render arrays, `hook_theme`, preprocess, cache tags/contexts/max-age, DRY|
|**G**|[[#G. Hooks, Events, Migrate API / Hooks, Events, Migrate API]]|Hook vs Event Subscriber, custom events, Migrate (CSV → Tasks)|
|**H**|[[#H. Workflow, Update Hooks, Access / Workflow, Update Hooks, Access]]|Content Moderation, `hook_update_N`, route access, Kanban/Scrum|
|**I**|[[#I. Figma MCP, фронтенд и AI-код / Figma MCP, Frontend & AI code]]|Figma → Drupal, Twig, CSS vars, accessibility, AI-review|
|**J**|[[#J. Playwright E2E-тестирование / Playwright E2E Testing]]|E2E, fixtures, locators, flaky, security, AI tests|
|**K**|[[#K. Шпаргалка / Cheat Sheet]]|Таблицы «X vs Y», сквозные принципы, фразы для интервью|

---

# A. Окружение и инструменты / Environment & Tooling

> [!info] За что отвечает блок / What this block covers
> 
> **RU:** Как развернуть воспроизводимое окружение (DDEV, Composer), что коммитить в git, как отлаживать (Xdebug) и как автоматически держать качество кода (PHPCS, PHPStan, GrumPHP).
> 
> **EN:** How to build a reproducible environment (DDEV, Composer), what to commit, how to debug (Xdebug) and how to enforce code quality automatically.

## A1. DDEV vs «голый» Docker Compose

**Q (RU):** Чем DDEV отличается от Docker Compose, который пишешь вручную для Drupal?

**Q (EN):** How is DDEV different from hand-written Docker Compose for a Drupal project?

**RU:** Docker Compose только запускает контейнеры — всё остальное пишешь и поддерживаешь сам. DDEV добавляет готовые пресеты проектов, автоматические HTTPS-адреса (`*.ddev.site`) и простые команды (`ddev import-db`, `ddev xdebug`); вся команда получает идентичное окружение из одного конфиг-файла.

**EN:** Compose only starts containers; you maintain everything else. DDEV adds project presets, automatic HTTPS URLs (`*.ddev.site`) and simple commands (`ddev import-db`, `ddev xdebug`), so the whole team gets an identical environment from one config file.

## A2. `composer create-project` vs архив ядра

**Q (RU):** Почему Drupal ставят через `composer create-project`, а не скачиванием архива?

**Q (EN):** Why deploy Drupal with `composer create-project` rather than downloading the core archive?

**RU:** Архив — статичный снимок без учёта зависимостей. Composer управляет ядром, модулями и библиотеками как графом зависимостей с lock-файлом → одинаковую сборку можно воспроизвести на любой машине.

**EN:** An archive is a static snapshot with no dependency tracking. Composer manages core, modules and libraries as a dependency graph with a lock file, so the exact same build can be reproduced anywhere.

## A3. `composer.json` vs `composer.lock`

**Q (RU):** В чём разница и почему коммитить оба?

**Q (EN):** What's the difference and why commit both?

**RU:** `composer.json` — что вы **хотите** (диапазоны версий). `composer.lock` — что **реально установлено** (точные версии). Без lock разные машины получат разные совместимые версии; оба в git → установка одинакова везде.

**EN:** `composer.json` states what you _want_ (version ranges); `composer.lock` records what was _installed_ (exact versions). Without the lock, machines may get different compatible versions; committing both makes installs identical.

## A4. Профиль Minimal vs Standard

**Q (RU):** Почему устанавливать с профилем Minimal?

**Q (EN):** Why install with the Minimal profile instead of Standard?

**RU:** Standard ставит демо-типы контента, views и модули, которые никто не просил — мусор для чистки. Minimal даёт чистый старт: всё в конфигурации создано командой осознанно.

**EN:** Standard pre-installs demo content types, views and modules nobody asked for. Minimal starts clean, so everything in config was deliberately built by the team.

## A5. Что в `.gitignore`

**Q (RU):** Почему `/web/core`, `/web/modules/contrib`, `/vendor/` — в `.gitignore`?

**Q (EN):** Why should `/web/core`, `/web/modules/contrib` and `/vendor/` be gitignored?

**RU:** Это сторонний код, полностью воспроизводимый из `composer.lock`. Коммит удваивает хранение, раздувает репозиторий и даёт огромные конфликты слияния при каждом обновлении.

**EN:** It's third-party code fully reproducible from `composer.lock`. Committing it doubles storage, bloats the repo and causes huge merge conflicts on every update.

## A6. `settings.php` vs `settings.local.php`

**Q (RU):** Почему `settings.local.php` не в git, а `settings.php` коммитится?

**Q (EN):** Why keep `settings.local.php` out of git but commit `settings.php`?

**RU:** `settings.php` — общая конфигурация, одинаковая везде. `settings.local.php` — значения конкретной машины и секреты (пароли БД): коммит утёк бы учётные данные или сломал настройки других разработчиков.

**EN:** `settings.php` holds shared config. `settings.local.php` holds machine-specific values and secrets (DB passwords); committing it would leak credentials or break other developers' setups.

## A7. Режимы Xdebug: `debug` vs `develop`/`coverage`/`profile`

**Q (RU):** Чем `debug` отличается от остальных и почему нужен именно он?

**Q (EN):** How does `debug` differ from `develop`/`coverage`/`profile`, and why is it needed?

**RU:** `develop` — лучше вывод ошибок; `coverage` — покрытие тестами; `profile` — производительность. Ни один не даёт взаимодействовать с работающим кодом. Только `debug` соединяется с IDE: точки останова и пошаговый просмотр.

**EN:** `develop` improves error output, `coverage` measures test coverage, `profile` measures performance — none lets you interact with running code. Only `debug` connects to the IDE for breakpoints and step-through.

## A8. Path mapping в PhpStorm

**Q (RU):** Зачем нужен path mapping и что ломается при ошибке?

**Q (EN):** Why is path mapping needed and what breaks if it's wrong?

**RU:** PHP работает в контейнере с путями, отличными от диска хоста; path mapping переводит одни в другие. При ошибке Xdebug подключается, но точки останова **молча никогда не срабатывают**.

**EN:** PHP runs in a container with different file paths than the host. Path mapping translates between them. If wrong, Xdebug still connects but breakpoints silently never trigger.

## A9. Пошаговая отладка vs `var_dump()`/логи

**Q (RU):** Почему step debugging лучше дампов для сложной логики?

**Q (EN):** Why is step debugging better than `var_dump()`/logs for complex logic?

**RU:** Дампы требуют заранее угадать, куда смотреть (пробы и ошибки). Отладчик позволяет остановиться где угодно, посмотреть любую переменную и проследить реальный путь выполнения.

**EN:** Dumps require guessing where to look in advance (slow trial and error). A debugger lets you pause anywhere, inspect any variable live and follow the real execution path.

## A10. Триггер Xdebug: web vs Drush

**Q (RU):** Чем отличается триггер подключения для веб-запроса и команды Drush?

**Q (EN):** How does Xdebug's trigger differ between a web request and a Drush command?

**RU:** Веб-запрос триггерится cookie или параметром URL. Drush — CLI-процесс без браузера: триггер должен прийти из переменной окружения (`XDEBUG_SESSION` / `XDEBUG_MODE`), заданной до запуска команды.

**EN:** A web request triggers via cookie or URL parameter. Drush is a CLI process with no browser, so the trigger must be an environment variable (`XDEBUG_SESSION`/`XDEBUG_MODE`) set before running.

## A11. Почему отладка переключаемая

**Q (RU):** Почему отладку нужно переключать, а не держать всегда включённой?

**Q (EN):** Why must debugging be toggled rather than always on?

**RU:** Xdebug добавляет накладные расходы на каждый запрос и постоянно пытался бы подключиться к IDE, которая не слушает — замедление и захламлённые логи.

**EN:** Xdebug adds overhead to every request and would constantly try connecting to an IDE that isn't listening, slowing things down and cluttering logs.

## A12. PHPCS vs PHPStan

**Q (RU):** Чем принципиально различаются PHPCS и PHPStan?

**Q (EN):** How are PHPCS and PHPStan fundamentally different?

**RU:** **PHPCS** — _стиль_ (форматирование, именование), не понимает логику. **PHPStan** — _корректность_ (типы, несуществующие методы), не заботится о форматировании. Вместе покрывают форму и суть.

**EN:** **PHPCS** checks _style_ (formatting, naming); **PHPStan** checks _correctness_ (types, undefined methods). Together they cover form and substance; alone each misses half.

## A13. Стандарты `Drupal` vs `DrupalPractice`

**Q (RU):** В чём разница?

**Q (EN):** What's the difference between the `Drupal` and `DrupalPractice` standards?

**RU:** `Drupal` — обязательный стандарт форматирования. `DrupalPractice` — рекомендательный: отмечает нежелательные паттерны и вероятные ошибки. Вместе ловят и плохой стиль, и плохие практики.

**EN:** `Drupal` is the mandatory formatting standard; `DrupalPractice` is advisory, flagging discouraged patterns and likely mistakes. Together they catch bad formatting and bad practices.

## A14. GrumPHP — только изменённые кастомные файлы

**Q (RU):** Почему GrumPHP проверяет только изменённые кастомные файлы?

**Q (EN):** Why does GrumPHP check only modified custom files?

**RU:** Core и contrib — не код команды: проверка бессмысленный шум. Сканирование всего дерева сделало бы каждый коммит мучительно медленным.

**EN:** Core and contrib aren't the team's code — checking them is noise, and scanning everything would make every commit painfully slow.

## A15. Зачем GrumPHP, если можно запускать вручную

**Q (RU):** Зачем GrumPHP вместо ручного запуска PHPCS/PHPStan?

**Q (EN):** Why use GrumPHP instead of running PHPCS/PHPStan manually?

**RU:** Ручной запуск легко забыть. GrumPHP (git hook) делает проверки автоматическими и обязательными для каждого коммита.

**EN:** Manual runs are easy to forget. GrumPHP (a git hook) makes checks automatic and mandatory for every commit.

## A16. `--no-verify`

**Q (RU):** Какие компромиссы у обхода хука?

**Q (EN):** What are the trade-offs of bypassing a hook with `--no-verify`?

**RU:** Полезно в настоящих экстренных случаях (хотфикс, ложное срабатывание), но можно злоупотреблять. Страховка — **CI-проверка на сервере**, ловящая пропущенное локально.

**EN:** Useful for genuine emergencies (hotfixes, false positives) but can be abused. The backstop is a **server-side CI check**.

## A17. Воспроизводимость инструментов через `composer install`

**Q (RU):** Почему конфигурация инструментов должна воспроизводиться через `composer install`?

**Q (EN):** Why must tooling config be reproducible via `composer install`, not set up manually?

**RU:** Ручная локальная настройка невидима новым разработчикам и CI — они не узнают правил и версий. Всё из закоммиченных файлов → одинаковые проверки везде.

**EN:** A manual local setup is invisible to new developers and CI. Everything coming from committed files guarantees identical checks everywhere.

---

# B. Управление конфигурацией / Configuration Management

> [!info] За что отвечает блок / What this block covers
> 
> **RU:** Как переносить структуру сайта между окружениями через YAML-файлы (`drush cex` / `drush cim`), чем конфигурация отличается от контента и какой рабочий процесс безопасен.
> 
> **EN:** How to move site structure between environments via YAML (`drush cex`/`cim`), config vs content, and the safe workflow.

## B1. Файлы (`cex`/`cim`) vs копия БД

**Q (RU):** Почему конфигурацию синхронизируют файлами, а не копированием БД?

**Q (EN):** Why sync config via files instead of copying the database?

**RU:** Копия БД перезапишет и контент — обычно нежелательно. Файлы можно диффить и ревьюить в git как обычный код и не нужен прямой доступ к БД между окружениями.

**EN:** A DB copy would overwrite content with config. Files are diffable and reviewable in git like code and need no direct DB access between environments.

## B2. Конфигурация vs контент

**Q (RU):** Чем отличаются и почему это важно?

**Q (EN):** How is configuration different from content and why does it matter?

**RU:** Конфигурация — структура сайта (одинакова везде). Контент — данные (уникальны в каждом окружении). Разделение гарантирует, что `cim` безопасно перепишет структуру, не трогая реальный контент.

**EN:** Config is structure (same everywhere); content is data (unique per environment). The split lets `cim` safely rewrite structure without touching real content.

## B3. «No differences» на странице Configuration Synchronization

**Q (RU):** Что значит «нет различий»?

**Q (EN):** What does "no differences" on the config sync page mean?

**RU:** Активная конфигурация в БД точно совпадает с YAML в папке синхронизации — импорт полностью удался.

**EN:** The active configuration in the DB exactly matches the YAML in the sync folder — import fully succeeded.

## B4. Папка sync не в `sites/default/files`

**Q (RU):** Почему папка синхронизации не должна быть в `sites/default/files`?

**Q (EN):** Why shouldn't the sync folder be inside `sites/default/files`?

**RU:** Эта папка публично доступна по вебу. Конфигурация раскрывает всю структуру сайта — скачать её можно для разведки.

**EN:** That folder is publicly web-accessible, and config reveals the site's entire structure (reconnaissance risk).

## B5. Config Split / Config Ignore

**Q (RU):** Какую проблему решают и почему пока не возникла?

**Q (EN):** What problem do Config Split/Config Ignore solve and why hasn't it appeared yet?

**RU:** Нужны, когда конфигурация должна законно различаться по окружениям или часть правится вживую на проде. Пока это простой пайплайн одного окружения.

**EN:** Needed when config legitimately differs per environment or part of it is edited live in production. Our pipeline is still single-environment.

## B6. Порядок: `git pull` → `cim`

**Q (RU):** Почему сначала `git pull` и `cim`, а не UI и коммит без экспорта?

**Q (EN):** Why `git pull` then `cim`, not edit via UI then commit without exporting?

**RU:** Git — источник истины: pull приносит нужное состояние, `cim` его применяет. Изменения в UI живут только в БД, пока не выполнен `cex`; коммит без экспорта ничего не сохранит.

**EN:** Git is the source of truth: pull brings intended state, `cim` applies it. UI changes exist only in the DB until `cex`, so committing without exporting saves nothing.

## B7. Забыли `config:export` перед коммитом

**Q (RU):** Что будет?

**Q (EN):** What happens if a developer forgets `config:export` before committing?

**RU:** Изменение остаётся только в локальной БД, невидимое git и команде. Хуже: следующий `cim` **молча перезапишет и потеряет** его, приводя БД к устаревшим YAML.

**EN:** The change stays only in the local DB. Worse, the next `cim` silently overwrites and loses it, forcing the DB to match the outdated YAML.

> [!tip] Workflow / Рабочий цикл `git pull` → `drush cim` → работа в UI → `drush cex` → `git add/commit/push`

---

# C. Контент-модель, Views, тема, AJAX / Content Modeling, Views, Theming, AJAX

> [!info] За что отвечает блок / What this block covers
> 
> **RU:** Как моделировать данные (Entity reference, List, Number, cardinality, Media, Paragraphs), как они хранятся в конфиге (`field.storage` vs `field.field`), как строится Kanban-доска на Views (contextual filter, AJAX, view modes) и как подключаются шаблоны, preprocess и JS (`Drupal.behaviors`).
> 
> **EN:** How to model data (Entity reference, List, Number, cardinality, Media, Paragraphs), how it's stored in config, how the Kanban board is built with Views, and how templates, preprocess and JS (`Drupal.behaviors`) are wired.

## C1. Entity reference vs свободный текст (Task → Project)

**Q (RU):** Почему связь Task → Project — Entity reference на node `project`, а не текст/автокомплит?

**Q (EN):** Why is Task → Project an Entity reference rather than a free-text/autocomplete field?

**RU:** Можно выбрать только реальный проект (без опечаток). Views строит списки/фильтры по настоящей связи. При переименовании проекта связь не ломается.

**EN:** Only real projects can be picked (no typos); Views can build lists/filters on the real relationship; renaming a project doesn't break the link.

> 📍 `field_project`, используется как contextual filter во View `task_board`.

## C2. `field_status` — List (text), а не taxonomy/Boolean

**Q (RU):** Почему статус — List (text) с машинными значениями?

**Q (EN):** Why is `field_status` a List (text) with machine values, not taxonomy or Boolean?

**RU:** Список маленький и фиксированный, меняется только разработчиками. Taxonomy нужна для списков, которые редакторы пополняют сами. Boolean — только 2 значения, а статусов больше.

**EN:** The list is small, fixed, developer-controlled. Taxonomy is for editor-extendable lists; Boolean has only two values.

> 📍 Значения = machine names колонок Kanban; AJAX-обновление при drag&drop.

## C3. Cardinality поля

**Q (RU):** Что такое cardinality и почему у `field_project` она = 1?

**Q (EN):** What is cardinality and why is `field_project` = 1?

**RU:** Cardinality — сколько значений хранит поле (1, N, unlimited). Задача принадлежит одному проекту; unlimited позволил бы привязать её к нескольким.

**EN:** Cardinality = how many values a field holds (one, N, unlimited). A task belongs to one project; unlimited would allow many.

## C4. `field_estimate` — Number (decimal)

**Q (RU):** Почему decimal, а не Integer/текст?

**Q (EN):** Why Number (decimal) rather than Integer or text?

**RU:** Оценка может быть дробной (0.5, 1.5 ч). Integer — только целые. Текст не валидируется как число, не сортируется и не считается.

**EN:** Estimates can be fractional. Integer stores only whole numbers; text can't be validated as a number, sorted or summed.

## C5. `field.storage.*.yml` vs `field.field.*.yml`

**Q (RU):** В чём разница и зачем оба файла?

**Q (EN):** What's the difference and why does a field need both?

**RU:** **storage** — поле на уровне БД: тип и cardinality (создаётся один раз). **field.field** — привязка к конкретному bundle (метка, настройки, default). Для каждого типа контента, использующего поле, нужен свой `field.field`.

**EN:** **storage** defines the field at DB level (type, cardinality) once. **field.field** attaches it to one bundle (label, settings, default). One `field.field` per bundle.

## C6. Default value и API-создание

**Q (RU):** Где задаётся предзаполнение Backlog и что будет при создании Task через API?

**Q (EN):** Where is the Backlog default set and what happens when a Task is created via API?

**RU:** В `field.field.*.yml` → `default_value`. Работает только через **веб-форму**. При программном создании (`Node::create()`) дефолт не применяется — поле останется пустым, пока код сам не задаст статус.

**EN:** In `field.field.*.yml` → `default_value`. It works only through the **web form**; programmatic creation skips it, so the field stays empty unless code sets it.

## C7. Media reference vs File field

**Q (RU):** Почему `field_attachments` — Media reference, а не File?

**Q (EN):** Why is `field_attachments` a Media reference instead of a File field?

**RU:** Файл можно переиспользовать, добавлять метаданные (alt, тип), есть единая медиабиблиотека, поддерживаются не-файлы (удалённое видео). File-поле — просто один файл без переиспользования.

**EN:** Media allows reuse, extra metadata, a central library and non-file items (remote video). A File field just stores one file.

## C8. Media vs File entity

**Q (RU):** Чем Media принципиально отличается от File?

**Q (EN):** How is the Media entity fundamentally different from File?

**RU:** File — простая запись об загруженном файле (путь, тип, размер), без доп. полей. Media — богатая сущность: bundles, свои поля, ревизии, использование во многих местах.

**EN:** File is a basic record (path, type, size). Media is a richer entity: bundles, custom fields, revisions, linkable from many places.

## C9. Отдельный media bundle Document

**Q (RU):** Почему не переиспользовать Image?

**Q (EN):** Why a separate Document bundle instead of reusing Image?

**RU:** У bundle свои поля и правила. Image ожидает alt/размер и только форматы изображений — не подходит для PDF/Word. Document позволяет задать нужные поля и расширения (pdf/docx/xlsx).

**EN:** Each bundle has its own fields/rules. Image expects image fields and formats; Document allows the right fields and file extensions.

## C10. Ограничение типов медиа в поле

**Q (RU):** Как технически ограничить Media-типы (Image, Document)?

**Q (EN):** How do you restrict allowed media types on an entity reference?

**RU:** В настройках поля → **Target bundles** (`handler_settings.target_bundles: [image, document]`).

**EN:** Via **Target bundles** in handler settings (`handler_settings.target_bundles: [image, document]`).

## C11. Media Library widget vs autocomplete

**Q (RU):** Чем отличается и почему нужен именно он?

**Q (EN):** How does the Media Library widget differ from autocomplete and why is it needed?

**RU:** Визуальная сетка с миниатюрами, поиском, фильтрами и загрузкой «на месте». Autocomplete требует набирать имя — неудобно для нетехнических редакторов.

**EN:** A visual grid with thumbnails, search, filters and inline upload vs typing a name — far better for non-technical editors.

## C12. Paragraphs vs большое WYSIWYG-поле

**Q (RU):** Зачем Paragraphs для страниц документации?

**Q (EN):** Why Paragraphs instead of one big WYSIWYG body?

**RU:** Страница из отдельных переиспользуемых блоков (code, callout, image) со своими полями, правилами и оформлением; блоки легко переставлять. Одно body — неструктурированный кусок HTML.

**EN:** The page is built from reusable blocks, each with its own fields, rules and styling, easy to reorder. A single body is an unstructured blob of HTML.

## C13. Entity reference revisions vs Entity reference

**Q (RU):** Чем отличается и почему `field_content` использует его?

**Q (EN):** How does Entity reference revisions differ from Entity reference?

**RU:** Ссылается на **конкретную ревизию** параграфа, а не на текущую. Старые версии страницы сохраняют именно тот вид параграфов. Обычная ссылка всегда на самую новую → история терялась бы.

**EN:** It points to a **specific revision** of the paragraph, so old page versions keep their exact paragraphs; a plain reference always points to the newest.

## C14. Зачем параграфам свои ревизии

**Q (RU):** Если родитель и так версионируется?

**Q (EN):** Why do paragraphs need their own revisions if the parent is revisioned?

**RU:** Параграфы хранятся отдельно. Без своих ревизий правка меняла бы их везде, а старые версии страницы не показали бы старое содержимое. Это нужно для истории и безопасного отката.

**EN:** Paragraphs are stored separately. Without own revisions, edits would change them everywhere and old page versions couldn't show old content; needed for history and rollback.

## C15. `field_language`/`field_callout_type` — List, не taxonomy

**Q (RU):** Почему List (text)?

**Q (EN):** Why List (text) rather than taxonomy?

**RU:** Списки маленькие, фиксированные и привязаны к коду (CSS-класс, шаблон). Изменение всё равно требует правки кода, поэтому главное преимущество taxonomy (редакторы добавляют термины) бесполезно — лишняя сложность.

**EN:** Small, fixed, tied to code (CSS class/template). Changes need code anyway, so taxonomy's main benefit is moot — just extra complexity.

## C16. Template suggestions

**Q (RU):** Как Drupal выбирает `paragraph--code.html.twig`, а не `paragraph.html.twig`?

**Q (EN):** How does Drupal choose `paragraph--code.html.twig` over `paragraph.html.twig`?

**RU:** Через **template suggestions**: строится список имён (база + bundle), берётся самый конкретный существующий файл; общий — запасной.

**EN:** Via **template suggestions**: a list of names (base + bundle) is built and the most specific existing file wins; the generic one is a fallback.

## C17. `hook_preprocess_paragraph()` и проверка bundle

**Q (RU):** Зачем считать CSS-класс в хуке и зачем проверять bundle?

**Q (EN):** Why compute the CSS class in preprocess and why check the bundle?

**RU:** Логика — в PHP, Twig только показывает. Хук срабатывает для **каждого** типа параграфа; без проверки `$paragraph->bundle() === 'code'` код читает `field_language` там, где его нет → ошибка.

**EN:** Logic in PHP, Twig only displays. The hook fires for **every** paragraph type; without a bundle check it reads a missing field and errors.

```php
function mytheme_preprocess_paragraph(array &$variables) {
  $paragraph = $variables['paragraph'];
  if ($paragraph->bundle() === 'code') {
    $variables['attributes']['class'][] = 'language-' . $paragraph->get('field_language')->value;
  }
}
```

## C18. Contextual filter vs fixed/exposed

**Q (RU):** Почему `task_board` использует contextual filter по `field_project`?

**Q (EN):** Why does `task_board` use a contextual filter on `field_project`?

**RU:** Значение берётся из URL (`/board/{project_id}`) → один View для любого проекта. Fixed привязал бы к одному проекту; exposed требует ручного выбора.

**EN:** The value comes from the URL, so one View serves any project. Fixed locks to one project; exposed needs manual input.

## C19. Нет аргумента у contextual filter

**Q (RU):** Что будет и что это контролирует?

**Q (EN):** What happens when the contextual filter gets no argument?

**RU:** Настройка **«When the filter value is NOT available»**: показать всё / ничего / 404 / сводку / значение по умолчанию. Мы выбрали **Page not found**, чтобы не смешивать задачи всех проектов.

**EN:** Controlled by **"When the filter value is NOT available"** (show all, nothing, 404, summary, default). We chose **Page not found** to avoid mixing projects.

## C20. AJAX во View

**Q (RU):** Зачем AJAX и что технически меняется?

**Q (EN):** Why enable AJAX and what changes technically?

**RU:** Фильтры/сортировка/пагинация обновляют только часть страницы в фоне, без полной перезагрузки — критично для Kanban.

**EN:** Filters/sorting/pagination refresh only part of the page in the background — essential for a Kanban board.

## C21. Teaser на доске, Full в модалке

**Q (RU):** Почему разные view modes?

**Q (EN):** Why Teaser on the board and Full in the modal?

**RU:** View modes показывают один контент с разной детализацией. Teaser — компактная карточка (заголовок, исполнитель, оценка); Full — детали (описание, вложения).

**EN:** View modes show the same content at different detail levels: compact Teaser vs detailed Full.

## C22. Frontend editing (модалка) vs `/node/{id}/edit`

**Q (RU):** В чём разница по UX и технически?

**Q (EN):** How does modal/inline editing differ from `/node/{id}/edit`?

**RU:** UX: пользователь остаётся на доске. Технически форма грузится AJAX-запросом в модальное окно (`use-ajax` + `OpenModalDialogCommand`), ответ обновляет только часть страницы.

**EN:** UX: the user stays on the board. Technically the form loads via AJAX into a modal (`use-ajax` + `OpenModalDialogCommand`) and only part of the page updates.

## C23. Drag&drop: AJAX к серверу и `Drupal.behaviors`

**Q (RU):** Зачем AJAX-запрос при drag&drop и почему `Drupal.behaviors`, а не `$(document).ready()`?

**Q (EN):** Why an AJAX request on drag&drop and why `Drupal.behaviors` over `$(document).ready()`?

**RU:** Визуальный перенос нигде не сохраняется — нужен запрос, который обновит `field_status` в БД. `Drupal.behaviors` перезапускаются при каждом добавлении контента через AJAX; `ready()` выполнится один раз и не привяжется к новым карточкам.

**EN:** A visual move saves nothing; a request must persist `field_status`. Behaviors re-run on every AJAX content insertion; `ready()` runs once and misses new cards.

---

# D. Content Entity, Entity API, Routing, Form API / Content Entity, Entity API, Routing, Form API

> [!info] За что отвечает блок / What this block covers
> 
> **RU:** Кастомная сущность `time_log` (учёт времени), программный CRUD и EntityQuery, роуты с upcasting и правами, форма списания времени (Form API, States API, валидация) и добавление ссылки «Remove time» через form alter.
> 
> **EN:** The custom `time_log` entity, programmatic CRUD and EntityQuery, routes with upcasting and permissions, the time write-off form (Form API, States API, validation) and adding a "Remove time" link via form alter.

## D1. Content Entity vs Config Entity

**Q (RU):** В чём разница и почему TimeLog — Content Entity?

**Q (EN):** Content vs Config entity, and why Content for TimeLog?

**RU:** Content Entity — **данные сайта** (задачи, записи времени). Config Entity — **конфигурация** (настройки, типы). TimeLog хранит реальные записи пользователей → Content.

**EN:** Content entities store **site data**; config entities store **configuration**. TimeLog holds real user records → Content.

## D2. Content Entity vs node

**Q (RU):** Почему не обычный content type?

**Q (EN):** How is a custom content entity different from a node and why not a content type?

**RU:** Node — один из видов Content Entity для обычного контента. TimeLog — техническая сущность: ей не нужны revisions, translations и стандартные списки контента.

**EN:** A node is one kind of content entity for editorial content. TimeLog is technical and needs no revisions, translations or content lists.

## D3. `revisionable = FALSE`, `translatable = FALSE`

**Q (RU):** Что значат и какие последствия?

**Q (EN):** What do they mean and what are the implications?

**RU:** Нет истории версий и нельзя переводить → нет revision history и translation UI у TimeLog.

**EN:** No revision history and no translations → no revision/translation UI for TimeLog.

## D4. Зачем явно отключать флаги

**Q (RU):** Почему в задании указано явно, а не по умолчанию генератора?

**Q (EN):** Why explicitly disable them instead of generator defaults?

**RU:** Чтобы явно показать, что возможности не нужны, и зафиксировать требование на уровне определения сущности, а не случайного поведения генератора.

**EN:** To make the requirement explicit at the entity-definition level rather than relying on generator defaults.

## D5. `task` и `uid` — entity reference

**Q (RU):** Почему не числовые поля с ID?

**Q (EN):** Why entity references rather than plain numeric ID fields?

**RU:** Drupal понимает связь: может загружать связанные сущности и использовать её в EntityQuery. Число хранит только ID.

**EN:** Drupal understands the relationship — loads related entities, supports EntityQuery joins. A number only stores an ID.

## D6. `hours` decimal vs `log_date` date-only

**Q (RU):** Почему такие типы и зачем дате без времени?

**Q (EN):** Why decimal `hours` and date-only `log_date`?

**RU:** `hours` — числовое (2.5) → decimal. `log_date` — календарный день (`2026-09-14`); важен день списания, не точное время.

**EN:** `hours` is numeric (2.5); `log_date` is a calendar day — the exact time isn't needed.

## D7. Генератор `drush generate:entity:content`

**Q (RU):** Почему генератор, а не вручную?

**Q (EN):** Why use the generator instead of hand-writing the entity?

**RU:** Он создаёт корректный каркас (класс, routing, forms, list builder); лишнее удаляется, остальное адаптируется.

**EN:** It scaffolds the correct structure (class, routing, forms, list); then you prune and adapt.

## D8. Upcasting `{task}` → `NodeInterface`

**Q (RU):** Почему через `options.parameters`, а не `load()` вручную?

**Q (EN):** Why upcast `{task}` via `options.parameters` instead of loading manually?

**RU:** Drupal сам превращает ID в объект, проверяет существование и bundle (`type: entity:node`, `bundle: task`). Контроллер сразу получает `NodeInterface`.

**EN:** Drupal converts the ID to an object and validates existence and bundle; the controller receives a ready `NodeInterface`.

```yaml
drupaljira.timelog_debug.crud:
  path: '/timelog-debug/{task}/crud'
  options:
    parameters:
      task:
        type: entity:node
        bundle: [task]
  requirements:
    _permission: 'access administration pages'
```

## D9. Upcasting и 404

**Q (RU):** Как связан upcasting и 404 вместо fatal error?

**Q (EN):** How does upcasting relate to a 404 for a non-existent `{task}`?

**RU:** Если сущность не найдена, Drupal останавливает обработку и возвращает **404**; контроллер не получает неверный объект.

**EN:** If the entity isn't found, Drupal stops and returns **404**; the controller never receives an invalid object.

## D10. `access administration pages` на debug-роутах

**Q (RU):** Почему не доступны всем авторизованным?

**Q (EN):** Why restrict `timelog-debug/*` with an admin permission?

**RU:** Это debug/demo-инструменты администратора; нельзя давать обычным пользователям запускать CRUD-тесты.

**EN:** They're admin debug tools; regular users shouldn't run CRUD tests or see internals.

## D11. EntityQuery vs прямой SQL

**Q (RU):** В чём разница и почему SQL запрещён?

**Q (EN):** EntityQuery vs direct SQL — why is SQL prohibited?

**RU:** EntityQuery работает через Entity API, знает сущности и поля, не зависит от структуры таблиц. SQL привязывает код к схеме БД и обходит абстракции.

**EN:** EntityQuery goes through the Entity API and is schema-agnostic; direct SQL couples code to table layout and bypasses abstractions.

## D12. Сумма часов: aggregate vs `loadMultiple()`

**Q (RU):** Почему достаточно агрегирующего запроса?

**Q (EN):** Why is an aggregate query enough instead of `loadMultiple()` + PHP summing?

**RU:** Не нужно загружать все сущности: БД вернёт одно итоговое значение — меньше данных и объектов в PHP.

**EN:** No need to load every entity; the DB returns one total — less data and fewer objects in PHP.

## D13. Render array вместо `echo`/`print`

**Q (RU):** Почему контроллер возвращает render array?

**Q (EN):** Why return a render array instead of echoing HTML?

**RU:** Это стандартный способ Drupal: система сама рендерит, кеширует, темизирует. `echo` смешивает логику и вывод.

**EN:** It's Drupal's standard way — rendering, caching and theming are handled by the system; `echo` mixes logic and output.

## D14. `create()` vs `save()`

**Q (RU):** Почему `create()` не сохраняет?

**Q (EN):** Why doesn't `create()` persist the entity?

**RU:** `create()` только создаёт объект в памяти; `save()` пишет в БД. Можно подготовить/изменить данные до сохранения.

**EN:** `create()` builds an in-memory object; `save()` writes to the DB, letting you prepare data first.

> CRUD: `create() → save() → load() → set()+save() → delete()`

## D15. Route requirement vs проверка в контроллере/`hook_form_alter`

**Q (RU):** Почему доступ в Task 3.2 через `_permission` в route?

**Q (EN):** Why use a route `_permission` requirement instead of a check in the controller?

**RU:** `hook_form_alter` меняет форму; route requirement решает, можно ли вообще открыть маршрут — Drupal проверяет **до** вызова контроллера.

**EN:** Form alter changes a form; a route requirement decides whether the route opens at all, checked **before** the controller runs.

## D16. `FormBase` vs `ConfirmFormBase`

**Q (RU):** Почему форма списания на `FormBase`?

**Q (EN):** Why `FormBase` and not `ConfirmFormBase`?

**RU:** `FormBase` — обычная форма ввода; `ConfirmFormBase` — подтверждение действия (удаление). У нас форма с несколькими полями.

**EN:** `FormBase` is for data-entry forms; `ConfirmFormBase` is for confirming an action (delete).

## D17. States API (`#states`)

**Q (RU):** Как работает для «Excess Reason» и почему это клиентское решение?

**Q (EN):** How does `#states` work and why is it client-side?

**RU:** Связывает состояние поля с другим элементом: при отметке checkbox поле становится видимым и обязательным прямо в браузере, без отправки формы и без своего JS.

**EN:** It binds one element's state to another: checking the box shows/requires the field in the browser, no round trip and no custom JS.

```php
$form['over_estimate_reason'] = [
  '#type' => 'textarea',
  '#states' => [
    'visible'  => [':input[name="over_estimate"]' => ['checked' => TRUE]],
    'required' => [':input[name="over_estimate"]' => ['checked' => TRUE]],
  ],
];
```

## D18. Почему нужна server-side validation

**Q (RU):** Почему States API недостаточно?

**Q (EN):** Why is States API not enough and why duplicate in `validateForm()`?

**RU:** States управляет интерфейсом, но не защищает данные: можно отправить изменённый HTTP-запрос. Сервер обязан проверить причину, дату и часы.

**EN:** States only controls the UI; a crafted HTTP request bypasses it. The server must validate reason, date and hours.

## D19. Валидация в `validateForm()`, а не constraints сущности

**Q (RU):** Почему проверки даты/часов в форме?

**Q (EN):** Why are `log_date`/`hours` checks in `validateForm()` instead of entity constraints?

**RU:** В рамках задачи это правила **сценария формы**: проверка перед созданием TimeLog с понятной ошибкой. (В общем случае constraints сущности защищают данные независимо от формы.)

**EN:** Here they're form-workflow rules with clear user errors. (In general, entity constraints protect data regardless of entry point.)

## D20. `validateForm()` vs `submitForm()`

**Q (RU):** Разница этапов и почему создание в `submitForm()`?

**Q (EN):** Difference between validate and submit stages?

**RU:** `validateForm()` проверяет и может остановить отправку; `submitForm()` выполняется после успешной проверки и делает основное действие — создаёт TimeLog.

**EN:** Validate checks and can halt; submit runs only after successful validation and performs the action.

## D21. Пользователь — программно в `submitForm()`

**Q (RU):** Почему не поле выбора пользователя?

**Q (EN):** Why bind the current user in code, not via a user select?

**RU:** TimeLog должен показывать, кто реально списал время; автора нельзя выбирать вручную. ID берётся из текущей сессии.

**EN:** TimeLog must record who actually logged the time; the author can't be user-selected — taken from the session.

## D22. `hook_form_alter()` vs `hook_form_FORM_ID_alter()` / `BASE_FORM_ID`

**Q (RU):** В чём разница и что лучше?

**Q (EN):** What's the difference and which is preferable?

**RU:** Общий `hook_form_alter()` затрагивает **все формы**; специфичные — только нужную (по base form id или form id). Меньше риска затронуть лишнее.

**EN:** The generic hook affects all forms; specific ones target a base/ID — safer.

## D23. Ссылка «Remove time» только для `task`

**Q (RU):** Как гарантируется?

**Q (EN):** How do you ensure the link appears only for `task` nodes?

**RU:** Проверка bundle: `$node->bundle() === 'task'` до добавления элемента.

**EN:** Check `$node->bundle() === 'task'` before adding the element.

## D24. Task ID — из entity формы, не из URL

**Q (RU):** Почему?

**Q (EN):** Why take the task ID from the form entity rather than the URL?

**RU:** Форма уже содержит конкретную node; это надёжнее разбора URL (формат/параметры могут отличаться).

**EN:** The form already holds the node; more reliable than parsing a URL whose format may vary.

## D25. Новая (несохранённая) Task

**Q (RU):** Почему ссылку не показываем/не ломаем?

**Q (EN):** Why must the link not appear/error on an unsaved task?

**RU:** У новой node ещё нет ID; маршрут `/task/{id}/log-time` требует существующую задачу.

**EN:** A new node has no ID yet; the route needs an existing task.

## D26. `Url::fromRoute()` vs конкатенация

**Q (RU):** Почему не `'/task/' . $nid`?

**Q (EN):** Why `Url::fromRoute()` over string concatenation?

**RU:** Используется имя route и параметры; Drupal строит корректный URL (префиксы, alias, смена пути). Безопаснее и соответствует Routing API.

**EN:** It uses the route name and params; Drupal builds the right URL (prefixes, aliases, path changes) — safer and idiomatic.

```php
$form['remove_time'] = [
  '#type' => 'link',
  '#title' => t('Remove time'),
  '#url' => Url::fromRoute('drupaljira.log_time', ['task' => $node->id()]),
];
```

## D27. Элемент через alter и `node_form`

**Q (RU):** Почему не нужно переопределять контроллер/route?

**Q (EN):** Why no need to override the edit controller/route?

**RU:** Alter получает готовый form array и добавляет элемент; базовый controller/route работают как раньше.

**EN:** The alter hook receives the existing form array and adds an element; the standard controller/route are untouched.

---

# E. Сервисы, DI, Plugin API / Services, DI, Plugin API

> [!info] За что отвечает блок / What this block covers
> 
> **RU:** `TaskStatService` (бизнес-логика статистики), Dependency Injection, собственная система плагинов отчётов (`ReportGenerator`) и три kernel-плагина: Block, Field Widget, Field Formatter.
> 
> **EN:** `TaskStatService` (statistics logic), Dependency Injection, a custom `ReportGenerator` plugin system and three core plugin types: Block, Field Widget, Field Formatter.

## E1. Сервис vs класс со статикой

**Q (RU):** Что такое сервис и чем отличается от класса со static-методами?

**Q (EN):** What is a service and how does it differ from a static-method class?

**RU:** Сервис — объект под управлением **Service Container**, для переиспользуемой логики, получает зависимости через DI. Статика работает без объекта и скрывает зависимости.

**EN:** A service is an object managed by the **Service Container**, reusable, with injected dependencies. Static methods hide dependencies and are hard to replace.

## E2. Зачем выносить в `TaskStatService`

**Q (RU):** Почему не дублировать расчёт?

**Q (EN):** Why extract time calculation into `TaskStatService`?

**RU:** Одна реализация: изменили в одном месте; меньше дублирования и ошибок.

**EN:** One implementation — change it once; less duplication and fewer bugs.

## E3. Entity API/EntityQuery в сервисе, а не SQL

**Q (RU):** Почему?

**Q (EN):** Why Entity API/EntityQuery rather than SQL in the service?

**RU:** Entity API абстрагирует структуру БД: код переносим и соответствует архитектуре Drupal.

**EN:** The Entity API abstracts the DB structure; code is portable and idiomatic.

## E4. `*.services.yml` vs `new`

**Q (RU):** Чем регистрация отличается от `new TaskStatService()`?

**Q (EN):** Registering in `services.yml` vs `new TaskStatService()`?

**RU:** Контейнер сам создаёт объект и подставляет зависимости; при `new` их передаёшь вручную.

**EN:** The container builds the object and injects dependencies; with `new` you wire them manually.

```yaml
services:
  drupaljira.task_stat:
    class: Drupal\drupaljira\Service\TaskStatService
    arguments: ['@entity_type.manager']
```

## E5. Когда ошибка в `services.yml` обнаружится

**Q (RU):** Несуществующий класс/синтаксис?

**Q (EN):** When is a missing class/syntax error in `services.yml` detected?

**RU:** Когда Drupal загружает/создаёт сервис: при `drush cr` или первом обращении (зависит от ошибки).

**EN:** When Drupal loads/instantiates the service — at `drush cr` or first use, depending on the error.

## E6. `\Drupal::` в конструкторе — временный компромисс

**Q (RU):** Почему это не хорошая практика?

**Q (EN):** Why is `\Drupal::` in the constructor only a temporary compromise?

**RU:** Это временный способ до полного DI; создаёт скрытые зависимости. Правильно — передавать их через конструктор.

**EN:** A stopgap before full DI; it hides dependencies. Pass them through the constructor.

## E7. Service Locator vs DI

**Q (RU):** Разница и почему DI лучше?

**Q (EN):** Service Locator vs DI — why DI?

**RU:** Locator достаёт зависимость внутри класса «по месту». DI передаёт через конструктор: зависимости явные, тесты проще, поведение предсказуемее.

**EN:** A locator fetches dependencies inside the class; DI passes them in — explicit, testable, predictable.

## E8. `\Drupal::` и unit-тесты

**Q (RU):** Почему усложняет?

**Q (EN):** Why does `\Drupal::` complicate unit testing?

**RU:** Класс обращается к глобальному контейнеру — mock подставить трудно. С DI mock передаётся в конструктор.

**EN:** The class reaches the global container, making mocking hard; with DI you pass a mock to the constructor.

## E9. DI сервиса vs DI формы

**Q (RU):** Чем отличаются механизмы?

**Q (EN):** DI in a service vs DI in a `FormBase` form?

**RU:** Сервис: зависимости в `services.yml` + конструктор. Форма: Drupal вызывает статический `create(ContainerInterface $container)`, который достаёт зависимости из контейнера.

**EN:** Service: `services.yml` + constructor. Form: Drupal calls static `create(ContainerInterface $container)` to pull dependencies.

## E10. `ContainerInjectionInterface` и `create()`

**Q (RU):** Что это?

**Q (EN):** What are `ContainerInjectionInterface` and `create()`?

**RU:** Интерфейс означает «объект получает зависимости из контейнера». `create()` получает контейнер и конструирует объект с нужными зависимостями.

**EN:** The interface marks container-injectable objects; `create()` receives the container and builds the object with its dependencies.

```php
public static function create(ContainerInterface $container) {
  return new static($container->get('drupaljira.task_stat'));
}
```

## E11. Неверный порядок `arguments`

**Q (RU):** Что будет?

**Q (EN):** What if `arguments` order doesn't match the constructor?

**RU:** Зависимости попадут не в те параметры → ошибка типов или неверное поведение.

**EN:** Dependencies hit the wrong parameters → type error or wrong behavior.

## E12. Стандарт «сервисы только через конструктор»

**Q (RU):** Какие проблемы решает?

**Q (EN):** Why require services only via the constructor?

**RU:** Явные зависимости, проще тесты, меньше скрытой связности, легче поддерживать.

**EN:** Explicit dependencies, easier tests, less hidden coupling, easier maintenance.

## E13. Plugin API

**Q (RU):** Что это и почему не `switch/if` по типу отчёта?

**Q (EN):** What is the Plugin API and why not a `switch` on report type?

**RU:** Автоматически находит и использует разные реализации одной задачи. Новые отчёты добавляются без правки большого `switch`; каждый отчёт — отдельный плагин.

**EN:** It auto-discovers different implementations of one job. New reports need no big `switch`; each report is its own plugin.

## E14. Три части ReportGenerator

**Q (RU):** Из чего состоит?

**Q (EN):** What three parts make up the plugin system?

**RU:** 1) `ReportGeneratorInterface` — контракт; 2) `ReportGeneratorManager` — поиск и управление плагинами; 3) плагины `ProjectSummaryReport`, `OverdueTasksReport`.

**EN:** 1) interface (contract); 2) manager (discovery/management); 3) concrete plugins.

## E15. Зачем интерфейс

**Q (RU):** Почему не просто метод `generate()`?

**Q (EN):** Why a formal interface?

**RU:** Гарантирует одинаковые методы и сигнатуры → менеджер работает со всеми плагинами одинаково.

**EN:** Guarantees identical methods/signatures so the manager treats all plugins uniformly.

## E16. `generate()` и `getLabel()` раздельно

**Q (RU):** Что это даёт?

**Q (EN):** Why split `generate()` and `getLabel()`?

**RU:** Разные обязанности: данные отчёта и его название — код проще (SRP).

**EN:** Separate responsibilities (data vs name) — simpler code (SRP).

## E17. Как `DefaultPluginManager` находит плагины

**Q (RU):** Откуда знает о `ProjectSummaryReport` без регистрации?

**Q (EN):** How does the manager discover plugins without registration?

**RU:** Сканирует namespace `Plugin/ReportGenerator` и ищет классы с Attribute `#[ReportGenerator]` (id, label). После `drush cr` плагин попадает в definitions.

**EN:** It scans the plugin namespace for classes with the `#[ReportGenerator]` attribute (id, label); after cache rebuild they appear in definitions.

## E18. Attributes vs Annotations

**Q (RU):** Почему Attributes в Drupal 11?

**Q (EN):** Why PHP attributes over docblock annotations in Drupal 11?

**RU:** Attributes — часть языка PHP, структурированный синтаксис; Drupal 11 использует их для discovery (annotations — устаревающий подход).

**EN:** Attributes are native PHP with structured syntax; Drupal 11 uses them for discovery (annotations are being phased out).

## E19. DI в плагинах через `create()`

**Q (RU):** Почему не `\Drupal::service()` в `generate()`?

**Q (EN):** Why inject `TaskStatService` via `create()` into plugins?

**RU:** Зависимость передаётся при создании, а не скрыто внутри метода → тестируемость и чистая структура.

**EN:** The dependency is provided at construction rather than hidden in a method — testable and clean.

## E20. Block vs Field Widget vs Field Formatter

**Q (RU):** Для чего каждый?

**Q (EN):** What is each plugin type for?

**RU:** **Block** — отдельный блок контента. **Widget** — _ввод_ значения поля в форме. **Formatter** — _отображение_ значения поля.

**EN:** **Block** outputs a content block; **Widget** controls field _input_; **Formatter** controls field _display_.

## E21. Project Statistics Block — определение проекта

**Q (RU):** Как блок находит проект и почему не парсить URL?

**Q (EN):** How does the block detect the project and why not parse the URL?

**RU:** Через `CurrentRouteMatch` берёт `node`: project → он сам; task → проект из `field_project`. Drupal уже предоставляет параметр route.

**EN:** `CurrentRouteMatch` gives the node: project → itself; task → its `field_project`. Drupal already exposes the route parameter.

## E22. `ContainerFactoryPluginInterface` vs `ContainerInjectionInterface`

**Q (RU):** В чём разница?

**Q (EN):** Difference between the two?

**RU:** Для плагинов: `create()` дополнительно получает `$configuration`, `$plugin_id`, `$plugin_definition`. Для форм/контроллеров — `ContainerInjectionInterface` с простым `create($container)`.

**EN:** Plugins: `create()` also receives configuration, plugin ID and definition. Forms/controllers: simple `create($container)`.

```php
public static function create(ContainerInterface $container, array $configuration, $plugin_id, $plugin_definition) {
  return new static($configuration, $plugin_id, $plugin_definition, $container->get('drupaljira.task_stat'));
}
```

## E23. Hours + Minutes widget — двусторонняя конвертация

**Q (RU):** Зачем конвертировать туда и обратно?

**Q (EN):** Why convert in both directions?

**RU:** БД хранит decimal (`2.5`), пользователю удобнее «2 ч 30 мин»: при сохранении объединяем, при редактировании разбиваем обратно.

**EN:** The DB stores decimals; users prefer hours+minutes — combine on save, split on edit.

## E24. Formatter получает `TaskStatService` через DI

**Q (RU):** Почему не считать в `viewElements()`?

**Q (EN):** Why inject the service instead of querying in `viewElements()`?

**RU:** Не дублировать бизнес-логику; formatter только форматирует готовые данные.

**EN:** Avoid duplicating business logic; the formatter only formats ready data.

## E25. Доступность widget/formatter для типов полей

**Q (RU):** Что определяет?

**Q (EN):** What determines which field types a widget/formatter supports?

**RU:** Параметр `field_types` в Attribute (у нас `['decimal']`).

**EN:** The `field_types` parameter of the plugin attribute (here `['decimal']`).

---

# F. Render API, Theme, Cache API / Render API, Theme, Cache API

> [!info] За что отвечает блок / What this block covers
> 
> **RU:** Render arrays, `#markup` vs `#theme`, `hook_theme()` + preprocess + Twig, переопределение шаблонов темой, **Cache API** (tags, contexts, max-age, bubbling, инвалидация TimeLog), рефакторинг `DurationFormatter` (DRY).
> 
> **EN:** Render arrays, theme hooks with preprocess and Twig, template overrides, the **Cache API** (tags, contexts, max-age, bubbling, TimeLog invalidation) and the `DurationFormatter` DRY refactoring.

## F1. Render Array

**Q (RU):** Что это и почему не склеивать HTML-строки?

**Q (EN):** What is a render array and why not concatenate HTML?

**RU:** Структурированный PHP-массив, описывающий, что рендерить. Даёт Render API, Twig, Theme API, cache metadata, переопределение тем — presentation не смешивается с PHP.

**EN:** A structured array describing what to render; enables Render API, Twig, Theme API, cache metadata and overrides, keeping presentation out of PHP.

## F2. `#markup` vs `#theme`

**Q (RU):** Когда что?

**Q (EN):** `#markup` vs `#theme`?

**RU:** `#markup` — небольшой готовый текст/разметка. `#theme` — **тематизированный компонент** через theme hook и (обычно) Twig-шаблон.

**EN:** `#markup` — small ready markup; `#theme` — a themed component via theme hook and usually Twig.

## F3. `hook_theme()` vs прямой Twig

**Q (RU):** Почему регистрировать hook, а не `twig_render_template()`?

**Q (EN):** Why register via `hook_theme()` instead of rendering Twig directly?

**RU:** Подключает шаблон к Theme API: preprocess, theme overrides, theme registry, стандартный рендеринг. Прямой вызов обходит архитектуру.

**EN:** It integrates with the Theme API: preprocess, overrides, registry, standard rendering; a direct call bypasses it.

```php
function drupaljira_theme() {
  return ['drupaljira_project_stats' => [
    'variables' => ['logged' => NULL, 'remaining' => NULL],
    'template' => 'drupaljira-project-stats',
  ]];
}
```

## F4. Тема переопределяет шаблон модуля

**Q (RU):** Какой шаблон выберет Drupal при одноимённых файлах?

**Q (EN):** Which template wins when module and theme both have the file?

**RU:** Модуль регистрирует hook и исходный шаблон; после построения theme registry активная тема **переопределяет** его — используется версия темы.

**EN:** The module registers the hook; the active theme's template overrides it in the theme registry.

## F5. `hook_preprocess_HOOK()`

**Q (RU):** Почему подготовка переменных именно там?

**Q (EN):** Why prepare variables in preprocess, not Twig?

**RU:** Preprocess готовит данные для presentation, Twig отображает. Расчёты, форматирование, вызовы сервисов — в PHP.

**EN:** Preprocess prepares data; Twig displays. Calculations, formatting, service calls stay in PHP.

## F6. Уже отформатированные значения в шаблон

**Q (RU):** Почему не сырые числа и при чём границы компонентов?

**Q (EN):** Why pass formatted values and what boundary is preserved?

**RU:** Чёткое разделение: `TaskStatService` считает, preprocess готовит для показа, Twig отвечает за markup.

**EN:** Clear separation: service calculates, preprocess prepares display values, Twig handles markup.

## F7. Отключение модуля

**Q (RU):** Что будет с блоком и почему без fatal?

**Q (EN):** What happens to the block if its module is disabled?

**RU:** Компонент исчезает (код недоступен), но fatal error быть не должно — Drupal корректно обрабатывает отсутствующий компонент.

**EN:** The component disappears, and there should be no fatal error — Drupal handles missing components gracefully.

## F8. Theme hook + preprocess vs строка в `build()`

**Q (RU):** Архитектурная разница?

**Q (EN):** Architectural difference vs returning a string in `build()`?

**RU:** Theme hook разделяет данные, подготовку и presentation (service → preprocess → Twig). Строка смешивает логику и вывод — сложнее переопределять и поддерживать.

**EN:** A theme hook separates data, preparation and presentation; a string mixes them — harder to override and maintain.

## F9. Cache Tags / Contexts / `max-age`

**Q (RU):** За что отвечает каждый?

**Q (EN):** Tags vs contexts vs max-age?

**RU:** **Tags** — что инвалидировать при изменении данных. **Contexts** — когда нужны разные варианты кеша (user, roles, url…). **max-age** — как долго кеш валиден.

**EN:** **Tags**: what to invalidate on data change. **Contexts**: when cache varies. **max-age**: how long it stays valid.

## F10. Custom tag `drupaljira_project_stats:{nid}`

**Q (RU):** Почему не `node:{nid}`?

**Q (EN):** Why a custom tag instead of `node:{nid}`?

**RU:** Инвалидировать нужно именно кеш статистики проекта, а не всё, связанное с node — точнее.

**EN:** We want to invalidate only that project's statistics, not everything tied to the node.

## F11. `Cache::invalidateTags()` vs `drush cr`

**Q (RU):** Почему при save/delete TimeLog через tags?

**Q (EN):** Why `invalidateTags()` on TimeLog save/delete instead of `drush cr`?

**RU:** Инвалидирует только связанные записи. `drush cr` чистит значительно шире и является административной операцией, не частью логики приложения.

**EN:** It invalidates only related entries; `drush cr` is a broad admin operation, not application logic.

```php
use Drupal\Core\Cache\Cache;
Cache::invalidateTags(['drupaljira_project_stats:' . $project_id]);
```

## F12. Изоляция кеша проектов A и B

**Q (RU):** Как проверить?

**Q (EN):** How do you verify tags don't overlap between projects?

**RU:** Прогреть кеш A и B, изменить TimeLog проекта A: A пересчитается, кеш B остаётся валидным.

**EN:** Warm both, change a TimeLog in A: A recalculates, B stays cached.

## F13. Cache bubbling

**Q (RU):** Что это и зачем?

**Q (EN):** What is cache bubbling?

**RU:** Cache metadata дочернего render array «всплывает» к родителю — Drupal знает, от чего зависит весь результат.

**EN:** Child cache metadata propagates to parents so Drupal knows what the whole output depends on.

## F14. Context `user` vs `max-age: 0`

**Q (RU):** Почему для Kanban context, а не отключение кеша?

**Q (EN):** Why a `user`/`user.roles` context for Kanban instead of `max-age: 0`?

**RU:** Нужно кешировать, но отдельно для пользователей/ролей. Context создаёт варианты; `max-age: 0` убил бы полезное кеширование.

**EN:** Keep caching, but per user/role; contexts create variants while `max-age: 0` kills caching.

## F15. `user` vs `user.roles`

**Q (RU):** Как выбирать?

**Q (EN):** How to choose between `user` and `user.roles`?

**RU:** От чего реально зависит output: от конкретного пользователя → `user`; только от роли → `user.roles` (меньше вариантов кеша).

**EN:** By what output truly depends on; narrower context = fewer variants.

## F16. Проверка, что context работает

**Q (RU):** Как убедиться на практике?

**Q (EN):** How to verify the context actually changes output?

**RU:** Открыть доску двумя пользователями с разными assigned-задачами; если каждый видит только свои как `my task` — варианты кеша разные.

**EN:** Open the board as two users with different assigned tasks; each should see only their own marked `my task`.

## F17. Anonymous без session

**Q (RU):** Зачем отдельная проверка?

**Q (EN):** Why test anonymous users separately?

**RU:** Нет обычной authenticated identity: нужно убедиться, что нет PHP-ошибок и нет персональной отметки `my task`.

**EN:** No normal identity — verify no PHP errors and no personal marker.

## F18. Entity hooks vs Event Subscriber для инвалидации

**Q (RU):** Разница и когда что?

**Q (EN):** Entity hooks vs event subscriber for invalidation?

**RU:** Hooks — простой Drupal-специфичный способ реагировать на конкретный entity type. Subscriber — объектный, удобен как отдельный класс и для нескольких событий.

**EN:** Hooks are simple and Drupal-specific; subscribers are OO, ideal for dedicated classes and multiple events.

## F19. Task 5.3 — что продублировано

**Q (RU):** Что именно и что НЕ продублировано?

**Q (EN):** What was duplicated and what wasn't?

**RU:** Дублировалось **форматирование времени** (округление, суффикс, строка). Расчёт часов — нет: всё берёт числа из `TaskStatService`.

**EN:** Duplicated: **time formatting** (rounding, suffix, string). Calculation wasn't — all use `TaskStatService`.

## F20. `DurationFormatter` не считает

**Q (RU):** Почему не обращается к `TaskStatService`?

**Q (EN):** Why shouldn't `DurationFormatter` call `TaskStatService`?

**RU:** Service = _calculation_, formatter = _formatting_ готовых чисел. Разделение ответственности сохраняется.

**EN:** Service = calculation; formatter = formatting ready numbers — separation of responsibilities.

## F21. DRY vs Strategy/Plugin

**Q (RU):** Почему 5.3 — DRY, а не Plugin?

**Q (EN):** Why is 5.3 DRY rather than Strategy/Plugin?

**RU:** Мы объединяем **одну и ту же** логику в одном месте, а не создаём взаимозаменяемые алгоритмы. Plugin/Strategy нужен для выбора между реализациями (как `ReportGenerator`).

**EN:** We consolidate one existing logic, not interchangeable algorithms; Plugin/Strategy is for choosing among implementations.

## F22. `formatSummary()` вызывает `format()`

**Q (RU):** Зачем и почему явно в задании?

**Q (EN):** Why must `formatSummary()` reuse `format()`?

**RU:** Правило форматирования должно жить **в одном месте**, без повторения округления и суффиксов.

**EN:** The formatting rule must exist in **one place**.

## F23. Проверка рефакторинга по коду

**Q (RU):** Какие признаки?

**Q (EN):** What code-level signs verify the refactoring?

**RU:** `DurationFormatter` — сервис; `format()` содержит round+suffix; `formatSummary()` вызывает `format()`; Field Formatter и preprocess используют сервис; у потребителей нет своих `round()`/суффиксов.

**EN:** Service registered; `format()` holds the logic; `formatSummary()` calls it; consumers use it and have no own rounding/suffix code.

## F24. Неизменное поведение и проверка без «на глаз»

**Q (RU):** Почему и как?

**Q (EN):** Why unchanged behavior and how to check objectively?

**RU:** Это refactoring, не изменение функциональности. Сравнить возвращаемые строки на одинаковых входах (2.5, 1.0, 1.5) и перезапустить тесты Field Formatter и Project Statistics.

**EN:** Refactoring ≠ functional change; compare returned strings for identical inputs and rerun existing tests.

---

# G. Hooks, Events, Migrate API / Hooks, Events, Migrate API

> [!info] За что отвечает блок / What this block covers
> 
> **RU:** Как расширять Drupal (hooks vs Event Subscribers, custom event для TimeLog) и как импортировать исторический backlog через **Migrate API** (CSV → Task: source/process/destination, `entity_lookup`, идемпотентность, rollback).
> 
> **EN:** How to extend Drupal (hooks vs event subscribers, a custom TimeLog event) and how to import a historical backlog through the **Migrate API** (CSV → Task).

## G1. Hook

**Q (RU):** Что такое hook и как Drupal находит реализации?

**Q (EN):** What is a hook and how are implementations found?

**RU:** Точка расширения. Core через **Module Handler** находит функции во включённых модулях по имени и вызывает их.

**EN:** An extension point; core uses the **Module Handler** to find matching functions in enabled modules and call them.

## G2. `hook_ENTITY_TYPE_insert()` vs `hook_entity_insert()`

**Q (RU):** Разница?

**Q (EN):** Difference?

**RU:** Первый — для конкретного типа (`hook_node_insert()`), второй — для всех entity.

**EN:** The first targets one entity type; the second fires for all entity types.

## G3. Hook vs Event Subscriber

**Q (RU):** Фундаментальная разница?

**Q (EN):** Fundamental difference?

**RU:** Hook — процедурный вызов функции. Subscriber — объектный подход: dispatcher передаёт объект события.

**EN:** A hook is a procedural function call; a subscriber is OO and receives an event object from the dispatcher.

## G4. Порядок вызова

**Q (RU):** Как определяется?

**Q (EN):** How is execution order determined?

**RU:** Subscribers — по `priority` (больше = раньше). Hooks — обычно по порядку модулей (weight/имя), не по priority.

**EN:** Subscribers by `priority` (higher first); hooks by module order.

## G5. Event лучше hook для логирования TimeLog

**Q (RU):** Почему при многих subscribers?

**Q (EN):** Why prefer an event for TimeLog logging with many subscribers?

**RU:** Несколько независимых subscribers реагируют на одно событие без изменения кода создания TimeLog.

**EN:** Independent subscribers react without changing the creation code.

## G6. Hook + event одновременно в production

**Q (RU):** Почему нельзя и как проверить?

**Q (EN):** Why not both hook and event in production?

**RU:** TimeLog может обработаться дважды. Создать один TimeLog и убедиться, что в логе одна запись.

**EN:** One TimeLog could be processed twice; verify a single log entry per creation.

## G7. DI для dispatch custom event

**Q (RU):** Что внедрить?

**Q (EN):** What to inject to dispatch a custom event?

**RU:** `EventDispatcherInterface` (сервис `event_dispatcher`) — явная зависимость, тестируемо, в духе DI.

**EN:** `EventDispatcherInterface` — explicit, testable, DI-style (not `\Drupal::service()`).

## G8. Custom Event class

**Q (RU):** Что содержит и зачем отдельный класс?

**Q (EN):** What goes in a custom event and why a separate class?

**RU:** Созданный TimeLog и нужные данные. Отдельный класс — собственный тип события и чёткий контракт для subscribers.

**EN:** The created TimeLog and required data; a dedicated class gives a distinct event type and clear contract.

## G9. Регистрация Event Subscriber

**Q (RU):** Что нужно?

**Q (EN):** How is a subscriber registered?

**RU:** Сервис в `services.yml` с tag `event_subscriber`; класс реализует `EventSubscriberInterface::getSubscribedEvents()`.

**EN:** A service in `services.yml` tagged `event_subscriber`, implementing `getSubscribedEvents()`.

```yaml
drupaljira.timelog_subscriber:
  class: Drupal\drupaljira\EventSubscriber\TimeLogSubscriber
  tags: [{ name: event_subscriber }]
```

## G10. Migrate API vs PHP-скрипт с `Node::create()`

**Q (RU):** Зачем Migrate API?

**Q (EN):** Why Migrate API over a `Node::create()` script?

**RU:** source/process/destination, tracking (map), повторный запуск и rollback — импорт воспроизводим и управляем.

**EN:** Source/process/destination, ID tracking, re-runs and rollback — reproducible and manageable.

## G11. Source / Process / Destination

**Q (RU):** Что значат?

**Q (EN):** What do they mean?

**RU:** **Source** — откуда данные. **Process** — как преобразуем. **Destination** — куда сохраняем.

**EN:** Source — where from; process — how transformed; destination — where stored.

## G12. Declarative YAML vs custom PHP class

**Q (RU):** Почему YAML?

**Q (EN):** Why YAML instead of a custom class?

**RU:** Проще читать, поддерживать, воспроизводить. Класс — только для сложной/нестандартной логики.

**EN:** Easier to read, maintain, reproduce; custom classes only for complex logic.

## G13. `entity_lookup` — связь Task с Project по имени

**Q (RU):** Как работает и почему лучше hard-coded ID?

**Q (EN):** How does lookup link a task to a project by name?

**RU:** Ищет `project` node с `title` = `project_name` и возвращает ID. Не зависит от конкретного NID.

**EN:** Finds a `project` node whose title matches and returns its ID; no dependency on specific NIDs.

```yaml
process:
  field_project:
    plugin: entity_lookup
    source: project_name
    entity_type: node
    bundle_key: type
    bundle: project
    value_key: title
  field_status:
    plugin: static_map
    source: status
    map: { open: backlog, working: in_progress, completed: done }
```

## G14. Повторный `migrate:import` без `--update`

**Q (RU):** Что происходит?

**Q (EN):** What happens on a re-run without `--update`?

**RU:** Импортированные source IDs записаны в **migration map**, дубликаты не создаются.

**EN:** Already-imported source IDs are in the migration map → no duplicates.

## G15. `migrate:rollback`

**Q (RU):** Что делает и почему не удаляет другие tasks?

**Q (EN):** What does rollback do and why isn't it destructive to other tasks?

**RU:** Удаляет entities, созданные именно этой migration (по её map). Остальные не затрагиваются.

**EN:** Removes only entities created by that migration (via its map).

## G16. Нормализация статуса в process

**Q (RU):** Почему там?

**Q (EN):** Why normalize status in the process pipeline?

**RU:** Source использует иные значения; `static_map` приводит их к валидным Drupal-значениям до сохранения.

**EN:** The source uses different values; `static_map` converts them before the entity is saved.

## G17. Idempotency vs rollback

**Q (RU):** Разница и зачем оба?

**Q (EN):** Idempotency vs rollback — why both?

**RU:** Idempotency — повтор импорта без дублей. Rollback — откат результатов. Для исторического backlog нужны оба.

**EN:** Idempotency allows safe re-runs; rollback undoes results; both are needed.

## G18. `migrate_plus` и `migrate_source_csv`

**Q (RU):** Роли относительно core?

**Q (EN):** Roles of `migrate_plus` / `migrate_source_csv` vs core?

**RU:** Core — базовый Migrate API. `migrate_plus` расширяет плагины/конфигурацию (migration groups, config entities); `migrate_source_csv` добавляет CSV source plugin.

**EN:** Core = base API; `migrate_plus` extends plugins/config; `migrate_source_csv` adds the CSV source.

## G19. Проверка через Drush

**Q (RU):** Почему YAML недостаточно?

**Q (EN):** Why check via Drush and not just trust the YAML?

**RU:** YAML не доказывает, что Drupal обнаружил migration; `drush migrate:status` подтверждает регистрацию.

**EN:** YAML alone doesn't prove discovery; `drush migrate:status` confirms registration.

---

# H. Workflow, Update Hooks, Access / Workflow, Update Hooks, Access

> [!info] За что отвечает блок / What this block covers
> 
> **RU:** Content Moderation + Workflows для статусов Task, тип проекта (kanban/scrum), миграция существующих данных через `hook_update_N()`, условный UI и **доступ к Sprints** через route Access API.
> 
> **EN:** Content Moderation + Workflows for Task statuses, project type (kanban/scrum), migrating existing data via `hook_update_N()`, conditional UI and **Sprints access** via the route Access API.

## H1. States vs transitions

**Q (RU):** Чем отличаются и почему это важнее одного текстового поля?

**Q (EN):** States vs transitions — why better than one status text field?

**RU:** **State** — текущее состояние (`review`). **Transition** — разрешённое действие между состояниями (`approve: review → done`). Drupal управляет допустимыми переходами и правами.

**EN:** A state is the current condition; a transition is an allowed move (`approve: review → done`) — Drupal enforces valid transitions and permissions.

## H2. List field vs `moderation_state`

**Q (RU):** Фундаментальная разница в защите от неверного значения?

**Q (EN):** List field vs `moderation_state` — what prevents invalid values?

**RU:** List проверяет лишь, что значение в списке. `moderation_state` связан с Workflow — Drupal проверяет, **разрешён ли переход** из текущего состояния.

**EN:** List only validates membership; `moderation_state` also validates that the **transition is allowed**.

## H3. Права на transitions

**Q (RU):** Где хранятся и проверяются?

**Q (EN):** Where are transition permissions stored/checked?

**RU:** Переходы — в **конфигурации Workflow**; доступ — через Access API и permissions Content Moderation. Лучше проверки роли в коде: не привязано к роли, переиспользуется формами/роутами.

**EN:** Transitions live in Workflow config; access is checked via Access API/permissions — better than hard-coded role checks.

## H4. Core Content Moderation vs `drupal/state_machine`

**Q (RU):** Почему core?

**Q (EN):** Why core Content Moderation instead of `state_machine`?

**RU:** Задача требует **core** Content Moderation и интеграции с revisionable content; `state_machine` — валидная contrib-альтернатива, но вне требуемой архитектуры.

**EN:** The task requires core integration with revisionable content; `state_machine` is a valid contrib alternative.

## H5. Task должен быть revisionable

**Q (RU):** Почему до подключения moderation?

**Q (EN):** Why must `task` be revisionable first?

**RU:** Moderation работает с ревизиями: разные версии — в разных состояниях. Без revisions корректно хранить moderated versions нельзя.

**EN:** Moderation operates on revisions; without them it can't store moderated versions.

## H6. Все 4 состояния published

**Q (RU):** Почему backlog/in_progress/review/done — published/default?

**Q (EN):** Why are all four states published/default revision?

**RU:** Они описывают **рабочий процесс**, не публичность: Task должна быть доступна на всех этапах. Это не схема `draft → published`.

**EN:** They model workflow, not visibility; a Task stays available at all stages.

## H7. `field_project_type` — List (kanban/scrum)

**Q (RU):** Почему не Boolean?

**Q (EN):** Why List (text) and not Boolean?

**RU:** Дискретный тип, который может расширяться (`waterfall`); явные machine values понятны коду.

**EN:** A discrete, extensible type with explicit machine values.

## H8. Почему не taxonomy/entity reference

**Q (RU):** Почему запрещено?

**Q (EN):** Why forbid taxonomy/entity reference here?

**RU:** `kanban`/`scrum` — фиксированные технические варианты поведения, не сущности/категории редакторов; reference — лишняя зависимость.

**EN:** They're fixed behavior types, not editor-managed entities; a reference adds needless dependency.

## H9. Default `kanban`

**Q (RU):** Почему не пусто?

**Q (EN):** Why default `kanban`?

**RU:** Система уже на Kanban — сохраняется **обратная совместимость**; новые проекты получают валидный тип автоматически.

**EN:** Preserves backward-compatible behavior; new projects get a valid type automatically.

## H10. Required field и существующие nodes

**Q (RU):** Почему не ломает?

**Q (EN):** Why doesn't adding a required field break existing nodes?

**RU:** `required` применяется при создании/редактировании формы; старые entities значение не получают автоматически — нужна отдельная миграция/update hook.

**EN:** `required` applies on create/edit; old entities don't get values automatically → need an update hook.

## H11. `hook_update_N()` vs `hook_post_update_NAME()`

**Q (RU):** Разница и почему здесь `hook_update_N()`?

**Q (EN):** Difference and why `hook_update_N()` here?

**RU:** `hook_update_N()` — версионируемое обновление БД с числом (последовательные schema/data updates). `post_update` — операции после numbered updates (обычно когда нужен полностью работающий API, кеш/конфиг). Мы делаем конкретную версионируемую миграцию данных.

**EN:** `hook_update_N()` — numbered, ordered schema/data update; `post_update` runs afterwards. We do a specific versioned data migration.

## H12. Почему не `drush scr`

**Q (RU):** Почему не одноразовый скрипт?

**Q (EN):** Why not a one-off `drush scr` script?

**RU:** Update hook — часть системы с **учётом выполненных обновлений**: Drupal знает, что уже выполнено, и сам применит на другом окружении. `scr` без tracking можно пропустить или запустить дважды.

**EN:** Update hooks are tracked by Drupal and applied per environment; scripts can be skipped or rerun.

## H13. `$sandbox` и batch

**Q (RU):** Зачем?

**Q (EN):** Why `$sandbox` and batching?

**RU:** `$sandbox` хранит прогресс между вызовами; batching снижает память и риск timeout. Нельзя рассчитывать, что всё безопасно загрузится одним `loadMultiple()`.

**EN:** `$sandbox` persists progress between runs; batching reduces memory/timeouts.

```php
function drupaljira_update_10002(&$sandbox) {
  if (!isset($sandbox['ids'])) {
    $sandbox['ids'] = \Drupal::entityQuery('node')->accessCheck(FALSE)->condition('type','task')->execute();
    $sandbox['total'] = count($sandbox['ids']);
  }
  foreach (array_splice($sandbox['ids'], 0, 50) as $nid) { /* migrate via Entity API */ }
  $sandbox['#finished'] = $sandbox['total'] ? 1 - count($sandbox['ids']) / $sandbox['total'] : 1;
}
```

## H14. Entity API vs прямой SQL UPDATE

**Q (RU):** Почему Entity API?

**Q (EN):** Entity API vs direct SQL UPDATE?

**RU:** Учитывает field storage, ревизии, hooks/events, валидацию, инвалидацию кеша; SQL обходит логику и ведёт к несогласованности.

**EN:** Handles storage, revisions, hooks/events, validation, cache invalidation; SQL bypasses it.

## H15. Идемпотентность update

**Q (RU):** Как обеспечивается?

**Q (EN):** How is update idempotency ensured?

**RU:** Обрабатываются только нужные entities с проверкой текущих данных; Drupal фиксирует выполненный update — `drush updatedb -y` повторно не запустит.

**EN:** Only entities needing change are processed; Drupal records completed updates.

## H16. Миграция по предыдущему `field_status`

**Q (RU):** Почему не всем `backlog`?

**Q (EN):** Why map from the previous status, not set everything to `backlog`?

**RU:** Нужно сохранить бизнес-логику: `working` → `in_progress`, `completed` → `done`. Иначе потеряем исторические состояния.

**EN:** Preserve business meaning (`working` → `in_progress`, `completed` → `done`).

## H17. `/admin/reports/status`

**Q (RU):** Что покажет, если не запустили `updatedb`?

**Q (EN):** What does the status report show if `updatedb` wasn't run?

**RU:** **Pending database updates** — код ждёт новой структуры/данных, а БД ещё на старой версии.

**EN:** Pending database updates — code expects newer DB state.

## H18. Доступ к Sprints через route Access API

**Q (RU):** Почему не проверка в контроллере?

**Q (EN):** Why route-level access rather than a controller check?

**RU:** Доступ проверяется **до** контроллера; access callback — стандартный Access API и защищает route при любом способе вызова.

**EN:** Access is enforced before the controller via the standard Access API.

## H19. Одно условие для tab и route

**Q (RU):** Почему?

**Q (EN):** Why use the same condition for the tab and the route?

**RU:** Чтобы UI и security не расходились: скрытая tab при открытом route позволяет зайти по прямому URL.

**EN:** So UI and access logic don't diverge — a hidden tab with an open route is bypassable.

## H20. CSS/JS `display:none` — запрещено

**Q (RU):** Почему?

**Q (EN):** Why forbid hiding the link with CSS/JS?

**RU:** Это presentation, не access control; URL можно открыть напрямую. Сервер решает доступ независимо от UI.

**EN:** Presentation ≠ access control; the server must enforce access.

## H21. Тип проекта + node access

**Q (RU):** Почему оба условия?

**Q (EN):** Why check both project type and node access?

**RU:** Нужно и право просматривать Project, и чтобы он был Scrum — **defense in depth**.

**EN:** Both must hold — defense in depth.

## H22. Stub-route для Scrum заранее

**Q (RU):** Зачем?

**Q (EN):** Why prepare a tab/stub route in advance?

**RU:** Создаёт **extension point** и архитектурную границу; следующий блок добавит sprint-логику без переделки Project UI/routing.

**EN:** Creates an extension point so the next block adds sprint logic without redesign.

## H23. Условный UI проекта

**Q (RU):** Почему не один универсальный?

**Q (EN):** Why not one universal Project UI?

**RU:** Kanban и Scrum имеют разные workflows и элементы (Sprints только для Scrum). Интерфейс показывает релевантное типу.

**EN:** Different workflows and features per type; show only relevant functionality.

## H24. Смена Kanban → Scrum

**Q (RU):** Почему Sprints появляются сразу?

**Q (EN):** Why does Sprints appear immediately after switching type?

**RU:** Доступность определяется **динамически по текущему значению поля**, не хардкодом. Rebuild routing нужен только при изменении самой конфигурации route.

**EN:** Availability is computed dynamically from the field value; routing rebuild is needed only when route config changes.

---

# I. Figma MCP, фронтенд и AI-код / Figma MCP, Frontend & AI code

> [!info] За что отвечает блок / What this block covers
> 
> **RU:** Как переносить дизайн из Figma в Drupal-тему без потери Drupal-функциональности (View остаётся источником данных, Twig только презентация), CSS custom properties, accessibility, cache и ревью AI-кода.
> 
> **EN:** How to port Figma designs into a Drupal theme without breaking Drupal behavior, plus CSS variables, accessibility, caching and AI-code review.

## I1. Figma MCP vs скриншот/ссылка

**Q (RU):** Что даёт AI-агенту?

**Q (EN):** What does Figma MCP give an agent that a screenshot/link doesn't?

**RU:** Структурированный доступ: компоненты, стили, размеры, цвета, layout. Скриншот — только внешний вид.

**EN:** Structured access to components, styles, dimensions, colors, layout; a screenshot is visual only.

## I2. Credentials MCP вне репозитория

**Q (RU):** Почему?

**Q (EN):** Why keep MCP credentials/config outside the repo?

**RU:** Токены и секреты нельзя в Git — попадут в историю и дадут доступ к Figma.

**EN:** Tokens/secrets in Git history grant others Figma access.

## I3. Figma MCP — dev dependency

**Q (RU):** Почему не runtime?

**Q (EN):** Why a development, not runtime, dependency?

**RU:** Нужен разработчику/AI при работе; production-Drupal не должен ходить в Figma для отображения сайта.

**EN:** Needed during development; production never needs Figma to render.

## I4. Компонент Figma → View/template/render array

**Q (RU):** Как сопоставить?

**Q (EN):** How to map a Figma component to Drupal?

**RU:** Определить, какие данные приходят из View; HTML-структуру реализовать в Twig/render array; заполнить реальными View fields.

**EN:** Identify View-provided data, implement markup in Twig/render array, fill with real fields.

## I5. `/board/%` View остаётся источником данных

**Q (RU):** Почему не копировать mock из Figma?

**Q (EN):** Why must the View remain the data source?

**RU:** View даёт реальные Tasks, фильтры и access. Mock в Twig делает дизайн статичным.

**EN:** The View supplies real tasks, filters and access; mock data makes it static.

## I6. View-specific template vs широкий override

**Q (RU):** Когда?

**Q (EN):** When use a View-specific template over page/node overrides?

**RU:** Когда изменение относится к конкретному View/display; общий page/node затронет другие страницы.

**EN:** When the change is specific to one View/display; broad overrides affect other pages.

## I7. CSS custom properties

**Q (RU):** Зачем для значений из Figma?

**Q (EN):** Why CSS custom properties for Figma values?

**RU:** Цвета, spacing в одном месте → меньше дублирования, проще менять design system.

**EN:** Shared values defined once — less duplication, easier design-system changes.

## I8. Что ломает удаление `attributes`/`children`/contextual placeholders

**Q (RU):** Что именно?

**Q (EN):** What breaks if you remove template attributes, children or contextual-link placeholders?

**RU:** Accessibility, field rendering, contextual links, cache metadata и др. Не удалять, не понимая назначения.

**EN:** Accessibility, field rendering, contextual links, cache metadata.

## I9. Entity loading/workflow/access не в Twig

**Q (RU):** Почему?

**Q (EN):** Why not do entity loading, workflow decisions or access checks in Twig?

**RU:** Twig — presentation; логика в PHP/сервисах — безопасно, тестируемо, поддерживаемо.

**EN:** Twig is presentation; logic belongs in PHP/services.

## I10. JS во View: behaviors и `once()`

**Q (RU):** Зачем?

**Q (EN):** Why Drupal behaviors and `once()`?

**RU:** Behaviors выполняются при загрузке и после AJAX; `once()` не даёт навесить обработчик на элемент повторно.

**EN:** Behaviors run on load and after AJAX; `once()` prevents duplicate handlers.

```js
Drupal.behaviors.taskBoard = {
  attach(context) {
    once('task-board', '.task-card', context).forEach((el) => { /* init drag&drop */ });
  }
};
```

## I11. Доказать live Drupal rendering

**Q (RU):** Как?

**Q (EN):** How prove the redesigned board still uses live rendering?

**RU:** Изменить реальные данные Task (название/статус/assignee), обновить: UI показывает новое без правки Twig/mock.

**EN:** Change real Task data and reload; the UI updates without touching Twig.

## I12. Скрытие ≠ access control

**Q (RU):** Почему?

**Q (EN):** Why isn't visually hiding an action access control?

**RU:** CSS/JS скрывают элемент, не запрещают запрос. Проверка — на сервере.

**EN:** They hide UI but don't block requests; check on the server.

## I13. Cache contexts/tags для board

**Q (RU):** Какие важны?

**Q (EN):** Which contexts/tags matter for the board?

**RU:** Contexts: `user.permissions`, параметры запроса (`url.query_args`); tags: конкретный project и Task entities.

**EN:** Contexts like `user.permissions`, query args; tags for the project and Task entities.

## I14. Fidelity vs semantic HTML/a11y

**Q (RU):** Когда уступать?

**Q (EN):** When should visual fidelity yield to semantics/a11y?

**RU:** Если копия ухудшает доступность — приоритет семантике: интерактивное = настоящая `<button>`, даже если в Figma `div`.

**EN:** Prefer semantics/a11y — an interactive element should be a real `<button>`.

## I15. Ручной review AI-кода

**Q (RU):** Почему, если проходит coding standards?

**Q (EN):** Why manually review AI code that passes coding standards?

**RU:** Standards проверяют стиль и часть техпроблем, но не архитектуру, security, access control, соответствие требованиям Drupal.

**EN:** Standards don't guarantee architecture, security or access correctness.

---

# J. Playwright E2E-тестирование / Playwright E2E Testing

> [!info] За что отвечает блок / What this block covers
> 
> **RU:** Что тестировать E2E и что на Unit/Kernel/Functional, стабильные fixtures, изоляция, locators, синхронизация, security-проверки (403), секреты, flaky/retries, AI-сгенерированные тесты, a11y, coverage matrix.
> 
> **EN:** What to test E2E vs Unit/Kernel/Functional, stable fixtures, isolation, locators, synchronization, security checks, secrets, flakiness, AI-generated tests, a11y and the coverage matrix.

## J1. Playwright vs Drupal-тесты

**Q (RU):** Какие дефекты где?

**Q (EN):** Which defects suit Playwright vs Unit/Kernel/Functional?

**RU:** Playwright — пользовательские сценарии (навигация, формы, кнопки, AJAX, permissions, отображение, сохранение после reload). Unit — изолированная PHP-логика; Kernel — Drupal API, entities, БД; Functional — HTTP/application-уровень.

**EN:** Playwright — real user journeys; Unit — isolated logic; Kernel — APIs/entities/DB; Functional — HTTP-level behavior.

## J2. «Покрыть весь проект» — risk-based

**Q (RU):** Почему?

**Q (EN):** Why is "cover the whole project" risk-based?

**RU:** E2E медленнее и дороже; выбираем сценарии с наибольшим бизнес-риском (проект, Task, статус, permissions); остальное — на подходящем уровне.

**EN:** E2E is slow/expensive; focus on highest-risk journeys.

## J3. Стабильные fixtures

**Q (RU):** Почему важнее количества тестов?

**Q (EN):** Why do stable fixtures matter more than test count?

**RU:** Тест должен стартовать из предсказуемого состояния; нестабильные fixtures дают случайные падения и сложный debugging.

**EN:** Predictable starting state; unstable fixtures cause random failures.

## J4. Секреты в Git

**Q (RU):** Риски storage state/cookies/credentials/traces/screenshots?

**Q (EN):** Risks of committing storage state, cookies, credentials, traces, screenshots?

**RU:** Активные сессии, токены, пароли, персональные данные, внутренние URL. Не коммитить без очистки.

**EN:** Live sessions, tokens, passwords, personal data, internal URLs.

## J5. Независимость тестов

**Q (RU):** Почему не зависеть от порядка?

**Q (EN):** Why must tests be order-independent?

**RU:** Playwright запускает параллельно и меняет порядок; зависимость даёт flaky. Тест сам создаёт данные или использует изолированный fixture.

**EN:** Parallel/reordered execution; each test creates its own data.

## J6. `waitForTimeout()` и auto-waiting

**Q (RU):** Почему источник flaky?

**Q (EN):** Why is `waitForTimeout()` flaky and what's better?

**RU:** Фиксированная пауза не знает, когда операция завершена. Лучше assertions + auto-waiting (`expect(locator).toBeVisible()`, `toHaveText()`).

**EN:** Fixed sleeps ignore real state; use web-first assertions and auto-waiting.

## J7. Accessible locators vs CSS/XPath

**Q (RU):** Почему надёжнее?

**Q (EN):** Why roles/labels over CSS/XPath?

**RU:** Основаны на семантике (button, textbox), меньше зависят от структуры/классов; тестируют глазами пользователя и поддерживают a11y.

**EN:** Semantic, resilient to redesign, user-centric.

```ts
await page.getByRole('button', { name: 'Save' }).click();
await expect(page.getByText('Task created')).toBeVisible();
```

## J8. `data-testid`

**Q (RU):** Когда оправдан?

**Q (EN):** When is `data-testid` justified?

**RU:** Когда нет стабильного семантического идентификатора (сложный компонент). Если кнопка находится `getByRole`, `testid` ради удобства может скрыть проблему a11y/разметки.

**EN:** When no stable semantic locator exists; otherwise it can hide a11y/markup problems.

## J9. Проверка после reload

**Q (RU):** Почему?

**Q (EN):** Why verify after reload?

**RU:** Изменение DOM не гарантирует сохранения в Drupal; reload заново берёт данные из backend.

**EN:** DOM change ≠ persisted; reload re-reads from backend.

## J10. Скрытая Approve ≠ защита

**Q (RU):** Почему недостаточно?

**Q (EN):** Why isn't a hidden Approve button proof of access control?

**RU:** Это UI-защита; можно вызвать URL/endpoint вручную. Нужен тест server-side: unauthorized → **HTTP 403**, состояние Task не меняется.

**EN:** Test the server: expect 403 and unchanged state.

## J11. Отличить 403 от необработанного exception

**Q (RU):** Как?

**Q (EN):** How distinguish expected 403 from an unhandled error page?

**RU:** Проверять **фактический HTTP status code**, не только текст «Access denied»; 500/200 — провал. Дополнительно body и логи.

**EN:** Assert the actual status code, not page text.

## J12. Реальные взаимодействия (Media Library, moderation, AJAX)

**Q (RU):** Почему не заменять API?

**Q (EN):** Why test via real interactions rather than API setup?

**RU:** Зависят от backend+JS+AJAX+forms+permissions; API проверит только состояние backend, не UI.

**EN:** API setup verifies backend only, not the integrated UI.

## J13. Fixtures через API/Drush

**Q (RU):** Когда допустимо?

**Q (EN):** When is API/Drush/DB setup appropriate?

**RU:** Для данных вне проверяемого сценария. Если тестируем создание Task — создавать через UI.

**EN:** For non-tested prerequisites; the tested journey must go through the UI.

## J14. Данные для LLM-диагностики

**Q (RU):** Что давать и что удалить?

**Q (EN):** What to give an LLM and what to remove?

**RU:** Код теста, error message, stack trace, locator, URL, шаг, trace/screenshot, ожидаемое/фактическое. Удалить пароли, токены, cookies, auth state, API keys, персональные данные.

**EN:** Test code, error, stack trace, locator, URL, step, trace; strip secrets and PII.

## J15. Зелёный, но бесполезный AI-тест

**Q (RU):** Почему возможен?

**Q (EN):** Why can an AI test pass yet assert nothing meaningful?

**RU:** Слабый assertion (заголовок существует) вместо проверки создания и сохранения Task.

**EN:** Weak assertions give false confidence.

## J16. Типичные ошибки AI-тестов

**Q (RU):** Какие?

**Q (EN):** Common AI mistakes?

**RU:** Хрупкие CSS/XPath, `waitForTimeout()`, общие fixtures, фиксированные имена, слабые assertions, нет cleanup, зависимости между тестами, проверка только UI без persistence.

**EN:** Fragile selectors, sleeps, shared fixtures, fixed names, weak assertions, no cleanup, UI-only checks.

## J17. AI-fix: проверять причину

**Q (RU):** Почему не принимать из-за «зелёного»?

**Q (EN):** Why validate an AI fix beyond a green test?

**RU:** AI мог ослабить assertion или увеличить timeout. Смотреть trace, воспроизвести failure, убедиться, что устранён root cause.

**EN:** Check the trace and root cause, not just green.

## J18. Traces/screenshots/video

**Q (RU):** Зачем только при failure и когда video?

**Q (EN):** Why keep artifacts on failure only and when add video?

**RU:** Меньше данных в CI. Trace — действия, network, DOM; screenshot — визуал. Video — для последовательностей, анимаций, timing, сложного UI.

**EN:** Less CI noise; video helps with sequences/animations/timing.

## J19. Retries

**Q (RU):** Польза и опасность?

**Q (EN):** Retries — useful vs dangerous?

**RU:** Показывают нестабильность (race/timing) — диагностика. Но если оставить как есть, CI зелёный, а проблема жива.

**EN:** Diagnostic signal, but can mask flakiness.

## J20. Workers и fixtures

**Q (RU):** Как влияют на naming/cleanup?

**Q (EN):** How do parallel workers affect fixture naming/cleanup?

**RU:** Уникальные имена/ID (worker/test id) и изолированный cleanup, не удаляющий чужие данные.

**EN:** Unique names and isolated cleanup.

## J21. Smoke vs regression

**Q (RU):** Что входит?

**Q (EN):** Smoke vs regression suite?

**RU:** Smoke — немного критичных быстрых (login, открыть Project, создать/изменить Task). Regression — шире: permissions, moderation, Media Library, edge cases, a11y, viewports.

**EN:** Smoke = few critical fast checks; regression = broad.

## J22. Accessibility: авто + keyboard + человек

**Q (RU):** Почему комбинировать?

**Q (EN):** Why combine automated a11y scans, keyboard tests and human review?

**RU:** Сканер находит техпроблемы (label, contrast), но не контекст; keyboard — focus/tab order; человек — реальный UX.

**EN:** Each catches what the others miss.

## J23. Viewports vs visual regression

**Q (RU):** Разница?

**Q (EN):** Viewport testing vs visual regression?

**RU:** Viewports — функциональность и адаптация на разных размерах. Visual regression — сравнение скриншотов с baseline (отступы, размеры).

**EN:** Viewports test function/adaptation; visual regression diffs screenshots.

## J24. Coverage matrix и gaps

**Q (RU):** Почему явно указывать manual checks и пробелы?

**Q (EN):** Why list manual checks and known gaps?

**RU:** Автотесты не дают полного покрытия; матрица показывает автоматическое, ручное и gaps — реалистичная картина.

**EN:** Honest view of coverage; helps plan next work.

---

# K. Шпаргалка / Cheat Sheet

## K1. Таблицы «X vs Y» / Comparison tables

|Тема / Topic|A|B|Ключевое отличие / Key difference|
|---|---|---|---|
|Composer|`composer.json`|`composer.lock`|желаемое vs точно установленное|
|Xdebug|`develop`/`coverage`/`profile`|`debug`|только `debug` даёт breakpoints|
|Статика кода|PHPCS|PHPStan|стиль vs корректность|
|Данные|Config|Content|структура (`cex/cim`) vs данные|
|Поля|`field.storage`|`field.field`|БД-уровень vs привязка к bundle|
|Media|File|Media|запись файла vs богатая сущность|
|Ссылки|Entity reference|Entity reference revisions|текущая vs конкретная ревизия|
|Сущности|Content Entity|Config Entity|данные vs конфигурация|
|Запросы|EntityQuery|SQL|Entity API vs схема таблиц|
|Формы|`#states`|`validateForm()`|клиент (UI) vs сервер (защита)|
|Формы|`FormBase`|`ConfirmFormBase`|ввод данных vs подтверждение|
|DI|Service (`services.yml`)|Form/Controller (`create()`)|контейнер vs `ContainerInjectionInterface`|
|DI|Plugin|Form|`ContainerFactoryPluginInterface` (+config) vs `ContainerInjectionInterface`|
|Зависимости|Service Locator|DI|скрыто vs явно|
|Плагины|Attributes|Annotations|современный PHP vs устаревающие docblocks|
|Плагины|Widget|Formatter|ввод vs отображение|
|Render|`#markup`|`#theme`|простой текст vs тематизированный компонент|
|Кеш|Tags|Contexts / max-age|что инвалидировать / варианты / время|
|Расширение|Hook|Event Subscriber|процедурный vs объектный, priority|
|Update|`hook_update_N`|`hook_post_update_NAME`|номерной vs после номерных|
|Update|`hook_update_N`|`drush scr`|tracked vs не tracked|
|Доступ|route `_permission`/access|проверка в контроллере / CSS-hide|до контроллера; UI ≠ защита|
|Тесты|E2E (Playwright)|Unit/Kernel/Functional|сценарии пользователя vs логика/API|
|Playwright|`waitForTimeout`|assertions + auto-wait|фикс. пауза vs реальное состояние|
|Playwright|Smoke|Regression|быстро-критичное vs широкое|
|Playwright|Viewports|Visual regression|функционал vs скриншот-diff|

## K2. Мини-чеклисты / Mini checklists

> [!example] Новая кастомная функциональность в Drupal / Adding custom functionality
> 
> 1. Логика → **сервис** (`services.yml`, DI через конструктор).
> 2. Данные → **Entity API / EntityQuery**, не SQL.
> 3. Доступ → **route requirement / Access API**, не CSS.
> 4. Вывод → **render array + `hook_theme` + preprocess + Twig**.
> 5. Кеш → **tags + contexts + max-age**, точная инвалидация.
> 6. Структура → **config (cex/cim)**, данные → **update hook / migration**.
> 7. Качество → **PHPCS + PHPStan + GrumPHP + CI**; UI-сценарии → **Playwright**.

> [!example] Чек «кеш сломан?» / Cache debugging
> 
> - Данные изменились, а вывод старый → не хватает **tag** или нет инвалидации.
> - Разным пользователям один вывод → не хватает **context**.
> - Всё слишком медленно → `max-age: 0` / слишком широкий context.

> [!example] Чек «update hook» / Update hook `$sandbox` + batch → Entity API → проверка текущих значений (идемпотентность) → `drush updatedb -y` → `/admin/reports/status`.

## K3. Фразы для собеседования / Interview soundbites

- **RU:** «Конфигурацию мы переносим файлами и ревьюим в git; контент не трогаем.» — **EN:** "We sync config through files reviewed in git and never touch content."
- **RU:** «Скрытие в UI — не безопасность; доступ проверяется на сервере (route access, 403).» — **EN:** "UI hiding isn't security; access is enforced server-side."
- **RU:** «Зависимости передаём через конструктор — так код тестируем и честен.» — **EN:** "Dependencies via constructor injection keep code testable and honest."
- **RU:** «Логика в PHP/сервисах, Twig только показывает.» — **EN:** "Logic lives in PHP/services; Twig only presents."
- **RU:** «Кеш инвалидируем точечными tags, а не `drush cr`.» — **EN:** "We invalidate with targeted cache tags, not `drush cr`."
- **RU:** «Миграции и update hooks дают воспроизводимость на всех окружениях.» — **EN:** "Migrations and update hooks give reproducibility across environments."
- **RU:** «Зелёный тест не значит, что проблема решена — смотрим trace и причину.» — **EN:** "A green test doesn't mean the problem is fixed — we check traces and root cause."
- **RU:** «AI-код проходит те же ревью и тесты, что и человеческий.» — **EN:** "AI-generated code gets the same review and tests as human code."

## K4. Топ-вопросы для самопроверки / Self-check

1. Config vs Content и безопасный workflow `cex`/`cim`? → [[#B. Управление конфигурацией / Configuration Management]]
2. Как работает DI в сервисе, форме и плагине? → [[#E. Сервисы, DI, Plugin API / Services, DI, Plugin API]]
3. Tags/contexts/max-age на примере Project Statistics? → [[#F. Render API, Theme, Cache API / Render API, Theme, Cache API]]
4. Почему upcasting даёт 404 и где проверять доступ? → [[#D. Content Entity, Entity API, Routing, Form API / Content Entity, Entity API, Routing, Form API]]
5. Hook vs Event Subscriber; Migrate: idempotency vs rollback? → [[#G. Hooks, Events, Migrate API / Hooks, Events, Migrate API]]
6. `hook_update_N`: `$sandbox`, Entity API, идемпотентность? → [[#H. Workflow, Update Hooks, Access / Workflow, Update Hooks, Access]]
7. Как проверить 403 и persistence в Playwright? → [[#J. Playwright E2E-тестирование / Playwright E2E Testing]]