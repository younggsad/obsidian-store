---
custom-width: 80
---
---
## Оглавление

1. [[#1. Основы]]
2. [[#2. Глобальные объекты и страница]]
3. [[#3. Курсы, записи, группы]]
4. [[#4. Контексты, роли, права]]
5. [[#5. Плагины]]
6. [[#6. База данных]]
7. [[#7. Параметры и безопасность]]
8. [[#8. Формы, вывод, строки, настройки]]
9. [[#9. События и cron]]
10. [[#10. Web services и JavaScript]]
11. [[#11. Hooks, файлы, кэш, сообщения, Privacy]]
12. [[#12. Модули и блоки]]
13. [[#13. Тесты, отладка, обновление]]
14. [[#14. Инфраструктура и интеграции]]
15. [[#15. Встроенные подсистемы]]
16. [[#16. Темы]]
17. [[#17. Работа с чужим кодом]]
18. [[#18. PHP, SQL, Git, веб]]

---

## 1. Основы

> [!question] Что такое Moodle?
> 
> LMS на PHP под лицензией GPL v3. Расширяется плагинами, ядро не правят.

> [!question] Что такое LMS?
> 
> Система управления обучением: курсы, записи на курс, тесты, задания, журнал оценок.

> [!question] Из чего состоит установка Moodle?
> 
> Из кода, базы данных и каталога moodledata. Переносить нужно все три части.

> [!question] Что такое moodledata (`$CFG->dataroot`)?
> 
> Каталог вне веб-корня с файлами пользователей, кэшами, временными файлами и сессиями.

> [!question] Почему moodledata не отдают напрямую через веб-сервер?
> 
> Там приватные файлы, прямая отдача обходит проверку прав. Файлы идут через `pluginfile.php`.

> [!question] Что значит «no core hacking»?
> 
> Ядро не правят. Любое изменение делают плагином, событием, хуком или темой.

> [!question] Как Moodle хранит файлы?
> 
> По SHA-1 содержимого (дедупликация): на диске один файл, в таблице `{files}` много записей.

> [!question] Что такое LTS?
> 
> Версия с долгой поддержкой безопасности (около 3 лет), выходит раз в два года: 4.1, 4.5, 5.3.

> [!question] Как часто выходят релизы Moodle?
> 
> Два раза в год: в апреле и в октябре.

> [!question] Как записывается версия ядра?
> 
> Числом `YYYYMMDDXX` (например `2026042000`) и читаемым `$release` (например `5.2.1`).

> [!question] Какой PHP нужен Moodle?
> 
> 4.5 — 8.1–8.3; 5.0 и 5.1 — 8.2–8.4; 5.2 и 5.3 — 8.3–8.4. Всегда 64-битный.

> [!question] Какие СУБД поддерживает Moodle?
> 
> MySQL/MariaDB, PostgreSQL, MS SQL Server. Oracle не поддерживается с 5.0.

> [!question] Что такое `public/` в Moodle 5.0?
> 
> Папка веб-корня. `config.php` и `vendor/` лежат выше, вне веб-корня. Внутри плагинов ничего не меняется.

> [!question] Что такое `config.php`?
> 
> Корневой конфиг: БД, `wwwroot`, `dataroot`, `prefix`. Последней строкой подключает `lib/setup.php`.

> [!question] Что делает `lib/setup.php`?
> 
> Запускает ядро: читает `$CFG`, подключает БД, стартует сессию, включает автозагрузку классов.

> [!question] Что такое moodle-docker?
> 
> Готовые контейнеры для разработки Moodle, в том числе для Behat.

> [!question] Как Moodle обрабатывает запрос?
> 
> Запускается конкретный PHP-скрипт → `config.php` → `require_login()` → проверка прав → настройка `$PAGE` → вывод через `$OUTPUT`.

> [!question] Есть ли в Moodle единый front controller?
> 
> Нет, каждая страница — отдельный PHP-файл (`view.php`, `index.php`). В 5.x добавлен роутер для новых точек входа.

---

## 2. Глобальные объекты и страница

> [!question] Что такое `$CFG`?
> 
> Объект настроек сайта.

> [!question] Что такое `$DB`?
> 
> Объект доступа к базе данных (`moodle_database`).

> [!question] Что такое `$USER`?
> 
> Текущий пользователь.

> [!question] Что такое `$PAGE`?
> 
> Состояние страницы: URL, контекст, заголовок, подключённые JS и CSS.

> [!question] Что такое `$OUTPUT`?
> 
> Объект вывода элементов интерфейса (`header()`, `footer()`, `render_from_template()`).

> [!question] Что такое `$COURSE`?
> 
> Текущий курс.

> [!question] Что такое `$SESSION`?
> 
> Данные текущей сессии пользователя.

> [!question] Что делает `require_login()`?
> 
> Проверяет, что пользователь вошёл. С `$course` и `$cm` проверяет ещё доступ к курсу и активности. Нужен на каждой странице.

> [!question] Чем `require_capability()` отличается от `has_capability()`?
> 
> `has_capability()` возвращает true/false, `require_capability()` бросает исключение, если права нет.

> [!question] В каком порядке строится страница плагина?
> 
> `config.php` → `require_login` → контекст → `require_capability` → `$PAGE` (url, context, title, heading) → `header` → контент → `footer`.

> [!question] Для чего `admin_externalpage_setup()`?
> 
> Настраивает `$PAGE` для страницы админки, описанной в `settings.php`.

> [!question] Почему глобальные объекты не любят в новом коде?
> 
> Они скрывают зависимости и мешают тестированию. Лучше передавать нужное параметрами или через DI.

---

## 3. Курсы, записи, группы

> [!question] Что такое cm (course module)?
> 
> Размещение активности в курсе, запись в `{course_modules}`. Её id — cmid.

> [!question] Что такое instance активности?
> 
> Сама активность: запись в таблице плагина (например `{forum}`). На неё ссылается `course_modules.instance`.

> [!question] Чем cmid отличается от instance id?
> 
> `cmid` — номер размещения в курсе (нужен для URL и контекста), `instance id` — номер записи в таблице модуля. Они не взаимозаменяемы.

> [!question] От какого id строится контекст модуля?
> 
> От cmid: `context_module::instance($cm->id)`.

> [!question] Что делает `get_fast_modinfo($course)`?
> 
> Возвращает кэшированную структуру курса без лишних запросов к БД. `->get_cm($cmid)` даёт `cm_info`.

> [!question] Что такое `cm_info`?
> 
> Объект активности из modinfo. Поле `uservisible` показывает, видна ли она пользователю.

> [!question] Что такое enrolment?
> 
> Запись пользователя на курс через плагин `enrol_*`. Роль назначается отдельно.

> [!question] Что такое `{enrol}`?
> 
> Экземпляр способа записи в конкретном курсе (например самозапись с ключом).

> [!question] Что такое `{user_enrolments}`?
> 
> Запись конкретного пользователя на курс через способ записи.

> [!question] Что такое `{role_assignments}`?
> 
> Назначенные роли пользователей в контекстах. Удаление записи не всегда убирает роль.

> [!question] Как проверить, что пользователь записан на курс?
> 
> `is_enrolled($context, $user, $withcapability)`.

> [!question] Как получить список записанных пользователей?
> 
> `get_enrolled_users($context, $withcapability, $groupid)`.

> [!question] Как правильно записать пользователя на курс из кода?
> 
> Через плагин записи: `enrol_get_plugin('manual')->enrol_user($instance, $userid, $roleid)`, не прямым `INSERT` (иначе не сработают события).

> [!question] Что такое группа (group)?
> 
> Набор пользователей внутри курса. У активности три режима: нет групп, раздельные, видимые.

> [!question] Что такое grouping?
> 
> Набор групп курса.

> [!question] Что такое cohort?
> 
> Глобальная группа пользователей (на уровне системы или категории), удобна для массовой записи.

> [!question] Для чего право `moodle/site:accessallgroups`?
> 
> Позволяет видеть все группы, независимо от режима групп.

> [!question] Что такое completion API?
> 
> Отслеживание завершения. Активность сообщает через `completion_info::update_state($cm, COMPLETION_COMPLETE, $userid)`.

> [!question] Что такое gradebook и grade API?
> 
> Gradebook — журнал оценок. Активность отправляет оценки через `grade_update()`, обычно в функциях `{mod}_grade_item_update()` и `{mod}_update_grades()`.

> [!question] Что такое плагины `availability_*`?
> 
> Правила ограничения доступа к активностям и разделам курса.

---

## 4. Контексты, роли, права

> [!question] Что такое контекст?
> 
> Узел дерева, к которому привязаны права: система → категория курсов → курс → модуль → блок. Есть ещё контекст пользователя.

> [!question] Что такое capability?
> 
> Конкретное право в формате `тип/имя:право`, например `mod/forum:replypost` или `local/hello:view`.

> [!question] Что такое роль?
> 
> Набор разрешений (teacher, student, manager). Назначается в контексте и действует вниз по дереву.

> [!question] Какие значения может иметь право?
> 
> inherit, allow, prevent, prohibit.

> [!question] Что значит prohibit?
> 
> Жёсткий запрет, который нельзя переопределить в нижних контекстах (prevent можно).

> [!question] Что такое override?
> 
> Переопределение прав роли на уровне курса или активности.

> [!question] Где объявляют права плагина?
> 
> В `db/access.php`, с `captype`, `contextlevel` и `archetypes`. Название для интерфейса: `$string['hello:view']`.

> [!question] Почему нельзя проверять права по имени роли?
> 
> Роли настраиваются на сайте. Код проверяет capability в нужном контексте.

> [!question] Что делать после изменения `access.php`?
> 
> Поднять `version` плагина и запустить `upgrade.php`.

> [!question] Почему преподаватель может видеть данные чужого курса?
> 
> Нет `require_login()` или `require_capability()`, проверка в системном контексте вместо курса, или в SQL нет фильтра по `courseid`/`userid`.

---

## 5. Плагины

> [!question] Что такое frankenstyle?
> 
> Схема имён `тип_имя` (`local_hello`, `mod_forum`). По ней строятся компонент, папка, namespace, префиксы функций и таблиц.

> [!question] Что обязательно в `version.php`?
> 
> `$plugin->component`, `version` (формат `YYYYMMDDXX`), `requires` (минимальная версия Moodle). Также `maturity` и `release`.

> [!question] Как Moodle решает, что плагин надо установить или обновить?
> 
> Сравнивает `version` в файле с `config_plugins`: нет записи — установка, больше — upgrade, меньше — ошибка.

> [!question] Когда поднимать версию плагина?
> 
> При изменении схемы БД, `access.php`, событий, задач, служб, кэшей, всего, что читается при установке или обновлении.

> [!question] Что обязательно для плагина кроме `version.php`?
> 
> Строка `pluginname` в языковом файле и Privacy-провайдер.

> [!question] Что такое `$plugin->dependencies`?
> 
> Зависимость от версии другого плагина: без неё плагин не установится.

> [!question] Что такое `$plugin->supported` и `incompatible`?
> 
> Диапазон версий Moodle, с которыми плагин совместим, и версии, с которыми он несовместим.

> [!question] Что такое подплагин?
> 
> Расширение внутри плагина-родителя (`assignsubmission_*`, `quizaccess_*`). Описывается в `db/subplugins.json`.

> [!question] Что такое плагин `local`?
> 
> Проектная доработка вне модели активностей: страницы, интеграции, фоновые обработчики.

> [!question] Что такое плагин `mod`?
> 
> Активность или ресурс в курсе (форум, тест, задание).

> [!question] Что такое плагин `block`?
> 
> Блок: виджет на странице.

> [!question] Что такое плагин `theme`?
> 
> Оформление сайта.

> [!question] Что такое плагин `auth`?
> 
> Способ входа (OAuth2, LDAP, SAML, LTI).

> [!question] Что такое плагин `enrol`?
> 
> Способ записи на курс.

> [!question] Что такое плагин `format`?
> 
> Формат курса. Папка `/course/format`.

> [!question] Что такое плагин `tool`?
> 
> Административный инструмент. Папка `/admin/tool`.

> [!question] Что такое плагин `report`?
> 
> Отчёт.

> [!question] Что такое плагин `filter`?
> 
> Фильтр, преобразующий текст при выводе.

> [!question] Что такое плагин `repository`?
> 
> Источник файлов (внешнее хранилище).

> [!question] Что такое плагин `qtype`?
> 
> Тип вопроса для банка вопросов. Папка `/question/type`.

> [!question] Какой тип плагина выбрать под задачу?
> 
> Активность → `mod`; виджет → `block`; внешний вид → `theme`; вход → `auth`; запись → `enrol`; отчёт → `report`; остальное проектное → `local`.

> [!question] Плагин скопирован, но Moodle его не видит. Причины?
> 
> Не та папка; имя папки не совпадает с `component`; нет `version.php`; нет `pluginname`; нет прав чтения у веб-сервера.

> [!question] Почему «Class not found» после создания класса?
> 
> Карта классов закэширована. Выполнить `purge_caches.php`, проверить namespace и регистр букв.

---

## 6. База данных

> [!question] Как работать с БД в Moodle?
> 
> Только через `$DB`. `PDO`, `mysqli_*`, `pg_*` запрещены.

> [!question] Зачем нужен `$DB`?
> 
> Он даёт префиксы таблиц, защиту от инъекций, совместимость СУБД и транзакции.

> [!question] Как писать имена таблиц в SQL?
> 
> В фигурных скобках: `{user}`. Префикс `mdl_` подставляется сам.

> [!question] Как передавать значения в запрос?
> 
> Только плейсхолдерами `:name` или `?`. Не смешивать их в одном запросе, не повторять одно имя.

> [!question] Что делает `get_record()`?
> 
> Возвращает одну запись по условию.

> [!question] Что делает `get_records()`?
> 
> Возвращает массив записей. Ключ массива — первая колонка (обычно `id`), если она не уникальна, данные теряются.

> [!question] Что делает `get_recordset()`?
> 
> Читает большую выборку потоком, без загрузки в память. После работы `$rs->close()`.

> [!question] Что значат `MUST_EXIST` и `IGNORE_MISSING`?
> 
> `MUST_EXIST` — исключение, если записи нет. `IGNORE_MISSING` — вернуть `false`.

> [!question] Что возвращает `insert_record()`?
> 
> Id новой записи.

> [!question] Что нужно для `update_record()`?
> 
> Объект с полем `id`.

> [!question] Как сделать постраничную выборку?
> 
> Параметры `$limitfrom` и `$limitnum` в `get_records*`.

> [!question] Зачем `sql_like()`, `sql_concat()`, `get_in_or_equal()`?
> 
> Кроссплатформенные фрагменты SQL и безопасная подстановка списка значений.

> [!question] Как выполнить транзакцию?
> 
> `$tx = $DB->start_delegated_transaction(); ... $tx->allow_commit();`

> [!question] В чём разница между `install.xml` и `upgrade.php`?
> 
> `install.xml` — актуальная схема для первой установки, `upgrade.php` — шаги для существующих баз. Менять нужно оба и поднимать версию.

> [!question] Что обязательно в шаге `upgrade.php`?
> 
> Проверка `$oldversion`, идемпотентные операции (`field_exists`) и `upgrade_plugin_savepoint(true, версия, 'local', 'hello')`.

> [!question] Что такое XMLDB editor?
> 
> Инструмент админки (Development) для корректного редактирования `install.xml`.

> [!question] Как называют таблицы плагина?
> 
> С префиксом компонента: `local_hello_messages`.

> [!question] Что такое N+1?
> 
> Запрос к БД внутри цикла. Лечится одним запросом с `JOIN` или `IN` либо `get_fast_modinfo()`.

---

## 7. Параметры и безопасность

> [!question] Как получать данные запроса?
> 
> Через `required_param()` и `optional_param()` с типом `PARAM_*`. `$_GET` и `$_POST` напрямую не используют.

> [!question] Что такое `PARAM_*`?
> 
> Типы очистки входных данных: `PARAM_INT`, `PARAM_TEXT`, `PARAM_ALPHANUMEXT`, `PARAM_URL`, `PARAM_BOOL` и др. Берут самый узкий.

> [!question] Чем `PARAM_TEXT` отличается от `PARAM_RAW`?
> 
> `PARAM_RAW` ничего не чистит, `PARAM_TEXT` убирает теги и оставляет безопасный текст.

> [!question] Как защититься от SQL-инъекции?
> 
> Через `$DB` и плейсхолдеры.

> [!question] Как защититься от XSS?
> 
> Типизировать вход (`PARAM_*`) и экранировать вывод: `s()`, `format_string()`, `format_text()`.

> [!question] Чем `{{ }}` отличается от `{{{ }}}` в Mustache?
> 
> `{{ }}` экранирует, `{{{ }}}` выводит как есть (только для очищенных данных).

> [!question] Как защититься от CSRF?
> 
> sesskey: `sesskey()` в форме или ссылке и `require_sesskey()` на сервере. `moodleform` делает это сам.

> [!question] Как защитить доступ?
> 
> `require_login()` и `require_capability()` в правильном контексте.

> [!question] Как безопасно отдавать файлы?
> 
> Через `{component}_pluginfile`: вход, права в контексте, затем `send_stored_file()`.

---

## 8. Формы, вывод, строки, настройки

> [!question] Что такое `moodleform`?
> 
> Базовый класс форм. Поля описываются в `definition()`, проверка в `validation()`. CSRF-защита встроена.

> [!question] Что возвращает `$form->get_data()`?
> 
> Данные формы или `null`, пока форма не отправлена или не прошла валидацию.

> [!question] Как построить страницу с формой?
> 
> `is_cancelled()` → редирект; `get_data()` → сохранить через `$DB`; иначе `$form->display()` между `header()` и `footer()`.

> [!question] Что такое dynamic form?
> 
> Форма на базе `moodleform` (`\core_form\dynamic_form`), которая рендерится и отправляется через AJAX, часто в модальном окне.

> [!question] Что такое renderer?
> 
> Класс в `classes/output/`, который отвечает за вывод. Его можно переопределить в теме.

> [!question] Что такое Mustache-шаблон в Moodle?
> 
> Файл `templates/*.mustache`, который только отображает данные. Логика остаётся в PHP.

> [!question] Что такое `renderable` и `templatable`?
> 
> Интерфейсы: объект выводим, и его данные готовятся для шаблона методом `export_for_template`.

> [!question] Как вывести шаблон?
> 
> `$OUTPUT->render_from_template('local_hello/card', $data)`.

> [!question] Зачем renderer и Mustache вместо `echo`?
> 
> Разделение логики и разметки, переопределение в темах, единое экранирование, те же шаблоны работают из JS.

> [!question] Что такое языковые строки?
> 
> Массив `$string['key']` в `lang/en/{component}.php`. Вывод: `get_string('key', 'component', $a)`. Текст в коде не пишут.

> [!question] Что такое AMOS?
> 
> Система переводов Moodle. В плагине достаточно английских строк.

> [!question] Что такое `settings.php`?
> 
> Файл настроек плагина в админке: `admin_settingpage`, `admin_setting_configtext`, `$ADMIN->add(...)` внутри `if ($hassiteconfig)`.

> [!question] Как читать и писать настройки плагина?
> 
> `get_config('local_hello', 'ключ')` и `set_config(...)`. Хранятся в `config_plugins`.

> [!question] Что такое `moodle_url`?
> 
> Класс для построения ссылок: `new moodle_url('/local/hello/index.php', ['id' => $id])`.

> [!question] Как сделать редирект с сообщением?
> 
> `redirect($url, 'Сохранено', null, \core\output\notification::NOTIFY_SUCCESS)`. Для сообщения на следующей странице `\core\notification::add()`.

> [!question] Что такое `html_writer`?
> 
> Класс для безопасной сборки HTML (`tag()`, `link()`), где шаблон избыточен.

> [!question] Как выбросить ошибку в Moodle?
> 
> `throw new \moodle_exception('код', 'компонент', $returnurl)`. Текст берётся из языковой строки с этим ключом.

> [!question] Как вывести таблицу с сортировкой и пагинацией?
> 
> `table_sql` или `flexible_table`. Для настраиваемых отчётов Report builder.

> [!question] Что такое Report builder?
> 
> Конструктор отчётов (с 4.0): плагин описывает источник данных, отчёты собирает админ.

---

## 9. События и cron

> [!question] Что такое событие (Events API)?
> 
> Класс в `classes/event/` (наследует `\core\event\base`), который сообщает, что что-то произошло. Попадает в логи. Отправка: `->trigger()`.

> [!question] Что такое наблюдатель (observer)?
> 
> Обработчик события, подписка в `db/events.php`. Работает синхронно в том же запросе.

> [!question] Почему в наблюдателе нельзя делать тяжёлое?
> 
> Он замедляет ответ пользователю, а сбой ломает запрос. Делают быструю операцию и ставят adhoc-задачу.

> [!question] Что такое scheduled task?
> 
> Задача по расписанию. Наследует `\core\task\scheduled_task` (`get_name()`, `execute()`), расписание в `db/tasks.php`.

> [!question] Что такое adhoc task?
> 
> Разовая задача из очереди (`adhoc_task`, `set_custom_data()`, `queue_adhoc_task()`). При сбое Moodle повторяет.

> [!question] Как часто запускается cron?
> 
> Каждую минуту: `php admin/cli/cron.php`.

> [!question] Как запустить задачу вручную?
> 
> `php admin/cli/scheduled_task.php --execute='\local_hello\task\cleanup'`.

> [!question] Как понять, что cron не работает?
> 
> В админке (Server → Tasks) видны время последнего запуска, логи и предупреждение. Проверить crontab, владельца `moodledata`, зависшую задачу.

---

## 10. Web services и JavaScript

> [!question] Что такое external function?
> 
> Функция, доступная извне и из JS. Описывается в `db/services.php`, класс в `classes/external/` наследует `\core_external\external_api`.

> [!question] Какие три метода у external-класса?
> 
> `execute_parameters()`, `execute()`, `execute_returns()`.

> [!question] Что обязательно делать внутри `execute()`?
> 
> `validate_parameters`, `validate_context`, `require_capability`, затем логика.

> [!question] Что значат флаги `ajax` и `type`?
> 
> `ajax => true` — вызов из JS, `type` — `read` или `write`.

> [!question] Что такое токен web service?
> 
> Ключ доступа внешней системы от имени пользователя. Это пароль: только HTTPS, не хранить в репозитории.

> [!question] Как внешняя система получает данные?
> 
> Включить web services и REST, создать пользователя-интегратора с минимальными правами и токен, вызвать `/webservice/rest/server.php?wstoken=...&wsfunction=...&moodlewsrestformat=json`.

> [!question] Где лежит JavaScript плагина?
> 
> Исходники в `amd/src/*.js` (ES6), сборка в `amd/build/` командой `npx grunt amd`.

> [!question] Как подключить JS на странице?
> 
> `$PAGE->requires->js_call_amd('local_hello/main', 'init', [$args]);`.

> [!question] Что такое `core/ajax`?
> 
> JS-модуль для вызова external functions.

> [!question] Что такое `core/templates`, `core/str`, `core/notification`, `core/modal`?
> 
> Рендер Mustache на клиенте, языковые строки, уведомления и модальные окна.

> [!question] Как сделать AJAX-запрос безопасно?
> 
> External function с `ajax => true` и проверкой контекста и прав. Не писать свой PHP-скрипт без sesskey.

> [!question] Изменил JS, а в браузере старое. Что делать?
> 
> `npx grunt amd`, сбросить кэши, на dev отключить `cachejs`.

---

## 11. Hooks, файлы, кэш, сообщения, Privacy

> [!question] Что такое callbacks в `lib.php`?
> 
> Старый способ расширения: функции вида `{component}_{callback}`.

> [!question] Что такое Hooks API?
> 
> Типизированная замена callbacks (с 4.3). Класс хука в ядре, подписка в `db/hooks.php`, обработчик в `classes/hook_callbacks.php`.

> [!question] Что идентифицирует файл в Files API?
> 
> `contextid`, `component`, `filearea`, `itemid`, `filepath`, `filename`.

> [!question] Что такое draft area?
> 
> Временная область, куда файл попадает из формы (`filemanager`, `filepicker`). Затем сохраняется `file_save_draft_area_files()`.

> [!question] Как работать с Files API?
> 
> `get_file_storage()`, затем `create_file_from_string/pathname`, `get_area_files`.

> [!question] Что такое Cache API (MUC)?
> 
> Абстракция кэша: определение в `db/caches.php`, использование `cache::make()`, `get()`, `set()`. Хранилища: файлы, Redis, Memcached.

> [!question] Какие режимы у MUC?
> 
> Application (общий), session, request.

> [!question] Что такое Message API?
> 
> Отправка уведомлений. Тип в `db/messages.php`, отправка объектом `\core\message\message` и `message_send()`. Канал выбирает пользователь.

> [!question] Что такое `email_to_user()`?
> 
> Низкоуровневая отправка письма.

> [!question] Что такое Privacy API?
> 
> Реализация GDPR: плагин описывает хранимые данные, умеет их экспортировать и удалять. Провайдер `classes/privacy/provider.php`.

> [!question] Что такое `null_provider`?
> 
> Privacy-провайдер для плагина, который не хранит личных данных.

---

## 12. Модули и блоки

> [!question] Что обязательно для плагина-активности (`mod`)?
> 
> `add_instance`, `update_instance`, `delete_instance`, `supports` в `lib.php`, `mod_form.php`, `view.php`, схема БД и права (`addinstance`, `view`).

> [!question] Что такое `mod_form.php`?
> 
> Форма настроек активности, наследует `moodleform_mod`.

> [!question] Что принимает `view.php` активности?
> 
> cmid (`view.php?id=cmid`).

> [!question] Зачем `backup/moodle2/` в активности?
> 
> Классы `backup_{mod}_activity_task` и `restore_{mod}_activity_task`. Без них активность не переносится с курсом.

> [!question] Как устроен блок?
> 
> Класс `block_xxx extends block_base`: `init()` задаёт заголовок, `get_content()` заполняет `$this->content`.

> [!question] Что такое `applicable_formats()` в блоке?
> 
> Где блок можно показывать.

---

## 13. Тесты, отладка, обновление

> [!question] Что такое PHPUnit в Moodle?
> 
> Тесты логики в `tests/*_test.php`: класс наследует `\advanced_testcase`, вызывает `resetAfterTest()`, данные создаёт через `getDataGenerator()`.

> [!question] Что такое Behat?
> 
> Браузерные сценарии на Gherkin в `tests/behat/*.feature`.

> [!question] Что такое moodle-plugin-ci?
> 
> Набор проверок плагина: phpcs, phpdoc, eslint, mustache lint, phpunit, behat.

> [!question] Как включить режим разработки?
> 
> В `config.php`: `$CFG->debug = (E_ALL | E_STRICT)`, `debugdisplay`, отключить `cachejs`, `langstringcache`, включить `themedesignermode`.

> [!question] Почему нельзя dev-настройки на проде?
> 
> Всё компилируется на каждый запрос (медленно), а ошибки раскрывают пути и структуру БД.

> [!question] Что делает `debugging()`?
> 
> Выводит предупреждение разработчика, например о deprecated функции.

> [!question] Что делает `purge_caches.php`?
> 
> Сбрасывает кэши: классов, шаблонов, строк, MUC.

> [!question] Что нужно, когда Moodle «не видит изменения»?
> 
> Класс, шаблон, `db/*.php` → `purge_caches`; схема или `access.php` → поднять версию и `upgrade.php`; `amd/src` → `grunt amd`; SCSS → `themedesignermode` или сброс кэшей.

> [!question] Как правильно обновить Moodle?
> 
> Режим обслуживания → бэкап БД и `moodledata` → новый код и совместимые плагины → `upgrade.php` → `purge_caches` → выключить обслуживание. Сначала на копии.

> [!question] Как включить режим обслуживания?
> 
> `php admin/cli/maintenance.php --enable`.

> [!question] Каков стиль кода Moodle?
> 
> 4 пробела, без закрывающего `?>`, phpdoc, заголовок GPL, `defined('MOODLE_INTERNAL') || die();` в скриптах без классов.

> [!question] Почему страница работает медленно?
> 
> Dev-режим на проде, N+1 запросы, тяжёлые операции в синхронном коде, нет кэша MUC, не настроены OPcache и Redis.

---

## 14. Инфраструктура и интеграции

> [!question] Что такое Lock API?
> 
> Блокировки от параллельного запуска: `lock_config::get_lock_factory(...)->get_lock('ключ', 10)` и `release()` в `finally`.

> [!question] Почему долгая страница тормозит другие вкладки пользователя?
> 
> PHP-сессия блокируется на время запроса. Помогает `\core\session\manager::write_close()`, после него в `$SESSION` писать нельзя.

> [!question] Как делать HTTP-запросы наружу?
> 
> `\core\http_client` или класс `curl`. Они учитывают прокси и защиту от SSRF. Не `file_get_contents()` и не «голый» `curl_*`.

> [!question] Что такое DI-контейнер в Moodle?
> 
> `\core\di::get(...)` (с 4.4): получение зависимостей без глобальных объектов.

> [!question] Где хранятся сессии?
> 
> В файлах, БД или Redis.

> [!question] Как настроить SSO?
> 
> Плагином `auth_*`: `auth_oauth2`, `auth_ldap`, SAML (чаще сторонний `auth_saml2`), `auth_lti`.

> [!question] Как логируются события?
> 
> В хранилище `logstore_standard`. В CLI и cron вывод через `mtrace()`.

> [!question] Как плагин поддерживает мобильное приложение?
> 
> Приложение работает поверх web services. Для плагина нужны `db/mobile.php` и шаблоны для приложения.

---

## 15. Встроенные подсистемы

> [!question] Что такое банк вопросов?
> 
> Хранилище вопросов по категориям. Вопросы попадают в тест через слоты.

> [!question] Что такое question engine?
> 
> Ядро ведения попыток: хранит шаги и ответы (`question_usage_by_activity`), оценка считается поведением вопроса.

> [!question] Что такое `qtype_*`, `qbehaviour_*`, `quizaccess_*`?
> 
> Типы вопросов, поведения вопросов и правила доступа к тесту.

> [!question] Что такое `assignsubmission_*` и `assignfeedback_*`?
> 
> Подплагины способов сдачи и обратной связи в `mod_assign`.

> [!question] Как добавить пункт в навигацию курса?
> 
> Callback `{component}_extend_navigation_course` (или хук) и проверка `has_capability` в контексте курса.

> [!question] Что такое глобальный поиск?
> 
> Плагин описывает область поиска (`search area`), индексация идёт по cron.

> [!question] Что такое календарь Moodle?
> 
> Подсистема событий (`{event}`). Активность подставляет действие через `mod_xxx_core_calendar_provide_event_action`.

> [!question] Что такое badges, competency, tag API?
> 
> Подсистемы значков, компетенций и тегов со своим API и правами.

> [!question] Как устроен backup/restore?
> 
> Структура описывается классами `backup_nested_element` и `restore_structure_step`. Пользовательские данные включаются по настройкам.

---

## 16. Темы

> [!question] Что такое Boost?
> 
> Базовая тема Moodle на Bootstrap.

> [!question] Что такое дочерняя тема?
> 
> Тема с `$THEME->parents = ['boost']`: наследует всё и переопределяет нужное.

> [!question] Что такое layouts?
> 
> Макеты страниц (`standard`, `course`, `admin`, `login`, `frontpage` и др.). Выбор через `$PAGE->set_pagelayout()`, описание в `$THEME->layouts`.

> [!question] Как переопределить шаблон ядра?
> 
> Положить одноимённый файл в `theme/xxx/templates/{компонент}/` и сбросить кэши.

> [!question] Как переопределить renderer?
> 
> Класс `theme_xxx\output\core_renderer` наследует `\theme_boost\output\core_renderer`.

---

## 17. Работа с чужим кодом

> [!question] Что такое Tracker?
> 
> Багтрекер Moodle (tracker.moodle.org), задачи `MDL-12345`. Туда заводят баги и патчи.

> [!question] Что значит deprecated?
> 
> Функция устарела и будет удалена через несколько релизов. Замену ищут в upgrade notes на moodledev.io.

> [!question] Как менять поведение без правки ядра?
> 
> Событие, хук или callback, настройка, capability, переопределение в дочерней теме, `local`-плагин. Крайний случай: форк с оформленным патчем.

> [!question] Как найти в коде, где что-то происходит?
> 
> Взять текст из интерфейса, найти ключ в `lang/en/*.php`, сделать `grep` по ключу. Также смотреть события, имя capability, Xdebug.

> [!question] Как внести вклад в ядро?
> 
> Задача в Tracker, pull request с веткой вида `MDL-12345-main-описание`, review и тестирование командой.

> [!question] Как называются ветки Moodle в git?
> 
> `main` и ветки на версию: `MOODLE_XXX_STABLE`.

> [!question] Где искать документацию?
> 
> moodledev.io (для разработчиков), docs.moodle.org (для админов), каталог плагинов moodle.org/plugins.

---

## 18. PHP, SQL, Git, веб

> [!question] Что нового в PHP 8?
> 
> Union types, `match`, `enum`, `readonly`, nullsafe `?->`, именованные аргументы, constructor promotion, attributes.

> [!question] Чем интерфейс отличается от абстрактного класса?
> 
> Интерфейс — контракт без реализации, можно реализовать несколько. Абстрактный класс содержит общий код и состояние, наследуется один.

> [!question] Что такое SOLID?
> 
> Пять принципов ООП: единственная ответственность, открытость/закрытость, подстановка Лисков, разделение интерфейсов, инверсия зависимостей.

> [!question] Что такое PSR-4 и Composer?
> 
> PSR-4 — стандарт автозагрузки классов по namespace. Composer — менеджер зависимостей PHP.

> [!question] Чем INNER JOIN отличается от LEFT JOIN?
> 
> INNER оставляет только совпадения, LEFT — все строки левой таблицы, недостающие значения правой NULL.

> [!question] Что такое индекс и как работает составной?
> 
> Структура для быстрого поиска. Составной индекс используется слева направо, порядок полей важен.

> [!question] Что делает `EXPLAIN`?
> 
> Показывает план выполнения запроса: какие индексы и сколько строк используются.

> [!question] Что такое ACID?
> 
> Атомарность, согласованность, изолированность, долговечность транзакции.

> [!question] Чем `merge` отличается от `rebase`?
> 
> `merge` сохраняет историю и делает коммит слияния, `rebase` переносит коммиты поверх новой базы. Опубликованную ветку не перебазируют.

> [!question] Что такое `cherry-pick`?
> 
> Перенос одного коммита в другую ветку.

> [!question] Что такое XSS?
> 
> Выполнение чужого скрипта в браузере жертвы. Защита: экранирование вывода.

> [!question] Что такое CSRF?
> 
> Браузер жертвы отправляет запрос от её имени. Защита: токен (в Moodle sesskey).

> [!question] Что такое SQL-инъекция?
> 
> Данные попадают в SQL как код. Защита: параметризованные запросы (плейсхолдеры).

> [!question] Что такое SSRF?
> 
> Сервер делает запрос на адрес, подсунутый атакующим. Защита: фильтры хостов, `http_client` Moodle.

---