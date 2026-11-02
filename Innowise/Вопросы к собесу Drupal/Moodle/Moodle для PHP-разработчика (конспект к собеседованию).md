---
custom-width: 80
---
---
## Оглавление

1. [[#1. Что такое Moodle и как он устроен]]
2. [[#2. Установка, config.php и окружение разработчика]]
3. [[#3. Структура каталогов]]
4. [[#4. Как обрабатывается запрос и глобальные объекты]]
5. [[#5. Предметная область: курсы, активности, cm и instance]]
6. [[#6. Контексты, роли и права]]
7. [[#7. Плагины: типы и frankenstyle]]
8. [[#8. Анатомия плагина, version.php и обновления]]
9. [[#9. Первый плагин local_hello шаг за шагом]]
10. [[#10. Работа с базой данных]]
11. [[#11. Параметры и безопасность]]
12. [[#12. Формы moodleform]]
13. [[#13. Вывод: страница, renderer, Mustache]]
14. [[#14. Языковые строки]]
15. [[#15. Настройки плагина]]
16. [[#16. События и наблюдатели]]
17. [[#17. Задачи cron: scheduled и adhoc]]
18. [[#18. Веб-сервисы и external functions]]
19. [[#19. JavaScript: AMD и ES6]]
20. [[#20. Hooks и callbacks]]
21. [[#21. Files API]]
22. [[#22. Cache API]]
23. [[#23. Message API]]
24. [[#24. Privacy API]]
25. [[#25. Модуль активности mod]]
26. [[#26. Блок block]]
27. [[#27. Другие типы плагинов]]
28. [[#28. Тестирование: PHPUnit и Behat]]
29. [[#29. Отладка и производительность]]
30. [[#30. Стиль кода, публикация, обновление сайта]]
31. [[#31. Сценарные вопросы]]
32. [[#32. Как отвечать, если опыта мало, и что спросить у интервьюера]]
33. [[#33. Шпаргалка перед собеседованием]]

---

## 1. Что такое Moodle и как он устроен

### Теория

Moodle это открытая система дистанционного обучения (LMS) на PHP. В ней есть курсы, активности (форумы, тесты, задания), записи пользователей на курсы, роли и права, журнал оценок, отчёты, уведомления. Лицензия GPL v3. Работает с MySQL/MariaDB, PostgreSQL и MS SQL Server.

Главная архитектурная идея: ядро не правят, всё расширяется плагинами. Даже форум и тест в самом Moodle это плагины. Плагин кладётся в нужную папку, подчиняется соглашениям об именах и файлах, и Moodle сам его находит, устанавливает, создаёт таблицы, права и строки.

Код исторически процедурный, со множеством глобальных объектов. Постепенно в него приходят классы, пространства имён, автозагрузка, Mustache-шаблоны, hooks и типизированные API. Поэтому в реальном проекте ты увидишь смесь старого и нового стиля, и это нормально.

Установка состоит из трёх частей. Код (каталог с исходниками). База данных. Каталог moodledata, внешняя папка, куда пишутся загруженные файлы, кэши, временные данные и (если файловые) сессии. Moodledata не должна быть доступна из веба напрямую. Если переносишь сайт, переносить надо все три части, иначе файлы пропадут.

Фронтенд: Mustache-шаблоны и JavaScript (AMD/ES6-модули), темы построены на Bootstrap.

### Вопросы и ответы

**Что такое Moodle и из чего он состоит?** Коротко: открытая LMS на PHP, ядро плюс плагины, данные в SQL-базе, файлы в moodledata. Глубже: три части установки (код, БД, moodledata). Расширение только плагинами разных типов. Ядро даёт API: база ($DB), права (capabilities), вывод ($OUTPUT, Mustache), формы, события, задачи, веб-сервисы, кэш, файлы, сообщения. Подвох: не называй Moodle «CMS». Это LMS с предметной областью обучения (курсы, записи, оценки), а не универсальное управление контентом.

**Чем Moodle отличается от обычной CMS вроде WordPress или Drupal с точки зрения разработчика?** Коротко: предметная область уже зашита (курсы, записи, роли в контекстах, журнал оценок), расширение идёт плагинами по строгим соглашениям, код ближе к процедурному стилю с глобальными объектами. Глубже: у Moodle свой слой БД ($DB) вместо ORM, своя система прав на контекстах, своя библиотека форм moodleform, свои Mustache-шаблоны. Единого роутера нет, у каждой страницы свой скрипт (в 4.5/5.0 появился новый routing для части маршрутов). В Drupal больше Symfony, внедрения зависимостей и YAML-конфигурации. Принцип «ядро не правим, пишем плагин» общий. Подвох: не критикуй Moodle как «старый». Скажи нейтрально: стиль исторически сложился, но современные версии активно движутся к классам, hooks и типизированным API.

**Что такое moodledata и зачем она нужна?** Коротко: каталог вне веб-корня для загруженных файлов, кэшей и временных данных, путь в $CFG->dataroot. Глубже: файлы пользователей лежат там под хэшированными именами (дедупликация по содержимому), метаданные в таблице files. Каталог доступен на запись веб-серверу и недоступен из веба, иначе утекут приватные файлы. В кластере moodledata делают общей (NFS и аналоги), а кэши и сессии выносят в Redis или Memcached. Подвох: нельзя «просто скопировать БД», нужно переносить и moodledata.

---

## 2. Установка, config.php и окружение разработчика

### Теория

Для разработки нужны PHP с расширениями (intl, mbstring, xml, zip, gd, curl, soap, sodium, opcache и др.), веб-сервер, СУБД (удобнее всего PostgreSQL или MariaDB), Git, Composer и Node.js для сборки JS. Проще всего поднять окружение через Docker от Moodle HQ (репозиторий moodlehq/moodle-docker): он даёт веб, БД и Selenium для Behat.

Конфигурация лежит в config.php: параметры БД, wwwroot, dataroot. Самая последняя строка config.php подключает lib/setup.php, это точка, где Moodle «оживает». Поэтому ничего не пиши в config.php после неё.

Для разработки включают режим отладки и отключают кэширование JS, строк и шаблонов темы, иначе будешь постоянно думать, почему изменения не видны.

### Код

Установка из командной строки:

```bash
php admin/cli/install.php --lang=ru --wwwroot=http://localhost:8080 \
  --dataroot=/var/moodledata --dbtype=pgsql --dbname=moodle \
  --dbuser=moodle --dbpass=secret --fullname="Test" --shortname="test" \
  --adminuser=admin --adminpass='Admin123!' --agree-license --non-interactive
```

Docker от Moodle HQ:

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

config.php для разработки:

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
$CFG->prefix    = 'mdl_';
$CFG->wwwroot   = 'http://localhost:8080';
$CFG->dataroot  = '/var/moodledata';
$CFG->admin     = 'admin';

// Режим разработки
$CFG->debug = (E_ALL | E_STRICT);
$CFG->debugdisplay = 1;
$CFG->cachejs = false;
$CFG->langstringcache = false;
$CFG->themedesignermode = true;
$CFG->perfdebug = 15;

require_once(__DIR__ . '/lib/setup.php'); // всегда последней строкой
```

Полезные CLI-команды:

```bash
php admin/cli/upgrade.php          # обновление БД после добавления или изменения плагина
php admin/cli/purge_caches.php     # сброс всех кэшей, самая частая команда
php admin/cli/cron.php             # запуск cron вручную
php admin/cli/scheduled_task.php --execute='\local_hello\task\cleanup'
php admin/cli/adhoc_task.php --execute
php admin/cli/maintenance.php --enable   # режим обслуживания
```

### Вопросы и ответы

**Что делает config.php и почему lib/setup.php должен быть последней строкой?** Коротко: config.php хранит настройки (БД, wwwroot, dataroot) и в конце подключает lib/setup.php, который поднимает ядро. Глубже: всё, что должно повлиять на загрузку ядра ($CFG->... настройки), надо задать до подключения setup.php, иначе оно уже не сработает. Подвох: настройки, добавленные после require setup.php, игнорируются.

**Что вы включаете на локальной машине для разработки?** Коротко: режим отладки DEVELOPER, отключение кэша JS, строк и шаблонов темы, Xdebug, PHPUnit и Behat окружение. Глубже: $CFG->debug, debugdisplay, cachejs=false, langstringcache=false, themedesignermode, perfdebug. На проде эти настройки должны быть выключены. Подвох: оставленный на проде режим дизайнера и отладки сильно тормозит сайт и показывает пользователям ошибки.

---

## 3. Структура каталогов

### Теория

Классическая раскладка Moodle 4.x: в корне лежат папки по типам плагинов и подсистем. admin (админка, в admin/tool плагины-утилиты), auth (аутентификация), blocks (блоки), course (код курсов, в course/format форматы), enrol (способы записи), filter (фильтры текста), grade (журнал оценок), lib (библиотеки ядра: setup.php, dml, form, classes), local (твои локальные плагины), mod (активности), question (банк вопросов и типы вопросов), report (отчёты), repository (источники файлов), theme (темы), user (профили).

Начиная с Moodle 5.0 появилась папка public/. Веб-доступный код переехал внутрь неё (public/local, public/mod, public/lib), а в корне остались config.php, vendor/ и то, что не должно быть видно из веба. Плагины теперь ставятся в public/<тип>/<имя>. Для разработчика плагина внутренняя структура плагина не меняется, меняются пути. Это сделано ради безопасности (в веб-корне только то, что должно быть доступно) и ради нового routing.

Плагины ставятся в папку по типу: local/имя, mod/имя, blocks/имя, admin/tool/имя, course/format/имя, question/type/имя.

### Вопросы и ответы

**Какова структура каталогов Moodle и что изменилось в 5.0?** Коротко: в корне папки по типам плагинов (mod, blocks, local, theme, auth, enrol, admin/tool, lib). В 5.0 веб-доступный код переехал в public/. Глубже: плагин теперь лежит в public/local/имя. Внутренняя структура плагина та же. Цель: безопасность и подготовка к routing. Подвох: если не уверен в деталях конкретного релиза, скажи это прямо и укажи, где проверишь (moodledev.io, release notes).

**Куда положить плагин типа tool и типа format?** Коротко: tool в admin/tool/имя, format в course/format/имя (в 5.x всё это внутри public/). Подвох: папка пути не совпадает с типом в frankenstyle, поэтому надо помнить таблицу соответствия типов и папок.

---

## 4. Как обрабатывается запрос и глобальные объекты

### Теория

Moodle «много-входной»: у каждой страницы свой php-файл (/local/hello/index.php, /mod/forum/view.php). Классического единого роутера нет (в 4.5/5.0 добавлен новый routing на базе Slim для части маршрутов, но для обычных плагинов отдельные страницы остаются нормой).

Типичный скрипт начинается с подключения config.php. Дальше lib/setup.php подключается к БД, поднимает сессию, загружает кэши, определяет пользователя, создаёт глобальные объекты. Потом скрипт проверяет вход и права, настраивает страницу через $PAGE и выводит шапку, содержимое и подвал через $OUTPUT.

Глобальные объекты: $CFG (настройки), $DB (база), $USER (текущий пользователь), $PAGE (описание страницы), $OUTPUT (вывод), $COURSE (текущий курс, заполняется после require_login с курсом), $SITE (курс сайта, запись курса с id=1), $SESSION (данные сессии). В функциях их надо объявлять через global. В современном коде их стараются не использовать глубоко внутри классов, а передавать нужные данные параметрами, чтобы код легче тестировался.

### Код

Скелет любой страницы плагина:

```php
<?php
require_once(__DIR__ . '/../../config.php');   // в 5.x путь может отличаться, проверяй

require_login();
$context = context_system::instance();
require_capability('local/xxx:view', $context);

$PAGE->set_url(new moodle_url('/local/xxx/index.php'));
$PAGE->set_context($context);
$PAGE->set_title('...');
$PAGE->set_heading('...');

echo $OUTPUT->header();
// содержимое
echo $OUTPUT->footer();
```

Полезные функции:

```php
$url = new moodle_url('/local/hello/index.php', ['id' => 5]);
echo html_writer::link($url, 'Ссылка');
redirect($url, 'Сохранено', null, \core\output\notification::NOTIFY_SUCCESS);

$course  = get_course($courseid);
$modinfo = get_fast_modinfo($course);   // быстрый доступ к активностям курса
is_siteadmin();
isloggedin();
is_enrolled($context, $user);
userdate(time());                        // дата с учётом часового пояса пользователя
```

### Вопросы и ответы

**Как Moodle обрабатывает запрос?** Коротко: скрипт подключает config.php, тот читает настройки и подключает lib/setup.php. Setup подключается к БД, создаёт сессию, загружает кэши, определяет пользователя, создаёт глобальные объекты. Потом выполняется код страницы. Глубже: дальше типичный скрипт вызывает require_login, require_capability, настраивает $PAGE (URL, контекст, заголовок), выводит header, содержимое и footer. Подвох: забытый $PAGE->set_url вызывает ошибку кодирования, а забытый require_login оставляет страницу открытой всем.

**Какие глобальные объекты есть и зачем?** Коротко: $CFG конфигурация, $DB база, $USER пользователь, $PAGE страница, $OUTPUT вывод, $COURSE курс, $SITE курс сайта, $SESSION сессия. Глубже: внутри функций объявляются через global. В современном коде их не тянут глубоко в классы, а передают параметрами. Подвох: $USER хранит данные сессии, а не всегда свежие данные из БД. Для актуальных данных пользователя берут запись из таблицы user.

---

## 5. Предметная область: курсы, активности, cm и instance

### Теория

Сайт содержит категории курсов, в категориях курсы, в курсах разделы, в разделах активности. Экземпляр активности привязан к курсу через course module (cm), таблица course_modules. Это один из главных источников путаницы: id экземпляра (в таблице самой активности, например forum) и id course module (в таблице course_modules) это два разных числа. В URL обычно приходит id cm (view.php?id=...), экземпляр получают как $cm->instance. Контекст модуля строится от cm: context_module::instance($cm->id).

Пользователь записывается на курс через плагин записи (enrol), ему назначается роль (студент, преподаватель и т.д.).

Основные сущности: Site, Category, Course, Section, Activity/Module, Course module, Resource (файл, страница, папка, тоже mod), User, Enrolment, Role, Capability, Context, Grade (журнал оценок), Cohort (глобальная группа пользователей), Group и Grouping (группы внутри курса).

### Код

Правильное получение курса и cm на странице активности:

```php
$id = required_param('id', PARAM_INT);   // id course module, не instance!
[$course, $cm] = get_course_and_cm_from_cmid($id, 'hello');
$instance = $DB->get_record('hello', ['id' => $cm->instance], '*', MUST_EXIST);

require_login($course, true, $cm);
$context = context_module::instance($cm->id);   // от cm->id, не от instance
```

### Вопросы и ответы

**В чём разница между id экземпляра и id course module?** Коротко: course module это привязка активности к курсу и разделу (таблица course_modules), instance это запись самой активности (например, в таблице forum). Числа разные. Глубже: в URL приходит cm id. Экземпляр: $cm->instance. Контекст: context_module::instance($cm->id). Подвох: поиск записи в таблице активности по cm id даёт чужую или пустую запись.

**Как проверить доступ пользователя к конкретной активности?** Коротко: получить cm, вызвать require_login($course, true, $cm), затем проверить право в context_module::instance($cm->id). Глубже: get_course_and_cm_from_cmid или get_fast_modinfo дают курс и cm. require_login учитывает запись на курс, видимость, ограничения доступа и режим гостя. Дальше проверяется конкретное право (view, submit и т.д.). Подвох: путаница cm id и instance id.

---

## 6. Контексты, роли и права

### Теория

Контекст это область, в которой действует право. Иерархия: система (CONTEXT_SYSTEM), категория (CONTEXT_COURSECAT), курс (CONTEXT_COURSE), модуль/активность (CONTEXT_MODULE), блок (CONTEXT_BLOCK). Отдельной веткой стоит контекст пользователя (CONTEXT_USER). Роль назначается в контексте, права наследуются вниз по дереву и могут быть переопределены.

Capability (право) это конкретное право вида компонент/путь:имя, например mod/forum:replypost. Оно объявляется в db/access.php плагина. У права есть тип (read или write), уровень контекста, архетипы (какие стандартные роли получают его по умолчанию) и битовая маска риска (например, RISK_XSS, RISK_PERSONAL), по которой админ видит, насколько право опасно.

Значения права для роли: CAP_ALLOW (разрешить), CAP_PREVENT (не разрешать, но можно разрешить ниже), CAP_PROHIBIT (жёсткий запрет, не переопределяется ниже), CAP_INHERIT (наследовать).

В коде проверяют право, а не роль. has_capability возвращает true/false, require_capability выбрасывает исключение. Для курсов и активностей перед этим вызывают require_login с курсом и cm: он проверяет вход, запись на курс, видимость.

### Код

Объявление права, db/access.php:

```php
<?php
defined('MOODLE_INTERNAL') || die();

$capabilities = [
    'local/hello:view' => [
        'captype'      => 'read',
        'contextlevel' => CONTEXT_SYSTEM,
        'archetypes'   => [
            'user'    => CAP_ALLOW,
            'manager' => CAP_ALLOW,
        ],
    ],
    'local/hello:manage' => [
        'captype'      => 'write',
        'contextlevel' => CONTEXT_COURSE,
        'riskbitmask'  => RISK_XSS,
        'archetypes'   => [
            'editingteacher' => CAP_ALLOW,
            'teacher'        => CAP_ALLOW,
            'manager'        => CAP_ALLOW,
        ],
    ],
];
```

Строки для прав в lang/en/local_hello.php:

```php
$string['hello:view']   = 'View hello page';
$string['hello:manage'] = 'Manage hello messages';
```

Проверки:

```php
$context = context_course::instance($courseid);

if (has_capability('local/hello:manage', $context)) { /* показать кнопку */ }
require_capability('local/hello:manage', $context);   // исключение, если нет права

// Для страницы активности:
require_login($course, true, $cm);
require_capability('mod/hello:view', context_module::instance($cm->id));
```

### Вопросы и ответы

**Как устроена система ролей и прав?** Коротко: права объявляются в db/access.php, роли получают их в контекстах (система, категория, курс, модуль, блок), права наследуются вниз и могут быть переопределены. В коде проверяют право, а не роль. Глубже: значения CAP_ALLOW, CAP_PREVENT, CAP_PROHIBIT, CAP_INHERIT. Архетипы задают, какие стандартные роли получают право по умолчанию. Админ может менять и переопределять права через интерфейс. Подвох: проверка вида «если у пользователя роль teacher» плохая практика, роли переименовывают и создают новые.

**В чём разница между require_login и require_capability?** Коротко: require_login проверяет, что пользователь вошёл (а при передаче курса и cm ещё и доступ к курсу и активности). require_capability проверяет конкретное право в контексте. Глубже: правильный порядок для страницы активности: require_login($course, true, $cm), затем require_capability('mod/x:view', context_module::instance($cm->id)). Когда нужен результат, а не исключение, используют has_capability. Подвох: require_login без проверки права открывает страницу любому вошедшему, включая студентов без нужной роли.

**Что такое контекст и зачем он нужен?** Коротко: область действия прав и назначений ролей, иерархия от системы до блока. Подвох: права проверяют в правильном контексте. Проверка в системном контексте для страницы курса даёт не тот результат, потому что на курсе права могли быть переопределены.

**Как добавить новое право в плагин и почему оно «не появилось»?** Коротко: объявить в db/access.php, добавить строку названия, поднять version и выполнить upgrade. Подвох: поменял access.php, но не поднял version, поэтому новое право не появится.

---

## 7. Плагины: типы и frankenstyle

### Теория

У каждого плагина есть тип и имя. Вместе они дают frankenstyle-имя: local_hello, mod_forum, block_html. Оно же называется component и используется везде: в строках, правах, событиях, кэшах, таблицах и пространствах имён. Правила: имя из маленьких латинских букв и цифр, начинается с буквы, без подчёркиваний внутри самой части «имя». Таблицы называются с префиксом компонента (local_hello_msg), функции в lib.php тоже (local_hello_extend_navigation), для mod функции начинаются с имени без mod_ (hello_add_instance). Пространство имён классов совпадает с компонентом.

Основные типы: mod (активности и ресурсы), block (блоки), local (универсальный тип для всего, что не подходит под другие), theme, auth (аутентификация), enrol (способы записи), filter (фильтры текста), format (форматы курсов), report (отчёты), tool (админ-утилиты в admin/tool), qtype (типы вопросов), repository, editor/tiny, availability (условия доступа), message (каналы доставки), webservice, cachestore, а также подплагины (assignsubmission, assignfeedback, gradereport).

Как выбрать тип. Нужна новая активность в курсе: mod. Блок на странице: block. Админская утилита: tool. Отчёт: report. Всё остальное (новые страницы, интеграции, обработчики событий, веб-сервисы): local. Для 80% задач хватает local, block, mod и tool.

### Вопросы и ответы

**Какие типы плагинов бывают и как выбрать тип?** Коротко: mod, block, local, theme, auth, enrol, filter, format, report, tool, qtype, repository. Новая активность в курсе: mod, блок на странице: block, остальное чаще всего local. Глубже: local универсален для страниц, обработчиков событий, веб-сервисов и интеграций. Выбор типа определяет папку и набор обязательных файлов. Подвох: не используй mod там, где достаточно local. Mod тянет за собой резервное копирование, оценки, завершение, права на экземпляр, это много работы.

**Что такое frankenstyle?** Коротко: схема именования тип_имя, например local_hello. Используется в строках, правах, событиях, кэшах, таблицах и пространствах имён. Подвох: в части «имя» не должно быть подчёркиваний, иначе Moodle не разберёт компонент.

**В чём разница между local, mod и block на практике?** Коротко: mod это активность внутри курса со своими экземплярами, оценками и резервными копиями. Block это виджет на странице. Local это всё остальное. Глубже: экземпляр mod создаёт преподаватель, он попадает в course_modules, участвует в резервном копировании и доступе. Блок добавляется на страницу и показывает контент. Local не имеет собственного контейнера: сам решает, какие страницы, пункты навигации и обработчики добавить. Подвох: для небольшой функции «страница, таблица и право» хватит local, не усложняй.

---

## 8. Анатомия плагина, version.php и обновления

### Теория

Минимальный плагин состоит из version.php, языкового файла lang/en/компонент.php с ключом pluginname и (для каталога обязательно) privacy provider. Остальное по необходимости.

В version.php задают component, version (формат ГГГГММДДNN), requires (минимальная версия Moodle), maturity, release, при необходимости dependencies. Moodle сравнивает version из файла с версией, записанной в БД. Нет записи: установка, число в файле больше: обновление, запускается db/upgrade.php. Параллельно регистрируются права из access.php, события, задачи, веб-сервисы, кэши. Поэтому после любого изменения схемы или этих файлов версию повышают и запускают php admin/cli/upgrade.php (или заходят в админку).

Автозагрузка классов. Папка classes/ подключается автоматически: namespace начинается с frankenstyle, остальное повторяет путь внутри classes/. Карта классов кэшируется, поэтому после добавления нового класса надо сбросить кэши.

Типичное содержимое плагина:

```
local/hello/
├── version.php              обязательно
├── lang/en/local_hello.php  обязательно (минимум pluginname)
├── classes/privacy/provider.php  обязательно для каталога
├── classes/                 PHP-классы (form, output, external, event, task...)
├── db/                      install.xml, upgrade.php, access.php, events.php,
│                            tasks.php, services.php, caches.php, hooks.php, messages.php
├── templates/               Mustache-шаблоны
├── amd/src, amd/build       JS-исходники и сборка
├── tests/                   PHPUnit и Behat
├── lib.php, settings.php, index.php, styles.css, pix/
```

### Код

version.php:

```php
<?php
defined('MOODLE_INTERNAL') || die();

$plugin->component = 'local_hello';     // совпадает с именем папки
$plugin->version   = 2026110200;        // YYYYMMDDXX
$plugin->requires  = 2024100700;        // минимальный build Moodle
$plugin->maturity  = MATURITY_STABLE;
$plugin->release   = '1.0.0';
// $plugin->dependencies = ['mod_forum' => 2022112800];
```

Соответствие классов и файлов:

```
\local_hello\manager               -> local/hello/classes/manager.php
\local_hello\form\message_form     -> local/hello/classes/form/message_form.php
\local_hello\task\cleanup          -> local/hello/classes/task/cleanup.php
```

### Вопросы и ответы

**Какие файлы обязательны в плагине?** Коротко: version.php и lang/en/компонент.php с ключом pluginname. Для публикации ещё privacy provider. Глубже: дальше по необходимости db/install.xml и upgrade.php, db/access.php, settings.php, lib.php, classes/, templates/, amd/, tests/. У mod ещё mod_form.php, view.php и функции add/update/delete instance. Подвох: если имя папки не совпадает с component в version.php, Moodle откажется устанавливать плагин.

**Как Moodle узнаёт, что плагин нужно установить или обновить?** Коротко: сравнивает version из version.php с версией в БД. Нет записи: установка, число больше: обновление. Глубже: при установке выполняется db/install.xml, при обновлении db/upgrade.php. Регистрируются права, события, задачи, веб-сервисы, кэши. Проверка срабатывает при заходе админа или через php admin/cli/upgrade.php. Подвох: поменял db/access.php и не поднял version: новое право не появится.

**Как работает автозагрузка классов?** Коротко: классы лежат в classes/, namespace начинается с компонента, остальное повторяет путь. Подвох: «Class not found» после добавления класса почти всегда лечится сбросом кэшей или исправлением namespace. Вручную через require подключают только устаревшие библиотеки (например, formslib для moodleform).

---

## 9. Первый плагин local_hello шаг за шагом

### Теория

Цель: страница /local/hello/index.php, которая показывает приветствие и доступна по праву. Порядок работы: создать папку, version.php, языковой файл, право, privacy provider, страницу, затем установить (upgrade). Это стандартный маршрут для любого простого плагина, и его полезно проговаривать на собеседовании.

### Код

Шаг 1. version.php, как в предыдущем разделе.

Шаг 2. lang/en/local_hello.php:

```php
<?php
defined('MOODLE_INTERNAL') || die();

$string['greeting']   = 'Hello, {$a}!';
$string['hello:view'] = 'View hello page';
$string['pluginname'] = 'Hello';
$string['privacy:metadata'] = 'The Hello plugin does not store personal data.';
```

Шаг 3. db/access.php (право local/hello:view, см. раздел 6).

Шаг 4. classes/privacy/provider.php:

```php
<?php
namespace local_hello\privacy;

class provider implements \core_privacy\local\metadata\null_provider {
    public static function get_reason(): string {
        return 'privacy:metadata';
    }
}
```

Шаг 5. index.php:

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

Шаг 6. Положить папку в local/hello (в 5.x public/local/hello), зайти как админ (Site administration, Notifications) или выполнить php admin/cli/upgrade.php, подтвердить установку, открыть страницу.

Цикл разработки: изменил PHP — обнови страницу. Изменил строки, шаблоны, db/*.php или добавил класс — php admin/cli/purge_caches.php. Поднял version — php admin/cli/upgrade.php.

### Вопросы и ответы

**Опишите, как создать простой плагин со страницей, доступной по праву.** Коротко: тип local, version.php, языковой файл, db/access.php с правом, privacy provider, index.php с require_login, require_capability, настройкой $PAGE и выводом через $OUTPUT. Затем upgrade. Подвох: забытая строка названия права показывает [[ключ]] в интерфейсе.

**Что делать, если плагин не появился на странице уведомлений?** Коротко: проверить путь (папка в нужном месте), совпадение имени папки с component, наличие version.php и lang/en/<component>.php, сбросить кэши.

---

## 10. Работа с базой данных

### Теория

Всё обращение к БД идёт через $DB. Прямой PDO или mysqli использовать нельзя: Moodle поддерживает несколько СУБД, и $DB скрывает различия, добавляет префиксы, плейсхолдеры, профилирование, транзакции и работу с тестовой БД. Прямой доступ ломает переносимость, тесты (PHPUnit использует отдельный префикс и откат) и безопасность.

Имена таблиц пишутся без префикса. В SQL таблицы берут в фигурные скобки {таблица}, Moodle сам добавит префикс. Значения подставляются через плейсхолдеры, знак вопроса или :имя, смешивать их нельзя. Для кроссплатформенных конструкций есть хелперы: sql_like, sql_concat, sql_compare_text, sql_cast_char2int.

Схема таблиц описывается в db/install.xml (создаётся при первой установке). Изменения для уже существующих сайтов оформляются в db/upgrade.php через XMLDB API с обязательным upgrade_plugin_savepoint. Удобнее всего генерировать XML и код миграций в админском XMLDB-редакторе (Site administration, Development, XMLDB editor). Поле id в таблице обязательно. Имя таблицы без префикса не длиннее 28 символов.

Методы чтения: get_record, get_records, get_field, get_records_sql, get_record_sql, get_recordset, count_records, record_exists. Записи: insert_record, update_record, delete_records. Для IN: get_in_or_equal.

Транзакции в Moodle делегированные и вложенные: реальная фиксация происходит на самом внешнем уровне. Если внутренний код сделал rollback, внешний уровень тоже откатывается.

### Код

install.xml:

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

upgrade.php:

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
        if (!$dbman->field_exists($table, $field)) {   // идемпотентность
            $dbman->add_field($table, $field);
        }
        upgrade_plugin_savepoint(true, 2026110300, 'local', 'hello');
    }
    return true;
}
```

Правило: для каждого изменения новый блок if ($oldversion < НОВАЯ_ВЕРСИЯ), внутри savepoint, и такая же версия в version.php. Параллельно обновляют install.xml, чтобы новые установки получали актуальную схему.

Основные методы:

```php
$rec  = $DB->get_record('user', ['id' => 5]);                    // false, если нет
$rec  = $DB->get_record('user', ['id' => 5], '*', MUST_EXIST);   // исключение, если нет
$recs = $DB->get_records('local_hello_msg', ['userid' => $USER->id], 'timecreated DESC');
$val  = $DB->get_field('user', 'email', ['id' => 5]);
$cnt  = $DB->count_records('local_hello_msg', ['userid' => 5]);
$has  = $DB->record_exists('local_hello_msg', ['id' => 3]);

$sql = "SELECT m.id, m.message, u.firstname
          FROM {local_hello_msg} m
          JOIN {user} u ON u.id = m.userid
         WHERE m.timecreated > :since";
$recs = $DB->get_records_sql($sql, ['since' => time() - DAY_IN_SECS]);

$id = $DB->insert_record('local_hello_msg', (object)[
    'userid' => $USER->id, 'message' => 'Привет', 'timecreated' => time(),
]);
$DB->update_record('local_hello_msg', (object)['id' => $id, 'message' => 'Новое']);
$DB->delete_records('local_hello_msg', ['id' => $id]);

[$insql, $inparams] = $DB->get_in_or_equal([1, 2, 3], SQL_PARAMS_NAMED);
$users = $DB->get_records_select('user', "id $insql", $inparams);

// Большая выборка без загрузки всего в память
$rs = $DB->get_recordset('local_hello_msg');
foreach ($rs as $row) { /* ... */ }
$rs->close();   // обязательно

// Кроссплатформенный LIKE
$DB->sql_like('message', ':p', false);
```

Транзакция:

```php
$transaction = $DB->start_delegated_transaction();
try {
    // несколько операций
    $transaction->allow_commit();
} catch (Exception $e) {
    $transaction->rollback($e);   // откат и повторный выброс
}
```

### Вопросы и ответы

**Почему нельзя использовать PDO или mysqli напрямую?** Коротко: Moodle поддерживает несколько СУБД, $DB скрывает различия, добавляет префиксы, плейсхолдеры, профилирование, транзакции и работу с тестовой БД. Глубже: прямой доступ ломает переносимость, префиксы, тесты и единую безопасность. Для кроссплатформенных конструкций есть sql_like, sql_concat, sql_compare_text, sql_cast_char2int. Подвох: на проверке кода прямое обращение к БД почти всегда отклоняют.

**get_record, get_records, get_recordset: в чём разница?** Коротко: get_record одна запись, get_records массив записей с ключом по первому полю, get_recordset итератор, не грузит всё в память. Глубже: recordset обязательно закрывают ($rs->close()). У get_record режимы IGNORE_MISSING (вернуть false), IGNORE_MULTIPLE (взять первую), MUST_EXIST (исключение, если нет). Подвох: в get_records первое поле выборки становится ключом массива. Если оно не уникально, строки перезапишут друг друга. В запросах с JOIN первым полем ставь уникальный id.

**Как безопасно подставлять значения в SQL и делать IN?** Коротко: только плейсхолдеры (? или :имя, не смешивая). Для IN get_in_or_equal. Глубже: таблицы пишутся как {имя}. Динамические имена колонок или сортировку по пользовательскому вводу сверяют по белому списку. Подвох: конкатенация пользовательских данных в SQL недопустима, даже «очищенных».

**Как работают транзакции?** Коротко: $DB->start_delegated_transaction(), операции, в конце allow_commit(), при исключении rollback($e). Транзакции вложенные, фиксация на внешнем уровне. Глубже: если внутренний код сделал rollback, внешний уровень тоже откатится. Наблюдатели событий с internal=false откладываются до фиксации. Подвох: нельзя оставлять транзакцию «висеть» без allow_commit или rollback.

**Как создать таблицу и как её менять?** Коротко: схема в db/install.xml (удобно через XMLDB-редактор), изменения через db/upgrade.php с повышением version. Подвох: менять только install.xml недостаточно, он выполняется лишь при первой установке. Забытый индекс по полю фильтрации самая частая причина медленных страниц на больших данных.

**Что значит ограничение на длину имени таблицы?** Коротко: имя без префикса не длиннее 28 символов, поэтому имена плагинов и таблиц делают короткими.

---

## 11. Параметры и безопасность

### Теория

Входные данные берутся через required_param и optional_param с типом PARAM_*, прямой доступ к $_GET и $_POST не используется. Тип выполняет очистку: PARAM_INT оставит число, PARAM_ALPHANUMEXT буквы, цифры, подчёркивание и дефис, PARAM_TEXT уберёт теги. PARAM_RAW не очищает, поэтому с ним нужна отдельная валидация и аккуратный вывод.

Защита от XSS: format_string для заголовков, format_text для HTML из редактора, s() для произвольных строк, автоэкранирование в Mustache ({{var}} экранирует, {{{var}}} нет). Защита от SQL-инъекций: только плейсхолдеры. Защита от CSRF: sesskey. Формы moodleform проверяют его сами, для ссылок с действиями используют sesskey() и require_sesskey(). Права проверяют на сервере всегда, даже если кнопку «не показали».

### Код

```php
$id     = required_param('id', PARAM_INT);
$action = optional_param('action', 'view', PARAM_ALPHA);
$text   = optional_param('text', '', PARAM_TEXT);

// Опасное действие по ссылке
$deleteurl = new moodle_url('/local/hello/delete.php',
    ['id' => $id, 'sesskey' => sesskey()]);

// На странице-обработчике
require_login();
require_sesskey();
require_capability('local/hello:manage', $context);
```

Вывод:

```php
echo format_string($record->name);              // заголовки
echo format_text($record->description, FORMAT_HTML);   // HTML из редактора
echo s($userinput);                              // произвольная строка
```

Частые типы PARAM: PARAM_INT, PARAM_FLOAT, PARAM_BOOL, PARAM_ALPHA, PARAM_ALPHANUMEXT, PARAM_TEXT, PARAM_RAW, PARAM_URL, PARAM_EMAIL, PARAM_COMPONENT.

### Вопросы и ответы

**Как получать параметры запроса?** Коротко: required_param и optional_param с типом PARAM_*. Прямое чтение $_GET и $_POST не используется. Подвох: в moodleform забывают setType для полей, из-за чего значение не очищается и при отладке появляется предупреждение.

**Как защититься от XSS, CSRF и SQL-инъекций?** Коротко: XSS: format_string, format_text, s(), автоэкранирование Mustache. CSRF: sesskey (moodleform делает сам, для ссылок require_sesskey). SQL: плейсхолдеры. Глубже: тройные скобки в Mustache допустимы только для уже безопасного HTML. Для чувствительных к XSS прав ставят riskbitmask (RISK_XSS), чтобы админ видел риск при выдаче. Подвох: скрыть кнопку в интерфейсе не защита. Проверка должна быть на сервере.

**Когда допустим PARAM_RAW?** Коротко: когда очистка невозможна по смыслу (например, вы сами позже валидируете структуру), но тогда вывод и валидация на вашей совести.

---

## 12. Формы moodleform

### Теория

Moodle имеет свой фреймворк форм moodleform: определение полей в definition(), типы данных через setType, валидация в validation(), обработка через is_cancelled() и get_data(). Форма сама проверяет sesskey, очищает данные по setType, показывает ошибки. Есть элементы для файлов (filemanager), редактора, дат, автодополнения. Для предзаполнения используют set_data. Для форм активностей (mod) наследуются от moodleform_mod, которая добавляет стандартные блоки (описание, доступность, завершение).

После успешного сохранения всегда делай redirect (паттерн Post/Redirect/Get), иначе при обновлении страницы форма отправится повторно.

### Код

classes/form/message_form.php:

```php
<?php
namespace local_hello\form;

defined('MOODLE_INTERNAL') || die();
require_once($CFG->libdir . '/formslib.php');

class message_form extends \moodleform {

    protected function definition() {
        $mform = $this->_form;

        $mform->addElement('text', 'message', get_string('message', 'local_hello'));
        $mform->setType('message', PARAM_TEXT);               // обязательно
        $mform->addRule('message', get_string('required'), 'required', null, 'client');
        $mform->addRule('message', null, 'maxlength', 255, 'client');

        $mform->addElement('hidden', 'id', $this->_customdata['id'] ?? 0);
        $mform->setType('id', PARAM_INT);

        $this->add_action_buttons();
    }

    public function validation($data, $files) {
        $errors = parent::validation($data, $files);
        if (trim($data['message']) === '') {
            $errors['message'] = get_string('required');
        }
        return $errors;
    }
}
```

На странице:

```php
$mform = new \local_hello\form\message_form(null, ['id' => $id]);

if ($mform->is_cancelled()) {
    redirect(new moodle_url('/local/hello/index.php'));
} else if ($data = $mform->get_data()) {
    $DB->insert_record('local_hello_msg', (object)[
        'userid' => $USER->id,
        'message' => $data->message,   // уже очищено по setType
        'timecreated' => time(),
    ]);
    redirect(new moodle_url('/local/hello/index.php'), 'Сохранено',
             null, \core\output\notification::NOTIFY_SUCCESS);
}

echo $OUTPUT->header();
$mform->display();
echo $OUTPUT->footer();
```

Полезное: setDefault, addHelpButton, hideIf, disabledIf, set_data. Типы элементов: text, textarea, editor, select, advcheckbox, date_selector, date_time_selector, filemanager, filepicker, passwordunmask, static, header, hidden, autocomplete, duration, radio, group.

### Вопросы и ответы

**Как делать формы в Moodle?** Коротко: класс на основе moodleform, поля в definition(), типы через setType, правила и validation(). На странице is_cancelled(), get_data(), display(). Подвох: форма, не обрабатывающая случай «данные не пришли», показывает пустую страницу после сохранения. Всегда redirect после успеха.

**Зачем setType?** Коротко: он задаёт очистку значения. Без него Moodle не знает, как чистить данные, и значение остаётся неочищенным.

---

## 13. Вывод: страница, renderer, Mustache

### Теория

На странице обязательно задать URL и контекст ($PAGE->set_url, $PAGE->set_context), затем заголовки. Дальше header, содержимое, footer.

Современный подход: данные готовит PHP-класс (renderable и templatable с методом export_for_template), а HTML делает Mustache-шаблон из templates/. Renderer связывает их. Mustache не содержит логики: всё, что нужно (флаги, отформатированные даты), готовится в PHP. В шаблоне обязателен docblock с @template и примером контекста в JSON, по нему строится документация и тесты. Один и тот же шаблон можно отрисовать и на сервере, и в JavaScript через core/templates, что упрощает AJAX-обновления.

Для простых случаев достаточно $OUTPUT->render_from_template('компонент/шаблон', $массив).

### Код

Подготовка страницы:

```php
$PAGE->set_url($url);                  // обязательно
$PAGE->set_context($context);          // обязательно
$PAGE->set_title('Заголовок вкладки');
$PAGE->set_heading('Заголовок страницы');
$PAGE->set_pagelayout('standard');
$PAGE->navbar->add('Крошка', $url);
$PAGE->requires->js_call_amd('local_hello/main', 'init', [$id]);
```

Готовые элементы:

```php
echo $OUTPUT->heading('Текст', 2);
echo $OUTPUT->notification('Ошибка', \core\output\notification::NOTIFY_ERROR);
echo $OUTPUT->single_button($url, 'Нажми');
echo $OUTPUT->render_from_template('local_hello/message_list', $data);
```

Шаблон templates/message_list.mustache:

```mustache
{{!
    @template local_hello/message_list

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

Основные конструкции Mustache: {{var}} вывод с экранированием, {{{var}}} без экранирования, {{#list}}...{{/list}} цикл или условие, {{^x}}...{{/x}} «если пусто», {{#str}} key, component {{/str}} строка, {{#pix}} i/edit, core {{/pix}} иконка, {{> core/loading }} partial.

Renderable, classes/output/message_list.php:

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

Renderer, classes/output/renderer.php:

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

### Вопросы и ответы

**Как выводить HTML: renderer, renderable, Mustache?** Коротко: PHP-класс (renderable/templatable) готовит данные в export_for_template, Mustache-шаблон делает HTML, renderer их связывает. Для простых случаев render_from_template. Глубже: в Mustache нет логики, все флаги и даты считаются в PHP. Тот же шаблон работает из JS через core/templates. Подвох: тройные фигурные скобки отключают экранирование, используй только для заведомо безопасного HTML.

**Почему в шаблоне нужен docblock с примером JSON?** Коротко: по нему строится документация шаблона, и он используется в тестах и в инструментах проверки (mustache lint).

---

## 14. Языковые строки

### Теория

Все тексты интерфейса лежат в lang/<язык>/<компонент>.php и получаются через get_string('ключ', 'компонент', $a). Английская версия обязательна, остальные языки по желанию. Строки не склеивают из кусков (в других языках другой порядок слов), а используют плейсхолдеры. Ключи расположены по алфавиту. Строки кэшируются, поэтому после правок сбрасывают кэш, иначе увидишь старое значение или [[ключ]]. В JS используют core/str.

Название права, описание события, имя провайдера сообщений, определение кэша и privacy-метаданные тоже живут в языковом файле. Для mod-плагина файл называется без префикса: lang/en/hello.php.

### Код

```php
// lang/en/local_hello.php
$string['count']    = 'You have {$a->count} messages from {$a->name}';
$string['greeting'] = 'Hello, {$a}!';
```

```php
get_string('greeting', 'local_hello', 'Иван');
get_string('count', 'local_hello', ['count' => 3, 'name' => 'Анна']);
get_string('save');              // строка ядра
```

```js
import {get_string as getString} from 'core/str';
getString('greeting', 'local_hello', 'Иван').then(s => { /* ... */ });
```

### Вопросы и ответы

**Как работают языковые строки?** Коротко: lang/<язык>/<компонент>.php, получение через get_string. Английский файл обязателен. Глубже: плейсхолдеры {$a} или {$a->поле}. Строки не склеиваются. Ключи по алфавиту. После правки сброс кэша. Подвох: забытая строка показывает [[ключ]] в интерфейсе. Это также относится к названиям прав и описаниям событий.

---

## 15. Настройки плагина

### Теория

Файл settings.php добавляет страницу настроек в админку. Условие $hassiteconfig проверяет, что у пользователя есть права настройки сайта. Значения читают через get_config и пишут через set_config. Ключ настройки имеет вид 'компонент/имя'.

### Код

```php
<?php
defined('MOODLE_INTERNAL') || die();

if ($hassiteconfig) {
    $settings = new admin_settingpage('local_hello',
        get_string('pluginname', 'local_hello'));
    $ADMIN->add('localplugins', $settings);

    $settings->add(new admin_setting_configtext(
        'local_hello/greeting',
        get_string('greeting_setting', 'local_hello'),
        get_string('greeting_setting_desc', 'local_hello'),
        'Hello',
        PARAM_TEXT
    ));

    $settings->add(new admin_setting_configcheckbox(
        'local_hello/enabled', 'Enabled', '', 1));
}
```

```php
$greeting = get_config('local_hello', 'greeting');
$all      = get_config('local_hello');
set_config('greeting', 'Привет', 'local_hello');
```

Своя админ-страница вместо панели настроек:

```php
$ADMIN->add('tools', new admin_externalpage('local_hello_admin',
    get_string('pluginname', 'local_hello'),
    new moodle_url('/local/hello/admin.php'), 'local/hello:manage'));
```

### Вопросы и ответы

**Как сделать настройки плагина?** Коротко: settings.php с admin_settingpage и admin_setting_config*, значения через get_config и set_config. Подвох: без проверки $hassiteconfig файл настроек может выполняться в контексте, где нет нужных прав.

---

## 16. События и наблюдатели

### Теория

Events API это шина событий: ядро и плагины бросают события («пользователь вошёл», «курс завершён»), другие плагины подписываются через db/events.php, обработчик это метод класса. События автоматически пишутся в журнал (Reports, Logs), поэтому у них должны быть осмысленные crud (c, r, u, d), edulevel, имя и описание. Наблюдатель получает объект события с данными (userid, relateduserid, objectid, contextid, other). По умолчанию наблюдатели выполняются сразу, но можно отложить до фиксации транзакции (internal=false).

Тяжёлую работу в наблюдателе делать не нужно: поставь adhoc-задачу в очередь.

### Код

Свой класс события, classes/event/message_created.php:

```php
<?php
namespace local_hello\event;

class message_created extends \core\event\base {

    protected function init() {
        $this->data['crud'] = 'c';
        $this->data['edulevel'] = self::LEVEL_OTHER;
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

Подписка, db/events.php:

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

classes/observer.php:

```php
<?php
namespace local_hello;

class observer {
    public static function course_completed(\core\event\course_completed $event) {
        $userid = $event->relateduserid;
        // минимум работы: записать и поставить adhoc-задачу
    }
}
```

### Вопросы и ответы

**Что такое Events API и как им пользоваться?** Коротко: ядро и плагины бросают события (класс наследует \core\event\base, ->trigger()), другие плагины подписываются через db/events.php, обработчик это метод класса. События пишутся в журнал. Подвох: тяжёлую работу в наблюдателе не делай, поставь adhoc-задачу.

**Чем событие отличается от hook?** Коротко: событие это уведомление «что-то произошло», пишется в журнал, наблюдатели не меняют результат. Hook это типизированная точка расширения, где плагин может влиять на поведение (например, добавить HTML).

---

## 17. Задачи cron: scheduled и adhoc

### Теория

Scheduled task выполняется по расписанию из db/tasks.php, подходит для регулярных операций (очистка, синхронизация). Adhoc task ставится в очередь для разового фонового выполнения, подходит для тяжёлых действий по требованию (рассылка, пересчёт), данные передаются через set_custom_data. Всё выполняет cron Moodle, который должен быть настроен в ОС каждую минуту. Задача должна быть идемпотентной, потому что при сбое её могут запустить повторно.

### Код

classes/task/cleanup.php:

```php
<?php
namespace local_hello\task;

class cleanup extends \core\task\scheduled_task {
    public function get_name() {
        return get_string('task_cleanup', 'local_hello');
    }

    public function execute() {
        global $DB;
        $DB->delete_records_select('local_hello_msg', 'timecreated < ?',
            [time() - YEAR_IN_SECS]);
    }
}
```

db/tasks.php:

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

Adhoc:

```php
namespace local_hello\task;

class send_report extends \core\task\adhoc_task {
    public function execute() {
        $data = $this->get_custom_data();
        // тяжёлая работа с $data->userid
    }
}

// Поставить в очередь:
$task = new \local_hello\task\send_report();
$task->set_custom_data(['userid' => $USER->id]);
\core\task\manager::queue_adhoc_task($task);
```

Cron в ОС: `* * * * * php /path/admin/cli/cron.php`

### Вопросы и ответы

**Какие бывают задачи cron и когда что использовать?** Коротко: scheduled по расписанию (db/tasks.php), adhoc для разовой фоновой работы. Выполняет cron Moodle. Подвох: задача должна быть идемпотентной, при сбое её могут запустить повторно.

**Почему не отправлять письмо прямо в обработчике запроса?** Коротко: это замедляет запрос пользователя и усложняет повтор при ошибке. Ставим adhoc-задачу.

---

## 18. Веб-сервисы и external functions

### Теория

External function это встроенный механизм API. Функция описывается в db/services.php и реализуется классом с методами execute_parameters, execute, execute_returns. Одна функция работает и через REST для внешних клиентов (в том числе мобильного приложения), и как AJAX-вызов из JS страницы (флаг ajax). Внутри execute порядок такой: validate_parameters, validate_context, require_capability, затем логика. Возвращаемая структура описывается через external_single_structure и external_multiple_structure.

До Moodle 4.2 классы external_* были глобальными, начиная с 4.2 они в пространстве core_external. Внешнему клиенту нужен токен: админ включает веб-сервисы и протокол (REST), создаёт сервис и токен.

### Код

db/services.php:

```php
<?php
defined('MOODLE_INTERNAL') || die();

$functions = [
    'local_hello_get_messages' => [
        'classname'    => 'local_hello\external\get_messages',
        'description'  => 'Returns messages of the current user.',
        'type'         => 'read',
        'ajax'         => true,
        'capabilities' => 'local/hello:view',
    ],
];

$services = [
    'Hello service' => [
        'functions' => ['local_hello_get_messages'],
        'restrictedusers' => 0,
        'enabled' => 1,
    ],
];
```

classes/external/get_messages.php:

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

        $params = self::validate_parameters(self::execute_parameters(), ['limit' => $limit]);

        $context = \context_system::instance();
        self::validate_context($context);
        require_capability('local/hello:view', $context);

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

Вызов внешним клиентом:

```
POST https://site/webservice/rest/server.php
  wstoken=...&wsfunction=local_hello_get_messages&moodlewsrestformat=json&limit=5
```

### Вопросы и ответы

**Как сделать API, чтобы внешний клиент и мой JS могли вызывать код плагина?** Коротко: external function: объявить в db/services.php, реализовать класс с execute_parameters, execute, execute_returns. Флаг ajax даёт вызов из JS, для внешних клиентов настраивают сервис и токен. Подвох: валидация параметров и проверка прав обязательны, функцию можно вызвать напрямую, минуя интерфейс.

**Как реализовать AJAX-кнопку, которая меняет данные без перезагрузки?** Коротко: external function с ajax=true, JS-модуль вызывает её через core/ajax, затем обновляет интерфейс через core/templates. На сервере проверка параметров и права. Не забыть собрать JS (grunt amd). Подвох: нельзя полагаться на проверку на клиенте.

---

## 19. JavaScript: AMD и ES6

### Теория

JS пишется модулями. Исходники лежат в amd/src (можно писать на ES6 с import/export), собираются Grunt-ом в amd/build, собранные файлы (.min.js и .map) коммитят в репозиторий, потому что на сервере сборки нет. Подключение со страницы: $PAGE->requires->js_call_amd('компонент/модуль', 'функция', [параметры]). Серверные вызовы идут через core/ajax к external functions, строки через core/str, шаблоны через core/templates, уведомления через core/notification. Другие стандартные модули: core/modal_factory, core/toast, core/pending (для Behat), core/fragment.

### Код

amd/src/main.js:

```js
import Ajax from 'core/ajax';
import Notification from 'core/notification';
import Templates from 'core/templates';

const loadMessages = async (limit) => {
    return Ajax.call([{
        methodname: 'local_hello_get_messages',
        args: {limit},
    }])[0];
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

Подключение и сборка:

```php
$PAGE->requires->js_call_amd('local_hello/main', 'init', [10]);
```

```bash
nvm install && nvm use      # версия Node из .nvmrc
npm install
npx grunt amd               # собрать JS (или: npx grunt amd --root=local/hello)
npx grunt watch
```

### Вопросы и ответы

**Как подключить JavaScript и как он собирается?** Коротко: исходники в amd/src, сборка Grunt-ом в amd/build, подключение через js_call_amd. Подвох: «не обновляется JS» обычно значит, что не выполнен grunt amd или не сброшены кэши (или включён cachejs).

---

## 20. Hooks и callbacks

### Теория

Старый способ расширения: функции с особым именем в lib.php плагина (callbacks). Новый способ, начиная с Moodle 4.3: типизированные hooks. Каждый hook это класс, подписка в db/hooks.php, обработчик это метод класса. Hooks дают явные типы, приоритеты, удобство тестирования и документацию. Старые callbacks постепенно помечаются устаревшими и удаляются. Не все точки ядра уже переведены на hooks, поэтому смотри документацию своей версии. Свои hooks можно объявлять в своём плагине, чтобы другие расширяли твой код.

### Код

Старый callback в lib.php:

```php
function local_hello_extend_navigation_course(navigation_node $parentnode,
        stdClass $course, context_course $context) {
    if (has_capability('local/hello:view', $context)) {
        $url = new moodle_url('/local/hello/index.php', ['id' => $course->id]);
        $parentnode->add(get_string('pluginname', 'local_hello'), $url,
            navigation_node::TYPE_CUSTOM);
    }
}
```

Другие известные callbacks: local_hello_extend_navigation, local_hello_extend_settings_navigation, local_hello_pluginfile, local_hello_before_footer.

Новый hook, db/hooks.php:

```php
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

classes/hook_callbacks.php:

```php
<?php
namespace local_hello;

class hook_callbacks {
    public static function before_footer(\core\hook\output\before_footer_html_generation $hook): void {
        $hook->add_html('<!-- hello -->');
    }
}
```

### Вопросы и ответы

**Hooks и callbacks в lib.php: в чём разница?** Коротко: callbacks старый способ, функции с особым именем в lib.php. Hooks (с 4.3) типизированные классы, подписка в db/hooks.php. Новое hooks, старые callbacks постепенно устаревают. Подвох: не все точки ядра переведены на hooks. Смотри документацию своей версии и не выдумывай имена hooks.

---

## 21. Files API

### Теория

Файл определяется набором: contextid, component, filearea, itemid, filepath, filename. Метаданные лежат в таблице files, содержимое в moodledata. Из формы файлы приходят в черновую область (draft), их переносят в постоянную через file_save_draft_area_files. Отдача идёт через callback pluginfile, где обязательно проверять вход и права, затем send_stored_file. Ссылку строят через moodle_url::make_pluginfile_url.

### Код

```php
// В форме
$mform->addElement('filemanager', 'attachments', 'Files', null,
    ['subdirs' => 0, 'maxfiles' => 5]);

// После get_data()
file_save_draft_area_files($data->attachments, $context->id,
    'local_hello', 'attachment', $itemid, ['subdirs' => 0, 'maxfiles' => 5]);

// Чтение
$fs = get_file_storage();
$files = $fs->get_area_files($context->id, 'local_hello', 'attachment',
    $itemid, 'filename', false);
foreach ($files as $file) {
    $url = moodle_url::make_pluginfile_url($file->get_contextid(),
        $file->get_component(), $file->get_filearea(), $file->get_itemid(),
        $file->get_filepath(), $file->get_filename());
}
```

Callback отдачи в lib.php:

```php
function local_hello_pluginfile($course, $cm, $context, $filearea, $args,
        $forcedownload, array $options = []) {
    if ($context->contextlevel != CONTEXT_SYSTEM) {
        return false;
    }
    require_login();
    require_capability('local/hello:view', $context);   // обязательно

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

### Вопросы и ответы

**Как Moodle хранит и отдаёт файлы?** Коротко: через Files API, файл задаётся contextid, component, filearea, itemid, filepath и filename. Метаданные в таблице files, содержимое в moodledata. Отдача через callback pluginfile. Подвох: отдача файла без проверки прав классическая дыра.

---

## 22. Cache API

### Теория

Cache API (MUC) хранит дорого вычисляемые данные. Определение в db/caches.php, строка cachedef_имя в языковом файле, использование через \cache::make(компонент, определение). Режимы: application (общий), session, request. Хранилище настраивается админом (файлы, Redis, Memcached). Можно задавать ttl, простые ключи (simplekeys), статическое ускорение.

### Код

```php
// db/caches.php
$definitions = [
    'messages' => [
        'mode' => cache_store::MODE_APPLICATION,
        'simplekeys' => true,
        'simpledata' => false,
        'ttl' => 3600,
    ],
];
```

```php
// lang/en/local_hello.php
$string['cachedef_messages'] = 'Messages cache';
```

```php
$cache = \cache::make('local_hello', 'messages');
$data = $cache->get($key);
if ($data === false) {
    $data = expensive_calculation();
    $cache->set($key, $data);
}
$cache->delete($key);
$cache->purge();
```

### Вопросы и ответы

**Как работает кэширование (MUC)?** Коротко: определение в db/caches.php, использование \cache::make, методы get, set, delete, purge. Режимы application, session, request. Подвох: кэшируя данные, зависящие от пользователя, учитывай контекст и права. Общий кэш не должен «подсовывать» одному пользователю чужие данные.

---

## 23. Message API

### Теория

Message API отправляет уведомления по каналам, настроенным у пользователя (email, push, внутренние сообщения). Тип сообщения объявляется в db/messages.php, для него нужна строка messageprovider:имя в языковом файле. Для разовых баннеров на странице используют \core\notification::success() или redirect с типом уведомления.

### Код

```php
// db/messages.php
$messageproviders = [
    'newmessage' => [],
];
// lang: $string['messageprovider:newmessage'] = 'New message notification';
```

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
$message->notification      = 1;
message_send($message);
```

### Вопросы и ответы

**Как отправить пользователю уведомление?** Коротко: объявить провайдер в db/messages.php, собрать \core\message\message и вызвать message_send. Подвох: без строки messageprovider:имя провайдер отображается некорректно.

---

## 24. Privacy API

### Теория

Privacy API нужен для соответствия GDPR: каждый плагин обязан описать, какие персональные данные он хранит, и уметь их экспортировать и удалять. Три случая. Плагин не хранит данные: null_provider. Плагин хранит данные: реализуются metadata\provider (описание) и plugin\provider (экспорт и удаление для пользователя и для контекста). Плагин только передаёт данные во внешнюю систему: metadata\provider с описанием внешней системы. Строки с описанием лежат в языковом файле (privacy:metadata:...).

### Код

Минимальный provider показан в разделе 9. Каркас для плагина с данными:

```php
namespace local_hello\privacy;

use core_privacy\local\metadata\collection;
use core_privacy\local\request\{approved_contextlist, approved_userlist, contextlist, userlist};

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

### Вопросы и ответы

**Зачем нужен Privacy API?** Коротко: GDPR. Плагин описывает, какие персональные данные хранит, умеет экспортировать и удалять их. Если данных нет, null_provider. Подвох: плагин без provider не пройдёт проверку каталога и вызовет проблемы на сайтах с запросами на удаление данных.

---

## 25. Модуль активности mod

### Теория

Самый «тяжёлый» тип плагина: активность в курсе, как форум или тест. Нужен, когда преподаватель добавляет её в курс, выставляет оценки, ограничивает доступ и включает в резервные копии. Для остального проще local.

Обязательные файлы: version.php, lib.php (hello_add_instance, hello_update_instance, hello_delete_instance, hello_supports), mod_form.php, view.php, index.php (список активностей в курсе), db/install.xml (таблица экземпляров), db/access.php (права addinstance и view), языковой файл lang/en/hello.php. Файл строк называется без префикса mod_, функции начинаются с hello_, а компонент mod_hello. Таблица экземпляров называется по имени модуля, в ней обязательны id, course, name, intro, introformat, timemodified.

Для полноценной интеграции добавляют резервное копирование (backup/moodle2), интеграцию с журналом оценок (grade_update, FEATURE_GRADE_HAS_GRADE) и отслеживание завершения (FEATURE_COMPLETION_*, custom completion). Чтобы не писать каркас вручную, используют tool_pluginskel или смотрят на mod_newmodule как на шаблон.

### Код

lib.php:

```php
<?php
defined('MOODLE_INTERNAL') || die();

function hello_supports($feature) {
    switch ($feature) {
        case FEATURE_MOD_INTRO:         return true;
        case FEATURE_BACKUP_MOODLE2:    return true;
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

mod_form.php:

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

        $this->standard_intro_elements();
        $this->standard_coursemodule_elements();
        $this->add_action_buttons();
    }
}
```

view.php:

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

### Вопросы и ответы

**Из чего состоит модуль активности?** Коротко: version.php, lib.php с add/update/delete instance и supports, mod_form.php, view.php, index.php, install.xml с таблицей экземпляров, access.php с правами, языковой файл. Подвох: файл строк mod-плагина называется hello.php, без префикса mod_.

**Как активность интегрируется с оценками и завершением?** Коротко: оценки через grade_update и функцию вроде hello_grade_item_update, завершение через FEATURE_COMPLETION_* в supports и классы custom_completion для своих правил. Подвох: для таких деталей на собеседовании честно говори, какие функции помнишь, а точные сигнатуры смотришь в документации.

**Когда выбирать mod, а когда local?** Коротко: mod если нужен вид активности, которую преподаватель добавляет в курс, оценивает, ограничивает по доступу и включает в бэкапы. Иначе local.

---

## 26. Блок block

### Теория

Блок это виджет в боковой панели или на страницах (профиль, курс, главная). Класс block_имя наследует block_base, файл blocks/имя/block_имя.php (имя файла равно имени класса). Методы: init (заголовок), get_content (содержимое), applicable_formats (где можно добавлять), has_config (включает settings.php), instance_allow_multiple (несколько экземпляров). Права addinstance и myaddinstance в access.php обязательны, иначе блок нельзя добавить. Для сложного контента используют renderer и Mustache.

### Код

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
        return true;
    }
}
```

Права db/access.php: block/hello:addinstance и block/hello:myaddinstance.

### Вопросы и ответы

**Как устроен блок?** Коротко: класс block_имя extends block_base с методами init, get_content, applicable_formats. Права addinstance и myaddinstance. Подвох: без права addinstance блок нельзя добавить на страницу.

---

## 27. Другие типы плагинов

### Теория

Theme: обычно наследуют Boost ($THEME->parents = ['boost']), переопределяют SCSS и шаблоны. Правки темы делают через дочернюю тему, а не правкой Boost. Auth: класс auth_plugin_xxx extends auth_plugin_base, метод user_login($username, $password). Enrol: класс enrol_xxx_plugin extends enrol_plugin. Filter: класс filter_xxx extends moodle_text_filter, метод filter($text, $options). Tool: как local, но в admin/tool, админ-утилиты. Report: отчёты в курсе или на сайте, добавляются в навигацию через report_xxx_extend_navigation_course. Format: класс format_xxx extends core_courseformat\base. Qtype: новые типы вопросов (questiontype.php, question.php, renderer.php).

### Вопросы и ответы

**Как бы вы сделали страницу отчёта только для преподавателей курса?** Коротко: плагин типа report (или local) со страницей, где require_login($course), контекст курса и require_capability с собственным правом, которое по архетипу получает editingteacher и teacher. Глубже: право в access.php (captype read, CONTEXT_COURSE). Страница получает id курса через required_param, строит SQL по записанным пользователям с учётом групп, выводит через Mustache, добавляет пункт в навигацию курса через callback или hook. Большие отчёты разбивают на страницы. Подвох: не забыть режим групп: преподаватель должен видеть только своих студентов.

---

## 28. Тестирование: PHPUnit и Behat

### Теория

PHPUnit для PHP: тесты лежат в tests/, имя файла кончается на _test.php, класс наследует advanced_testcase, resetAfterTest откатывает БД после теста, генераторы создают пользователей, курсы, записи, активности. PHPUnit использует отдельную БД и dataroot, они настраиваются в config.php ($CFG->phpunit_prefix, $CFG->phpunit_dataroot), инициализация командой admin/tool/phpunit/cli/init.php.

Behat для сценариев в браузере на Gherkin (Selenium/Chrome). CI для плагинов: moodle-plugin-ci (phpcs, phpdoc, phpunit, behat, grunt, mustache lint).

### Код

```php
<?php
namespace local_hello;

/**
 * @covers \local_hello\manager
 */
final class manager_test extends \advanced_testcase {

    public function test_add_message(): void {
        global $DB;
        $this->resetAfterTest();
        $user = $this->getDataGenerator()->create_user();
        $this->setUser($user);

        $id = manager::add_message($user->id, 'Hi');

        $this->assertTrue($DB->record_exists('local_hello_msg', ['id' => $id]));
    }
}
```

Генераторы: create_user(), create_course(), create_module('forum', [...]), enrol_user(...).

```bash
php admin/tool/phpunit/cli/init.php
vendor/bin/phpunit --filter manager_test
vendor/bin/phpunit local/hello/tests/manager_test.php
```

Behat, tests/behat/hello.feature:

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

```bash
php admin/tool/behat/cli/init.php
vendor/bin/behat --tags=@local_hello
```

### Вопросы и ответы

**Как тестировать плагины?** Коротко: PHPUnit для PHP (advanced_testcase, resetAfterTest, генераторы), Behat для сценариев в браузере, CI на moodle-plugin-ci. Подвох: забытый resetAfterTest приводит к «протеканию» данных между тестами. Если тест падает с «database not initialised», не выполнен phpunit init.

---

## 29. Отладка и производительность

### Теория

Отладка: режим DEVELOPER (в настройках или в config.php), логи веб-сервера и PHP-FPM, debugging('message', DEBUG_DEVELOPER), Xdebug с IDE, сброс кэшей, режим дизайнера темы, perfdebug для информации о производительности внизу страницы. Для проверки стиля: Code checker (local_codechecker) и phpcs с набором правил Moodle.

Чек-лист «ничего не обновляется». Изменил PHP: обнови страницу. Изменил строки, шаблоны, классы или db/*.php: сбрось кэши. Изменил схему или права: подними version и запусти upgrade. Изменил JS: grunt amd.

Оптимизация медленной страницы. Сначала измерения (профилирование, perfdebug), потом правки. Убрать запросы в цикле (N+1), использовать JOIN, get_fast_modinfo вместо отдельных запросов по активностям, recordset для больших выборок, постраничность. Добавить индексы по полям фильтрации. Кэшировать дорогие данные. Вынести тяжёлое в adhoc-задачи. На проде кэши и сессии в Redis, включить OPcache. Для тяжёлых отчётов предрасчёт по cron.

### Вопросы и ответы

**Как отлаживать и что делать, если «ничего не обновляется»?** Коротко: режим DEVELOPER, логи, Xdebug. Для «не обновляется»: purge_caches, поднять version при изменении db/, пересобрать JS. Подвох: правка файла на сервере без сброса кэша причина большинства «Moodle не видит мои изменения».

**Как оптимизировать медленную страницу?** Коротко: найти узкое место, убрать N+1, добавить индексы, кэшировать, вынести тяжёлое в adhoc-задачи. Подвох: не оптимизировать вслепую, сначала измерения.

---

## 30. Стиль кода, публикация, обновление сайта

### Теория

Стиль Moodle. Отступ 4 пробела, строки желательно не длиннее ~132 символов, файл PHP без закрывающего ?>. Каждый файл начинается с GPL-заголовка и docblock с @package, @copyright, @license. В файлах без чистых классов: defined('MOODLE_INTERNAL') || die();. Docblock и комментарии на английском. В новом коде допустим PSR-12-подобный стиль, смотри актуальные рекомендации на moodledev.io.

Публикация в каталоге плагинов: репозиторий в Git, проходят проверки moodle-plugin-ci, есть README, CHANGES, лицензия GPL v3+, privacy provider, английские строки, регистрация на moodle.org/plugins.

Обновление сайта с плагинами. Правки ядра ломаются при обновлении и делают поддержку невозможной, поэтому все изменения делают плагинами (в том числе дочерней темой вместо правки Boost). Перед обновлением Moodle: резервная копия БД и moodledata, тест на копии, проверка совместимости плагинов (requires и supported-версии), затем php admin/cli/maintenance.php --enable, php admin/cli/upgrade.php, отключение режима обслуживания.

### Вопросы и ответы

**Почему нельзя править ядро и как обновлять сайт с плагинами?** Коротко: правки ядра пропадают или ломаются при обновлении. Всё делаем плагином. Перед апгрейдом проверяем совместимость плагинов и тестируем на копии. Подвох: правки темы делают через дочернюю тему, а не правкой Boost.

---

## 31. Сценарные вопросы

**Сделать: при завершении курса отправлять письмо и записывать факт в свою таблицу.** Коротко: local-плагин с наблюдателем на \core\event\course_completed в db/events.php. В обработчике запись в свою таблицу и постановка adhoc-задачи на отправку письма через Message API. Глубже: таблица в install.xml, право и privacy provider сразу. Обработчик делает минимум (insert и queue_adhoc_task), письмо уходит в задаче через message_send. Для настроек (включить, текст письма) settings.php. PHPUnit-тест на наблюдатель. Подвох: не отправляй письмо прямо в наблюдателе, это замедлит запрос пользователя и усложнит повтор при ошибке.

Скелет решения:

```php
// db/events.php
$observers = [[
    'eventname' => '\core\event\course_completed',
    'callback'  => '\local_hello\observer::course_completed',
]];

// classes/observer.php
public static function course_completed(\core\event\course_completed $event) {
    global $DB;
    $DB->insert_record('local_hello_log', (object)[
        'userid' => $event->relateduserid,
        'courseid' => $event->courseid,
        'timecreated' => time(),
    ]);
    $task = new \local_hello\task\send_completion_mail();
    $task->set_custom_data(['userid' => $event->relateduserid, 'courseid' => $event->courseid]);
    \core\task\manager::queue_adhoc_task($task);
}
```

**Сделать страницу отчёта для преподавателей курса.** См. раздел 27: report или local, require_login($course), право с архетипом editingteacher, учёт групп, постраничность.

**Сделать AJAX-кнопку.** См. раздел 18: external function с ajax, JS через core/ajax, обновление через core/templates, проверка прав на сервере.

**Страница открывается без входа / пользователь видит чужие данные. Что проверить?** Коротко: require_login, require_capability в правильном контексте, фильтрация данных по $USER->id или по курсу, проверка в external functions и pluginfile.

**Плагин ведёт себя по-разному на разных СУБД. Почему?** Коротко: использован SQL, специфичный для одной СУБД, не применены $DB->sql_* хелперы.

Таблица частых симптомов:

```
Плагин не виден на странице уведомлений    папка не там, имя папки не равно component, нет version.php или lang/en
Class not found после добавления класса    не сброшены кэши, неверный namespace или путь
Строка показывается как [[key]]            нет ключа в lang/en или не сброшен кэш строк
Новое право не работает                    не поднята version, не запущен upgrade, нет строки названия
Изменения install.xml не применились       он выполняется только при первой установке, нужен upgrade.php и version
Coding error: page url not set             не вызван $PAGE->set_url()
Потерялись строки из get_records           первое поле выборки не уникально
Invalid sesskey                            опасное действие без sesskey
JS не обновляется                          не выполнен grunt amd, не сброшены кэши, включён cachejs
Страница открывается без входа             забыли require_login()
Пользователь видит чужие данные            нет require_capability или фильтрации по пользователю/курсу
```

**Какие самые частые ошибки новичков?** Коротко: забывают require_login и require_capability, обращаются к $_GET, склеивают SQL, не поднимают version, не сбрасывают кэши, путают cm id и instance, не пишут Privacy provider. Глубже: ещё не задают $PAGE->set_url, выводят HTML без экранирования, используют PARAM_RAW везде, не закрывают recordset, не собирают JS, пишут тексты в коде вместо языковых строк, правят ядро. Подвох: на собеседовании полезно говорить не только «как правильно», но и «как это обычно ломается». Это показывает практический опыт.

---

## 32. Как отвечать, если опыта мало, и что спросить у интервьюера

**Что говорить, если не знаешь или не помнишь деталей API?** Коротко: честно обозначить границу и показать, как ты это решаешь: назвать подсистему и принцип, сказать, что точную сигнатуру проверишь на moodledev.io или по исходникам ядра. Шаблон: «Принцип такой... Точное имя метода в этой версии я бы проверил в документации. Если нужно, посмотрю, как это сделано в ядре, например в mod_forum». Подвох: не выдумывай названия функций. Лучше «не помню точно», чем несуществующее имя.

**Какие вопросы задать интервьюеру?** Какая версия Moodle в проде и как часто обновляетесь. Есть ли PHPUnit и Behat в CI. Какие плагины свои, а какие сторонние. Есть ли форки ядра (хороший признак, если нет). Как организованы отладка и деплой плагинов. Инфраструктура: кэши, cron, масштабирование. Как устроен code review. Вопросы про зарплату и график оставь на конец или на отдельный этап.

Образцы для разбора в исходниках ядра: mod/page (простой mod), mod/forum (сложный), blocks/html, примеры в admin/tool.

---

## 33. Шпаргалка перед собеседованием

Где что объявляется:

```
Версия и зависимости        version.php
Таблицы                     db/install.xml, миграции db/upgrade.php
Права                       db/access.php
Строки                      lang/en/компонент.php
Настройки                   settings.php
Подписка на события         db/events.php
Задачи                      db/tasks.php и classes/task/
Веб-сервисы                 db/services.php и classes/external/
Кэши                        db/caches.php
Сообщения                   db/messages.php
Hooks                       db/hooks.php
Шаблоны                     templates/*.mustache
JS                          amd/src, затем amd/build
Privacy                     classes/privacy/provider.php
```

Минимальный набор файлов local_xxx: version.php, lang/en/local_xxx.php (pluginname), classes/privacy/provider.php.

Цикл разработки: правка кода, затем при изменении db, lang, классов, шаблонов сброс кэшей, при изменении version или db запуск upgrade, затем проверка в браузере.

Пять фраз, которые стоит произнести в ответах:

1. «Ядро не правим, расширяем плагином».
2. «Права проверяем на сервере, в нужном контексте, через capability, а не через роль».
3. «Весь SQL через $DB и плейсхолдеры, вывод экранируем».
4. «Изменил схему или права: поднимаю version и делаю upgrade».
5. «Тяжёлое выношу в adhoc-задачи, часто читаемое кэширую».

Где проверить детали: https://moodledev.io (документация разработчика), tracker.moodle.org (баги), форумы moodle.org.