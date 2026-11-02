---
custom-width: 80
---
---
# Moodle для PHP-разработчика

> [!info] 
> О конспекте Конспект построен от простого к сложному: сначала что такое Moodle и как он устроен, потом ядро API, потом плагины и каждая подсистема по отдельности. В конце есть сквозной пример и шпаргалка. Версии: примеры рассчитаны на Moodle 4.x и 5.x. В 5.0 поменялась структура каталогов (появился `public/`), и API постоянно эволюционирует, поэтому всё спорное сверяй с официальной документацией разработчика: **https://moodledev.io**.

## Оглавление

1. [[#1. Что такое Moodle]]
2. [[#2. Установка и окружение для разработки]]
3. [[#3. Структура каталогов]]
4. [[#4. Как Moodle обрабатывает запрос]]
5. [[#5. Ключевые сущности предметной области]]
6. [[#6. Глобальные объекты и базовые функции]]
7. [[#7. Плагины: типы и именование]]
8. [[#8. Анатомия плагина]]
9. [[#9. Первый плагин local_hello шаг за шагом]]
10. [[#10. Работа с базой данных]]
11. [[#11. Параметры, безопасность, права доступа]]
12. [[#12. Формы moodleform]]
13. [[#13. Вывод: страница, renderer, Mustache]]
14. [[#14. Языковые строки]]
15. [[#15. Настройки плагина]]
16. [[#16. События и наблюдатели]]
17. [[#17. Задачи по расписанию и фоновые задачи]]
18. [[#18. Веб-сервисы и external functions]]
19. [[#19. JavaScript в Moodle AMD ES6]]
20. [[#20. Hooks и callbacks в lib.php]]
21. [[#21. Файлы Files API]]
22. [[#22. Кэш Cache API]]
23. [[#23. Сообщения и уведомления]]
24. [[#24. Privacy API]]
25. [[#25. Модуль активности mod]]
26. [[#26. Блок block]]
27. [[#27. Другие типы плагинов коротко]]
28. [[#28. Тестирование PHPUnit и Behat]]
29. [[#29. Отладка и инструменты]]
30. [[#30. Стиль кода и публикация]]
31. [[#31. Типичные ошибки]]
32. [[#32. Шпаргалка]]

---

## 1. Что такое Moodle

**Moodle** это открытая LMS (Learning Management System, система управления обучением): курсы, задания, тесты, форумы, оценки, записи на курсы, отчёты.

- Язык: **PHP** (процедурный код в основе, постепенно переходит на классы, пространства имён, Mustache, hooks).
- БД: MySQL/MariaDB, PostgreSQL, MS SQL Server (разные уровни поддержки). Доступ **только через свой слой абстракции** `$DB`.
- Лицензия: GPL v3.
- Расширяется **плагинами** (их в каталоге тысячи): ядро сам Moodle тоже построен из плагинов (форум, задание, тест это плагины типа `mod`).
- Фронтенд: Mustache-шаблоны + JavaScript (AMD/ES6 модули), темы на Bootstrap.

> [!tip] Главная идея 
> В Moodle **ядро не правят**. Всё своё делается плагином, который лежит в нужной папке и подчиняется строгим соглашениям об именах и файлах. Moodle сам находит плагин, ставит его, создаёт таблицы, подключает права и строки.

---

## 2. Установка и окружение для разработки

### Что нужно

- PHP (версия зависит от Moodle, смотри требования релиза), с расширениями: `intl`, `mbstring`, `xml`, `zip`, `gd`, `curl`, `soap`, `sodium`, `opcache` и др.
- Веб-сервер (Apache/nginx) или встроенный для разработки.
- СУБД (PostgreSQL или MariaDB удобнее всего).
- Git, Composer, Node.js (для сборки JS).

### Способ 1. Docker от Moodle HQ (проще всего)

Репозиторий `moodlehq/moodle-docker` поднимает веб, БД, Selenium для Behat и т.д.

```bash
git clone https://github.com/moodle/moodle.git
export MOODLE_DOCKER_WWWROOT=/путь/к/moodle
export MOODLE_DOCKER_DB=pgsql
git clone https://github.com/moodlehq/moodle-docker.git
cd moodle-docker
cp config.docker-template.php $MOODLE_DOCKER_WWWROOT/config.php
bin/moodle-docker-compose up -d
bin/moodle-docker-wait-for-db
```

### Способ 2. Вручную

1. Скачать код: `git clone -b MOODLE_405_STABLE https://github.com/moodle/moodle.git` (ветка зависит от версии).
2. Создать каталог **moodledata** (вне веб-корня, с правами записи для веб-сервера): там лежат загруженные файлы, кэш, сессии, временные файлы.
3. Создать пустую БД (UTF-8).
4. Запустить установщик в браузере или через CLI:

```bash
php admin/cli/install.php --lang=ru --wwwroot=http://localhost:8080 \
  --dataroot=/var/moodledata --dbtype=pgsql --dbname=moodle \
  --dbuser=moodle --dbpass=secret --fullname="Test" --shortname="test" \
  --adminuser=admin --adminpass='Admin123!' --agree-license --non-interactive
```

> [!note] Путь к CLI-скриптам 
> В 5.x часть кода переехала в `public/`, расположение CLI-скриптов (`admin/cli/...`) проверяй в своей версии.

### config.php: настройки разработчика

```php
<?php
unset($CFG);
global $CFG;
$CFG = new stdClass();

$CFG->dbtype    = 'pgsql';
$CFG->dbhost    = 'localhost';
$CFG->dbname    = 'moodle';
$CFG->dbuser    = 'moodle';
$CFG->dbpass    = 'secret';
$CFG->prefix    = 'mdl_';          // префикс таблиц
$CFG->wwwroot   = 'http://localhost:8080';
$CFG->dataroot  = '/var/moodledata';
$CFG->admin     = 'admin';

// ---- Режим разработки ----
$CFG->debug = (E_ALL | E_STRICT);  // показывать все ошибки
$CFG->debugdisplay = 1;
$CFG->cachejs = false;             // не кэшировать JS
$CFG->langstringcache = false;     // не кэшировать строки
$CFG->themedesignermode = true;    // не кэшировать CSS/шаблоны темы
$CFG->perfdebug = 15;              // инфо о производительности внизу страницы

require_once(__DIR__ . '/lib/setup.php'); // ВСЕГДА последней строкой
```

### Полезные CLI-команды

|Команда|Что делает|
|---|---|
|`php admin/cli/upgrade.php`|Обновление БД после добавления/изменения плагина|
|`php admin/cli/purge_caches.php`|Сброс всех кэшей (самая частая команда)|
|`php admin/cli/cron.php`|Запуск cron вручную|
|`php admin/cli/scheduled_task.php --execute='\local_hello\task\cleanup'`|Выполнить одну задачу|
|`php admin/cli/adhoc_task.php --execute`|Выполнить фоновые (adhoc) задачи|

---

## 3. Структура каталогов

Классическая раскладка (Moodle 4.x), корень проекта = `dirroot`:

```
moodle/
├── config.php             ← настройки (создаётся при установке)
├── version.php            ← версия самого ядра
├── admin/                 ← админка; admin/tool/ плагины-утилиты
├── auth/                  ← плагины аутентификации
├── blocks/                ← блоки
├── course/                ← код курсов; course/format/ форматы курсов
├── enrol/                 ← способы записи на курс
├── filter/                ← фильтры текста
├── grade/                 ← журнал оценок; grade/report/ отчёты
├── lib/                   ← библиотеки ядра (setup.php, dml/, form/, classes/...)
├── local/                 ← ТВОИ локальные плагины (универсальный тип)
├── mod/                   ← модули активностей (форум, тест, задание)
├── question/              ← банк вопросов и типы вопросов
├── report/                ← отчёты
├── repository/            ← репозитории файлов
├── theme/                 ← темы оформления
├── user/                  ← профили пользователей
└── ...
```

> [!warning] Moodle 5.0 и новее 
> Появилась папка **`public/`**: веб-доступный код переехал внутрь неё (`public/local/`, `public/mod/`, `public/lib/` и т.д.), а в корне остались `config.php`, `vendor/`, конфигурация и код, который не должен быть доступен из веба. Плагины теперь ставятся в `public/<тип>/<имя>`. Принципы разработки те же, меняются только пути.

### Что где в каждой типичной папке плагина

Внутри плагина (например, `local/hello/`) структура одинаковая для всех типов, об этом в разделе [[#8. Анатомия плагина]].

---

## 4. Как Moodle обрабатывает запрос

Почти каждый PHP-скрипт Moodle начинается так:

```php
require_once(__DIR__ . '/../../config.php');
```

Дальше цепочка такая:

1. `config.php` читает настройки и в конце подключает `lib/setup.php`.
2. `lib/setup.php`: подключается к БД, поднимает сессию, загружает кэши, определяет текущего пользователя, создаёт глобальные объекты (`$CFG`, `$DB`, `$USER`, `$PAGE`, `$OUTPUT`, `$SESSION`, `$SITE`).
3. Твой скрипт проверяет вход и права (`require_login()`, `require_capability()`).
4. Настраивает страницу (`$PAGE`).
5. Выводит `$OUTPUT->header()`, содержимое, `$OUTPUT->footer()`.

> [!tip] Идея Moodle по-прежнему «много-входной»: у каждой страницы свой `.php`-файл (`/local/hello/index.php`, `/mod/forum/view.php`). Классического единого роутера нет (в 4.5/5.0 добавили новый routing на базе Slim для части маршрутов, но для обычных плагинов отдельные страницы остаются нормой).

---

## 5. Ключевые сущности предметной области

|Сущность|Что это|
|---|---|
|**Site**|Весь сайт Moodle (`$SITE`, это тоже запись курса с id=1)|
|**Category**|Категория курсов (дерево)|
|**Course**|Курс: у него разделы и активности|
|**Section**|Раздел (тема/неделя) курса|
|**Activity / Module**|Активность в курсе: форум, тест, задание. Это экземпляр плагина типа `mod`|
|**Course module (cm)**|Привязка экземпляра активности к курсу и разделу (таблица `course_modules`)|
|**Resource**|Ресурс (файл, страница, папка), по сути тоже `mod`|
|**User**|Пользователь (`$USER`)|
|**Enrolment**|Запись пользователя на курс (через плагин `enrol_*`)|
|**Role**|Роль (студент, преподаватель, менеджер, админ)|
|**Capability**|Конкретное право, например `mod/forum:replypost`|
|**Context**|Область действия прав (см. ниже)|
|**Grade**|Оценка (журнал оценок, gradebook)|
|**Cohort**|Глобальная группа пользователей|
|**Group / Grouping**|Группа внутри курса|

### Контексты (очень важно)

**Контекст** определяет, где действует право. Они образуют иерархию:

```
CONTEXT_SYSTEM (весь сайт)
 └─ CONTEXT_COURSECAT (категория)
     └─ CONTEXT_COURSE (курс)
         └─ CONTEXT_MODULE (активность)
             └─ CONTEXT_BLOCK (блок)
CONTEXT_USER (профиль пользователя) — отдельная ветка
```

Роль назначается **в контексте**, права наследуются вниз по дереву (если не переопределены).

```php
$context = context_system::instance();
$context = context_course::instance($courseid);
$context = context_module::instance($cmid);
$context = context_user::instance($userid);
```

### Права доступа: роли и capabilities

- **Capability** (право) объявляется в `db/access.php` плагина.
- Роли получают право (`CAP_ALLOW`), запрет (`CAP_PREVENT`), жёсткий запрет (`CAP_PROHIBIT`) или ничего (`CAP_INHERIT`).
- В коде проверяют **право, а не роль**:

```php
if (has_capability('local/hello:manage', $context)) { ... }  // вернёт true/false
require_capability('local/hello:manage', $context);           // выбросит исключение, если нет права
```

---

## 6. Глобальные объекты и базовые функции

В Moodle есть набор глобалов. В функциях их надо объявлять: `global $DB, $USER;`.

|Объект|Назначение|
|---|---|
|`$CFG`|Конфигурация: `$CFG->wwwroot`, `$CFG->dirroot`, `$CFG->dataroot`, `$CFG->libdir`|
|`$DB`|Доступ к БД (DML)|
|`$USER`|Текущий пользователь (`$USER->id`, `->username`, `->email`)|
|`$PAGE`|Настройки текущей страницы (заголовок, URL, контекст, JS, навигация)|
|`$OUTPUT`|Рендеринг HTML (шапка, подвал, уведомления, шаблоны)|
|`$COURSE`|Текущий курс (после `require_login($course)`)|
|`$SITE`|Курс уровня сайта (id=1)|
|`$SESSION`|Данные сессии|

Полезные функции:

```php
$url = new moodle_url('/local/hello/index.php', ['id' => 5]);   // построение URL
echo html_writer::link($url, 'Ссылка');                          // HTML-ссылка
redirect($url, 'Сохранено', null, \core\output\notification::NOTIFY_SUCCESS);

$course = get_course($courseid);                // объект курса
$modinfo = get_fast_modinfo($course);           // быстрый доступ к активностям курса
is_siteadmin();                                 // админ ли пользователь
isloggedin();                                   // вошёл ли
is_enrolled($context, $user);                   // записан ли на курс
format_string($str);                            // безопасный вывод заголовков
format_text($text, FORMAT_HTML);                // безопасный вывод HTML (применит фильтры)
s($text);                                       // экранирование (аналог htmlspecialchars)
get_string('key', 'component');                 // языковая строка
userdate(time());                               // дата с учётом часового пояса пользователя
```

---

## 7. Плагины: типы и именование

### Frankenstyle (именование)

Каждый плагин имеет имя вида **`тип_имя`**, называемое **frankenstyle**, например `local_hello`, `mod_forum`, `block_myblock`. Это:

- имя компонента (`component`) во всех API (строки, права, события, кэши, таблицы);
- префикс для функций и таблиц;
- пространство имён PHP-классов (`namespace local_hello;`).

Правила:

- имя: маленькие латинские буквы и цифры, начинается с буквы; **без подчёркиваний** внутри самого «имени» (кроме подтипов);
- таблицы: `local_hello_...` (**не длиннее 28 символов** без префикса `mdl_`);
- функции в `lib.php`: `local_hello_имя()`; для `mod`: `hello_имя()` (без `mod_`).

### Основные типы плагинов

|Тип|Папка|Назначение|
|---|---|---|
|`mod`|`mod/`|Активности и ресурсы в курсе (форум, тест)|
|`block`|`blocks/`|Блоки в боковой панели|
|`local`|`local/`|**Универсальный тип**: любые доработки, не вписывающиеся в другие типы|
|`theme`|`theme/`|Темы оформления|
|`auth`|`auth/`|Способы аутентификации (LDAP, OAuth2)|
|`enrol`|`enrol/`|Способы записи на курс|
|`filter`|`filter/`|Фильтры текста (LaTeX, автоссылки)|
|`format`|`course/format/`|Форматы курсов (темы, недели)|
|`report`|`report/`|Отчёты|
|`tool`|`admin/tool/`|Административные утилиты|
|`qtype`|`question/type/`|Типы вопросов|
|`repository`|`repository/`|Источники файлов|
|`editor` / `tiny`|`lib/editor/...`|Редакторы текста и их плагины|
|`assignsubmission`, `assignfeedback`|`mod/assign/...`|Подплагины задания|
|`gradereport`|`grade/report/`|Отчёты журнала оценок|
|`availability`|`availability/condition/`|Условия доступа к активностям|
|`message`|`message/output/`|Каналы доставки сообщений|
|`webservice`|`webservice/`|Протоколы веб-сервисов|
|`cachestore`|`cache/stores/`|Хранилища кэша|

> [!tip] С чего начинать Для 80% задач хватает `local`, `block`, `mod` и `tool`. Когда не знаешь, какой тип выбрать: если нужна **новая активность в курсе** то `mod`, **блок на странице** то `block`, **всё остальное** (новые страницы, интеграции, обработчики событий, веб-сервисы) то `local`.

---

## 8. Анатомия плагина

Типичное содержимое (не все файлы обязательны):

```
local/hello/
├── version.php            ← ОБЯЗАТЕЛЬНО: версия и метаданные
├── lang/
│   └── en/
│       └── local_hello.php ← ОБЯЗАТЕЛЬНО: языковые строки (минимум pluginname)
├── lib.php                ← функции-callbacks (старый стиль) и общие функции
├── settings.php           ← страница настроек в админке
├── index.php              ← страница плагина (твой скрипт)
├── classes/               ← PHP-классы с автозагрузкой (namespace local_hello\...)
│   ├── form/              ← формы
│   ├── output/            ← renderer и renderable
│   ├── external/          ← функции веб-сервисов
│   ├── event/             ← события
│   ├── task/              ← задачи cron
│   ├── privacy/provider.php ← ОБЯЗАТЕЛЬНО (Privacy API)
│   └── hook_callbacks.php ← обработчики hooks
├── db/
│   ├── install.xml        ← схема таблиц
│   ├── upgrade.php        ← миграции
│   ├── access.php         ← права (capabilities)
│   ├── events.php         ← подписка на события
│   ├── tasks.php          ← расписание cron
│   ├── services.php       ← веб-сервисы
│   ├── caches.php         ← определения кэшей
│   ├── hooks.php          ← подписки на hooks
│   └── messages.php       ← типы сообщений
├── templates/             ← Mustache-шаблоны (*.mustache)
├── amd/
│   ├── src/               ← исходники JS
│   └── build/             ← собранный JS (*.min.js)
├── tests/                 ← PHPUnit и Behat
├── pix/                   ← иконки
└── styles.css             ← CSS (для некоторых типов)
```

### version.php

```php
<?php
defined('MOODLE_INTERNAL') || die();

$plugin->component = 'local_hello';     // frankenstyle, должно совпадать с папкой
$plugin->version   = 2026110200;        // YYYYMMDDXX, при каждом изменении схемы ПОВЫШАЙ
$plugin->requires  = 2024100700;        // минимальная версия Moodle (build-номер релиза)
$plugin->maturity  = MATURITY_STABLE;   // ALPHA / BETA / RC / STABLE
$plugin->release   = '1.0.0';           // человекочитаемая версия
// $plugin->dependencies = ['mod_forum' => 2022112800];  // зависимости
```

> [!important] Как Moodle понимает, что плагин надо обновить Он сравнивает `$plugin->version` из файла с версией, сохранённой в БД. Больше число значит запустятся миграции из `db/upgrade.php`. Поэтому **после любой правки схемы БД или прав обязательно увеличивай `version`** и запускай `php admin/cli/upgrade.php` (или заходи на `/admin`).

### Автозагрузка классов

Папка `classes/` подключается автоматически по правилу:

|Класс|Файл|
|---|---|
|`\local_hello\manager`|`local/hello/classes/manager.php`|
|`\local_hello\form\message_form`|`local/hello/classes/form/message_form.php`|
|`\local_hello\task\cleanup`|`local/hello/classes/task/cleanup.php`|

Namespace всегда начинается с frankenstyle, остальное повторяет путь внутри `classes/`. После добавления нового класса **сбрасывай кэши** (карта автозагрузки кэшируется).

---

## 9. Первый плагин local_hello шаг за шагом

Цель: страница `/local/hello/index.php`, которая показывает приветствие и список сообщений из своей таблицы. Доступ по праву.

### Шаг 1. Папка и version.php

`local/hello/version.php`: как в разделе выше.

### Шаг 2. Языковые строки

`local/hello/lang/en/local_hello.php`:

```php
<?php
defined('MOODLE_INTERNAL') || die();

$string['pluginname'] = 'Hello';
$string['greeting']   = 'Hello, {$a}!';
$string['hello:view'] = 'View hello page';
$string['privacy:metadata'] = 'The Hello plugin does not store personal data.';
```

> [!note] Название права в строках Строка для права называется `имя_компонента:право` без префикса пути: для `local/hello:view` строка `$string['hello:view']`.

### Шаг 3. Право

`local/hello/db/access.php`:

```php
<?php
defined('MOODLE_INTERNAL') || die();

$capabilities = [
    'local/hello:view' => [
        'captype'      => 'read',
        'contextlevel' => CONTEXT_SYSTEM,
        'archetypes'   => [
            'user'    => CAP_ALLOW,   // все авторизованные
            'manager' => CAP_ALLOW,
        ],
    ],
];
```

### Шаг 4. Privacy (минимальный)

`local/hello/classes/privacy/provider.php`:

```php
<?php
namespace local_hello\privacy;

class provider implements \core_privacy\local\metadata\null_provider {
    public static function get_reason(): string {
        return 'privacy:metadata';
    }
}
```

### Шаг 5. Страница

`local/hello/index.php`:

```php
<?php
require_once(__DIR__ . '/../../config.php');

require_login();                                          // только вошедшие
$context = context_system::instance();
require_capability('local/hello:view', $context);         // проверка права

$PAGE->set_url(new moodle_url('/local/hello/index.php'));
$PAGE->set_context($context);
$PAGE->set_title(get_string('pluginname', 'local_hello'));
$PAGE->set_heading(get_string('pluginname', 'local_hello'));

echo $OUTPUT->header();
echo $OUTPUT->heading(get_string('greeting', 'local_hello', fullname($USER)));
echo $OUTPUT->footer();
```

### Шаг 6. Установка

1. Положить папку в `local/hello` (в 5.x это `public/local/hello`).
2. Зайти как админ: _Site administration → Notifications_ или выполнить `php admin/cli/upgrade.php`.
3. Подтвердить установку.
4. Открыть `http://localhost:8080/local/hello/index.php`.

> [!tip] Цикл разработки Изменил PHP: обновил страницу. Изменил строки, шаблоны, `db/*.php` или добавил класс: `php admin/cli/purge_caches.php`. Поднял `version`: `php admin/cli/upgrade.php`.

---

## 10. Работа с базой данных

### Основы

- Все обращения только через **`$DB`**, без `mysqli`/`PDO`.
- Имена таблиц пишутся **без префикса** (`'local_hello_msg'`, не `'mdl_local_hello_msg'`).
- Поле `id` (автоинкремент) в таблицах обязательно.
- Значения подставляются через **плейсхолдеры**, это защита от SQL-инъекций.

### Описание таблиц: `db/install.xml`

Руками XML лучше не писать: в админке есть **XMLDB-редактор** (_Site administration → Development → XMLDB editor_), он генерирует файл и код миграций. Пример результата:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<XMLDB PATH="local/hello/db" VERSION="2026110200"
       COMMENT="XMLDB file for Moodle local/hello"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:noNamespaceSchemaLocation="../../../lib/xmldb/xmldb.xsd">
  <TABLES>
    <TABLE NAME="local_hello_msg" COMMENT="Messages">
      <FIELDS>
        <FIELD NAME="id" TYPE="int" LENGTH="10" NOTNULL="true" SEQUENCE="true"/>
        <FIELD NAME="userid" TYPE="int" LENGTH="10" NOTNULL="true" DEFAULT="0" SEQUENCE="false"/>
        <FIELD NAME="message" TYPE="text" NOTNULL="true" SEQUENCE="false"/>
        <FIELD NAME="timecreated" TYPE="int" LENGTH="10" NOTNULL="true" DEFAULT="0" SEQUENCE="false"/>
      </FIELDS>
      <KEYS>
        <KEY NAME="primary" TYPE="primary" FIELDS="id"/>
        <KEY NAME="userid" TYPE="foreign" FIELDS="userid" REFTABLE="user" REFFIELDS="id"/>
      </KEYS>
      <INDEXES>
        <INDEX NAME="timecreated" UNIQUE="false" FIELDS="timecreated"/>
      </INDEXES>
    </TABLE>
  </TABLES>
</XMLDB>
```

`install.xml` выполняется **только при первой установке**. Для существующих сайтов изменения идут через `db/upgrade.php`.

### Миграции: `db/upgrade.php`

```php
<?php
defined('MOODLE_INTERNAL') || die();

function xmldb_local_hello_upgrade($oldversion) {
    global $DB;
    $dbman = $DB->get_manager();

    if ($oldversion < 2026110300) {
        $table = new xmldb_table('local_hello_msg');
        $field = new xmldb_field('status', XMLDB_TYPE_INTEGER, '2', null,
                                 XMLDB_NOTNULL, null, '0', 'message');

        if (!$dbman->field_exists($table, $field)) {
            $dbman->add_field($table, $field);
        }
        // Обязательная «точка сохранения»: записывает новую версию в БД
        upgrade_plugin_savepoint(true, 2026110300, 'local', 'hello');
    }

    return true;
}
```

Правило: для каждого изменения новый блок `if ($oldversion < НОВАЯ_ВЕРСИЯ)`, внутри `upgrade_plugin_savepoint`, и **такая же версия в `version.php`**.

### Основные методы `$DB`

```php
// ---- Чтение ----
$rec  = $DB->get_record('user', ['id' => 5]);                       // одна запись или false
$rec  = $DB->get_record('user', ['id' => 5], '*', MUST_EXIST);      // исключение, если нет
$recs = $DB->get_records('local_hello_msg', ['userid' => $USER->id], 'timecreated DESC');
$val  = $DB->get_field('user', 'email', ['id' => 5]);               // одно значение
$cnt  = $DB->count_records('local_hello_msg', ['userid' => 5]);
$has  = $DB->record_exists('local_hello_msg', ['id' => 3]);

// ---- SQL-запросы (с плейсхолдерами!) ----
$sql = "SELECT m.*, u.firstname
          FROM {local_hello_msg} m
          JOIN {user} u ON u.id = m.userid
         WHERE m.timecreated > :since";
$recs = $DB->get_records_sql($sql, ['since' => time() - DAY_IN_SECS]);
$rec  = $DB->get_record_sql($sql, $params);

// ---- Запись ----
$id = $DB->insert_record('local_hello_msg', (object)[
    'userid' => $USER->id, 'message' => 'Привет', 'timecreated' => time(),
]);
$DB->update_record('local_hello_msg', (object)['id' => $id, 'message' => 'Новое']);
$DB->delete_records('local_hello_msg', ['id' => $id]);

// ---- IN (...) ----
[$insql, $inparams] = $DB->get_in_or_equal([1, 2, 3], SQL_PARAMS_NAMED);
$recs = $DB->get_records_select('user', "id $insql", $inparams);
```

Нюансы:

- В SQL таблицы пишутся в фигурных скобках `{таблица}`, Moodle сам добавит префикс.
- Плейсхолдеры: `?` (порядковые) или `:имя` (именованные), смешивать нельзя.
- Первое поле в `get_records*` должно быть **уникальным**: оно становится ключом массива результатов (поэтому часто пишут `SELECT id, ...` или `DISTINCT` осторожно).
- Для больших выборок: `$DB->get_recordset(...)` (итератор, не грузит всё в память), закрывать через `$rs->close()`.
- Сравнение и LIKE кроссплатформенно: `$DB->sql_like('field', ':p', false)`, `$DB->sql_concat(...)`, `$DB->sql_compare_text(...)`.

### Транзакции

```php
$transaction = $DB->start_delegated_transaction();
try {
    // несколько операций...
    $transaction->allow_commit();
} catch (Exception $e) {
    $transaction->rollback($e);   // откат и повторный выброс исключения
}
```

---

## 11. Параметры, безопасность, права доступа

### Получение входных данных

**Никогда не используй `$_GET`/`$_POST` напрямую.**

```php
$id     = required_param('id', PARAM_INT);                 // обязательный параметр
$action = optional_param('action', 'view', PARAM_ALPHA);   // необязательный с дефолтом
$text   = optional_param('text', '', PARAM_TEXT);
```

Тип `PARAM_*` очищает значение. Частые:

|Константа|Что оставляет|
|---|---|
|`PARAM_INT`|целое число|
|`PARAM_FLOAT`|дробное|
|`PARAM_BOOL`|0/1|
|`PARAM_ALPHA`|только буквы|
|`PARAM_ALPHANUMEXT`|буквы, цифры, `_`, `-`|
|`PARAM_TEXT`|текст без HTML-тегов (кроме мультиязычных)|
|`PARAM_RAW`|без очистки (будь осторожен!)|
|`PARAM_URL`|валидный URL|
|`PARAM_EMAIL`|email|
|`PARAM_COMPONENT`|frankenstyle-имя|

### Защита страниц

```php
require_login();                              // вошёл ли пользователь
require_login($course, true, $cm);            // + доступ к курсу/активности
require_capability('local/hello:manage', $context);
require_sesskey();                            // защита от CSRF (для действий по ссылке/POST)
```

- В ссылках на опасные действия добавляй `sesskey`: `new moodle_url($url, ['sesskey' => sesskey(), 'delete' => $id])`.
- `moodleform` проверяет sesskey автоматически.

### Защита вывода (XSS)

|Что выводишь|Чем оборачивать|
|---|---|
|Короткая строка/заголовок|`format_string($s)`|
|Текст с HTML (из редактора)|`format_text($text, $format)`|
|Произвольная строка в атрибут/текст|`s($s)`|
|Шаблоны Mustache|`{{var}}` экранирует автоматически, `{{{var}}}` нет|

### Защита SQL

Только плейсхолдеры (`?` / `:name`), **никогда** конкатенация пользовательских данных в SQL.

---

## 12. Формы moodleform

Moodle имеет свой фреймворк форм (`moodleform`): рендер, валидация, sesskey, обработка файлов, редакторы.

`local/hello/classes/form/message_form.php`:

```php
<?php
namespace local_hello\form;

defined('MOODLE_INTERNAL') || die();
require_once($CFG->libdir . '/formslib.php');    // moodleform лежит тут

class message_form extends \moodleform {

    protected function definition() {
        $mform = $this->_form;

        $mform->addElement('text', 'message', get_string('message', 'local_hello'));
        $mform->setType('message', PARAM_TEXT);                 // ОБЯЗАТЕЛЬНО указать тип
        $mform->addRule('message', get_string('required'), 'required', null, 'client');
        $mform->addRule('message', null, 'maxlength', 255, 'client');

        $mform->addElement('hidden', 'id', $this->_customdata['id'] ?? 0);
        $mform->setType('id', PARAM_INT);

        $this->add_action_buttons();                            // Save + Cancel
    }

    // Серверная валидация
    public function validation($data, $files) {
        $errors = parent::validation($data, $files);
        if (trim($data['message']) === '') {
            $errors['message'] = get_string('required');
        }
        return $errors;
    }
}
```

Использование на странице:

```php
$mform = new \local_hello\form\message_form(null, ['id' => $id]);

if ($mform->is_cancelled()) {
    redirect(new moodle_url('/local/hello/index.php'));
} else if ($data = $mform->get_data()) {
    // $data->message уже очищено по setType
    $DB->insert_record('local_hello_msg', (object)[
        'userid' => $USER->id,
        'message' => $data->message,
        'timecreated' => time(),
    ]);
    redirect(new moodle_url('/local/hello/index.php'), 'Сохранено',
             null, \core\output\notification::NOTIFY_SUCCESS);
}

echo $OUTPUT->header();
$mform->display();
echo $OUTPUT->footer();
```

Типы элементов: `text`, `textarea`, `editor`, `select`, `advcheckbox`, `date_selector`, `date_time_selector`, `filemanager`, `filepicker`, `passwordunmask`, `static`, `header`, `hidden`, `autocomplete`, `duration`, `radio`, `group`.

Полезное: `$mform->setDefault(...)`, `$mform->addHelpButton(...)`, `$mform->hideIf(...)`, `$mform->disabledIf(...)`, `$mform->set_data($obj)` для предзаполнения.

---

## 13. Вывод: страница, renderer, Mustache

### Подготовка страницы `$PAGE`

```php
$PAGE->set_url($url);                       // ОБЯЗАТЕЛЬНО
$PAGE->set_context($context);               // ОБЯЗАТЕЛЬНО
$PAGE->set_title('Заголовок вкладки');
$PAGE->set_heading('Заголовок страницы');
$PAGE->set_pagelayout('standard');          // admin, course, incourse, base, popup...
$PAGE->navbar->add('Крошка', $url);         // хлебные крошки
$PAGE->requires->js_call_amd('local_hello/main', 'init', [$id]);   // JS
$PAGE->requires->css('/local/hello/extra.css');
```

### `$OUTPUT`: готовые элементы

```php
echo $OUTPUT->header();
echo $OUTPUT->heading('Текст', 2);
echo $OUTPUT->notification('Ошибка', \core\output\notification::NOTIFY_ERROR);
echo $OUTPUT->single_button($url, 'Нажми');
echo $OUTPUT->render_from_template('local_hello/message_list', $data);  // Mustache
echo $OUTPUT->footer();
```

### Mustache-шаблоны

Шаблон `local/hello/templates/message_list.mustache`:

```mustache
{{!
    @template local_hello/message_list

    Список сообщений.

    Example context (json):
    {
        "heading": "Messages",
        "messages": [
            {"text": "Hello", "date": "01.11.2026"},
            {"text": "World", "date": "02.11.2026"}
        ],
        "hasmessages": true
    }
}}
<div class="local-hello">
    <h3>{{heading}}</h3>
    {{#hasmessages}}
    <ul class="list-group">
        {{#messages}}
        <li class="list-group-item">
            {{text}} <small class="text-muted">{{date}}</small>
        </li>
        {{/messages}}
    </ul>
    {{/hasmessages}}
    {{^hasmessages}}
        <p>{{#str}} nomessages, local_hello {{/str}}</p>
    {{/hasmessages}}
</div>
```

Основные конструкции:

|Синтаксис|Смысл|
|---|---|
|`{{var}}`|вывод с экранированием|
|`{{{var}}}`|вывод HTML без экранирования (только для безопасного HTML)|
|`{{#list}}...{{/list}}`|цикл по массиву или условие (если истинно)|
|`{{^x}}...{{/x}}`|«если пусто/ложно»|
|`{{#str}} key, component {{/str}}`|языковая строка|
|`{{#pix}} i/edit, core {{/pix}}`|иконка|
|`{{> core/loading }}`|подключить другой шаблон (partial)|

> [!note] Особенность Mustache В Mustache нет логики. Всё, что нужно (`hasmessages`, готовые даты), подготавливается в PHP. Шаблон должен содержать docblock с `@template` и примером JSON, по нему строится документация и тесты.

### Renderer и renderable (правильная архитектура)

Подход Moodle: данные готовит класс `renderable`, а HTML делает `renderer`.

`classes/output/message_list.php` (данные):

```php
<?php
namespace local_hello\output;

use renderable;
use templatable;
use renderer_base;

class message_list implements renderable, templatable {
    private array $messages;

    public function __construct(array $messages) {
        $this->messages = $messages;
    }

    public function export_for_template(renderer_base $output): array {
        $items = [];
        foreach ($this->messages as $m) {
            $items[] = [
                'text' => format_string($m->message),
                'date' => userdate($m->timecreated),
            ];
        }
        return [
            'heading' => get_string('pluginname', 'local_hello'),
            'messages' => $items,
            'hasmessages' => !empty($items),
        ];
    }
}
```

`classes/output/renderer.php`:

```php
<?php
namespace local_hello\output;

class renderer extends \plugin_renderer_base {
    public function render_message_list(message_list $list) {
        return $this->render_from_template('local_hello/message_list',
            $list->export_for_template($this));
    }
}
```

Вызов:

```php
$renderer = $PAGE->get_renderer('local_hello');
echo $renderer->render(new \local_hello\output\message_list($records));
```

> [!tip] Упрощение Для простых случаев достаточно `$OUTPUT->render_from_template('компонент/шаблон', $массив)` и renderable не нужен.

---

## 14. Языковые строки

Все тексты UI лежат в файлах `lang/<код_языка>/<component>.php`. **Английская версия обязательна**, остальные языки опциональны.

```php
// lang/en/local_hello.php
$string['greeting'] = 'Hello, {$a}!';
$string['count']    = 'You have {$a->count} messages from {$a->name}';
```

Использование:

```php
get_string('greeting', 'local_hello', 'Иван');
get_string('count', 'local_hello', ['count' => 3, 'name' => 'Анна']);   // или объект
get_string('save');                       // строка ядра (компонент 'moodle' по умолчанию)
get_string('delete', 'core');
```

В JS:

```js
import {get_string as getString} from 'core/str';
getString('greeting', 'local_hello', 'Иван').then(s => ...);
```

Правила:

- ключи строк **по алфавиту** (так требует стиль кода);
- строки не склеивай из кусков (в других языках порядок слов другой), используй плейсхолдеры `{$a}`;
- русский перевод: `lang/ru/local_hello.php` (в каталоге плагинов переводы обычно ведёт сообщество через AMOS, но для своего плагина можно положить файл локально).

---

## 15. Настройки плагина

`local/hello/settings.php` добавляет страницу в админку:

```php
<?php
defined('MOODLE_INTERNAL') || die();

if ($hassiteconfig) {   // только если у пользователя есть права настройки сайта
    $settings = new admin_settingpage('local_hello',
        get_string('pluginname', 'local_hello'));
    $ADMIN->add('localplugins', $settings);   // раздел «Локальные плагины»

    $settings->add(new admin_setting_configtext(
        'local_hello/greeting',                       // component/имя
        get_string('greeting_setting', 'local_hello'),
        get_string('greeting_setting_desc', 'local_hello'),
        'Hello',                                      // значение по умолчанию
        PARAM_TEXT
    ));

    $settings->add(new admin_setting_configcheckbox(
        'local_hello/enabled', 'Enabled', '', 1));
}
```

Чтение и запись:

```php
$greeting = get_config('local_hello', 'greeting');      // одно значение
$all      = get_config('local_hello');                  // объект со всеми настройками
set_config('greeting', 'Привет', 'local_hello');
```

Типы настроек: `admin_setting_configtext`, `configcheckbox`, `configselect`, `configtextarea`, `configpasswordunmask`, `configduration`, `configmulticheckbox`, `heading` и др.

---

## 16. События и наблюдатели

**Events API** это шина событий: ядро и плагины бросают события («пользователь вошёл», «курс создан»), а другие плагины подписываются.

### Свой класс события

`classes/event/message_created.php`:

```php
<?php
namespace local_hello\event;

class message_created extends \core\event\base {

    protected function init() {
        $this->data['crud'] = 'c';                                  // c/r/u/d
        $this->data['edulevel'] = self::LEVEL_OTHER;                // LEVEL_TEACHING / PARTICIPATING / OTHER
        $this->data['objecttable'] = 'local_hello_msg';
    }

    public static function get_name() {
        return get_string('event_message_created', 'local_hello');
    }

    public function get_description() {
        return "The user with id '$this->userid' created message '$this->objectid'.";
    }
}
```

Бросить событие:

```php
$event = \local_hello\event\message_created::create([
    'objectid' => $id,
    'context'  => context_system::instance(),
]);
$event->trigger();
```

### Подписка на чужое событие

`db/events.php`:

```php
<?php
defined('MOODLE_INTERNAL') || die();

$observers = [
    [
        'eventname' => '\core\event\course_completed',
        'callback'  => '\local_hello\observer::course_completed',
    ],
];
```

`classes/observer.php`:

```php
<?php
namespace local_hello;

class observer {
    public static function course_completed(\core\event\course_completed $event) {
        $userid = $event->relateduserid;
        // реакция на завершение курса
    }
}
```

> [!note] Логи События автоматически пишутся в журнал (_Reports → Logs_), поэтому у них должны быть осмысленные `crud`, `edulevel`, описание и название.

---

## 17. Задачи по расписанию и фоновые задачи

### Scheduled task (по cron)

`classes/task/cleanup.php`:

```php
<?php
namespace local_hello\task;

class cleanup extends \core\task\scheduled_task {
    public function get_name() {
        return get_string('task_cleanup', 'local_hello');
    }

    public function execute() {
        global $DB;
        $DB->delete_records_select('local_hello_msg', 'timecreated < ?', [time() - YEAR_IN_SECS]);
    }
}
```

`db/tasks.php`:

```php
<?php
defined('MOODLE_INTERNAL') || die();

$tasks = [
    [
        'classname' => 'local_hello\task\cleanup',
        'blocking'  => 0,
        'minute'    => '0',
        'hour'      => '3',
        'day'       => '*',
        'month'     => '*',
        'dayofweek' => '*',
    ],
];
```

### Adhoc task (разовая фоновая задача)

Подходит для тяжёлых операций, которые не надо делать в запросе пользователя (отправка писем, пересчёт).

```php
namespace local_hello\task;

class send_report extends \core\task\adhoc_task {
    public function execute() {
        $data = $this->get_custom_data();      // то, что передали при постановке
        // тяжёлая работа с $data->userid
    }
}

// Поставить в очередь из любого места кода:
$task = new \local_hello\task\send_report();
$task->set_custom_data(['userid' => $USER->id]);
\core\task\manager::queue_adhoc_task($task);
```

Обе разновидности выполняет **cron Moodle** (должен быть настроен в системе: `* * * * * php /path/admin/cli/cron.php`).

---

## 18. Веб-сервисы и external functions

Moodle имеет встроенный механизм API: **external functions**. Одна функция работает и через REST/SOAP (внешние клиенты, мобильное приложение), и как AJAX-вызов из своего JS.

### Объявление: `db/services.php`

```php
<?php
defined('MOODLE_INTERNAL') || die();

$functions = [
    'local_hello_get_messages' => [
        'classname'    => 'local_hello\external\get_messages',
        'description'  => 'Returns messages of the current user.',
        'type'         => 'read',          // read или write
        'ajax'         => true,            // доступна из JS страницы Moodle
        'capabilities' => 'local/hello:view',
    ],
];

// Готовый «сервис» для внешних клиентов (опционально)
$services = [
    'Hello service' => [
        'functions' => ['local_hello_get_messages'],
        'restrictedusers' => 0,
        'enabled' => 1,
    ],
];
```

### Реализация

`classes/external/get_messages.php`:

```php
<?php
namespace local_hello\external;

use core_external\external_api;
use core_external\external_function_parameters;
use core_external\external_multiple_structure;
use core_external\external_single_structure;
use core_external\external_value;

class get_messages extends external_api {

    public static function execute_parameters(): external_function_parameters {
        return new external_function_parameters([
            'limit' => new external_value(PARAM_INT, 'Max items', VALUE_DEFAULT, 10),
        ]);
    }

    public static function execute(int $limit = 10): array {
        global $DB, $USER;

        // 1. Валидация параметров
        $params = self::validate_parameters(self::execute_parameters(), ['limit' => $limit]);

        // 2. Контекст и права
        $context = \context_system::instance();
        self::validate_context($context);
        require_capability('local/hello:view', $context);

        // 3. Логика
        $records = $DB->get_records('local_hello_msg', ['userid' => $USER->id],
                                    'timecreated DESC', '*', 0, $params['limit']);
        $out = [];
        foreach ($records as $r) {
            $out[] = ['id' => $r->id, 'message' => format_string($r->message)];
        }
        return $out;
    }

    public static function execute_returns(): external_multiple_structure {
        return new external_multiple_structure(
            new external_single_structure([
                'id'      => new external_value(PARAM_INT, 'Message id'),
                'message' => new external_value(PARAM_TEXT, 'Message text'),
            ])
        );
    }
}
```

> [!warning] Пространство имён До Moodle 4.2 классы `external_*` были глобальными (`external_api`, `external_value`...), начиная с 4.2 они в `core_external\`. Если пишешь под старую версию, используй старые имена.

Внешнему клиенту нужен токен: админ включает веб-сервисы, протокол (REST), создаёт сервис и токен пользователя. Вызов:

```
POST https://site/webservice/rest/server.php
  wstoken=...&wsfunction=local_hello_get_messages&moodlewsrestformat=json&limit=5
```

---

## 19. JavaScript в Moodle AMD ES6

Moodle использует **AMD-модули**, а исходники можно писать на **ES6-модулях** (`import`/`export`), их соберёт Grunt/Babel.

- Исходники: `local/hello/amd/src/main.js`
- Собранное: `local/hello/amd/build/main.min.js` (+ `.map`), **нужно коммитить в репозиторий**.

### Пример модуля

```js
// amd/src/main.js
import Ajax from 'core/ajax';
import Notification from 'core/notification';
import Templates from 'core/templates';
import {get_string as getString} from 'core/str';

const loadMessages = async (limit) => {
    const response = await Ajax.call([{
        methodname: 'local_hello_get_messages',   // имя external function
        args: {limit},
    }])[0];
    return response;
};

export const init = (limit) => {
    const button = document.querySelector('#local-hello-load');
    button?.addEventListener('click', async() => {
        try {
            const messages = await loadMessages(limit);
            const {html, js} = await Templates.renderForPromise(
                'local_hello/message_list', {messages, hasmessages: messages.length > 0});
            Templates.replaceNodeContents('#local-hello-list', html, js);
        } catch (e) {
            Notification.exception(e);
        }
    });
};
```

Подключение из PHP:

```php
$PAGE->requires->js_call_amd('local_hello/main', 'init', [10]);
```

### Сборка

```bash
cd /путь/к/moodle
nvm install && nvm use     # версия Node берётся из .nvmrc
npm install
npx grunt amd              # собрать JS всех плагинов (или: npx grunt amd --root=local/hello)
npx grunt watch            # пересобирать при изменении
```

Полезные стандартные модули: `core/ajax`, `core/notification`, `core/str`, `core/templates`, `core/modal_factory`, `core/toast`, `core/pending` (для Behat-тестов), `core/fragment`.

---

## 20. Hooks и callbacks в lib.php

Плагины встраиваются в ядро двумя способами.

### Старый способ: функции в `lib.php`

Moodle ищет функции с определённым именем в `lib.php` каждого плагина. Примеры:

```php
// lib.php  — добавить пункт в навигацию курса
function local_hello_extend_navigation_course(navigation_node $parentnode,
        stdClass $course, context_course $context) {
    if (has_capability('local/hello:view', $context)) {
        $url = new moodle_url('/local/hello/index.php', ['id' => $course->id]);
        $parentnode->add(get_string('pluginname', 'local_hello'), $url,
            navigation_node::TYPE_CUSTOM);
    }
}
```

Другие известные callbacks: `local_hello_extend_navigation`, `local_hello_extend_settings_navigation`, `local_hello_pluginfile` (отдача файлов), `local_hello_before_footer`, `local_hello_user_preferences` и др. Для типа `mod` набор богаче: `hello_add_instance`, `hello_update_instance`, `hello_delete_instance`, `hello_supports`.

### Новый способ: Hooks API (с Moodle 4.3)

Ядро постепенно переводит callbacks на **типизированные hooks**: каждый hook это класс, подписка объявляется в `db/hooks.php`, обработчик это обычный метод класса.

```php
// db/hooks.php
<?php
defined('MOODLE_INTERNAL') || die();

$callbacks = [
    [
        'hook'     => \core\hook\output\before_footer_html_generation::class,
        'callback' => [\local_hello\hook_callbacks::class, 'before_footer'],
        'priority' => 0,
    ],
];
```

```php
// classes/hook_callbacks.php
<?php
namespace local_hello;

class hook_callbacks {
    public static function before_footer(\core\hook\output\before_footer_html_generation $hook): void {
        $hook->add_html('<!-- hello -->');
    }
}
```

> [!important] Что выбирать Если для нужной точки расширения в твоей версии Moodle есть hook, **используй hook**, а старый callback в `lib.php` помечен устаревшим (deprecated) и со временем удаляется. Список актуальных hooks и callbacks смотри на moodledev.io в разделе Hooks / Callbacks, имена классов hooks меняются от версии к версии.

Свои hooks можно объявлять в своём плагине, чтобы другие могли расширять твой код.

---

## 21. Файлы Files API

Файлы Moodle хранятся в `moodledata`, но в БД есть таблица `files` с метаданными. Каждый файл однозначно задаётся **пятёркой**: `contextid`, `component`, `filearea`, `itemid`, `filepath` + `filename`.

### Сохранение файла из формы

```php
// В форме:
$mform->addElement('filemanager', 'attachments', 'Files', null, ['subdirs' => 0, 'maxfiles' => 5]);

// После get_data():
file_save_draft_area_files($data->attachments, $context->id,
    'local_hello', 'attachment', $itemid, ['subdirs' => 0, 'maxfiles' => 5]);
```

### Чтение и отдача

```php
$fs = get_file_storage();
$files = $fs->get_area_files($context->id, 'local_hello', 'attachment', $itemid,
                             'filename', false);   // false = без директорий
foreach ($files as $file) {
    $url = moodle_url::make_pluginfile_url($file->get_contextid(), $file->get_component(),
        $file->get_filearea(), $file->get_itemid(), $file->get_filepath(), $file->get_filename());
}
```

Чтобы файлы отдавались, в `lib.php` нужен callback с **обязательной проверкой прав**:

```php
function local_hello_pluginfile($course, $cm, $context, $filearea, $args, $forcedownload, array $options = []) {
    if ($context->contextlevel != CONTEXT_SYSTEM) {
        return false;
    }
    require_login();
    require_capability('local/hello:view', $context);

    $fs = get_file_storage();
    $itemid = array_shift($args);
    $filename = array_pop($args);
    $filepath = $args ? '/' . implode('/', $args) . '/' : '/';
    $file = $fs->get_file($context->id, 'local_hello', $filearea, $itemid, $filepath, $filename);
    if (!$file) {
        return false;
    }
    send_stored_file($file, 0, 0, $forcedownload, $options);
}
```

---

## 22. Кэш Cache API

Для данных, которые дорого считать, есть **Cache API** с настраиваемыми хранилищами (файлы, Redis, Memcached).

Определение: `db/caches.php`

```php
<?php
defined('MOODLE_INTERNAL') || die();

$definitions = [
    'messages' => [
        'mode' => cache_store::MODE_APPLICATION,   // APPLICATION (общий), SESSION, REQUEST
        'simplekeys' => true,
        'simpledata' => false,
        'ttl' => 3600,                             // время жизни, секунды
    ],
];
```

Строка в `lang`: `$string['cachedef_messages'] = 'Messages cache';`

Использование:

```php
$cache = \cache::make('local_hello', 'messages');
$data = $cache->get($key);
if ($data === false) {
    $data = дорогой_расчёт();
    $cache->set($key, $data);
}
$cache->delete($key);
$cache->purge();
```

---

## 23. Сообщения и уведомления

**Message API** отправляет уведомления пользователям по настроенным каналам (email, мобильные push, внутренние сообщения).

Объявление типа: `db/messages.php`

```php
<?php
defined('MOODLE_INTERNAL') || die();

$messageproviders = [
    'newmessage' => [],     // имя провайдера; строка: $string['messageprovider:newmessage']
];
```

Отправка:

```php
$message = new \core\message\message();
$message->component         = 'local_hello';
$message->name              = 'newmessage';
$message->userfrom          = \core_user::get_noreply_user();
$message->userto            = $user;
$message->subject           = 'Тема';
$message->fullmessage       = 'Текст';
$message->fullmessageformat = FORMAT_PLAIN;
$message->fullmessagehtml   = '<p>Текст</p>';
$message->smallmessage      = 'Коротко';
$message->notification      = 1;   // уведомление, а не личное сообщение
message_send($message);
```

Для разовых уведомлений на странице (баннер): `\core\notification::success('Готово');` или `redirect($url, $msg, null, \core\output\notification::NOTIFY_SUCCESS);`

---

## 24. Privacy API

**Каждый плагин обязан** иметь `classes/privacy/provider.php` (требование GDPR и каталога плагинов). Есть три случая:

1. **Плагин не хранит персональные данные**: `null_provider` (пример выше).
2. **Плагин хранит данные**: нужно реализовать `metadata\provider` (описание того, что хранится) и `plugin\provider` (экспорт и удаление данных пользователя).
3. **Только передаёт данные во внешнюю систему**: `metadata\provider` с описанием внешней системы.

Каркас для случая 2:

```php
namespace local_hello\privacy;

use core_privacy\local\metadata\collection;
use core_privacy\local\request\{approved_contextlist, approved_userlist, contextlist, userlist,
                                writer, transform};

class provider implements
    \core_privacy\local\metadata\provider,
    \core_privacy\local\request\plugin\provider,
    \core_privacy\local\request\core_userlist_provider {

    public static function get_metadata(collection $collection): collection {
        $collection->add_database_table('local_hello_msg', [
            'userid'      => 'privacy:metadata:msg:userid',
            'message'     => 'privacy:metadata:msg:message',
            'timecreated' => 'privacy:metadata:msg:timecreated',
        ], 'privacy:metadata:msg');
        return $collection;
    }

    public static function get_contexts_for_userid(int $userid): contextlist { /* ... */ }
    public static function export_user_data(approved_contextlist $list) { /* ... */ }
    public static function delete_data_for_all_users_in_context(\context $context) { /* ... */ }
    public static function delete_data_for_user(approved_contextlist $list) { /* ... */ }
    public static function get_users_in_context(userlist $userlist) { /* ... */ }
    public static function delete_data_for_users(approved_userlist $userlist) { /* ... */ }
}
```

Не забудь строки `privacy:metadata:*` в языковом файле.

---

## 25. Модуль активности mod

Самый «тяжёлый» тип плагина: активность в курсе (как форум или тест). Пример: `mod_hello`.

### Обязательные файлы

```
mod/hello/
├── version.php           ← component = 'mod_hello'
├── lib.php               ← hello_add_instance, hello_update_instance, hello_delete_instance, hello_supports
├── mod_form.php          ← форма настроек активности
├── view.php              ← страница просмотра активности
├── index.php             ← список всех активностей этого типа в курсе
├── db/install.xml        ← таблица `hello` (экземпляры)
├── db/access.php         ← права: mod/hello:addinstance, mod/hello:view
├── lang/en/hello.php     ← ВНИМАНИЕ: файл строк называется БЕЗ префикса mod_
└── pix/monologo.svg      ← иконка (в новых версиях: monologo.svg)
```

> [!warning] Особенность именования Для `mod` файл строк называется `hello.php` (не `mod_hello.php`), функции начинаются с `hello_`, а компонент для `get_string` всё равно `mod_hello` (или просто `hello` для строк плагина).

### Что требуется в БД

Таблица экземпляров называется по имени модуля (`hello`), в ней обязательные поля `id`, `course`, `name`, `intro`, `introformat`, `timemodified`.

### lib.php: минимум

```php
<?php
defined('MOODLE_INTERNAL') || die();

function hello_supports($feature) {
    switch ($feature) {
        case FEATURE_MOD_INTRO:         return true;   // поле описания
        case FEATURE_BACKUP_MOODLE2:    return true;   // резервное копирование
        case FEATURE_SHOW_DESCRIPTION:  return true;
        case FEATURE_MOD_PURPOSE:       return MOD_PURPOSE_CONTENT;
        default:                        return null;
    }
}

function hello_add_instance($data, $mform = null) {
    global $DB;
    $data->timecreated  = time();
    $data->timemodified = time();
    return $DB->insert_record('hello', $data);   // вернуть id нового экземпляра
}

function hello_update_instance($data, $mform = null) {
    global $DB;
    $data->timemodified = time();
    $data->id = $data->instance;
    return $DB->update_record('hello', $data);
}

function hello_delete_instance($id) {
    global $DB;
    if (!$DB->record_exists('hello', ['id' => $id])) {
        return false;
    }
    $DB->delete_records('hello', ['id' => $id]);
    return true;
}
```

### mod_form.php

```php
<?php
defined('MOODLE_INTERNAL') || die();
require_once($CFG->dirroot . '/course/moodleform_mod.php');

class mod_hello_mod_form extends moodleform_mod {
    public function definition() {
        $mform = $this->_form;
        $mform->addElement('header', 'general', get_string('general', 'form'));

        $mform->addElement('text', 'name', get_string('name'), ['size' => '64']);
        $mform->setType('name', PARAM_TEXT);
        $mform->addRule('name', null, 'required', null, 'client');

        $this->standard_intro_elements();        // поле «Описание»
        $this->standard_coursemodule_elements(); // доступность, группы, завершение
        $this->add_action_buttons();
    }
}
```

### view.php

```php
<?php
require_once(__DIR__ . '/../../config.php');

$id = required_param('id', PARAM_INT);   // id course module
[$course, $cm] = get_course_and_cm_from_cmid($id, 'hello');
$instance = $DB->get_record('hello', ['id' => $cm->instance], '*', MUST_EXIST);

require_login($course, true, $cm);
$context = context_module::instance($cm->id);
require_capability('mod/hello:view', $context);

$PAGE->set_url('/mod/hello/view.php', ['id' => $cm->id]);
$PAGE->set_title(format_string($instance->name));
$PAGE->set_heading(format_string($course->fullname));

echo $OUTPUT->header();
echo $OUTPUT->heading(format_string($instance->name));
echo format_module_intro('hello', $instance, $cm->id);
echo $OUTPUT->footer();
```

> [!tip] Когда нужен mod Если нужен **новый вид активности**, которую преподаватель добавляет в курс, выставляет оценки, ограничивает доступ и включает в резервные копии. Для остального проще `local`. Чтобы не писать каркас вручную, используют генератор **tool_pluginskel** (плагин-«скелет», создаёт файлы по YAML-рецепту) или смотрят на `mod_newmodule` как на шаблон.

---

## 26. Блок block

Блок это виджет в боковой панели или на страницах (профиль, курс, главная).

```
blocks/hello/
├── version.php            ← component = 'block_hello'
├── block_hello.php        ← главный класс (имя файла = имя класса)
├── db/access.php          ← права: block/hello:myaddinstance, block/hello:addinstance
└── lang/en/block_hello.php
```

`blocks/hello/block_hello.php`:

```php
<?php
defined('MOODLE_INTERNAL') || die();

class block_hello extends block_base {

    public function init() {
        $this->title = get_string('pluginname', 'block_hello');
    }

    public function get_content() {
        if ($this->content !== null) {
            return $this->content;
        }
        $this->content = new stdClass();
        $this->content->text = 'Привет из блока!';
        $this->content->footer = '';
        return $this->content;
    }

    public function applicable_formats() {
        return ['course-view' => true, 'site' => true, 'my' => true];
    }

    public function has_config() {
        return true;                    // есть settings.php
    }
}
```

Права для блока (`db/access.php`) должны содержать `block/hello:addinstance` (и `myaddinstance` для страницы «Моё»), иначе блок нельзя будет добавить.

---

## 27. Другие типы плагинов коротко

|Тип|Ключевой файл/класс|Суть|
|---|---|---|
|**theme**|`config.php`, `lib.php`, `scss/`, `templates/`, `layout/`|Обычно наследуют **Boost** (`$THEME->parents = ['boost']`), переопределяют SCSS и шаблоны|
|**auth**|`auth.php` класс `auth_plugin_xxx extends auth_plugin_base`|Метод `user_login($username, $password)` проверяет вход|
|**enrol**|`lib.php` класс `enrol_xxx_plugin extends enrol_plugin`|Способ записи (вручную, по ключу, по cohort)|
|**filter**|`filter.php` класс `filter_xxx extends moodle_text_filter`|Метод `filter($text, $options)` преобразует текст (например, `[[shortcode]]`)|
|**tool**|как `local`, но в `admin/tool/`|Админ-утилиты; можно добавлять в админ-дерево через `settings.php` с `admin_externalpage`|
|**report**|`index.php`, `lib.php`|Отчёты в курсе или на сайте; добавляются в навигацию через `report_xxx_extend_navigation_course`|
|**format**|`format.php`, `lib.php`, класс `format_xxx extends core_courseformat\base`|Как выглядит и ведёт себя страница курса|
|**qtype**|`questiontype.php`, `question.php`, `renderer.php`|Новые типы вопросов для теста|

Админ-страница через `settings.php` (для `tool`/`local`, когда не нужна «панель настроек», а нужна своя страница):

```php
$ADMIN->add('tools', new admin_externalpage('local_hello_admin',
    get_string('pluginname', 'local_hello'),
    new moodle_url('/local/hello/admin.php'), 'local/hello:manage'));
```

---

## 28. Тестирование PHPUnit и Behat

### PHPUnit (модульные и интеграционные тесты PHP)

Тесты лежат в `tests/`, имя файла кончается на `_test.php`, класс наследует `advanced_testcase`.

```php
<?php
namespace local_hello;

/**
 * @covers \local_hello\manager
 */
final class manager_test extends \advanced_testcase {

    public function test_add_message(): void {
        global $DB;
        $this->resetAfterTest();                         // откат изменений БД после теста
        $user = $this->getDataGenerator()->create_user();
        $this->setUser($user);

        $id = manager::add_message($user->id, 'Hi');

        $this->assertTrue($DB->record_exists('local_hello_msg', ['id' => $id]));
    }
}
```

Генераторы данных: `$this->getDataGenerator()->create_user()`, `create_course()`, `create_module('forum', [...])`, `enrol_user(...)`.

Запуск (разовая инициализация тестовой среды в отдельной БД/dataroot, указываются в `config.php`: `$CFG->phpunit_prefix`, `$CFG->phpunit_dataroot`):

```bash
php admin/tool/phpunit/cli/init.php
vendor/bin/phpunit --filter manager_test
vendor/bin/phpunit local/hello/tests/manager_test.php
```

### Behat (приёмочные тесты в браузере)

Сценарии на языке Gherkin в `tests/behat/*.feature`:

```gherkin
@local @local_hello
Feature: Hello page
  Scenario: A student sees the greeting
    Given the following "users" exist:
      | username | firstname |
      | student1 | Anna      |
    And I log in as "student1"
    When I am on site homepage
    And I navigate to "Hello" in site administration
    Then I should see "Hello, Anna"
```

Запуск: `php admin/tool/behat/cli/init.php`, затем `vendor/bin/behat --tags=@local_hello`.

---

## 29. Отладка и инструменты

- **Режим отладки**: _Site administration → Development → Debugging_ (уровень DEVELOPER, показывать ошибки) или в `config.php` (см. раздел 2).
- **Лог ошибок**: лог веб-сервера/PHP-FPM; `error_log()`; для отладки в коде: `debugging('message', DEBUG_DEVELOPER);`.
- **Xdebug** + IDE (VS Code с PHP Debug, PhpStorm): пошаговая отладка.
- **Purge caches**: _Site administration → Development → Purge caches_ или CLI.
- **Theme designer mode**: чтобы не сбрасывать кэш CSS/шаблонов.
- **Performance info**: `$CFG->perfdebug = 15` и `$CFG->debugperformance`.
- **moodle-plugin-ci**: CI-набор для проверок плагина (phpcs, phpdoc, phpunit, behat, mustache lint, grunt).
- **Code checker** (`local_codechecker`): проверка стиля прямо в Moodle.
- **Plugin skeleton** (`tool_pluginskel`) и **Moodle Docker**: быстрый старт.
- **Dev docs**: https://moodledev.io, **Moodle Tracker** (tracker.moodle.org) для багов, форумы moodle.org.

В VS Code стоит настроить PHP-расширение и phpcs с набором правил Moodle, тогда стиль кода будет проверяться на лету.

---

## 30. Стиль кода и публикация

### Стиль (Moodle Coding Style)

- Отступ 4 пробела, длинные строки до ~132 (лучше короче), `<?php` без закрывающего `?>`.
- Каждый PHP-файл начинается с **GPL-заголовка** (лицензия, copyright) и docblock с `@package`, `@copyright`, `@license`.
- В файлах, не содержащих чистых классов, в начале: `defined('MOODLE_INTERNAL') || die();`.
- Функции, классы, методы: docblock на английском.
- Комментарии и строки кода на английском (русский только в языковых пакетах `lang/ru`).
- Имена: классы и функции `snake_case` для функций, `lower_case` для методов в ядре (в новом коде допустим PSR-12-стиль, смотри актуальные рекомендации на moodledev.io).

### Публикация в каталоге плагинов

1. Репозиторий в Git (GitHub).
2. Проходят все проверки `moodle-plugin-ci`.
3. Есть `README`, `CHANGES`, лицензия GPL v3+, privacy provider, английские строки.
4. Регистрация на https://moodle.org/plugins, загрузка версий; каталог проверяет совместимость и безопасность.

---

## 31. Типичные ошибки

|Симптом|Причина|
|---|---|
|Плагин не появляется на странице уведомлений|Папка не в нужном месте, имя папки не совпадает с `component`, нет `version.php` или `lang/en/<component>.php`|
|«Class not found» после добавления класса|Не сброшены кэши (карта автозагрузки); неверный namespace/путь|
|Строка выводится как `[[key]]`|Нет ключа в `lang/en/...` или не сброшен кэш строк|
|Новое право не работает|Не поднята `version` или не запущен upgrade; не добавлена строка названия права|
|Изменения `install.xml` не применились|`install.xml` работает только при первой установке, нужен `upgrade.php` + повышение `version`|
|«Coding error: page url not set»|Не вызван `$PAGE->set_url()`|
|Пустые данные из `get_records` (потерялись строки)|Первое поле выборки не уникально, ключи массива перезаписались|
|Ошибка «Invalid sesskey»|Опасное действие без `sesskey` в ссылке/форме|
|JS не обновляется|Не собран `amd/build` (`grunt amd`), не сброшены кэши, включён `cachejs`|
|Тест падает «database not initialised»|Не выполнен `phpunit init` для текущей версии плагина/кода|
|Работает у разработчика, ломается на проде|Использован SQL, специфичный для одной СУБД; не использованы `$DB->sql_*` хелперы|
|Страница открывается без входа|Забыли `require_login()`|
|Пользователь видит чужие данные|Нет проверки прав/контекста (`require_capability`, фильтрация по `$USER->id`/курсу)|

---

## 32. Шпаргалка

### Минимальный набор файлов плагина `local_xxx`

```
version.php
lang/en/local_xxx.php          // pluginname
classes/privacy/provider.php   // privacy
```

### Скелет страницы

```php
require_once(__DIR__ . '/../../config.php');
require_login();
$context = context_system::instance();
require_capability('local/xxx:view', $context);
$PAGE->set_url(new moodle_url('/local/xxx/index.php'));
$PAGE->set_context($context);
$PAGE->set_title('...');
$PAGE->set_heading('...');
echo $OUTPUT->header();
// ...
echo $OUTPUT->footer();
```

### Где что объявляется

|Что нужно|Где|
|---|---|
|Версия, зависимости|`version.php`|
|Таблицы|`db/install.xml` (+ `db/upgrade.php`)|
|Права|`db/access.php`|
|Строки|`lang/en/<component>.php`|
|Настройки|`settings.php`|
|Подписка на события|`db/events.php`|
|Cron-задачи|`db/tasks.php` (+ `classes/task/`)|
|API для JS/внешних клиентов|`db/services.php` (+ `classes/external/`)|
|Кэши|`db/caches.php`|
|Типы сообщений|`db/messages.php`|
|Подписки на hooks|`db/hooks.php`|
|Шаблоны|`templates/*.mustache`|
|JS|`amd/src/*.js` → `amd/build/*.min.js`|
|Privacy|`classes/privacy/provider.php`|

### Ежедневный цикл

```
правка кода → (если db/lang/классы/шаблоны) purge_caches → (если version/db) upgrade → проверка в браузере
```

### Принципы

1. **Ядро не трогаем**, расширяем плагинами.
2. Всё через API: `$DB`, `required_param`, `moodleform`, `$OUTPUT`, `get_string`.
3. **Права проверяем всегда**: `require_login` + `require_capability` в правильном контексте.
4. Весь HTML-вывод экранируем (`format_string`, `s`, Mustache `{{ }}`), SQL только через плейсхолдеры.
5. Изменил схему или права: повысь `version`.
6. Пиши Privacy provider, тесты и docblock'и с первого дня.

### Куда идти дальше

- **moodledev.io**: официальная документация разработчика (Plugin development, APIs, Guides).
- Исходники ядра как образец: `mod/page` (простой mod), `mod/forum` (сложный), `blocks/html`, `local/` примеры в `admin/tool/`.
- Плагин-образцы: `mod_newmodule`, `local_hello`-подобные примеры в документации.