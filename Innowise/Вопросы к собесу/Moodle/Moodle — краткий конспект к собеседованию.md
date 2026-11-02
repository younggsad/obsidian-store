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
17. [[#17. Сценарные вопросы]]
18. [[#18. Как отвечать и что спросить]]
19. [[#19. Шпаргалка]]

---

## 1. Что такое Moodle

### Теория

- Moodle — **LMS** (система управления обучением): курсы, записи на курс (enrolments), тесты, задания, журнал оценок, компетенции. Это **не** универсальная CMS.
- Открытый код, PHP, лицензия **GPL v3**. СУБД: MySQL/MariaDB, PostgreSQL, MS SQL Server (Oracle не поддерживается с версии 5.0).
- Главное правило: **ядро не правим (no core hacking)**. Всё расширяется плагинами. Даже форум и тест — это плагины.
- Установка = **три части**: код, база данных, каталог **moodledata** (`$CFG->dataroot`).
- moodledata лежит **вне веб-корня**. Там файлы пользователей, кэши, временные файлы, сессии.
- Файлы хранятся по SHA-1 хэшу содержимого (**дедупликация**): одинаковый файл на диске один, а в таблице `{files}` на него много записей.
- Стек фронтенда: Mustache-шаблоны, JavaScript в виде AMD/ES6-модулей (сборка через Grunt), темы на Bootstrap.
- Стиль кода исторически процедурный и с глобальными объектами. Новый код: namespaces, автозагрузка классов, Hooks API, типизация.
- Нет единого front controller: **каждая страница — отдельный PHP-файл** (`view.php`, `index.php`). В новых версиях точечно появляется маршрутизация.
- **Требования к PHP растут с версиями:** 4.5 — 8.1–8.3; 5.0 и 5.1 — 8.2–8.4; 5.2 и 5.3 — 8.3–8.4 (всегда 64-битный PHP).

### Вопросы и ответы

> [!question] Из чего состоит установка Moodle?
> 
> Из трёх частей: кодовая база (ядро и плагины), база данных и каталог `moodledata`. При переносе сайта нужно перенести все три. **Подвох:** если забыть `moodledata`, сайт поднимется, но файлы, аватары и вложения будут битыми.

> [!question] Чем Moodle отличается от WordPress или Drupal?
> 
> - Предметная область — обучение (курсы, оценки, записи).
> - Свой слой работы с БД `$DB` (не ORM). Схема описывается в XML (`install.xml`), миграции — в `upgrade.php`.
> - Нет единого фронт-контроллера: каждая страница — отдельный скрипт.
> - Права строятся на **контекстах** (дерево: система → категория → курс → активность → блок) и **capabilities**. **Подвох:** не называй Moodle «устаревшим монолитом». Лучше: зрелая платформа с сильной обратной совместимостью, которая движется к типизированным API и Hooks.

> [!question] Почему к moodledata нельзя давать прямой веб-доступ?
> 
> Там приватные файлы (ответы на тесты, загруженные работы). Прямая отдача мимо Moodle обходит проверку прав. Файлы отдаются только через PHP-скрипты после проверок (`pluginfile.php`).

> [!question] Что такое LTS и какую версию ставить на прод?
> 
> LTS — версия с длинной поддержкой безопасности, выходит раз в два года. Для прода обычно берут LTS или последнюю стабильную, если команда готова обновляться два раза в год. **Подвох:** плагины и PHP должны подходить под выбранную версию (`$plugin->requires`, список поддерживаемых версий PHP). Если не помнишь точные даты поддержки, так и скажи и назови, где проверяешь.

---

## 2. Окружение, config.php, каталоги

### Теория

- Нужно: PHP нужной версии (см. раздел 1; для 5.x это 8.2–8.4) с расширениями `intl`, `mbstring`, `xml`, `gd`, `zip`, `curl`, `sodium`, `opcache` и др.; PostgreSQL или MariaDB; Node.js (версия из `.nvmrc`) для сборки JS.
- Для разработки удобно **moodle-docker** (`moodlehq/moodle-docker`): готовые контейнеры, в том числе для Behat.
- `config.php` — корневой конфиг: БД, `$CFG->wwwroot`, `$CFG->dataroot`, `$CFG->prefix`. **Последняя строка:** `require_once(__DIR__ . '/lib/setup.php');`
- Режим разработки в `config.php`:

```php
$CFG->debug = (E_ALL | E_STRICT);
$CFG->debugdisplay = 1;
$CFG->cachejs = false;
$CFG->langstringcache = false;
$CFG->themedesignermode = true;
```

- Основные CLI-команды:

```bash
php admin/cli/install.php ...        # установка
php admin/cli/upgrade.php            # применить обновления и миграции
php admin/cli/purge_caches.php       # сбросить кэши
php admin/cli/cron.php               # запустить cron
php admin/cli/maintenance.php --enable   # режим обслуживания
```

|Папка|Что внутри|
|:--|:--|
|`/admin`|админка, CLI, утилиты (`/admin/tool`)|
|`/auth`|способы входа (ручная регистрация, OAuth2, LDAP, SAML)|
|`/blocks`|блоки|
|`/course`|управление курсами, форматы курса (`/course/format`)|
|`/enrol`|способы записи на курс|
|`/lib`|библиотеки ядра (`setup.php`, DML/DDL, формы)|
|`/local`|локальные проектные плагины|
|`/mod`|активности и ресурсы (форум, тест, задание)|
|`/theme`|темы (Boost и дочерние)|

- **Moodle 5.0:** добавлена папка `public/`. Веб-сервер смотрит в неё, а `config.php` и `vendor/` остаются выше, вне веб-корня. Внутри плагина ничего не меняется, меняется только место на диске.
- Полезные пути в `$CFG`: `wwwroot` (URL сайта), `dirroot` (путь к коду на диске), `dataroot` (moodledata), `tempdir` и `cachedir` (временные файлы и кэши внутри moodledata), `localcachedir` (локальный кэш каждого сервера). Пути строят от `$CFG->dirroot`, а не от относительных `../`.

### Вопросы и ответы

> [!question] Почему `require_once(.../lib/setup.php)` должен быть последней строкой `config.php`?
> 
> `setup.php` запускает ядро: читает `$CFG`, подключает БД, стартует сессию, включает автозагрузку классов. Всё, что написано после, ядро уже не увидит.

> [!question] Что включают в config.php для разработки и почему нельзя на проде?
> 
> Показ всех ошибок, отключение кэша JS, строк, шаблонов и тем. На проде это даёт: (1) сильное падение скорости, всё компилируется заново на каждый запрос; (2) утечку информации: ошибки показывают пути и структуру БД.

> [!question] Куда класть плагины типа `tool` и `format`?
> 
> `tool` → `/admin/tool/имя`, `format` → `/course/format/имя`.
> 
> **Подвох:** папка не всегда равна типу из имени. Например `qtype` → `/question/type/`, `assignsubmission` → `/mod/assign/submission/`.

---

## 3. Запрос и глобальные объекты

### Теория

**Жизненный цикл страницы плагина:**

1. `require_once(config.php)` → запускается `lib/setup.php` (БД, сессия, пользователь).
2. `require_login()` — проверка, что пользователь вошёл.
3. `require_capability(...)` — проверка права в нужном контексте.
4. Настройка `$PAGE`: `set_url`, `set_context`, `set_title`, `set_heading`.
5. Вывод: `$OUTPUT->header()` → содержимое → `$OUTPUT->footer()`.

**Глобальные объекты:**

|Объект|Что это|
|:--|:--|
|`$CFG`|настройки сайта|
|`$DB`|доступ к базе данных|
|`$USER`|текущий пользователь|
|`$PAGE`|состояние страницы (URL, контекст, заголовок, подключённый JS/CSS)|
|`$OUTPUT`|вывод элементов интерфейса|
|`$COURSE`|текущий курс|
|`$SESSION`|данные текущей сессии|

- `$PAGE->set_pagelayout('standard')` выбирает макет (standard, admin, report, popup...).
- Варианты входа: `require_login()` (только вход), `require_login($course)` (вход + доступ к курсу), `require_login($course, false, $cm)` (вход + доступ к активности, учитывает видимость и ограничения).
- Страницы админки: вместо ручной настройки `$PAGE` вызывают `admin_externalpage_setup('имя_страницы')` (страница должна быть описана в `settings.php`).

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
> Веб-сервер запускает конкретный PHP-скрипт → он подключает `config.php` → ядро поднимает БД, сессию, `$USER` → скрипт делает `require_login()` и проверку прав → настраивает `$PAGE` → выводит страницу через `$OUTPUT`.

> [!question] Почему глобальные объекты не любят в новом коде?
> 
> Они скрывают зависимости, и класс сложно тестировать. В классах глобалы берут явно через `global $DB;` внутри метода (и стараются передавать нужное в параметрах), в новом коде используют внедрение зависимостей. **Подвох:** `require_login()` нужен на **каждой** странице. Без него страница открыта гостю.

---

## 4. Курсы, cm и instance

### Теория

- Структура: **курс → секции → активности** (`course_modules`).
- **Активность** состоит из двух частей:
    - **`cm` (course module)** — «размещение» активности в курсе. Таблица `{course_modules}`, id = **cmid**.
    - **`instance`** — сама активность в таблице плагина (например `{forum}`). `course_modules.instance` ссылается на неё.
- В URL активности обычно приходит **cmid** (`view.php?id=cmid`). По нему находят и cm, и instance:

```php
$cm = get_coursemodule_from_id('forum', $id, 0, false, MUST_EXIST);
$course = get_course($cm->course);
require_login($course, false, $cm);
$context = context_module::instance($cm->id);
```

- Структуру курса быстро получают через `get_fast_modinfo($courseid)` (кэшированный объект, без лишних запросов к БД).
- **Enrolment** — запись пользователя на курс (через плагины `enrol_*`). Роль пользователя в курсе даётся отдельно.
- **Gradebook** — журнал оценок; активности отправляют туда оценки через grade API.
- **Таблицы записи:** `{enrol}` — способ записи, включённый в конкретном курсе (экземпляр), `{user_enrolments}` — запись конкретного пользователя, `{role_assignments}` — назначенная роль. Запись и роль — разные вещи.
- **Enrolment API:** `is_enrolled($context, $user, $withcapability)`, `get_enrolled_users($context, $withcapability, $groupid)`, `enrol_get_my_courses()`, `enrol_get_users_courses($userid)`. Записать в код: `$plugin = enrol_get_plugin('manual'); $plugin->enrol_user($instance, $userid, $roleid);`. Не вставляй строки в `{user_enrolments}` руками: потеряются события и права.
- **Группы:** режимы активности (нет групп, раздельные, видимые), `groups_get_all_groups($courseid, $userid)`, `groups_get_activity_group($cm)`. Право `moodle/site:accessallgroups` позволяет видеть все группы. **Grouping** — набор групп. **Cohort** — глобальная группа пользователей (на уровне системы или категории), часто используется для массовой записи.
- **Completion API:** класс `completion_info`, активность сообщает о выполнении через `$completion->update_state($cm, COMPLETION_COMPLETE, $userid)`. Для своих правил завершения в новых версиях делают класс `custom_completion`.
- **Grade API:** активность создаёт элемент журнала через `grade_update(...)`, обычно оформленный функциями `{mod}_grade_item_update()` и `{mod}_update_grades()` в `lib.php`.
- **Ограничения доступа:** плагины `availability_*`. В коде видимость активности для пользователя проверяют через `cm_info`: `$modinfo = get_fast_modinfo($course); $cminfo = $modinfo->get_cm($cmid); $cminfo->uservisible`.

### Вопросы и ответы

> [!question] Чем отличаются `cmid` и `instance id`?
> 
> `cmid` — номер размещения активности в курсе (`course_modules.id`), нужен для URL и контекста. `instance id` — номер записи в таблице самого модуля (например `forum.id`). Один не заменяет другой. **Подвох:** контекст модуля строится от **cmid**: `context_module::instance($cm->id)`, а не от instance id.

> [!question] Как получить студентов курса и проверить, что пользователь записан?
> 
> `$context = context_course::instance($courseid)`; `is_enrolled($context, $USER, 'mod/forum:viewdiscussion')` для проверки и `get_enrolled_users($context, 'mod/assign:submit')` для списка. Для больших курсов не гонять это в цикле, брать одним запросом. **Подвох:** «записан» не значит «имеет право»: права проверяются отдельно через capability.

> [!question] Чем отличается способ записи от записи пользователя?
> 
> `{enrol}` — это экземпляр способа записи в курсе (например, самозапись с ключом). `{user_enrolments}` — конкретная запись пользователя через этот способ. Роль назначается отдельно (`{role_assignments}`), поэтому удаление записи не всегда убирает роль и наоборот.

> [!question] Как активность сообщает о завершении и об оценке?
> 
> О завершении — через `completion_info::update_state`, об оценке — через grade API (`grade_update`). Обе операции вызывают после действия пользователя (отправка, просмотр), а не по cron, если это возможно.

---

## 5. Контексты, роли, capabilities

### Теория

- **Контекст** — узел дерева, к которому привязаны права: **система → категория курсов → курс → модуль (активность) → блок**. Есть ещё контекст пользователя.
- **Capability** — конкретное право, например `mod/forum:replypost`. Имя в формате `тип/имя:право`, у local-плагина: `local/hello:view`.
- **Роль** — набор разрешений (teacher, student, manager). Роль назначают **в контексте**, и она действует вниз по дереву.
- Значения прав: _inherit_ (наследовать), _allow_, _prevent_, _prohibit_ (запрет, который нельзя переопределить ниже).
- Права можно переопределять (override) на уровне курса или активности.
- Проверка в коде:
    - `has_capability('local/hello:view', $context)` — вернёт true/false.
    - `require_capability(...)` — выбросит исключение, если права нет.
    - Проверять надо в **правильном контексте**.
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

- Название права для интерфейса: строка `$string['hello:view']` в языковом файле.
- После изменения `access.php`: поднять `version` и запустить `upgrade.php`.

### Вопросы и ответы

> [!question] Как в Moodle устроена проверка прав?
> 
> Права привязаны к capability, а не к имени роли. Роль даёт набор capabilities в конкретном контексте и действует на вложенные. Код проверяет `has_capability()` / `require_capability()` в нужном контексте. **Подвох:** нельзя проверять «пользователь = преподаватель» по имени роли. Только по capability.

> [!question] Преподаватель видит данные чужого курса. Где ошибка?
> 
> Типичные причины: (1) нет `require_login()`; (2) нет `require_capability()`; (3) проверка в системном контексте вместо курса (`context_course::instance($courseid)`); (4) в SQL нет фильтра по `courseid` или `userid`.

> [!question] Что значит prohibit?
> 
> Это жёсткий запрет: его нельзя переопределить в нижних контекстах, в отличие от prevent.

---

## 6. Плагины: типы, frankenstyle, структура, версии

### Теория

- **Frankenstyle** — правило имени: `тип_имя`. Например `local_hello`, `mod_forum`, `block_html`, `theme_boost`, `tool_xxx`. Имя папки, `component` в `version.php`, namespace и префиксы функций должны совпадать.
- **Основные типы и папки:**

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
├── version.php            # версия, component, requires
├── lang/en/local_hello.php# строки (обязателен pluginname)
├── db/
│   ├── install.xml        # таблицы
│   ├── upgrade.php        # миграции
│   ├── access.php         # права
│   ├── events.php, tasks.php, services.php, hooks.php, caches.php
├── classes/               # классы (автозагрузка по namespace)
├── templates/             # Mustache
├── amd/src, amd/build     # JS
├── settings.php           # настройки в админке
├── lib.php                # callbacks (устаревает в пользу hooks)
├── tests/                 # PHPUnit, Behat
└── index.php / view.php   # страницы
```

- `version.php`:

```php
$plugin->component = 'local_hello';
$plugin->version   = 2026110200;  // YYYYMMDDXX
$plugin->requires  = 2024100700;  // минимальная версия Moodle
$plugin->maturity  = MATURITY_STABLE;
$plugin->release   = 'v1.0.0';
```

- **Версия и обновление:** ядро сравнивает `version` в файле с версией в таблице `config_plugins`:
    - записи нет → **установка** (`install.xml`, регистрация прав, событий, кэшей);
    - в файле больше → **обновление** (`xmldb_local_hello_upgrade($oldversion)`);
    - в файле меньше → **ошибка**, даунгрейд не поддерживается.
- Классы в `classes/` ищутся по namespace, соответствующему компоненту. Карта классов кэшируется, поэтому после добавления нового класса нужен `purge_caches.php`.
- Обязательное: `pluginname` в языковом файле и Privacy-провайдер.
- **Зависимости и совместимость:** `$plugin->dependencies = ['mod_forum' => 2022112800];` — плагин не установится без нужной версии другого. Также есть `$plugin->supported` (диапазон поддерживаемых версий Moodle) и `$plugin->incompatible`.
- **Подплагины:** у активностей и не только есть свои расширения (например, `assignsubmission` и `assignfeedback` в `mod_assign`, `quizaccess` в `mod_quiz`). Описываются в `db/subplugins.json` (раньше `subplugins.php`). Папка подплагина лежит внутри родительского.

### Вопросы и ответы

> [!question] Что такое frankenstyle?
> 
> Единая схема имён: `тип_имя`. По ней строятся имя компонента, папка, namespace (`local_hello\...`), префиксы функций и таблиц. Благодаря этому ядро находит плагины и их классы.

> [!question] Как Moodle понимает, что плагин нужно установить или обновить?
> 
> Сравнивает `$plugin->version` в `version.php` с версией в БД (`config_plugins`). Нет записи — установка, число больше — upgrade, меньше — ошибка.

> [!question] Плагин скопирован, а Moodle его не видит. Причины?
> 
> Не та папка; имя папки не совпадает с `component`; нет `version.php`; нет строки `pluginname`; у веб-сервера нет прав чтения на папку.

> [!question] После создания класса в `classes/` — «Class not found». Почему?
> 
> Карта классов закэширована. Сбросить кэши `php admin/cli/purge_caches.php`. Также проверить namespace и регистр букв в пути.

> [!question] Когда поднимать версию плагина?
> 
> Когда изменилась схема БД, `access.php`, определения событий, задач, служб, кэшей, то есть всё, что читается при установке или обновлении. Без этого изменения не применятся.

---

## 7. База данных

### Теория

- Только через **`$DB`** (класс `moodle_database`). `PDO`, `mysqli_*`, `pg_*` напрямую — **запрещено**. `$DB` даёт префиксы, защиту от инъекций, совместимость между СУБД, транзакции.
- Таблицы пишут в фигурных скобках: `{user}`. Префикс (`mdl_`) подставляется сам.
- Параметры — **плейсхолдеры**: именованные `:name` или `?`. **Не смешивать** в одном запросе.
- Основные методы:

|Метод|Что делает|
|:--|:--|
|`get_record($table, $cond, $fields, MUST_EXIST)`|одна запись|
|`get_records(...)`|массив записей, ключ = первая колонка (обычно `id`)|
|`get_records_sql($sql, $params)`|запрос на SQL|
|`get_field`, `count_records`, `record_exists`|значение, количество, проверка|
|`get_recordset(...)`|потоковое чтение больших выборок|
|`insert_record`, `update_record` (нужен `id`), `set_field`|запись|
|`delete_records`|удаление|

- `MUST_EXIST` — выбросить исключение, если записи нет. `IGNORE_MISSING` — вернуть `false`.
- Если в `get_records_sql` первая колонка не уникальна, **записи перезапишутся**. Первой колонкой ставят уникальную (`id`).
- Кроссплатформенные куски SQL: `$DB->sql_like()`, `$DB->sql_concat()`, `$DB->get_in_or_equal()`.
- **Транзакции:** `$tx = $DB->start_delegated_transaction(); ... $tx->allow_commit();`
- **Схема:**
    - `db/install.xml` — **актуальная** схема, применяется при первой установке сайта или плагина.
    - `db/upgrade.php` — шаги для **существующих** баз (`$dbman = $DB->get_manager()`, `add_field`, `field_exists`) и обязательный `upgrade_plugin_savepoint(true, версия, 'local', 'hello')`.

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

- Имена таблиц плагина начинаются с компонента: `local_hello_messages`.
- Постраничная выборка: у `get_records*` есть параметры `$limitfrom` и `$limitnum`, например `$DB->get_records_sql($sql, $params, 0, 20)`.
- Другие полезные методы: `$DB->execute($sql, $params)` (произвольный запрос без выборки), `get_records_select($table, $select, $params)`, `delete_records_select(...)`, `insert_record` возвращает **id новой записи**.
- Одно и то же имя именованного плейсхолдера лучше не повторять в запросе. Если нужно несколько `IN`, имена получают через `$DB->get_in_or_equal($ids, SQL_PARAMS_NAMED, 'prefix')` или `$DB->get_unique_param()`.
- Схему редактируют не руками в XML, а через XMLDB editor (`Site administration → Development → XMLDB editor`): он генерирует корректный `install.xml`. Индексы добавляют на поля, по которым часто ищут (`courseid`, `userid`).

### Вопросы и ответы

> [!question] Как безопасно работать с БД в Moodle?
> 
> Только через `$DB`, таблицы в `{}`, значения только через плейсхолдеры. Никакой конкатенации значений в SQL.

> [!question] В чём разница между `install.xml` и `upgrade.php`?
> 
> `install.xml` — итоговая схема для **новой** установки. `upgrade.php` — пошаговые изменения для **уже существующих** сайтов. Менять схему нужно в обоих местах и поднимать версию.

> [!question] Что такое N+1 и как избежать?
> 
> Запрос к БД внутри цикла (`get_record` на каждого пользователя). Решение: один запрос с `JOIN` или `IN`, либо `get_fast_modinfo()` для структуры курса.

> [!question] Чем `get_records` опасен?
> 
> Ключ массива — первая колонка. Если она не уникальна, часть данных потеряется.

> [!question] Как сделать постраничный вывод и безопасно взять большой объём данных?
> 
> Для страницы — `LIMIT` через параметры `$limitfrom` и `$limitnum`. Для больших выборок — `get_recordset()`: он читает потоком и не грузит всё в память, после работы обязательно `$rs->close()`.

---

## 8. Параметры и безопасность

### Теория

- Данные из запроса берут **только** через `required_param()` / `optional_param()` с типом `PARAM_*` (`PARAM_INT`, `PARAM_TEXT`, `PARAM_ALPHANUMEXT`, `PARAM_URL`, `PARAM_BOOL` ...). `$_GET`/`$_POST` напрямую не используют.
- **SQL-инъекции** → плейсхолдеры.
- **XSS** → экранировать вывод: `s()`, `format_string()`, `format_text()`. В Mustache `{{ }}` экранирует, `{{{ }}}` — нет (использовать осторожно).
- **CSRF** → sesskey: в форме/ссылке `sesskey()`, на сервере `require_sesskey()`. Формы `moodleform` делают это сами.
- **Доступ** → `require_login()` + `require_capability()` в верном контексте.
- **Файлы** → отдача только через `pluginfile` с проверками.

### Вопросы и ответы

> [!question] Как в Moodle защищаются от основных атак?
> 
> SQL-инъекции: `$DB` и плейсхолдеры. XSS: `PARAM_*` на входе и экранирование на выходе. CSRF: sesskey. Доступ: `require_login` и capabilities. Файлы: отдача через PHP после проверки.

> [!question] Чем `PARAM_TEXT` отличается от `PARAM_RAW`?
> 
> `PARAM_RAW` не чистит ничего (опасно, только если дальше будет очистка). `PARAM_TEXT` убирает теги, оставляя безопасный текст. Всегда выбирай самый узкий тип.

---

## 9. Формы, вывод, строки, настройки

### Теория

**Формы (`moodleform`):**

- Класс наследует `moodleform` (файл `lib/formslib.php`). Поля в `definition()` через `$this->_form`.
- `validation($data, $files)` возвращает массив ошибок.
- Использование: `$form->is_cancelled()`, `$form->get_data()` (null, пока не отправлена или не прошла валидацию), `$form->set_data(...)`.
- Защита от CSRF встроена.

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

**Вывод:**

- Простой вывод — `$OUTPUT`.
- Правильный путь: **renderer** (`plugin_renderer_base`, `classes/output/renderer.php`) + **Mustache-шаблон** (`templates/*.mustache`).
- Данные для шаблона готовит класс, реализующий `renderable` и `templatable` (метод `export_for_template`).
- Вызов: `$OUTPUT->render_from_template('local_hello/card', $data)`.
- В шаблоне: `{{переменная}}`, `{{#список}}...{{/список}}`, `{{#str}}ключ, компонент{{/str}}`.
- Логика в PHP, а шаблон только отображает.

**Языковые строки:**

- Файл `lang/en/local_hello.php`, массив `$string['key'] = '...'`. Обязательна `pluginname`.
- Вывод: `get_string('key', 'local_hello', $a)`; параметр подставляется в `{$a}`.
- Другие языки подключаются через AMOS (в плагине достаточно `en`).
- **Никакого текста для пользователя прямо в коде.**

**Настройки плагина (`settings.php`):**

- Страница админки: `admin_settingpage`, поля `admin_setting_configtext` и др., добавление `$ADMIN->add('localplugins', $settings)` внутри `if ($hassiteconfig)`.
- Чтение: `get_config('local_hello', 'ключ')`, запись: `set_config(...)`. Хранится в `config_plugins`.

**Вспомогательные функции вывода и навигации:**

- `new moodle_url('/local/hello/index.php', ['id' => $id])` — построение ссылок, а не конкатенация строк.
- `redirect($url, 'Сохранено', null, \core\output\notification::NOTIFY_SUCCESS);` — редирект с сообщением. `\core\notification::add(...)` — сообщение, которое покажется на следующей странице.
- `html_writer::tag()`, `html_writer::link()` — безопасная сборка HTML там, где шаблон избыточен.
- Ошибки: `throw new \moodle_exception('код', 'local_hello', $returnurl)`. Текст берётся из языковой строки с этим ключом.

**Таблицы, отчёты, динамические формы:**

- Таблицы с сортировкой и постраничным выводом — `table_sql` и `flexible_table` (`lib/tablelib.php`). Вручную `<table>` в цикле не строят.
- **Report builder** (с 4.0) — конструктор отчётов: плагин описывает источник данных (`datasource`) или системный отчёт, а администратор собирает отчёты в интерфейсе. Для типовых отчётов лучше, чем писать страницу с нуля.
- **Dynamic forms** (`\core_form\dynamic_form`) — формы в модальном окне, которые работают через AJAX без перезагрузки страницы.

### Вопросы и ответы

> [!question] Как построить страницу с формой?
> 
> Класс формы наследует `moodleform` → в скрипте создать форму → `if ($form->is_cancelled())` редирект → `elseif ($data = $form->get_data())` сохранить через `$DB` → иначе `$form->display()` между `header()` и `footer()`.

> [!question] Зачем renderer и Mustache, если можно `echo`?
> 
> Разделение логики и разметки, возможность переопределять вывод в темах, единый стиль и экранирование, использование тех же шаблонов из JavaScript.

> [!question] Как выводить текст на разных языках?
> 
> Через языковые строки: `get_string('ключ', 'компонент')`. Строки лежат в `lang/en/компонент.php`, переводы подключаются отдельно.

> [!question] Как вывести таблицу данных с сортировкой и пагинацией?
> 
> `table_sql` (или `flexible_table`) либо Report builder, если нужен настраиваемый отчёт. Свою вёрстку на `foreach` и `echo` не делают: теряется сортировка, экспорт, доступность и единый вид.

> [!question] Что такое dynamic form и когда она нужна?
> 
> Форма на базе `moodleform`, которая рендерится и отправляется через AJAX (часто в модальном окне). Подходит для быстрых правок без перехода на другую страницу. Проверка прав и контекста обязательна, как в external function.

---

## 10. События и cron-задачи

### Теория

**События (Events API):**

- Событие — класс в `classes/event/`, наследует `\core\event\base`. В `init()` задают `crud` (c/r/u/d), `edulevel`, `objecttable`.
- Отправка: `\local_hello\event\message_sent::create([...])->trigger();`
- Подписка (наблюдатель) в `db/events.php`:

```php
$observers = [
    [
        'eventname' => '\core\event\course_completed',
        'callback'  => '\local_courseaudit\observer::course_completed',
    ],
];
```

- Наблюдатели срабатывают **синхронно** в том же запросе. Поэтому внутри делают только быстрое, тяжёлое выносят в задачу.
- События попадают в логи и отчёты (стандартный лог-хранилище).

**Задачи cron:**

- **Scheduled task** — по расписанию. Класс наследует `\core\task\scheduled_task`, методы `get_name()` и `execute()`. Расписание в `db/tasks.php` (`minute`, `hour`, `day`...).
- **Adhoc task** — разовая задача из очереди. Наследует `\core\task\adhoc_task`, данные через `set_custom_data()`, постановка `\core\task\manager::queue_adhoc_task($task)`. При сбое Moodle повторяет позже.
- Cron на сервере запускается **каждую минуту** (`php admin/cli/cron.php`).
- Вручную: `php admin/cli/scheduled_task.php --execute='\local_hello\task\cleanup'`.

### Вопросы и ответы

> [!question] Зачем нужны события и чем они лучше правки ядра?
> 
> Это точка расширения: ядро объявляет, что что-то произошло, плагин подписывается и реагирует, не меняя ядро. События также пишутся в лог.

> [!question] Чем scheduled task отличается от adhoc?
> 
> Scheduled — повторяется по расписанию. Adhoc — одноразовая, ставится в очередь из кода (например из наблюдателя), с повтором при ошибке.

> [!question] Почему не отправлять письмо прямо в наблюдателе?
> 
> Наблюдатель работает синхронно: письмо замедлит ответ пользователю, а сбой почты может сломать запрос. Правильно: в наблюдателе быстрая операция + постановка adhoc-задачи.

> [!question] Cron не работает. Как это понять и починить?
> 
> В админке (`Site administration → Server → Tasks`) видно время последнего запуска и логи задач, плюс появляется предупреждение о том, что cron давно не запускался. Проверить, что в системном crontab каждую минуту вызывается `php admin/cli/cron.php`, что пользователь совпадает с владельцем `moodledata`, и что нет зависших задач (долгая задача блокирует себя и не стартует повторно).

---

## 11. Web services и JavaScript

### Теория

**Web services (external functions):**

- Функция описывается в `db/services.php`; класс в `classes/external/` наследует `\core_external\external_api` (в старых версиях `external_api`).
- Три метода: `execute_parameters()`, `execute()`, `execute_returns()`.
- В `execute()`: `validate_parameters` → `validate_context` → `require_capability` → логика.
- Флаги: `'ajax' => true` (вызов из JS), `'type' => 'read'/'write'`.
- Доступ внешних клиентов: протоколы REST/SOAP и **токены**.

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

**Внешний доступ (services и токены):**

- В `db/services.php` рядом с `$functions` описывают `$services` (набор функций под одним именем). Администратор включает web services и нужный протокол (REST), создаёт пользователя-интегратора с минимальными правами и **токен**.
- Вызов REST: `/webservice/rest/server.php?wstoken=...&wsfunction=local_hello_get_messages&moodlewsrestformat=json`.
- Токен — это пароль: передавать только по HTTPS, не хранить в репозитории, ограничивать права пользователя. Мобильное приложение работает поверх этого же слоя.

Каркас external-класса:

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

**JavaScript:**

- Исходники в `amd/src/*.js` (ES6-модули), сборка в `amd/build/` командой `npx grunt amd`.
- Подключение со страницы: `$PAGE->requires->js_call_amd('local_hello/main', 'init', [$args]);`
- Полезные модули: `core/ajax` (вызов external functions), `core/templates` (рендер Mustache на клиенте), `core/str` (строки), `core/notification`, `core/modal`.

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
> Объявить external function (`ajax => true`), в JS вызвать через `core/ajax`. Внутри функции обязательно проверить контекст и capability. **Подвох:** не писать «свой» PHP-скрипт, принимающий AJAX без sesskey и проверок.

> [!question] Изменил JS, а в браузере старое. Что делать?
> 
> Пересобрать `npx grunt amd` (нужны файлы в `amd/build`), сбросить кэши `purge_caches.php`, на dev отключить `cachejs`.

> [!question] Как внешняя система получит данные из Moodle?
> 
> Описать external function (и при необходимости сервис), включить web services и REST, создать выделенного пользователя с минимальными правами и токен. Функция сама проверяет контекст и capability, потому что токен даёт доступ от имени пользователя. Для массовых выгрузок учесть нагрузку и пагинацию.

---

## 12. Hooks, Files, Cache, Message, Privacy

### Теория

**Hooks и callbacks:**

- Старый способ: функции в `lib.php` с именем по шаблону `{component}_{callback}` (например `local_hello_extend_navigation`).
- Новый способ (**Hooks API**, с Moodle 4.3): класс хука в ядре, подписка в `db/hooks.php`, обработчик в `classes/hook_callbacks.php`. Постепенно заменяет callbacks.
- Если можно — используй Hooks.

**Files API:**

- Файл идентифицируется набором: `contextid, component, filearea, itemid, filepath, filename`.
- Хранится в `{files}`, содержимое на диске по `contenthash`.
- Работа: `get_file_storage()` → `create_file_from_string/pathname`, `get_area_files`.
- Отдача пользователю: функция `local_hello_pluginfile(...)` в `lib.php` → `require_login`, проверка прав → `send_stored_file()`.
- В формах: `filemanager`, `filepicker`; сначала файл попадает в **draft area**, затем сохраняется `file_save_draft_area_files()`.

**Cache API (MUC):**

- Определение в `db/caches.php`, режимы: **application** (общий), **session**, **request**.
- Использование: `$cache = cache::make('local_hello', 'имя'); $cache->get($key); $cache->set($key, $val);`
- Хранилища: файловое, Redis, Memcached. Сброс: `purge_caches.php`.

**Message API:**

- Тип сообщений объявляется в `db/messages.php`.
- Отправка: объект `\core\message\message` (`component`, `name`, `userfrom`, `userto`, `subject`, `fullmessage`, `fullmessagehtml`, `smallmessage`, `notification`) и `message_send($message)`.
- Канал (email, popup, mobile) выбирает пользователь в настройках. Низкоуровнево есть `email_to_user()`.

**Privacy API (GDPR):**

- Каждый плагин обязан иметь `classes/privacy/provider.php`.
- Не хранит личные данные → `null_provider`.
- Хранит → `metadata\provider` (описание данных) + реализация: найти контексты пользователя, экспортировать данные, удалить данные пользователя / всех в контексте.

### Вопросы и ответы

> [!question] Как отдать пользователю файл безопасно?
> 
> Через функцию `{component}_pluginfile`: проверить `require_login`, права в контексте, потом `send_stored_file`. Прямую ссылку на диск не отдаём.

> [!question] Зачем Moodle нужен Cache API, если есть PHP-кэш?
> 
> MUC абстрагирует хранилище: один код работает на файлах, Redis или Memcached, поддерживает режимы (общий, сессия, запрос) и централизованный сброс.

> [!question] Что такое Privacy API и почему он обязателен?
> 
> Это реализация требований GDPR: система должна уметь описать, какие данные пользователя хранит плагин, экспортировать и удалить их. Без провайдера плагин не проходит проверки.

> [!question] Чем Hooks API лучше callbacks в lib.php?
> 
> Хуки типизированы, лежат в классах (автозагрузка, не нужно грузить весь `lib.php`), легче тестировать и задавать порядок.

---

## 13. Модуль mod, блок block, другие плагины

### Теория

**Модуль активности (`mod`):**

- Обязательные части:
    - `lib.php`: `{mod}_add_instance`, `{mod}_update_instance`, `{mod}_delete_instance` (возвращает id / true) и `{mod}_supports()` (флаги `FEATURE_*`);
    - `mod_form.php` — форма настроек, наследует `moodleform_mod`;
    - `view.php` — главная страница активности (принимает **cmid**);
    - `db/install.xml` — таблица модуля (поля `course`, `name`, `intro`, `introformat`, `timemodified`);
    - `db/access.php` — права, в том числе `mod/{name}:addinstance` и `:view`;
    - `backup/moodle2/` — резервное копирование и восстановление.
- Поддержка завершения (completion), оценок (grade API), событий, логирования.

**Блок (`block`):**

- Класс `block_xxx extends block_base` в `block_xxx.php`.
- `init()` задаёт `$this->title`. `get_content()` заполняет `$this->content->text` и `->footer`.
- Дополнительно: `applicable_formats()` (где блок можно показывать), `has_config()`, `instance_allow_multiple()`, `specialization()`, форма настроек `edit_form.php`.
- Права: `block/xxx:addinstance`, `block/xxx:myaddinstance`.

**Резервное копирование активности:** в `backup/moodle2/` лежат классы `backup_{mod}_activity_task` и `restore_{mod}_activity_task` и описание структуры данных (какие таблицы и файлы входят в копию). Без них активность не переносится вместе с курсом.

**Другие типы:** `theme` (дочерняя тема от Boost, SCSS, шаблоны), `auth`, `enrol`, `format`, `report`, `filter`, `repository`, `qtype`, `availability` (ограничения доступа), `tool` (админ-инструменты), `editor`.

### Вопросы и ответы

> [!question] Какие функции обязательны для плагина-активности?
> 
> `add_instance`, `update_instance`, `delete_instance`, `supports`, плюс `mod_form.php`, `view.php`, схема БД и права. Без этого активность нельзя создать, изменить или удалить.

> [!question] Чем `local` отличается от `mod`?
> 
> `mod` — активность в курсе, имеет cm, настройки, backup, оценки. `local` — любая проектная доработка вне модели активностей: страницы, интеграции, фоновые обработчики.

> [!question] Как выбрать тип плагина под задачу?
> 
> Новая активность в курсе → `mod`; виджет сбоку → `block`; внешний вид → `theme`; вход через внешнюю систему → `auth`; способ записи на курс → `enrol`; отчёт → `report`; всё остальное проектное → `local`.

> [!question] Что такое подплагин и зачем он нужен?
> 
> Расширение внутри плагина-родителя, например способ сдачи работы в `mod_assign` (`assignsubmission_*`) или правило доступа к тесту (`quizaccess_*`). Родитель сам определяет точки расширения, а подплагины подключаются через `db/subplugins.json`.

---

## 14. Тесты, отладка, стиль, обновление

### Теория

**Тестирование:**

- **PHPUnit** (`tests/*_test.php`): класс наследует `\advanced_testcase`, вызывает `$this->resetAfterTest()`, данные создаёт через `$this->getDataGenerator()->create_user()/create_course()`.
- Запуск: `php admin/tool/phpunit/cli/init.php` (один раз), потом `vendor/bin/phpunit local/hello/tests/manager_test.php`.
- **Behat** (`tests/behat/*.feature`): сценарии на Gherkin, браузерные проверки, окружение Selenium (удобно в moodle-docker).

**Отладка и производительность:**

- `debugging('текст', DEBUG_DEVELOPER)`, `$CFG->debug`, `$CFG->perfdebug`, Xdebug, логи.
- Главный враг — **N+1 запросы**. Структуру курса брать через `get_fast_modinfo()`.

**Чек-лист «Moodle не видит изменения»:**

1. Менял класс, шаблон, `db/*.php` → `purge_caches.php`.
2. Менял схему или `access.php` → поднять версию и `upgrade.php`.
3. Менял `amd/src` → `npx grunt amd`.
4. Менял CSS/SCSS → включить `themedesignermode` или сбросить кэши.

**Стиль кода:**

- 4 пробела (без табов), закрывающий `?>` не ставится.
- В файле-скрипте без классов: `defined('MOODLE_INTERNAL') || die();`
- Проверка линтерами: **moodle-plugin-ci** (`phpcs`, `phpdoc`, `eslint`, `mustache lint` и др.).

**Безопасное обновление сайта:**

1. Режим обслуживания: `php admin/cli/maintenance.php --enable`
2. Бэкап БД **и moodledata**.
3. Заменить код и плагины на совместимые.
4. `php admin/cli/upgrade.php --non-interactive`
5. `php admin/cli/purge_caches.php`, затем `maintenance.php --disable`.

### Вопросы и ответы

> [!question] Как тестировать плагин?
> 
> Логика → PHPUnit с генераторами данных, пользовательские сценарии → Behat. Перед проверкой кода — `moodle-plugin-ci` (стиль, phpdoc, линтеры).

> [!question] Как правильно обновить Moodle?
> 
> Режим обслуживания → бэкап БД и `moodledata` → новый код и совместимые версии плагинов → `upgrade.php` → сброс кэшей → выключить обслуживание. Сначала проверить на копии.

> [!question] Страница работает медленно. Что проверить?
> 
> Не остался ли dev-режим в `config.php`; N+1 запросы; не прогревается ли кэш; тяжёлые операции в синхронном коде (перенести в adhoc); кэширование через MUC; настройки OPcache и Redis.

---

## 15. Интеграции, сессии, блокировки, инфраструктура

### Теория

- **Исключения:** `throw new \moodle_exception('код', 'local_hello', $returnurl, $a);`. Текст ошибки лежит в языковом файле под этим ключом. Необработанное исключение в странице покажет страницу ошибки (в dev-режиме со стеком), в adhoc-задаче означает «задача упала» и повтор позже.
- **HTTP-запросы наружу:** в новых версиях `\core\http_client` (на базе Guzzle) или старый класс `curl` из `lib/filelib.php`. Они учитывают прокси и защитные настройки Moodle (например, чёрные списки хостов и портов), которые защищают от SSRF. Не использовать `file_get_contents($url)` и «голый» `curl_*` с адресом от пользователя.
- **Lock API:** защита от параллельного выполнения.

```php
$factory = \core\lock\lock_config::get_lock_factory('local_hello');
if ($lock = $factory->get_lock('sync_users', 10)) {
    try {
        // работа, которую нельзя запускать параллельно
    } finally {
        $lock->release();
    }
} else {
    // не удалось получить блокировку за 10 секунд
}
```

- **Сессии:** хранятся в файлах, БД или Redis. PHP-сессия блокируется на время запроса, поэтому долгая страница тормозит остальные запросы того же пользователя. В долгой операции после чтения нужных данных вызывают `\core\session\manager::write_close()`. После этого писать в `$SESSION` уже нельзя.
- **Внедрение зависимостей:** в новых версиях (с 4.4) есть DI-контейнер: `\core\di::get(\some\service::class)`. Нужен, чтобы не тянуть глобальные объекты в классы.
- **Маршрутизация:** в Moodle 5.x добавлен новый роутер для новых точек входа, но большинство страниц по-прежнему отдельные скрипты. Детали сверяй на moodledev.io.
- **Логи:** события пишутся в хранилище логов (`logstore_standard`), для CLI и cron есть `mtrace()`, для предупреждений `debugging()`. Логи задач смотрят в админке (раздел Server → Tasks).
- **Единый вход (SSO):** плагины `auth_*`: `auth_oauth2` (Google, Microsoft, свой провайдер), `auth_ldap`, SAML (обычно сторонний `auth_saml2`), `auth_lti`.
- **Мобильное приложение:** работает поверх web services. Для поддержки своего плагина в приложении добавляют `db/mobile.php` и специальные шаблоны.

### Вопросы и ответы

> [!question] Как из плагина безопасно вызвать внешний API?
> 
> Через `\core\http_client` или `curl`, а не напрямую. Ключи хранить в настройках плагина (`get_config`, поле-пароль в `settings.php`), вызов выносить в adhoc или scheduled-задачу, а не в синхронный код страницы. Задать таймаут, обработать ошибки, не писать секреты в лог.

> [!question] Пользователь запустил долгую страницу, остальные вкладки «зависли». Почему?
> 
> Сессия блокируется на время запроса. Решение: прочитать нужное из сессии и вызвать `\core\session\manager::write_close()`, либо вынести тяжёлую работу в фоновую задачу.

> [!question] Как не допустить параллельного запуска синхронизации?
> 
> Lock API (`lock_config::get_lock_factory`) с понятным ключом и обязательным `release()` в `finally`. Scheduled-задачи ядро само не запускает параллельно, но ручной CLI-запуск или два пути вызова кода это не закрывают.

> [!question] Как организовать вход через корпоративный SSO?
> 
> Плагин `auth_*` под протокол: OAuth2 (`auth_oauth2`), LDAP, SAML (чаще через сторонний `auth_saml2`). Настройка сопоставления полей профиля и ролей. Свой плагин `auth` пишут, только если готовых нет.

---

## 16. Чужой код, совместимость, вклад в проект

### Теория

- **Где искать:** moodledev.io (документация для разработчиков, API, upgrade notes), docs.moodle.org (для администраторов), Tracker (tracker.moodle.org, задачи `MDL-12345`), каталог плагинов moodle.org/plugins.
- **Как разобраться в незнакомом коде:** искать по ключу языковой строки, имени capability, имени события; смотреть, как то же сделано в ядре (`mod_assign`, `mod_forum`, простой `mod_choice`, `block_html`). Для отладки страницы: debug-режим, `$CFG->perfdebug`, Xdebug, логи.
- **Как менять поведение без правки ядра:** события, хуки и callbacks, настройки и capabilities; для внешнего вида — переопределение renderer или шаблона в дочерней теме (класс `theme_xxx\output\core_renderer` наследует `\theme_boost\output\core_renderer`); `local`-плагин для логики.
- **Совместимость:** функции и классы помечаются **deprecated** и удаляются через несколько релизов. В dev-режиме об этом предупреждает `debugging()`, список изменений есть в upgrade notes на moodledev.io. Плагин под несколько версий Moodle ведут ветками в git (по ветке на версию) и указывают `$plugin->requires` и `$plugin->supported`.
- **Стиль и проверки:** стандарт кода Moodle (4 пробела, phpdoc, заголовок GPL и `@package`, `@copyright` в каждом файле), проверка через `moodle-plugin-ci` (phpcs, phpdoc, mustache, grunt, phpunit, behat).
- **Вклад в ядро:** задача в Tracker (`MDL-...`), патч в виде pull request с веткой по шаблону вида `MDL-12345-main-описание`, дальше code review, интеграция и тестирование командой Moodle.
- **Инструменты разработчика в админке:** `Site administration → Development` (сброс кэшей, отладка, XMLDB editor, Template library, генератор тестовых курсов `tool_generator`, тестовый клиент web services).

### Вопросы и ответы

> [!question] Нужно изменить поведение стороннего плагина. Что делать?
> 
> Не править файлы плагина на месте: любое обновление сотрёт правку. Сначала искать точку расширения (событие, хук, настройка). Если её нет, форк плагина с оформленным патчем и желательно заявкой автору или в его репозиторий. Правка должна быть воспроизводимой, например через Composer или отдельный git-репозиторий.

> [!question] Как найти в коде Moodle, где что-то происходит?
> 
> Взять видимый текст из интерфейса, найти его ключ в `lang/en/*.php`, затем по ключу найти место использования (`grep`). Для действия — смотреть события в логах, имя capability, ссылку на страницу и её скрипт. Для пошаговой отладки Xdebug.

> [!question] Что значит «функция deprecated» и как реагировать?
> 
> Функция устарела и будет удалена через несколько релизов. В dev-режиме Moodle выдаёт предупреждение. Нужно найти замену в upgrade notes и поправить плагин до перехода на версию, где её удалят.

---

## 17. Сценарные вопросы

> [!question] При завершении курса записать данные в свою таблицу аудита и отправить письмо с уникальным кодом. Как спроектировать?
> 
> 1. Плагин `local_courseaudit`.
> 2. В `db/install.xml` — таблица лога (`userid`, `courseid`, `certcode`, `timecreated`).
> 3. В `db/events.php` — наблюдатель события `\core\event\course_completed`.
> 4. В наблюдателе **только** быстрая вставка `$DB->insert_record()` и постановка adhoc-задачи (`set_custom_data` с `userid` и `courseid`, `queue_adhoc_task`).
> 5. В задаче: сформировать текст через `get_string()` и отправить через Message API (`message_send`) или `email_to_user()`. Если отправка не удалась, бросить исключение: тогда Moodle повторит задачу позже.
> 6. Добавить Privacy-провайдер (таблица хранит данные пользователя), права и тесты.

> [!question] Нужен свой отчёт «время активности студентов» по курсу. Как?
> 
> Плагин `report_` (или `local_` со страницей). Страница: `require_login($course)`, `require_capability('report/xxx:view', context_course::instance($courseid))`. Данные одним `JOIN`-запросом по логам с плейсхолдерами и фильтром по курсу, вывод через renderer + Mustache, большие выборки через `get_recordset`. Права, строки и Privacy — как у любого плагина.

> [!question] Нужно добавить пункт в навигацию курса только для преподавателей.
> 
> Callback `local_xxx_extend_navigation_course` (или современный хук), пункт добавляется только если `has_capability('local/xxx:view', $context)`. Права объявить в `access.php` для архетипа `editingteacher`.

> [!question] Нужна кнопка, которая сохраняет данные без перезагрузки страницы.
> 
> External function (`ajax => true`, с проверкой прав и контекста) + AMD-модуль на `core/ajax` + подключение через `js_call_amd`. Сообщение об ошибке — `core/notification`.

> [!question] Нужно регулярно синхронизировать пользователей и записи на курсы из внешней системы. Как?
> 
> Плагин `local_` или `tool_` со scheduled task (и CLI-скриптом для ручного запуска). Данные читать порциями (`get_recordset`), работу делать **идемпотентной** (повторный запуск не ломает данные). Пользователей создавать через API (`user_create_user()`), запись на курс через `enrol_user()` плагина записи, а не прямыми `INSERT`: так срабатывают события и проверки. Параллельный запуск закрыть Lock API, результаты и ошибки писать в лог.

> [!question] Moodle тормозит в пик (экзамены, тесты). Что проверять?
> 
> Не включён ли dev-режим. OPcache. Redis для сессий и MUC. Медленные запросы в БД (slow query log) и индексы. N+1 в своих плагинах. Блокировки сессий. Тяжёлые отчёты и письма вынести в adhoc-задачи. Cron на отдельном процессе или сервере. Общий `moodledata` при нескольких веб-серверах. Для проверки нагрузки есть генератор тестовых курсов `tool_generator`.

> [!question] После обновления Moodle сторонний плагин падает с ошибкой. Что делать?
> 
> Не править на проде. Воспроизвести на копии, включить debug-режим, прочитать стек. Проверить, что версия плагина поддерживает новую версию Moodle, и нет ли в логе предупреждений о deprecated функциях. Сбросить кэши, запустить `upgrade.php`. Обновить плагин до совместимой версии или временно откатиться на бэкап. Если плагин заброшен, форкнуть и исправить, оформив патч.

> [!question] Нужно изменить поведение ядра или чужого плагина, но править код нельзя. Как?
> 
> Искать точку расширения: событие, хук или callback, настройка, capability. Для внешнего вида — переопределить renderer или шаблон в дочерней теме. Если точки нет, свой `local`-плагин, а как крайний случай — форк с оформленным патчем и заявкой в Tracker. Правка ядра на проде делает любое обновление болезненным.

---

## 18. Как отвечать и что спросить

### Если опыта мало

- Говори честно, но с пониманием механики. Пример:
    
    > «Точное название функции сейчас не вспомню, но знаю, как это устроено: данные приходят через `required_param` с типом, права проверяются в контексте курса. Сигнатуру сверяю по moodledev.io или смотрю в `mod_assign` / `mod_forum`.»
    
- Каждый ответ привязывай к безопасности:
    
    - про SQL → `$DB` и плейсхолдеры;
    - про параметры → `required_param` и `PARAM_*`;
    - про права → контексты и `has_capability`;
    - про изменения → «через плагин, не правка ядра».
- Если не знаешь, называй, **где ищешь**: moodledev.io, код ядра, тесты, `moodle-docker`.
    
- Можно сравнить с Drupal (у тебя есть опыт): хуки/события ↔ Hooks/Events API, плагины ↔ модули, `$DB` ↔ Database API, шаблоны ↔ Twig/Mustache.
    

### Что спросить у интервьюера

1. Какая версия Moodle на проде, планируется ли переход на 5.x?
2. Есть ли правки ядра или всё в плагинах?
3. Есть ли CI (moodle-plugin-ci, phpcs) и PHPUnit/Behat?
4. Как устроена инфраструктура: Redis для кэшей, общая файловая система для moodledata?
5. Какие плагины самописные, а какие сторонние, и кто их обновляет при смене версии Moodle?
6. Как устроены релизы: есть тестовый стенд, как выкатываются обновления?

---

## 19. Шпаргалка

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
Шаблоны               ──► templates/*.mustache
JavaScript            ──► amd/src/ → amd/build/
Privacy               ──► classes/privacy/provider.php
Подплагины            ──► db/subplugins.json
Мобильное приложение  ──► db/mobile.php
External functions    ──► classes/external/*.php
События (классы)      ──► classes/event/*.php
Задачи (классы)       ──► classes/task/*.php
Обработчики хуков     ──► classes/hook_callbacks.php
Рендереры, шаблоны    ──► classes/output/, templates/
Backup/restore        ──► backup/moodle2/
```

### Пять золотых правил

1. **Ядро не трогаем.** Любая задача решается плагином нужного типа.
2. **Права по контексту.** `has_capability` / `require_capability` в правильном контексте, не по имени роли.
3. **Безопасная работа с данными.** Только `$DB` и плейсхолдеры; вход — `required_param` + `PARAM_*`; выход — экранирование.
4. **Изменил схему или `access.php` — подними версию и запусти `upgrade.php`.**
5. **Тяжёлое — в фон.** Письма, пересчёты, интеграции — через adhoc-задачи, а не в запросе пользователя.

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
- Имя права записано с `_` вместо `/`: правильно `local/hello:view`.
- Долгая операция держит блокировку сессии и тормозит остальные вкладки пользователя.
- Прямые `INSERT` в таблицы записей и пользователей вместо API (теряются события).
- `{{{ }}}` в Mustache с пользовательскими данными без очистки.
- Один и тот же именованный плейсхолдер повторяется в запросе.
- Не вызван `$PAGE->set_url()`: возникают предупреждения и сломанные ссылки.
- Правка стороннего кода «на месте» вместо точки расширения.