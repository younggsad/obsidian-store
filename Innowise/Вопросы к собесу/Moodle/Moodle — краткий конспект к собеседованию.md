---
custom-width: 80
---
---
## Оглавление

1. [[#1. Что такое Moodle]]
2. [[#2. Окружение, config.php, каталоги]]
3. [[#3. Запрос и глобальные объекты]]
4. [[#4. Курсы, cm и instance]]
5. [[#5. Контексты, роли, capabilities]]
6. [[#6. Плагины: типы, frankenstyle, структура, версии]]
7. [[#7. База данных]]
8. [[#8. Параметры и безопасность]]
9. [[#9. Формы, вывод, строки, настройки]]
10. [[#10. События и cron-задачи]]
11. [[#11. Web services и JavaScript]]
12. [[#12. Hooks, Files, Cache, Message, Privacy]]
13. [[#13. Модуль mod, блок block, другие плагины]]
14. [[#14. Тесты, отладка, стиль, обновление]]
15. [[#15. Интеграции, сессии, блокировки, инфраструктура]]
16. [[#16. Чужой код, совместимость, вклад в проект]]
17. [[#17. Темы и внешний вид]]
18. [[#18. Встроенные подсистемы Moodle]]
19. [[#19. PHP, SQL, Git, веб для собеседования]]
20. [[#20. Сценарные вопросы]]
21. [[#21. Как отвечать и что спросить]]
22. [[#22. Шпаргалка]]

---

## 1. Что такое Moodle

### Теория

- Moodle — **LMS** (система управления обучением): курсы, записи на курс (enrolments), тесты, задания, журнал оценок, компетенции. Не универсальная CMS.
- Открытый код, PHP, **GPL v3**. СУБД: MySQL/MariaDB, PostgreSQL, MS SQL Server (Oracle убран в 5.0).
- **Ядро не правим (no core hacking).** Всё расширяется плагинами, даже форум и тест.
- Установка = **код + база данных + каталог moodledata** (`$CFG->dataroot`). moodledata лежит вне веб-корня: файлы пользователей, кэши, временные файлы, сессии.
- Файлы хранятся по SHA-1 содержимого (**дедупликация**): на диске один файл, в таблице `{files}` на него много записей.
- Фронтенд: Mustache-шаблоны, JS как AMD/ES6-модули (сборка Grunt), темы на Bootstrap.
- Нет единого front controller: **каждая страница — отдельный PHP-файл** (`view.php`, `index.php`). Новый код: namespaces, автозагрузка, Hooks API, типизация. В 5.x добавлен роутер для новых точек входа.

### Вопросы и ответы

> [!question] Из чего состоит установка Moodle?
> 
> Из трёх частей: код (ядро и плагины), база данных и `moodledata`. При переносе нужны все три. **Подвох:** без `moodledata` сайт поднимется, но файлы, аватары и вложения будут битыми.

> [!question] Чем Moodle отличается от WordPress или Drupal?
> 
> - Предметная область: обучение (курсы, оценки, записи).
> - Свой слой БД `$DB` (не ORM), схема в `install.xml`, миграции в `upgrade.php`.
> - Нет единого фронт-контроллера.
> - Права на контекстах (система → категория → курс → активность → блок) и capabilities.
> 
> **Подвох:** не называй Moodle «устаревшим монолитом». Скажи: зрелая платформа с сильной обратной совместимостью, которая движется к типизированным API и Hooks.

> [!question] Почему к moodledata нельзя давать прямой веб-доступ?
> 
> Там приватные файлы (ответы на тесты, работы). Прямая отдача обходит проверку прав. Файлы отдаются только через PHP (`pluginfile.php`) после проверок.

> [!question] Что такое LTS и какую версию ставить на прод?
> 
> LTS — версия с долгой поддержкой безопасности, выходит раз в два года. На прод берут LTS или последнюю стабильную, если готовы обновляться дважды в год. **Подвох:** плагины и PHP должны подходить под версию (`$plugin->requires`).

---

## 2. Окружение, config.php, каталоги

### Теория

- Нужно: PHP (версия из раздела 1) с расширениями `intl`, `mbstring`, `xml`, `gd`, `zip`, `curl`, `sodium`, `opcache`; PostgreSQL или MariaDB; Node.js (версия из `.nvmrc`) для сборки JS.
- Для разработки: **moodle-docker** (`moodlehq/moodle-docker`), в нём же окружение для Behat.
- `config.php`: БД, `$CFG->wwwroot`, `dataroot`, `prefix`. **Последняя строка:** `require_once(__DIR__ . '/lib/setup.php');`
- Пути в `$CFG`: `wwwroot` (URL), `dirroot` (код на диске), `dataroot`, `tempdir`, `cachedir`, `localcachedir` (локальный кэш сервера). Пути строят от `$CFG->dirroot`.
- Режим разработки:

```php
$CFG->debug = (E_ALL | E_STRICT);
$CFG->debugdisplay = 1;
$CFG->cachejs = false;
$CFG->langstringcache = false;
$CFG->themedesignermode = true;
```

- CLI:

```bash
php admin/cli/install.php ...            # установка
php admin/cli/upgrade.php                # применить обновления и миграции
php admin/cli/purge_caches.php           # сбросить кэши
php admin/cli/cron.php                   # запустить cron
php admin/cli/maintenance.php --enable   # режим обслуживания
```

|Папка|Что внутри|
|:--|:--|
|`/admin`|админка, CLI, утилиты (`/admin/tool`)|
|`/auth`|способы входа (OAuth2, LDAP, SAML)|
|`/blocks`|блоки|
|`/course`|управление курсами, форматы (`/course/format`)|
|`/enrol`|способы записи на курс|
|`/lib`|библиотеки ядра (`setup.php`, DML/DDL, формы)|
|`/local`|локальные проектные плагины|
|`/mod`|активности и ресурсы|
|`/theme`|темы (Boost и дочерние)|

- **Moodle 5.0:** появилась папка `public/`. Веб-сервер смотрит в неё, а `config.php` и `vendor/` лежат выше, вне веб-корня. Внутри плагина ничего не меняется.

### Вопросы и ответы

> [!question] Почему `require_once(.../lib/setup.php)` должен быть последней строкой `config.php`?
> 
> `setup.php` запускает ядро: читает `$CFG`, подключает БД, стартует сессию, включает автозагрузку. Всё написанное после ядро уже не увидит.

> [!question] Что включают в config.php для разработки и почему нельзя на проде?
> 
> Показ всех ошибок, отключение кэшей JS, строк, шаблонов и тем. На проде это даёт падение скорости (всё компилируется на каждый запрос) и утечку путей и структуры БД через ошибки.

> [!question] Куда класть плагины типа `tool` и `format`?
> 
> `tool` → `/admin/tool/имя`, `format` → `/course/format/имя`. **Подвох:** папка не всегда равна типу из имени: `qtype` → `/question/type/`, `assignsubmission` → `/mod/assign/submission/`.

---

## 3. Запрос и глобальные объекты

### Теория

**Жизненный цикл страницы плагина:**

1. `require_once(config.php)` → `lib/setup.php` (БД, сессия, пользователь).
2. `require_login()` — пользователь вошёл.
3. `require_capability(...)` — право в нужном контексте.
4. Настройка `$PAGE`: `set_url`, `set_context`, `set_title`, `set_heading`.
5. Вывод: `$OUTPUT->header()` → содержимое → `$OUTPUT->footer()`.

**Глобальные объекты:**

|Объект|Что это|
|:--|:--|
|`$CFG`|настройки сайта|
|`$DB`|доступ к БД|
|`$USER`|текущий пользователь|
|`$PAGE`|состояние страницы (URL, контекст, заголовок, JS/CSS)|
|`$OUTPUT`|вывод элементов интерфейса|
|`$COURSE`|текущий курс|
|`$SESSION`|данные сессии|

- Варианты входа: `require_login()` (только вход), `require_login($course)` (вход + доступ к курсу), `require_login($course, false, $cm)` (+ видимость и ограничения активности).
- `$PAGE->set_pagelayout('standard')` выбирает макет (standard, admin, report, popup...).
- Страницы админки: `admin_externalpage_setup('имя')` вместо ручной настройки `$PAGE` (страница описана в `settings.php`).

Скелет страницы:

```php
<?php
require_once(__DIR__ . '/../../config.php');
require_login();
$context = context_system::instance();
require_capability('local/hello:view', $context);

$PAGE->set_url(new moodle_url('/local/hello/index.php'));
$PAGE->set_context($context);
$PAGE->set_title(get_string('pluginname', 'local_hello'));
$PAGE->set_heading(get_string('pluginname', 'local_hello'));

echo $OUTPUT->header();
echo $OUTPUT->heading(get_string('greeting', 'local_hello', fullname($USER)));
echo $OUTPUT->footer();
```

### Вопросы и ответы

> [!question] Как Moodle обрабатывает запрос?
> 
> Веб-сервер запускает конкретный PHP-скрипт → он подключает `config.php` → ядро поднимает БД, сессию, `$USER` → `require_login()` и проверка прав → настройка `$PAGE` → вывод через `$OUTPUT`.

> [!question] Почему глобальные объекты не любят в новом коде?
> 
> Они скрывают зависимости и мешают тестированию. В классах глобалы берут явно (`global $DB;` в методе), лучше передавать нужное параметрами или через внедрение зависимостей. **Подвох:** `require_login()` нужен на **каждой** странице, иначе она открыта гостю.

---

## 4. Курсы, cm и instance

### Теория

- Структура: **курс → секции → активности** (`course_modules`).
- Активность = две части:
    - **cm (course module)** — размещение активности в курсе, таблица `{course_modules}`, id = **cmid**;
    - **instance** — сама активность в таблице плагина (например `{forum}`), на неё ссылается `course_modules.instance`.
- В URL обычно **cmid** (`view.php?id=cmid`):

```php
$cm = get_coursemodule_from_id('forum', $id, 0, false, MUST_EXIST);
$course = get_course($cm->course);
require_login($course, false, $cm);
$context = context_module::instance($cm->id);
```

- `get_fast_modinfo($course)` — кэшированная структура курса без лишних запросов. `->get_cm($cmid)` даёт `cm_info` (`->uservisible` — видна ли активность пользователю).
- **Запись на курс:** `{enrol}` — способ записи в конкретном курсе, `{user_enrolments}` — запись пользователя, `{role_assignments}` — роль. Роль назначается отдельно от записи.
- **Enrolment API:** `is_enrolled($context, $user, $withcapability)`, `get_enrolled_users($context, $withcapability, $groupid)`, `enrol_get_my_courses()`, `enrol_get_users_courses($userid)`. Записывать через плагин: `enrol_get_plugin('manual')->enrol_user($instance, $userid, $roleid)`, не прямым `INSERT` (иначе потеряются события).
- **Группы:** режимы активности (нет, раздельные, видимые), `groups_get_all_groups($courseid, $userid)`, `groups_get_activity_group($cm)`. Право `moodle/site:accessallgroups` видит все группы. **Grouping** — набор групп. **Cohort** — глобальная группа пользователей (система или категория), удобна для массовой записи.
- **Completion API:** `completion_info`, `$completion->update_state($cm, COMPLETION_COMPLETE, $userid)`. Свои правила завершения — класс `custom_completion`.
- **Grade API:** `grade_update(...)`, обычно через функции `{mod}_grade_item_update()` и `{mod}_update_grades()` в `lib.php`. **Gradebook** — журнал оценок.
- **Ограничения доступа:** плагины `availability_*`.

### Вопросы и ответы

> [!question] Чем отличаются `cmid` и `instance id`?
> 
> `cmid` — номер размещения активности в курсе (`course_modules.id`), нужен для URL и контекста. `instance id` — номер записи в таблице модуля (`forum.id`). Они не взаимозаменяемы. **Подвох:** контекст модуля строится от **cmid**: `context_module::instance($cm->id)`.

> [!question] Как получить студентов курса и проверить, что пользователь записан?
> 
> `$context = context_course::instance($courseid)`; проверка `is_enrolled($context, $USER, 'mod/forum:viewdiscussion')`, список `get_enrolled_users($context, 'mod/assign:submit')`. **Подвох:** «записан» не значит «имеет право», права проверяются через capability.

> [!question] Чем способ записи отличается от записи пользователя?
> 
> `{enrol}` — экземпляр способа записи в курсе (например, самозапись с ключом), `{user_enrolments}` — запись конкретного пользователя. Роль в `{role_assignments}`, поэтому удаление записи не всегда убирает роль.

> [!question] Как активность сообщает о завершении и об оценке?
> 
> О завершении — `completion_info::update_state`, об оценке — `grade_update` (grade API). Вызывать сразу после действия пользователя, а не по cron.

---

## 5. Контексты, роли, capabilities

### Теория

- **Контекст** — узел дерева, к которому привязаны права: **система → категория курсов → курс → модуль → блок**, отдельно контекст пользователя.
- **Capability** — конкретное право: `mod/forum:replypost`, у local-плагина `local/hello:view` (формат `тип/имя:право`).
- **Роль** — набор разрешений (teacher, student, manager). Назначается **в контексте** и действует вниз по дереву.
- Значения прав: _inherit_, _allow_, _prevent_, _prohibit_ (запрет, который нельзя переопределить ниже). Права можно переопределять (override) на уровне курса или активности.
- Проверка в коде: `has_capability('local/hello:view', $context)` (true/false), `require_capability(...)` (исключение, если права нет). Контекст должен быть **правильным**.
- Объявление в `db/access.php`:

```php
$capabilities = [
    'local/hello:view' => [
        'captype' => 'read',
        'contextlevel' => CONTEXT_SYSTEM,
        'archetypes' => ['manager' => CAP_ALLOW],
    ],
];
```

- Название права в интерфейсе: `$string['hello:view']` в языковом файле.
- После изменения `access.php`: поднять `version` и запустить `upgrade.php`.

### Вопросы и ответы

> [!question] Как в Moodle устроена проверка прав?
> 
> Права привязаны к capability, а не к имени роли. Роль даёт набор capabilities в контексте и действует на вложенные. Код проверяет `has_capability()` / `require_capability()` в нужном контексте. **Подвох:** нельзя проверять «пользователь = преподаватель» по имени роли.

> [!question] Преподаватель видит данные чужого курса. Где ошибка?
> 
> Типичные причины: нет `require_login()`; нет `require_capability()`; проверка в системном контексте вместо `context_course::instance($courseid)`; в SQL нет фильтра по `courseid` или `userid`.

> [!question] Что значит prohibit?
> 
> Жёсткий запрет: в отличие от prevent, его нельзя переопределить в нижних контекстах.

---

## 6. Плагины: типы, frankenstyle, структура, версии

### Теория

- **Frankenstyle** — имя `тип_имя`: `local_hello`, `mod_forum`, `block_html`, `theme_boost`. Имя папки, `component` в `version.php`, namespace и префиксы функций должны совпадать.

|Тип|Папка|Для чего|
|:--|:--|:--|
|`mod`|`/mod`|активности и ресурсы|
|`block`|`/blocks`|блоки|
|`local`|`/local`|проектные доработки|
|`theme`|`/theme`|оформление|
|`auth`|`/auth`|способы входа|
|`enrol`|`/enrol`|способы записи|
|`format`|`/course/format`|форматы курса|
|`tool`|`/admin/tool`|админские инструменты|
|`report`|`/report`|отчёты|
|`filter`|`/filter`|фильтры текста|
|`repository`|`/repository`|источники файлов|
|`qtype`|`/question/type`|типы вопросов|

- **Структура плагина:**

```text
local/hello/
├── version.php             # версия, component, requires
├── lang/en/local_hello.php # строки (обязателен pluginname)
├── db/
│   ├── install.xml         # таблицы
│   ├── upgrade.php         # миграции
│   ├── access.php          # права
│   └── events.php, tasks.php, services.php, hooks.php, caches.php
├── classes/                # классы (автозагрузка по namespace)
├── templates/              # Mustache
├── amd/src, amd/build      # JS
├── settings.php            # настройки в админке
├── lib.php                 # callbacks (уступает место hooks)
├── tests/                  # PHPUnit, Behat
└── index.php / view.php    # страницы
```

- `version.php`:

```php
$plugin->component = 'local_hello';
$plugin->version   = 2026110200;  // YYYYMMDDXX
$plugin->requires  = 2024100700;  // минимальная версия Moodle
$plugin->maturity  = MATURITY_STABLE;
$plugin->release   = 'v1.0.0';
```

- **Установка и обновление:** ядро сравнивает `version` в файле с `config_plugins`: записи нет → установка (`install.xml`, права, события, кэши); в файле больше → обновление (`xmldb_local_hello_upgrade($oldversion)`); меньше → ошибка (даунгрейда нет).
- Классы в `classes/` ищутся по namespace компонента. Карта классов кэшируется, после нового класса нужен `purge_caches.php`.
- Обязательно: `pluginname` в языковом файле и Privacy-провайдер.
- **Зависимости:** `$plugin->dependencies = ['mod_forum' => 2022112800];`. Совместимость: `$plugin->supported` (диапазон версий Moodle) и `$plugin->incompatible`.
- **Подплагины** (например `assignsubmission_*` в `mod_assign`, `quizaccess_*` в `mod_quiz`) описываются в `db/subplugins.json` (раньше `subplugins.php`), папка лежит внутри родителя.

### Вопросы и ответы

> [!question] Что такое frankenstyle?
> 
> Единая схема имён `тип_имя`. По ней строятся компонент, папка, namespace (`local_hello\...`), префиксы функций и таблиц. Так ядро находит плагины и их классы.

> [!question] Как Moodle понимает, что плагин надо установить или обновить?
> 
> Сравнивает `$plugin->version` с версией в `config_plugins`. Нет записи — установка, число больше — upgrade, меньше — ошибка.

> [!question] Плагин скопирован, а Moodle его не видит. Причины?
> 
> Не та папка; имя папки не совпадает с `component`; нет `version.php`; нет строки `pluginname`; у веб-сервера нет прав чтения.

> [!question] После создания класса в `classes/` — «Class not found». Почему?
> 
> Карта классов закэширована: `php admin/cli/purge_caches.php`. Проверить namespace и регистр букв в пути.

> [!question] Когда поднимать версию плагина?
> 
> При изменении схемы БД, `access.php`, определений событий, задач, служб, кэшей, то есть всего, что читается при установке или обновлении. Иначе изменения не применятся.

> [!question] Что такое подплагин?
> 
> Расширение внутри плагина-родителя (способ сдачи в `mod_assign`, правило доступа в `mod_quiz`). Родитель задаёт точки расширения, подплагины подключаются через `db/subplugins.json`.

---

## 7. База данных

### Теория

- Только через **`$DB`** (`moodle_database`). `PDO`, `mysqli_*`, `pg_*` напрямую **запрещены**: `$DB` даёт префиксы, защиту от инъекций, совместимость СУБД, транзакции.
- Таблицы в фигурных скобках: `{user}` (префикс `mdl_` подставляется сам).
- Параметры — **плейсхолдеры**: `:name` или `?`. Не смешивать в одном запросе. Одно имя `:name` лучше не повторять, имена получают через `$DB->get_in_or_equal(..., SQL_PARAMS_NAMED, 'prefix')` или `$DB->get_unique_param()`.

|Метод|Что делает|
|:--|:--|
|`get_record($table, $cond, $fields, MUST_EXIST)`|одна запись|
|`get_records(...)`|массив записей, ключ = первая колонка|
|`get_records_sql($sql, $params, $limitfrom, $limitnum)`|запрос на SQL, с пагинацией|
|`get_records_select`, `delete_records_select`|условие в виде SQL-строки|
|`get_field`, `count_records`, `record_exists`|значение, количество, проверка|
|`get_recordset(...)`|потоковое чтение больших выборок (потом `$rs->close()`)|
|`insert_record` (возвращает id), `update_record` (нужен `id`), `set_field`|запись|
|`delete_records`, `execute`|удаление, произвольный запрос|

- `MUST_EXIST` — исключение, если записи нет. `IGNORE_MISSING` — вернуть `false`.
- В `get_records_sql` первая колонка должна быть уникальной (`id`), иначе записи перезапишутся.
- Кроссплатформенный SQL: `$DB->sql_like()`, `$DB->sql_concat()`, `$DB->get_in_or_equal()`.
- **Транзакции:** `$tx = $DB->start_delegated_transaction(); ... $tx->allow_commit();`
- **Схема:**
    - `db/install.xml` — актуальная схема, применяется при первой установке сайта или плагина. Редактировать удобно через XMLDB editor (Site administration → Development).
    - `db/upgrade.php` — шаги для существующих баз (`$dbman = $DB->get_manager()`, `add_field`, `field_exists`) и обязательный `upgrade_plugin_savepoint(true, версия, 'local', 'hello')`.

```php
function xmldb_local_hello_upgrade($oldversion) {
    global $DB;
    $dbman = $DB->get_manager();
    if ($oldversion < 2026110300) {
        $table = new xmldb_table('local_hello_messages');
        $field = new xmldb_field('status', XMLDB_TYPE_INTEGER, '2', null, XMLDB_NOTNULL, null, '0', 'message');
        if (!$dbman->field_exists($table, $field)) {   // идемпотентность
            $dbman->add_field($table, $field);
        }
        upgrade_plugin_savepoint(true, 2026110300, 'local', 'hello');
    }
    return true;
}
```

- Имена таблиц плагина начинаются с компонента: `local_hello_messages`. Индексы ставят на поля частого поиска (`courseid`, `userid`).

### Вопросы и ответы

> [!question] Как безопасно работать с БД в Moodle?
> 
> Только через `$DB`, таблицы в `{}`, значения только плейсхолдерами. Никакой конкатенации значений в SQL.

> [!question] В чём разница между `install.xml` и `upgrade.php`?
> 
> `install.xml` — итоговая схема для новой установки, `upgrade.php` — пошаговые изменения для существующих сайтов. Схему меняют в обоих местах и поднимают версию.

> [!question] Что такое N+1 и как избежать?
> 
> Запрос в цикле (`get_record` на каждого пользователя). Решение: один запрос с `JOIN` или `IN`, для структуры курса `get_fast_modinfo()`.

> [!question] Чем `get_records` опасен?
> 
> Ключ массива — первая колонка. Если она не уникальна, часть данных потеряется.

> [!question] Как взять большой объём данных или сделать постраничный вывод?
> 
> Пагинация: `$limitfrom` и `$limitnum`. Большие выборки: `get_recordset()` читает потоком и не грузит всё в память, в конце `$rs->close()`.

---

## 8. Параметры и безопасность

### Теория

- Данные запроса берут **только** через `required_param()` / `optional_param()` с типом `PARAM_*` (`PARAM_INT`, `PARAM_TEXT`, `PARAM_ALPHANUMEXT`, `PARAM_URL`, `PARAM_BOOL`...). `$_GET` и `$_POST` напрямую не используют.
- **SQL-инъекции** → плейсхолдеры.
- **XSS** → экранировать вывод: `s()`, `format_string()`, `format_text()`. В Mustache `{{ }}` экранирует, `{{{ }}}` нет (только для очищенных данных).
- **CSRF** → sesskey: `sesskey()` в форме или ссылке, `require_sesskey()` на сервере. `moodleform` делает это сам.
- **Доступ** → `require_login()` + `require_capability()` в верном контексте.
- **Файлы** → отдача только через `pluginfile` с проверками.

### Вопросы и ответы

> [!question] Как в Moodle защищаются от основных атак?
> 
> SQL-инъекции: `$DB` и плейсхолдеры. XSS: `PARAM_*` на входе и экранирование на выходе. CSRF: sesskey. Доступ: `require_login` и capabilities. Файлы: отдача через PHP после проверки.

> [!question] Чем `PARAM_TEXT` отличается от `PARAM_RAW`?
> 
> `PARAM_RAW` ничего не чистит (опасно). `PARAM_TEXT` убирает теги, оставляя безопасный текст. Всегда выбирай самый узкий тип.

---

## 9. Формы, вывод, строки, настройки

### Теория

**Формы (`moodleform`):**

- Класс наследует `moodleform`, поля в `definition()` через `$this->_form`, `validation($data, $files)` возвращает массив ошибок.
- Использование: `$form->is_cancelled()`, `$form->get_data()` (null, пока форма не отправлена или не прошла валидацию), `$form->set_data(...)`. Защита от CSRF встроена.

```php
class message_form extends \moodleform {
    protected function definition() {
        $mform = $this->_form;
        $mform->addElement('text', 'message', get_string('message', 'local_hello'));
        $mform->setType('message', PARAM_TEXT);
        $mform->addRule('message', null, 'required', null, 'client');
        $this->add_action_buttons();
    }
}
```

- **Dynamic forms** (`\core_form\dynamic_form`) — формы в модальном окне через AJAX, без перезагрузки. Проверка прав и контекста обязательна.

**Вывод:**

- Простой вывод: `$OUTPUT`. Правильный путь: **renderer** (`classes/output/renderer.php`) + **Mustache-шаблон** (`templates/*.mustache`).
- Данные для шаблона готовит класс, реализующий `renderable` и `templatable` (`export_for_template`). Вызов: `$OUTPUT->render_from_template('local_hello/card', $data)`.
- В шаблоне: `{{переменная}}`, `{{#список}}...{{/список}}`, `{{#str}}ключ, компонент{{/str}}`. Логика в PHP, шаблон только отображает.
- Помощники: `new moodle_url('/local/hello/index.php', ['id' => $id])` (ссылки), `redirect($url, 'Сохранено', null, \core\output\notification::NOTIFY_SUCCESS)`, `\core\notification::add(...)` (сообщение на следующей странице), `html_writer::tag()` / `::link()` (где шаблон избыточен).
- Ошибки: `throw new \moodle_exception('код', 'local_hello', $returnurl)`, текст берётся из языковой строки с этим ключом.
- **Таблицы:** `table_sql` / `flexible_table` (сортировка, пагинация, экспорт). **Report builder** (с 4.0) — конструктор отчётов: плагин описывает источник данных, отчёты собирает админ. Вручную `<table>` в цикле не строят.

**Языковые строки:**

- Файл `lang/en/local_hello.php`, `$string['key'] = '...'`, обязательна `pluginname`. Вывод: `get_string('key', 'local_hello', $a)` (параметр подставляется в `{$a}`).
- Другие языки подключаются через AMOS (в плагине достаточно `en`). **Никакого текста для пользователя прямо в коде.**

**Настройки плагина (`settings.php`):**

- `admin_settingpage`, поля `admin_setting_configtext` и др., добавление `$ADMIN->add('localplugins', $settings)` внутри `if ($hassiteconfig)`.
- Чтение `get_config('local_hello', 'ключ')`, запись `set_config(...)`. Хранится в `config_plugins`.

### Вопросы и ответы

> [!question] Как построить страницу с формой?
> 
> Класс формы наследует `moodleform` → в скрипте создать форму → `if ($form->is_cancelled())` редирект → `elseif ($data = $form->get_data())` сохранить через `$DB` → иначе `$form->display()` между `header()` и `footer()`.

> [!question] Зачем renderer и Mustache, если можно `echo`?
> 
> Логика отделена от разметки, вывод можно переопределить в теме, единый стиль и экранирование, те же шаблоны работают из JavaScript.

> [!question] Как выводить текст на разных языках?
> 
> Через языковые строки `get_string('ключ', 'компонент')` из `lang/en/компонент.php`, переводы подключаются отдельно.

> [!question] Как вывести таблицу с сортировкой и пагинацией?
> 
> `table_sql` (или `flexible_table`), для настраиваемых отчётов Report builder. Свой `foreach` с `echo` теряет сортировку, экспорт и единый вид.

> [!question] Что такое dynamic form и когда она нужна?
> 
> Форма на базе `moodleform`, которая рендерится и отправляется через AJAX (часто в модальном окне). Подходит для быстрых правок без перехода на другую страницу.

---

## 10. События и cron-задачи

### Теория

**События (Events API):**

- Класс в `classes/event/`, наследует `\core\event\base`; в `init()` задают `crud` (c/r/u/d), `edulevel`, `objecttable`.
- Отправка: `\local_hello\event\message_sent::create([...])->trigger();`
- Наблюдатель в `db/events.php`:

```php
$observers = [
    [
        'eventname' => '\core\event\course_completed',
        'callback'  => '\local_courseaudit\observer::course_completed',
    ],
];
```

- Наблюдатели работают **синхронно** в том же запросе: внутри только быстрое, тяжёлое выносят в задачу. События попадают в логи и отчёты.

**Задачи cron:**

- **Scheduled task** — по расписанию. Класс наследует `\core\task\scheduled_task` (`get_name()`, `execute()`), расписание в `db/tasks.php`.
- **Adhoc task** — разовая задача из очереди. Наследует `\core\task\adhoc_task`, данные через `set_custom_data()`, постановка `\core\task\manager::queue_adhoc_task($task)`. При сбое Moodle повторяет позже.
- Cron на сервере запускается **каждую минуту** (`php admin/cli/cron.php`). Вручную: `php admin/cli/scheduled_task.php --execute='\local_hello\task\cleanup'`.

### Вопросы и ответы

> [!question] Зачем нужны события и чем они лучше правки ядра?
> 
> Это точка расширения: ядро объявляет, что что-то произошло, плагин подписывается и реагирует без правки ядра. События ещё и пишутся в лог.

> [!question] Чем scheduled task отличается от adhoc?
> 
> Scheduled повторяется по расписанию. Adhoc одноразовая, ставится в очередь из кода (например из наблюдателя), при ошибке повторяется.

> [!question] Почему не отправлять письмо прямо в наблюдателе?
> 
> Наблюдатель синхронный: письмо замедлит ответ, а сбой почты может сломать запрос. Правильно: быстрая операция + постановка adhoc-задачи.

> [!question] Cron не работает. Как понять и починить?
> 
> В админке (Site administration → Server → Tasks) видны время последнего запуска, логи задач и предупреждение, что cron давно не запускался. Проверить, что в crontab каждую минуту вызывается `php admin/cli/cron.php`, что пользователь совпадает с владельцем `moodledata`, и нет ли зависшей долгой задачи.

---

## 11. Web services и JavaScript

### Теория

**External functions:**

- Описание в `db/services.php`, класс в `classes/external/` наследует `\core_external\external_api`.
- Три метода: `execute_parameters()`, `execute()`, `execute_returns()`. В `execute()`: `validate_parameters` → `validate_context` → `require_capability` → логика.
- Флаги: `'ajax' => true` (вызов из JS), `'type' => 'read'/'write'`.

```php
$functions = [
    'local_hello_get_messages' => [
        'classname'   => 'local_hello\external\get_messages',
        'description' => 'Get messages',
        'type'        => 'read',
        'ajax'        => true,
    ],
];
```

```php
namespace local_hello\external;

use core_external\external_api;
use core_external\external_function_parameters;
use core_external\external_multiple_structure;
use core_external\external_single_structure;
use core_external\external_value;

class get_messages extends external_api {
    public static function execute_parameters(): external_function_parameters {
        return new external_function_parameters([
            'courseid' => new external_value(PARAM_INT, 'Course id'),
        ]);
    }

    public static function execute(int $courseid): array {
        global $DB;
        $params = self::validate_parameters(self::execute_parameters(), ['courseid' => $courseid]);
        $context = \context_course::instance($params['courseid']);
        self::validate_context($context);
        require_capability('local/hello:view', $context);

        $records = $DB->get_records('local_hello_messages', ['courseid' => $params['courseid']]);
        return array_map(fn($r) => ['id' => $r->id, 'message' => $r->message], array_values($records));
    }

    public static function execute_returns(): external_multiple_structure {
        return new external_multiple_structure(new external_single_structure([
            'id' => new external_value(PARAM_INT, 'Id'),
            'message' => new external_value(PARAM_TEXT, 'Text'),
        ]));
    }
}
```

**Внешний доступ:**

- В `db/services.php` рядом с `$functions` описывают `$services` (набор функций под одним именем). Админ включает web services и REST, создаёт пользователя-интегратора с минимальными правами и **токен**.
- Вызов REST: `/webservice/rest/server.php?wstoken=...&wsfunction=local_hello_get_messages&moodlewsrestformat=json`.
- Токен — это пароль: только HTTPS, не хранить в репозитории. Мобильное приложение работает поверх этого же слоя.

**JavaScript:**

- Исходники `amd/src/*.js` (ES6-модули), сборка в `amd/build/` командой `npx grunt amd`.
- Подключение: `$PAGE->requires->js_call_amd('local_hello/main', 'init', [$args]);`
- Модули: `core/ajax` (external functions), `core/templates` (Mustache на клиенте), `core/str`, `core/notification`, `core/modal`.

```js
import Ajax from 'core/ajax';
export const init = () => {
    Ajax.call([{methodname: 'local_hello_get_messages', args: {}}])[0]
        .then(data => console.log(data));
};
```

### Вопросы и ответы

> [!question] Как сделать AJAX-запрос в Moodle?
> 
> Объявить external function (`ajax => true`), в JS вызвать через `core/ajax`, внутри проверить контекст и capability. **Подвох:** не писать свой PHP-скрипт для AJAX без sesskey и проверок.

> [!question] Изменил JS, а в браузере старое. Что делать?
> 
> Пересобрать `npx grunt amd` (нужны файлы в `amd/build`), сбросить кэши, на dev отключить `cachejs`.

> [!question] Как внешняя система получит данные из Moodle?
> 
> Описать external function (и сервис), включить web services и REST, создать выделенного пользователя с минимальными правами и токен. Функция сама проверяет контекст и capability, потому что токен даёт доступ от имени пользователя. Для больших выгрузок учесть нагрузку и пагинацию.

---

## 12. Hooks, Files, Cache, Message, Privacy

### Теория

**Hooks и callbacks:**

- Старый способ: функции в `lib.php` вида `{component}_{callback}` (например `local_hello_extend_navigation`).
- **Hooks API** (с 4.3): класс хука в ядре, подписка в `db/hooks.php`, обработчик в `classes/hook_callbacks.php`. Постепенно заменяет callbacks, по возможности используй его.

**Files API:**

- Файл идентифицируется набором `contextid, component, filearea, itemid, filepath, filename`. Хранится в `{files}`, содержимое на диске по `contenthash`.
- Работа: `get_file_storage()` → `create_file_from_string/pathname`, `get_area_files`.
- Отдача: функция `local_hello_pluginfile(...)` в `lib.php` → `require_login`, проверка прав → `send_stored_file()`.
- В формах: `filemanager`, `filepicker`. Файл сначала попадает в **draft area**, затем сохраняется `file_save_draft_area_files()`.

**Cache API (MUC):**

- Определение в `db/caches.php`, режимы: **application** (общий), **session**, **request**.
- `$cache = cache::make('local_hello', 'имя'); $cache->get($key); $cache->set($key, $val);`
- Хранилища: файловое, Redis, Memcached. Сброс: `purge_caches.php`.

**Message API:**

- Тип сообщений в `db/messages.php`. Отправка: `\core\message\message` (`component`, `name`, `userfrom`, `userto`, `subject`, `fullmessage`, `fullmessagehtml`, `smallmessage`, `notification`) и `message_send($message)`.
- Канал (email, popup, mobile) выбирает пользователь. Низкоуровнево есть `email_to_user()`.

**Privacy API (GDPR):**

- У каждого плагина обязателен `classes/privacy/provider.php`. Нет личных данных → `null_provider`.
- Есть данные → `metadata\provider` (описание) + найти контексты пользователя, экспортировать данные, удалить данные пользователя или всех в контексте.

### Вопросы и ответы

> [!question] Как безопасно отдать файл пользователю?
> 
> Через `{component}_pluginfile`: `require_login`, права в контексте, затем `send_stored_file`. Прямую ссылку на диск не отдаём.

> [!question] Зачем Cache API, если есть PHP-кэш?
> 
> MUC абстрагирует хранилище: один код работает на файлах, Redis или Memcached, есть режимы (общий, сессия, запрос) и централизованный сброс.

> [!question] Что такое Privacy API и почему он обязателен?
> 
> Реализация требований GDPR: плагин должен описать, какие данные пользователя хранит, уметь их экспортировать и удалить. Без провайдера плагин не проходит проверки.

> [!question] Чем Hooks API лучше callbacks в `lib.php`?
> 
> Хуки типизированы, лежат в классах (автозагрузка, не грузится весь `lib.php`), их проще тестировать и упорядочивать.

---

## 13. Модуль mod, блок block, другие плагины

### Теория

**Модуль активности (`mod`):**

- Обязательные части:
    - `lib.php`: `{mod}_add_instance`, `{mod}_update_instance`, `{mod}_delete_instance` и `{mod}_supports()` (флаги `FEATURE_*`);
    - `mod_form.php` (наследует `moodleform_mod`), `view.php` (принимает **cmid**);
    - `db/install.xml` с таблицей модуля (`course`, `name`, `intro`, `introformat`, `timemodified`);
    - `db/access.php` с правами, включая `mod/{name}:addinstance` и `:view`;
    - `backup/moodle2/` — классы `backup_{mod}_activity_task` и `restore_{mod}_activity_task` и описание данных. Без них активность не переносится с курсом.
- Поддержка завершения (completion), оценок (grade API), событий, логирования.

**Блок (`block`):**

- Класс `block_xxx extends block_base` в `block_xxx.php`. `init()` задаёт `$this->title`, `get_content()` заполняет `$this->content->text` и `->footer`.
- Дополнительно: `applicable_formats()`, `has_config()`, `instance_allow_multiple()`, `specialization()`, `edit_form.php`. Права: `block/xxx:addinstance`, `block/xxx:myaddinstance`.

**Другие типы:** `theme`, `auth`, `enrol`, `format`, `report`, `filter`, `repository`, `qtype`, `availability` (ограничения доступа), `tool` (админ-инструменты), `editor`.

### Вопросы и ответы

> [!question] Какие функции обязательны для плагина-активности?
> 
> `add_instance`, `update_instance`, `delete_instance`, `supports`, плюс `mod_form.php`, `view.php`, схема БД и права. Без этого активность нельзя создать, изменить или удалить.

> [!question] Чем `local` отличается от `mod`?
> 
> `mod` — активность в курсе (cm, настройки, backup, оценки). `local` — любая проектная доработка вне модели активностей: страницы, интеграции, фоновые обработчики.

> [!question] Как выбрать тип плагина под задачу?
> 
> Активность в курсе → `mod`; виджет сбоку → `block`; внешний вид → `theme`; вход через внешнюю систему → `auth`; способ записи → `enrol`; отчёт → `report`; остальное проектное → `local`.

---

## 14. Тесты, отладка, стиль, обновление

### Теория

**Тестирование:**

- **PHPUnit** (`tests/*_test.php`): класс наследует `\advanced_testcase`, вызывает `$this->resetAfterTest()`, данные создаёт через `$this->getDataGenerator()->create_user()/create_course()`.
- Запуск: один раз `php admin/tool/phpunit/cli/init.php`, потом `vendor/bin/phpunit local/hello/tests/manager_test.php`.
- **Behat** (`tests/behat/*.feature`): сценарии на Gherkin, браузерные проверки, Selenium (удобно в moodle-docker).

**Отладка и производительность:**

- `debugging('текст', DEBUG_DEVELOPER)`, `$CFG->debug`, `$CFG->perfdebug`, Xdebug, логи.
- Главный враг — **N+1 запросы**. Структуру курса брать через `get_fast_modinfo()`.

**Чек-лист «Moodle не видит изменения»:**

1. Менял класс, шаблон, `db/*.php` → `purge_caches.php`.
2. Менял схему или `access.php` → поднять версию и `upgrade.php`.
3. Менял `amd/src` → `npx grunt amd`.
4. Менял CSS/SCSS → `themedesignermode` или сброс кэшей.

**Стиль кода:**

- 4 пробела (без табов), закрывающий `?>` не ставится. В скрипте без классов: `defined('MOODLE_INTERNAL') || die();`
- Линтеры: **moodle-plugin-ci** (`phpcs`, `phpdoc`, `eslint`, `mustache lint` и др.).

**Безопасное обновление сайта:**

1. `php admin/cli/maintenance.php --enable`
2. Бэкап БД **и moodledata**.
3. Заменить код и плагины на совместимые.
4. `php admin/cli/upgrade.php --non-interactive`
5. `php admin/cli/purge_caches.php`, затем `maintenance.php --disable`.

### Вопросы и ответы

> [!question] Как тестировать плагин?
> 
> Логику — PHPUnit с генераторами данных, пользовательские сценарии — Behat. Перед проверкой кода прогнать `moodle-plugin-ci`.

> [!question] Как правильно обновить Moodle?
> 
> Режим обслуживания → бэкап БД и `moodledata` → новый код и совместимые плагины → `upgrade.php` → сброс кэшей → выключить обслуживание. Сначала проверить на копии.

> [!question] Страница работает медленно. Что проверить?
> 
> Не остался ли dev-режим в `config.php`; N+1 запросы; тяжёлые операции в синхронном коде (вынести в adhoc); кэширование через MUC; OPcache и Redis.

---

## 15. Интеграции, сессии, блокировки, инфраструктура

### Теория

- **Исключения:** `throw new \moodle_exception('код', 'local_hello', $returnurl, $a);`. Необработанное исключение показывает страницу ошибки (в dev-режиме со стеком), в adhoc-задаче означает «задача упала», повтор позже.
- **HTTP наружу:** `\core\http_client` (на базе Guzzle, в новых версиях) или класс `curl` из `lib/filelib.php`. Они учитывают прокси и защитные настройки Moodle (чёрные списки хостов и портов) против SSRF. Не использовать `file_get_contents($url)` и «голый» `curl_*` с адресом от пользователя.
- **Lock API:** защита от параллельного запуска.

```php
$factory = \core\lock\lock_config::get_lock_factory('local_hello');
if ($lock = $factory->get_lock('sync_users', 10)) {
    try {
        // работа, которую нельзя запускать параллельно
    } finally {
        $lock->release();
    }
} else {
    // не получили блокировку за 10 секунд
}
```

- **Сессии:** файлы, БД или Redis. PHP-сессия блокируется на время запроса, поэтому долгая страница тормозит остальные запросы того же пользователя. В долгой операции после чтения нужного из сессии вызывают `\core\session\manager::write_close()`, после этого писать в `$SESSION` нельзя.
- **Внедрение зависимостей:** DI-контейнер (с 4.4): `\core\di::get(\some\service::class)`.
- **Маршрутизация:** в 5.x новый роутер для новых точек входа, основная масса страниц остаётся отдельными скриптами. Детали на moodledev.io.
- **Логи:** события пишутся в `logstore_standard`, в CLI и cron есть `mtrace()`, предупреждения через `debugging()`. Логи задач: Site administration → Server → Tasks.
- **SSO:** плагины `auth_*`: `auth_oauth2` (Google, Microsoft, свой провайдер), `auth_ldap`, SAML (обычно сторонний `auth_saml2`), `auth_lti`.
- **Мобильное приложение:** работает поверх web services, для своего плагина нужны `db/mobile.php` и шаблоны приложения.

### Вопросы и ответы

> [!question] Как из плагина безопасно вызвать внешний API?
> 
> Через `\core\http_client` или `curl`. Ключи хранить в настройках плагина (`get_config`, поле-пароль). Вызов выносить в adhoc или scheduled-задачу, задать таймаут, обработать ошибки, не писать секреты в лог.

> [!question] Пользователь запустил долгую страницу, остальные вкладки «зависли». Почему?
> 
> Сессия блокируется на время запроса. Прочитать нужное из сессии и вызвать `write_close()` либо вынести тяжёлую работу в фоновую задачу.

> [!question] Как не допустить параллельного запуска синхронизации?
> 
> Lock API с понятным ключом и `release()` в `finally`. Scheduled-задачи ядро само параллельно не запускает, но ручной CLI-запуск или второй путь вызова кода это не закрывает.

> [!question] Как организовать вход через корпоративный SSO?
> 
> Плагин `auth_*` под протокол: OAuth2, LDAP, SAML (чаще сторонний `auth_saml2`), с сопоставлением полей профиля и ролей. Свой плагин `auth` пишут, только если готового нет.

---

## 16. Чужой код, совместимость, вклад в проект

### Теория

- **Где искать:** moodledev.io (API, upgrade notes), docs.moodle.org (для админов), Tracker (tracker.moodle.org, задачи `MDL-12345`), каталог плагинов moodle.org/plugins.
- **Разобраться в незнакомом коде:** искать по ключу языковой строки, имени capability или события; смотреть, как сделано в ядре (`mod_assign`, `mod_forum`, простой `mod_choice`, `block_html`); отладка: debug-режим, `$CFG->perfdebug`, Xdebug, логи.
- **Менять поведение без правки ядра:** события, хуки и callbacks, настройки и capabilities; внешний вид — переопределение renderer или шаблона в дочерней теме; логика — `local`-плагин.
- **Совместимость:** функции и классы помечаются **deprecated** и удаляются через несколько релизов. В dev-режиме предупреждает `debugging()`, список изменений в upgrade notes на moodledev.io. Плагин под несколько версий ведут ветками git (по ветке на версию), указывают `$plugin->requires` и `$plugin->supported`.
- **Стиль и проверки:** стандарт кода Moodle (4 пробела, phpdoc, заголовок GPL, `@package`, `@copyright`), `moodle-plugin-ci` (phpcs, phpdoc, mustache, grunt, phpunit, behat).
- **Вклад в ядро:** задача в Tracker (`MDL-...`), pull request с веткой вида `MDL-12345-main-описание`, затем review, интеграция и тестирование командой Moodle.
- **Инструменты в админке** (Site administration → Development): сброс кэшей, отладка, XMLDB editor, Template library, генератор тестовых курсов `tool_generator`, тестовый клиент web services.

### Вопросы и ответы

> [!question] Нужно изменить поведение стороннего плагина. Что делать?
> 
> Не править файлы на месте: обновление сотрёт правку. Сначала искать точку расширения (событие, хук, настройка). Нет её — форк плагина с оформленным патчем и заявкой автору. Правка должна быть воспроизводимой (отдельный git-репозиторий или Composer).

> [!question] Как найти в коде Moodle, где что-то происходит?
> 
> Взять текст из интерфейса, найти его ключ в `lang/en/*.php`, затем `grep` по ключу. Для действия смотреть события в логах, имя capability, ссылку и её скрипт. Для пошагового разбора Xdebug.

> [!question] Что значит «функция deprecated» и как реагировать?
> 
> Функция устарела и будет удалена через несколько релизов. В dev-режиме Moodle предупреждает. Найти замену в upgrade notes и поправить плагин до перехода на версию, где её удалят.

---

## 17. Темы и внешний вид

### Теория

- Тема — плагин `theme_xxx`. **Boost** — базовая тема на Bootstrap, своя тема обычно **дочерняя** от неё: наследует всё и переопределяет нужное.
- Файлы: `version.php`, `config.php` (`$THEME->name`, `$THEME->parents = ['boost']`, `$THEME->scss`, `$THEME->layouts`), `lib.php` (функции SCSS), `scss/`, `templates/`, `classes/output/core_renderer.php`, `pix/`, `lang/`.
- **Макеты (layouts):** `standard`, `course`, `incourse`, `admin`, `login`, `frontpage`, `mydashboard` и др. Выбор: `$PAGE->set_pagelayout()`, описание в `$THEME->layouts`.
- **Переопределение шаблона:** файл `theme/xxx/templates/{компонент}/{имя}.mustache` заменяет шаблон ядра или плагина.
- **Переопределение renderer:** класс `theme_xxx\output\core_renderer` наследует `\theme_boost\output\core_renderer`.
- SCSS компилируется на сервере и кэшируется: после правок сбросить кэши (на dev `themedesignermode`).

### Вопросы и ответы

> [!question] Как изменить внешний вид без правки ядра?
> 
> Дочерняя тема от Boost: свой SCSS, переопределённые шаблоны и renderer. Ядро и Boost не трогаем, обновления проходят безболезненно.

> [!question] Как переопределить шаблон ядра или плагина?
> 
> Положить одноимённый файл в `templates/{компонент}/` своей темы, затем сбросить кэши.

---

## 18. Встроенные подсистемы Moodle

### Теория

- **Тест (`mod_quiz`):** вопросы в **банке вопросов**, в тест попадают через слоты. Попытка — `quiz_attempts`, состояние ведёт **question engine** (`question_usage_by_activity`). Расширения: типы вопросов `qtype_*`, поведения `qbehaviour_*`, правила доступа `quizaccess_*`, отчёты `quiz_*`.
- **Задание (`mod_assign`):** сдачи в `assign_submission`, способы сдачи `assignsubmission_*`, обратная связь `assignfeedback_*`, оценки уходят через grade API.
- **Форум (`mod_forum`):** обсуждения и посты, подписки, режимы групп, рассылка через Message API.
- **Navigation API:** пункты меню плагин добавляет callbacks `{component}_extend_navigation_course`, `_extend_navigation_user_settings`, `_extend_settings_navigation` (или хуками). Право проверять обязательно.
- **Календарь:** события в `{event}`, активность подставляет действие через `mod_xxx_core_calendar_provide_event_action`.
- **Значки (badges), компетенции (competency), теги (tag API), комментарии** — подсистемы со своим API и правами.
- **Глобальный поиск:** плагин описывает область поиска (`search area`), индексация идёт по cron.
- **Backup/restore:** структура описывается классами backup (`backup_nested_element`) и restore (`restore_structure_step`). Пользовательские данные включаются по настройкам.

### Вопросы и ответы

> [!question] Как устроена попытка теста?
> 
> Тест хранит слоты с вопросами из банка. При старте создаётся попытка (`quiz_attempts`) и использование вопросов (question usage). Ответы и шаги ведёт question engine, оценку считает поведение вопроса (`qbehaviour_*`), результат уходит в журнал оценок.

> [!question] Как добавить свой тип вопроса или правило доступа к тесту?
> 
> Плагин `qtype_*` или `quizaccess_*`. Это подплагины с точками расширения в `mod_quiz` и ядре вопросов, сам тест править не нужно.

> [!question] Как добавить пункт в меню курса?
> 
> Callback `{component}_extend_navigation_course` (или хук) и `has_capability` в контексте курса. Право объявить в `db/access.php`.

---

## 19. PHP, SQL, Git, веб для собеседования

### Теория

- **PHP 8:** типизация (скалярные, union, nullable), `readonly`, `enum`, `match`, `?->`, именованные аргументы, constructor promotion, attributes.
- **ООП:** SOLID, интерфейс против абстрактного класса, трейты, внедрение зависимостей, исключения. Автозагрузка PSR-4, пакеты через Composer.
- **SQL:** виды `JOIN`, `GROUP BY` и `HAVING`, индексы (порядок полей в составном), `EXPLAIN`, транзакции и ACID, нормализация, N+1.
- **Git:** ветки, `merge` против `rebase`, `cherry-pick`, откат. В Moodle `main` и ветки на версию `MOODLE_XXX_STABLE`.
- **Веб:** методы и коды HTTP, cookies и сессии, CORS, XSS, CSRF, SQL-инъекция, SSRF.

### Вопросы и ответы

> [!question] Чем интерфейс отличается от абстрактного класса?
> 
> Интерфейс — контракт без реализации, класс может реализовать несколько. Абстрактный класс может содержать общий код и состояние, наследуется один. Интерфейс берут, когда важен контракт, абстрактный класс, когда есть общая реализация.

> [!question] Как ускорить медленный SQL-запрос?
> 
> Посмотреть план (`EXPLAIN`), добавить индекс на поля фильтра и соединения, убрать лишние поля и подзапросы, устранить N+1 (один запрос с `JOIN` или `IN`). Составной индекс работает слева направо, порядок полей важен.

> [!question] Чем `merge` отличается от `rebase`?
> 
> `merge` сохраняет историю и добавляет коммит слияния. `rebase` переписывает коммиты поверх новой базы, история линейная. Опубликованную ветку не перебазируют.

> [!question] Чем отличаются XSS, CSRF и SQL-инъекция?
> 
> XSS: чужой скрипт выполняется в браузере жертвы (защита: экранирование вывода). CSRF: браузер жертвы отправляет запрос от её имени (защита: токен, в Moodle sesskey). SQL-инъекция: данные попадают в SQL как код (защита: плейсхолдеры).

---

## 20. Сценарные вопросы

> [!question] При завершении курса записать данные в свою таблицу аудита и отправить письмо с уникальным кодом. Как спроектировать?
> 
> 1. Плагин `local_courseaudit`.
> 2. В `db/install.xml` таблица лога (`userid`, `courseid`, `certcode`, `timecreated`).
> 3. В `db/events.php` наблюдатель `\core\event\course_completed`.
> 4. В наблюдателе **только** быстрая вставка `$DB->insert_record()` и постановка adhoc-задачи (`set_custom_data` с `userid` и `courseid`, `queue_adhoc_task`).
> 5. В задаче сформировать текст через `get_string()` и отправить через `message_send` или `email_to_user()`. При неудаче бросить исключение: Moodle повторит задачу позже.
> 6. Добавить Privacy-провайдер, права и тесты.

> [!question] Нужен свой отчёт «время активности студентов» по курсу. Как?
> 
> Плагин `report_` (или `local_` со страницей). Страница: `require_login($course)`, `require_capability('report/xxx:view', context_course::instance($courseid))`. Данные одним `JOIN`-запросом по логам с плейсхолдерами и фильтром по курсу, вывод через `table_sql` или renderer + Mustache, большие выборки через `get_recordset`. Права, строки и Privacy как у любого плагина.

> [!question] Нужно добавить пункт в навигацию курса только для преподавателей.
> 
> Callback `local_xxx_extend_navigation_course` (или хук), пункт добавлять только если `has_capability('local/xxx:view', $context)`. Право объявить в `access.php` для архетипа `editingteacher`.

> [!question] Нужна кнопка, которая сохраняет данные без перезагрузки страницы.
> 
> External function (`ajax => true`, с проверкой контекста и прав) + AMD-модуль на `core/ajax`, подключение через `js_call_amd`. Ошибки показывать через `core/notification`.

> [!question] Как с нуля сделать минимальный плагин `local` со страницей, формой и таблицей?
> 
> 1. Папка `local/hello`, `version.php` (`component`, `version`, `requires`).
> 2. `lang/en/local_hello.php` со строкой `pluginname`.
> 3. `db/access.php` с правом `local/hello:view`.
> 4. `db/install.xml` с таблицей `local_hello_messages`.
> 5. `index.php`: `require_login`, `require_capability`, `$PAGE`, вывод через `$OUTPUT`.
> 6. Форма `moodleform` в `classes/form/`, сохранение через `$DB->insert_record`.
> 7. Таблица `table_sql` или Mustache-шаблон.
> 8. `classes/privacy/provider.php`.
> 9. Войти администратором (запустится установка) или `php admin/cli/upgrade.php`.
> 10. PHPUnit-тест на класс с логикой.

> [!question] Нужно регулярно синхронизировать пользователей и записи на курсы из внешней системы. Как?
> 
> Плагин `local_` или `tool_` со scheduled task (и CLI-скриптом для ручного запуска). Читать порциями (`get_recordset`), делать **идемпотентно** (повторный запуск ничего не ломает). Пользователей создавать через `user_create_user()`, записывать через `enrol_user()` плагина записи, не прямым `INSERT` (иначе потеряются события). Параллельный запуск закрыть Lock API, результаты и ошибки писать в лог.

> [!question] Moodle тормозит в пик (экзамены, тесты). Что проверять?
> 
> Не включён ли dev-режим; OPcache; Redis для сессий и MUC; медленные запросы и индексы в БД; N+1 в своих плагинах; блокировки сессий; тяжёлые отчёты и письма в adhoc; cron на отдельном процессе; общий `moodledata` при нескольких веб-серверах. Для нагрузочных проверок есть генератор курсов `tool_generator`.

> [!question] После обновления Moodle сторонний плагин падает. Что делать?
> 
> Не править на проде. Воспроизвести на копии с debug-режимом и прочитать стек. Проверить, что версия плагина поддерживает новую версию Moodle, и нет ли предупреждений о deprecated. Сбросить кэши, запустить `upgrade.php`. Обновить плагин или откатиться на бэкап. Если плагин заброшен, форкнуть и исправить с оформленным патчем.

> [!question] Нужно изменить поведение ядра или чужого плагина, но править код нельзя. Как?
> 
> Искать точку расширения: событие, хук или callback, настройка, capability. Для внешнего вида переопределить renderer или шаблон в дочерней теме. Нет точки — свой `local`-плагин, крайний случай: форк с оформленным патчем и заявкой в Tracker.

---

## 21. Как отвечать и что спросить

### Если опыта мало

- Говори честно, но с пониманием механики. Пример:
    
    > «Точное название функции сейчас не вспомню, но знаю, как это устроено: данные приходят через `required_param` с типом, права проверяются в контексте курса. Сигнатуру сверяю по moodledev.io или смотрю в `mod_assign` / `mod_forum`.»
    
- Привязывай каждый ответ к безопасности:
    
    - про SQL → `$DB` и плейсхолдеры;
    - про параметры → `required_param` и `PARAM_*`;
    - про права → контексты и `has_capability`;
    - про изменения → «через плагин, не правка ядра».
- Если не знаешь, называй, **где ищешь**: moodledev.io, код ядра, тесты, `moodle-docker`.
    
- Сравнение с Drupal (есть опыт): хуки и события ↔ Hooks/Events API, модули ↔ плагины, Database API ↔ `$DB`, Twig ↔ Mustache.
    

### Что спросить у интервьюера

1. Какая версия Moodle на проде, планируется ли переход на 5.x?
2. Есть ли правки ядра или всё в плагинах?
3. Есть ли CI (moodle-plugin-ci, phpcs) и PHPUnit/Behat?
4. Как устроена инфраструктура: Redis для кэшей, общая файловая система для moodledata?
5. Какие плагины самописные, какие сторонние, и кто обновляет их при смене версии Moodle?
6. Как устроены релизы: есть ли тестовый стенд, как выкатываются обновления?

---

## 22. Шпаргалка

### Где что лежит в плагине

```text
Таблицы БД            ──► db/install.xml
Миграции схемы        ──► db/upgrade.php
Права                 ──► db/access.php
Языковые строки       ──► lang/en/{component}.php
Настройки в админке   ──► settings.php
Подписка на события   ──► db/events.php
Подписка на хуки      ──► db/hooks.php
Cron-задачи           ──► db/tasks.php
Web services / AJAX   ──► db/services.php
Кэши (MUC)            ──► db/caches.php
Типы сообщений        ──► db/messages.php
Подплагины            ──► db/subplugins.json
Мобильное приложение  ──► db/mobile.php
Шаблоны               ──► templates/*.mustache
JavaScript            ──► amd/src/ → amd/build/
Privacy               ──► classes/privacy/provider.php
External functions    ──► classes/external/*.php
Классы событий, задач ──► classes/event/, classes/task/
Обработчики хуков     ──► classes/hook_callbacks.php
Рендереры             ──► classes/output/
Backup/restore        ──► backup/moodle2/
```

### Пять золотых правил

1. **Ядро не трогаем.** Любая задача решается плагином нужного типа.
2. **Права по контексту.** `has_capability` / `require_capability` в правильном контексте, не по имени роли.
3. **Безопасная работа с данными.** Только `$DB` с плейсхолдерами; вход через `required_param` + `PARAM_*`; выход экранируем.
4. **Изменил схему или `access.php` — подними версию и запусти `upgrade.php`.**
5. **Тяжёлое — в фон.** Письма, пересчёты, интеграции через adhoc-задачи, а не в запросе пользователя.

### Мини-чек-лист «страница плагина»

`config.php` → `require_login` → контекст → `require_capability` → `$PAGE` (url, context, title, heading) → `header` → контент → `footer`.

### Частые ловушки

- Забыли `moodledata` при переносе.
- Dev-настройки на проде.
- Код после `require_once(.../setup.php)` в `config.php`.
- Нет `require_login()` или проверка прав не в том контексте.
- Путаница `cmid` и `instance id`.
- Не поднята версия после правки схемы или прав.
- Не собран JS или не сброшены кэши.
- Запрос к БД в цикле (N+1).
- Письма и тяжёлая логика в синхронном наблюдателе.
- Имя права с `_` вместо `/`: правильно `local/hello:view`.
- Долгая операция держит блокировку сессии.
- Прямые `INSERT` в таблицы записей и пользователей вместо API.
- `{{{ }}}` в Mustache с пользовательскими данными.
- Повтор одного именованного плейсхолдера в запросе.
- Нет `$PAGE->set_url()`.
- Правка стороннего кода «на месте» вместо точки расширения.