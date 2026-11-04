---
custom-width: 80
---
---
## Оглавление

1. [[#Часть A. Проект 1 — медицинская платформа]]
2. [[#Часть B. Проект 2 — платформа для госсектора]]
3. [[#Часть C. Технологии: вопросы и ответы]]
4. [[#Часть D. Как отвечать и чего избегать]]

---

# Часть A. Проект 1 — медицинская платформа

## A0. Проект в одной минуте

> Платформа непрерывного медицинского образования на Moodle. **Одна установка обслуживает сеть организаций**: у каждой свои категории курсов, группы слушателей и оформление, а контент, многоязычность и безопасность общие. Основные направления: автоматический перевод через DeepL, поиск на Solr, вход через SAML, собственная тема на Boost, модуль учёта клинических случаев, REST-функции для синхронизации с внешними системами, перенос тяжёлых операций в фоновые задачи, мониторинг через Grafana и Sentry.

**Схема архитектуры (словами):**

- Пользователь → веб-сервер (Apache/Nginx + PHP-FPM) → Moodle.
- Moodle ↔ БД (MySQL), `moodledata` (файлы), кеши.
- Moodle ↔ Solr (поиск).
- Moodle ↔ IdP организаций (SAML).
- Moodle → DeepL API (перевод).
- Внешние системы ↔ Moodle через REST веб-сервисы.
- Cron запускает плановые и ad-hoc задачи.
- Sentry собирает ошибки, Grafana показывает метрики.

---

## A1. Взаимодействие с партнёрами и дорожная карта

**Контекст:** несколько организаций-партнёров, у каждой свои пожелания. Разработка ограничена, поэтому нужен порядок.

**Что входит в задачу:**

1. Сбор запросов от представителей организаций (созвоны, письма, тикеты).
2. Перевод формулировок вида «хотим удобнее вести слушателей» в конкретные задачи: что именно изменится, какой результат, как проверить.
3. Оценка трудоёмкости и рисков, предложение порядка.
4. Общие потребности (многоязычность, поиск, вход) реализуются как платформенные функции, а не как отдельные доработки под одного партнёра.

**Критерии приоритета (можно назвать):** сколько организаций затронуто, влияние на слушателей, безопасность и соответствие требованиям, трудоёмкость, зависимость от других задач.

> [!question] Пример: как потребность превращается в задачу
>
> **Ответ:** Партнёры жаловались, что материалы на разных языках готовятся вручную и месяцами. Потребность: «быстро публиковать курсы на нескольких языках». Задачи: плагин интеграции с DeepL, перенос перевода в фон, фильтры многоязычности, проверка качества перевода редактором. Критерий готовности: курс переводится на выбранные языки без ручного копирования.
>
>

---

## A2. Языковые пакеты и многоязычный контент

**Контекст:** организации из разных регионов, слушатели читают на разных языках.

**Два разных механизма — их важно различать:**

| Что | Механизм | Где настраивается |
| --- | --- | --- |
| Интерфейс Moodle (кнопки, меню) | **Языковые пакеты** | Установка через `tool_langimport`, настройки языков сайта |
| Контент курсов (названия, описания, страницы) | **Фильтр многоязычного контента** | Включение фильтра, разметка текста |

**Как это работает:**

- Языковые пакеты ставятся в админке (Site administration → Language → Language packs). Пользователь выбирает язык в профиле или на странице входа.
- Встроенный фильтр Multi-language content использует разметку:

```html
<span lang="en" class="multilang">Course title</span>
<span lang="de" class="multilang">Kurstitel</span>
```

- Популярный плагин `filter_multilang2` использует короткие теги:

```
{mlang en}Course title{mlang}{mlang de}Kurstitel{mlang}
```

- Фильтр включают на уровне сайта и применяют к заголовкам и содержимому (опция «Content and headings»).
- Свои строки плагинов лежат в `lang/<код языка>/<плагин>.php`, получаются через `get_string()`.

**Локализованные каталоги курсов:** для организаций разных регионов — отдельные категории курсов (`course_categories`) со своими курсами на нужном языке; пользователям из региона показывается соответствующий раздел. Доступ к категориям регулируется ролями и группами пользователей (cohorts).

> [!question] В чём разница между языковым пакетом и фильтром multilang?
>
> **Ответ:** Языковой пакет переводит интерфейс, то есть строки, которые написали разработчики Moodle и плагинов. Фильтр переводит контент, который создали авторы курсов. Это разные данные: строки лежат в файлах `lang`, контент лежит в БД.

---

## A3. Local-плагин для DeepL

**Контекст:** учебные материалы нужно переводить на несколько языков. Вручную — долго и дорого.

**Идея решения:** плагин `local_deepl` берёт тексты курса, отправляет в DeepL API и сохраняет перевод. Тяжёлую часть запускает в фоне.

**Структура плагина:**

```
local/deepl/
├── version.php
├── settings.php                  # ключ API, список языков
├── lang/en/local_deepl.php
├── db/
│   ├── install.xml               # таблица журнала переводов
│   ├── access.php                # право запускать перевод
│   └── upgrade.php
├── classes/
│   ├── deepl_client.php          # HTTP-клиент
│   ├── content_collector.php     # собирает тексты курса
│   ├── translator.php            # логика: что, куда, как сохранить
│   └── task/translate_course.php # ad-hoc задача
├── index.php                     # форма запуска (выбор курса и языков)
└── templates/
```

**Настройки (`settings.php`):**

```php
$settings = new admin_settingpage('local_deepl', get_string('pluginname', 'local_deepl'));
$ADMIN->add('localplugins', $settings);

$settings->add(new admin_setting_configpasswordunmask(
    'local_deepl/apikey',
    get_string('apikey', 'local_deepl'),
    get_string('apikey_desc', 'local_deepl'),
    ''
));
```

**Клиент (`deepl_client.php`):**

```php
namespace local_deepl;

class deepl_client {
    private string $key;
    private string $url;

    public function __construct() {
        $this->key = get_config('local_deepl', 'apikey');
        // У бесплатного и платного тарифов разные адреса API.
        $this->url = 'https://api.deepl.com/v2/translate';
    }

    public function translate(array $texts, string $target, ?string $source = null): array {
        $curl = new \curl();
        $curl->setHeader([
            'Authorization: DeepL-Auth-Key ' . $this->key,
            'Content-Type: application/json',
        ]);
        $payload = [
            'text' => array_values($texts),
            'target_lang' => strtoupper($target),
            'tag_handling' => 'html', // сохраняем разметку
        ];
        if ($source) {
            $payload['source_lang'] = strtoupper($source);
        }
        $response = $curl->post($this->url, json_encode($payload));
        $info = $curl->get_info();
        if ((int)$info['http_code'] !== 200) {
            throw new \moodle_exception('deeplerror', 'local_deepl', '', $info['http_code']);
        }
        $data = json_decode($response, true);
        return array_column($data['translations'], 'text');
    }
}
```

**Что переводим:** названия и описания курса и секций, подписи (label), страницы (page), описания активностей — всё, что хранится как текст в БД. Содержимое берём через DML API.

**Как сохраняем перевод:** в многоязычном формате в то же поле (`{mlang de}…{mlang}`), чтобы фильтр показывал нужный язык. Возможен и вариант «копия курса на другом языке».

**Как не переводить дважды:** в таблице журнала храним хеш исходного текста (`sha1`), язык и дату. Если хеш не изменился — пропускаем.

**Разбиение на задачи:** большой курс = много текстов, поэтому одна ad-hoc задача на курс (или на пачку элементов), внутри — пакетные запросы к DeepL.

```php
$task = new \local_deepl\task\translate_course();
$task->set_custom_data(['courseid' => $courseid, 'langs' => ['de', 'fr']]);
$task->set_component('local_deepl');
\core\task\manager::queue_adhoc_task($task);
```

**Обработка ошибок:** повтор при временных ошибках (сеть, 429/5xx), логирование, продолжение с места остановки.

**Подводные камни:**

- HTML: без `tag_handling` теги ломаются.
- Лимиты и стоимость: DeepL считает символы, поэтому важно не переводить повторно.
- Названия, термины и дозировки: медицинский контент — нужен глоссарий и проверка человеком.
- Ключ API не хранить в коде — только в настройках или переменных окружения.

> [!question] Почему отдельный local-плагин, а не правка ядра или модуля?
>
> **Ответ:** Перевод — сервис, не привязанный к одному типу элементов курса. `local` подходит для такой сквозной функциональности. Ядро не трогаем, чтобы обновления проходили без конфликтов.

> [!question] Как защитить ключ API?
>
> **Ответ:** Хранится в настройках плагина как поле-пароль (`admin_setting_configpasswordunmask`), не в репозитории. Доступ к странице запуска ограничен capability.

> [!question] Что делать, если DeepL недоступен?
>
> **Ответ:** Задача падает с исключением, Moodle повторяет ad-hoc задачу с нарастающей задержкой. Ошибки видны в логах и Sentry. Уже переведённое не теряется благодаря журналу.

---

## A4. Global Search + Apache Solr

**Контекст:** стандартный поиск Moodle работает на БД и медленный на больших объёмах.

**Что входит в задачу:**

1. **Сервер Solr:** развернуть (например, в Docker), создать ядро (core) под Moodle.
2. **Плагин в Moodle:** Site administration → Plugins → Search → Manage global search → выбрать движок Solr, указать хост, порт, имя индекса. Для работы нужно PHP-расширение Solr.
3. **Схема:** поля Moodle для документов создаются при настройке.
4. **Поисковые области:** включить области курсов, форумных сообщений, ресурсов.
5. **Индексация:** первичная полная, затем дополнительная через cron.

```bash
php search/cli/indexer.php --force
```

**Как устроена индексация:**

- У каждой поисковой области (`\core_search\base`) есть метод, возвращающий записи, изменённые после последнего индексирования.
- Из записей делаются «документы» (`\core_search\document`) с полями: заголовок, содержимое, курс, контекст, время.
- Документы отправляются в Solr.
- При поиске Solr возвращает кандидатов, а Moodle **проверяет доступ** пользователя к каждому результату.

**Если надо добавить свою поисковую область** (например, для модуля клинических случаев):

```php
namespace mod_clinicalcase\search;

class activity extends \core_search\base_activity {
    // Для типового модуля достаточно базового класса:
    // он индексирует название и описание экземпляра.
}
```

**Почему стало быстрее:** Solr использует обратный индекс (по словам), а не `LIKE '%слово%'` по таблицам.

**Подводные камни:** актуальность индекса (cron), размер индекса, права при выдаче, морфология для разных языков.

> [!question] Как Moodle не показывает пользователю то, к чему у него нет доступа?
>
> **Ответ:** Права проверяются после выдачи из Solr: для каждого результата вызывается проверка доступа в поисковой области. Плюс в индексе хранится контекст документа.

> [!question] Как измерить ускорение поиска?
>
> **Ответ:** Сравнить время ответа поиска до и после на одинаковых запросах (логи и график в Grafana).

---

## A5. Тема на основе Boost

**Контекст:** страницы курсов каждой организации должны быть в её фирменном стиле, при этом тема одна.

**Структура:**

```
theme/orgboost/
├── version.php
├── config.php
├── lib.php                 # колбэки SCSS
├── settings.php
├── lang/en/theme_orgboost.php
├── scss/
│   ├── pre.scss            # переменные
│   └── post.scss           # правила
├── templates/
│   └── theme_boost/        # переопределения шаблонов Boost
│       └── drawer.mustache
└── pix/
```

**`config.php`:**

```php
defined('MOODLE_INTERNAL') || die();

$THEME->name = 'orgboost';
$THEME->parents = ['boost'];          // берём всё от Boost
$THEME->sheets = [];
$THEME->scss = function ($theme) {
    return theme_orgboost_get_main_scss_content($theme);
};
$THEME->prescsscallback = 'theme_orgboost_get_pre_scss';
$THEME->extrascsscallback = 'theme_orgboost_get_extra_scss';
```

**Переопределение шаблона:** копия шаблона Boost кладётся в `theme/orgboost/templates/theme_boost/…` и правится. Moodle сначала ищет шаблон в теме, потом у родителя.

**Как сделан фирменный стиль организации (вариант):**

- В настройках темы — цвета, логотип, шрифт для каждой организации.
- В `get_extra_scss` определяется, к какой организации относится текущая страница (например, по категории курса), и добавляю нужные переменные.
- Стили собираются в CSS и кешируются.

**Почему дочерняя тема:** обновления Boost подхватываются автоматически, а собственные правки изолированы.

> [!question] Как переопределить шаблон в теме?
>
> **Ответ:** Положить Mustache-файл с тем же именем и компонентом в `templates/<компонент>/` своей темы. Moodle найдёт его первым. После изменений нужно сбросить кеши.

> [!question] Зачем SCSS-колбэки?
>
> **Ответ:** `pre` задаёт переменные Bootstrap до сборки (цвета и т.д.), `post`/`extra` добавляет правила после. Так мы меняем внешний вид без копирования всех стилей Boost.

---

## A6. Task API: плановые и ad-hoc задачи

**Контекст:** массовый перевод и массовая запись на курсы нельзя делать в запросе пользователя: тайм-аут и нагрузка.

**Как это устроено в Moodle:**

- Cron каждую минуту запускает `admin/cli/cron.php`.
- Cron выполняет плановые задачи по расписанию и ad-hoc задачи из очереди.

**Плановая задача:**

```php
// db/tasks.php
$tasks = [[
    'classname' => 'local_deepl\task\retry_failed',
    'blocking'  => 0,
    'minute'    => '*/15',
    'hour'      => '*',
    'day'       => '*',
    'month'     => '*',
    'dayofweek' => '*',
]];
```

```php
namespace local_deepl\task;

class retry_failed extends \core\task\scheduled_task {
    public function get_name() {
        return get_string('task_retry_failed', 'local_deepl');
    }
    public function execute() {
        // повторить неудавшиеся переводы
    }
}
```

**Ad-hoc задача (массовая запись на курсы):**

```php
namespace local_sync\task;

class bulk_enrol extends \core\task\adhoc_task {
    public function execute() {
        $data = $this->get_custom_data();
        foreach (array_chunk($data->userids, 100) as $chunk) {
            // записываем пачкой, логируем прогресс
        }
    }
}
```

**Принципы устойчивой задачи:**

- **Идемпотентность:** повторный запуск не создаёт дублей (проверка «уже записан»).
- **Порционность:** пачки по 50–100 записей.
- **Логирование:** `mtrace()` и свои логи.
- **Ошибки:** исключение → Moodle повторит ad-hoc задачу позже.
- **Параллельность:** для тяжёлых задач можно ограничить конкурентность.

> [!question] Чем ad-hoc отличается от scheduled?
>
> **Ответ:** Плановая живёт по расписанию в `db/tasks.php`. Ad-hoc создаётся из кода в ответ на событие (пользователь нажал кнопку) с данными и исчезает после выполнения.

> [!question] Что если cron не запущен?
>
> **Ответ:** Задачи копятся в очереди и не выполняются. Поэтому длину очереди и дату последнего запуска cron нужно мониторить (например, графиком в Grafana).

---

## A7. SAML SSO

**Контекст:** слушатели должны входить через учётные записи своих организаций.

**Роли:**

- **SP (Service Provider)** — наша платформа.
- **IdP (Identity Provider)** — система организации-партнёра.

**Шаги настройки (типичный вариант с плагином `auth_saml2`):**

1. Установить и включить плагин аутентификации.
2. Получить у организации метаданные IdP (XML или ссылка).
3. Указать метаданные в настройках плагина.
4. Отправить организации метаданные нашего SP (`…/auth/saml2/sp/metadata.php`).
5. Сопоставить атрибуты: логин, email, имя, фамилия.
6. Включить создание пользователя при первом входе.
7. Проверить вход, выход, ошибки.

**Как идёт вход:**

1. Пользователь жмёт «Войти через организацию».
2. Moodle отправляет запрос на IdP.
3. Пользователь входит у себя.
4. IdP отдаёт подписанное утверждение с атрибутами.
5. Moodle проверяет подпись, находит или создаёт пользователя и открывает сессию.

**Типичные проблемы:**

- Несовпадение времени серверов (утверждение считается просроченным).
- Истёкший сертификат.
- Другой формат идентификатора пользователя (email вместо логина) — дубликаты пользователей.
- Несколько IdP: кнопки выбора организации на странице входа.

> [!question] Что такое метаданные SAML?
>
> **Ответ:** XML с описанием участника: идентификатор (entityID), адреса точек входа, сертификаты для подписи. SP и IdP обмениваются ими, чтобы доверять друг другу.

> [!question] Как не создать дублей пользователей?
>
> **Ответ:** Выбрать стабильный атрибут для сопоставления (обычно email или уникальный идентификатор организации) и настроить, по какому полю искать существующего пользователя.

---

## A8. Собственный модуль активности — разборы клинических случаев

**Контекст:** нужно учитывать разборы клинических случаев и прогресс в научном обучении. Стандартных элементов не хватает.

**Что получилось:** плагин `mod_clinicalcase` — элемент курса, который преподаватель добавляет в секцию.

**Структура:**

```
mod/clinicalcase/
├── version.php
├── lib.php                      # add/update/delete instance, supports
├── mod_form.php                 # форма настроек
├── view.php                     # страница активности
├── db/
│   ├── install.xml              # таблицы
│   ├── upgrade.php
│   ├── access.php               # права
│   └── services.php             # (если есть внешние функции)
├── classes/
│   ├── output/                  # renderable-объекты
│   ├── privacy/provider.php     # GDPR
│   └── search/activity.php
├── backup/moodle2/              # резервное копирование
├── templates/
└── lang/en/clinicalcase.php
```

**Таблицы (пример):**

- `clinicalcase` — экземпляр активности (курс, название, описание, максимальный балл, срок).
- `clinicalcase_attempt` — разбор слушателя (пользователь, статус, ответ, оценка, время).
- `clinicalcase_review` — комментарий проверяющего.

**`db/install.xml` (фрагмент):**

```xml
<TABLE NAME="clinicalcase_attempt" COMMENT="Case submissions">
  <FIELDS>
    <FIELD NAME="id" TYPE="int" LENGTH="10" NOTNULL="true" SEQUENCE="true"/>
    <FIELD NAME="clinicalcaseid" TYPE="int" LENGTH="10" NOTNULL="true"/>
    <FIELD NAME="userid" TYPE="int" LENGTH="10" NOTNULL="true"/>
    <FIELD NAME="status" TYPE="char" LENGTH="20" NOTNULL="true" DEFAULT="draft"/>
    <FIELD NAME="grade" TYPE="number" LENGTH="10" DECIMALS="2" NOTNULL="false"/>
    <FIELD NAME="timecreated" TYPE="int" LENGTH="10" NOTNULL="true" DEFAULT="0"/>
  </FIELDS>
  <KEYS>
    <KEY NAME="primary" TYPE="primary" FIELDS="id"/>
    <KEY NAME="fk_cc" TYPE="foreign" FIELDS="clinicalcaseid" REFTABLE="clinicalcase" REFFIELDS="id"/>
  </KEYS>
  <INDEXES>
    <INDEX NAME="user_cc" UNIQUE="false" FIELDS="userid, clinicalcaseid"/>
  </INDEXES>
</TABLE>
```

**Права (`db/access.php`):**

```php
$capabilities = [
    'mod/clinicalcase:addinstance' => [
        'captype' => 'write',
        'contextlevel' => CONTEXT_COURSE,
        'archetypes' => ['editingteacher' => CAP_ALLOW, 'manager' => CAP_ALLOW],
        'clonepermissionsfrom' => 'moodle/course:manageactivities',
    ],
    'mod/clinicalcase:submit' => [
        'captype' => 'write',
        'contextlevel' => CONTEXT_MODULE,
        'archetypes' => ['student' => CAP_ALLOW],
    ],
    'mod/clinicalcase:review' => [
        'captype' => 'write',
        'contextlevel' => CONTEXT_MODULE,
        'archetypes' => ['editingteacher' => CAP_ALLOW, 'teacher' => CAP_ALLOW],
    ],
];
```

**Обязательные функции в `lib.php`:**

```php
function clinicalcase_add_instance($data, $mform = null) {
    global $DB;
    $data->timecreated = time();
    $data->id = $DB->insert_record('clinicalcase', $data);
    return $data->id;
}

function clinicalcase_update_instance($data, $mform = null) {
    global $DB;
    $data->timemodified = time();
    $data->id = $data->instance;
    return $DB->update_record('clinicalcase', $data);
}

function clinicalcase_delete_instance($id) {
    global $DB;
    if (!$DB->record_exists('clinicalcase', ['id' => $id])) {
        return false;
    }
    $DB->delete_records('clinicalcase_attempt', ['clinicalcaseid' => $id]);
    $DB->delete_records('clinicalcase', ['id' => $id]);
    return true;
}

function clinicalcase_supports($feature) {
    switch ($feature) {
        case FEATURE_MOD_INTRO: return true;
        case FEATURE_BACKUP_MOODLE2: return true;
        case FEATURE_GRADE_HAS_GRADE: return true;
        default: return null;
    }
}
```

**Страница `view.php` (схема):**

1. Получить `id` модуля через `required_param('id', PARAM_INT)`.
2. `get_course_and_cm_from_cmid()`.
3. `require_login($course, true, $cm)`.
4. `$context = context_module::instance($cm->id); require_capability('mod/clinicalcase:view', $context);` (если описано такое право).
5. Подготовить данные, вывести через renderer и шаблон Mustache.

**Что ещё входит в зрелый плагин:** Privacy API (какие данные о пользователе хранит и как их экспортировать или удалить), Backup/Restore, события (`\mod_clinicalcase\event\attempt_submitted`), интеграция с журналом оценок, поисковая область.

> [!question] Какие таблицы и права нужны такому модулю?
>
> **Ответ:** Таблицы экземпляров, попыток и проверок. Права: добавлять активность, отправлять разбор, проверять. Слушатель видит только свои данные, проверяющий — всех в курсе.

> [!question] Что обязан реализовать модуль, чтобы Moodle его принял?
>
> **Ответ:** `version.php`, схему БД (`install.xml`), функции `add/update/delete_instance` в `lib.php`, форму `mod_form.php`, языковой файл со строкой `modulename`, права `addinstance`. Для production — ещё Privacy API и Backup.

> [!question] Как обновить схему БД в уже работающей системе?
>
> **Ответ:** Поднять версию в `version.php` и добавить шаг в `db/upgrade.php` с использованием DDL (`$dbman->add_field()` и т.д.). Moodle выполнит миграцию при обновлении.

---

## A9. REST веб-сервисы

**Контекст:** внешние системы должны синхронизировать учётные записи и записи на курсы в реальном времени.

**Подход:** собственные внешние функции в `local_sync` + обычные встроенные функции Moodle там, где их хватает.

**Файлы:**

```
local/sync/
├── db/services.php
├── db/access.php
└── classes/external/
    ├── upsert_users.php
    └── enrol_users.php
```

**Внешняя функция (Moodle 4.2+):**

```php
namespace local_sync\external;

use core_external\external_api;
use core_external\external_function_parameters;
use core_external\external_multiple_structure;
use core_external\external_single_structure;
use core_external\external_value;

class enrol_users extends external_api {

    public static function execute_parameters(): external_function_parameters {
        return new external_function_parameters([
            'enrolments' => new external_multiple_structure(
                new external_single_structure([
                    'userid'   => new external_value(PARAM_INT, 'User id'),
                    'courseid' => new external_value(PARAM_INT, 'Course id'),
                    'roleid'   => new external_value(PARAM_INT, 'Role id'),
                ])
            ),
        ]);
    }

    public static function execute(array $enrolments): array {
        $params = self::validate_parameters(self::execute_parameters(), ['enrolments' => $enrolments]);

        $context = \context_system::instance();
        self::validate_context($context);
        require_capability('local/sync:manage', $context);

        $result = [];
        foreach ($params['enrolments'] as $e) {
            // записать пользователя, вернуть статус по каждой записи
            $result[] = ['userid' => $e['userid'], 'status' => 'ok'];
        }
        return $result;
    }

    public static function execute_returns(): external_multiple_structure {
        return new external_multiple_structure(
            new external_single_structure([
                'userid' => new external_value(PARAM_INT, 'User id'),
                'status' => new external_value(PARAM_TEXT, 'ok or error'),
            ])
        );
    }
}
```

**Регистрация (`db/services.php`):**

```php
$functions = [
    'local_sync_enrol_users' => [
        'classname'   => 'local_sync\external\enrol_users',
        'description' => 'Enrol users into courses',
        'type'        => 'write',
        'ajax'        => false,
        'capabilities'=> 'local/sync:manage',
    ],
];

$services = [
    'Sync service' => [
        'functions' => ['local_sync_enrol_users'],
        'restrictedusers' => 1,
        'enabled' => 1,
    ],
];
```

**Вызов снаружи:**

```bash
curl -X POST "https://example.org/webservice/rest/server.php" \
  -d "wstoken=TOKEN" \
  -d "wsfunction=local_sync_enrol_users" \
  -d "moodlewsrestformat=json" \
  -d "enrolments[0][userid]=5" -d "enrolments[0][courseid]=2" -d "enrolments[0][roleid]=5"
```

**Что включить на стороне сайта:** веб-сервисы, протокол REST, создать сервис и сервисного пользователя, выдать минимальные права, создать токен.

**Хорошие практики:**

- Для записи на курсы использовать API записей Moodle (плагин `enrol_manual`, `enrol_get_plugin('manual')->enrol_user(...)`), а не писать в таблицы напрямую.
- Возвращать понятные коды ошибок.
- Идемпотентность: повторный вызов не создаёт дублей.
- Пакетная обработка: списком, а не по одному.
- Документировать: описания полей в `external_value`, плюс описание для интеграторов (примеры запросов и ответов).

> [!question] Как защищены функции?
>
> **Ответ:** Токен привязан к сервисному пользователю с минимальными правами, сервис ограничен (`restrictedusers`), внутри функции проверка контекста и capability, входные данные проходят `validate_parameters`.

> [!question] Почему не писать в БД напрямую?
>
> **Ответ:** Обходятся события, кеши, права и связанные данные. Используем API Moodle (записи на курсы, создание пользователей), тогда всё остаётся согласованным.

---

## A10. Мониторинг: Grafana и Sentry

**Контекст:** хочется узнавать о проблемах раньше пользователей.

**Sentry — «что сломалось и где»:**

- Ставится SDK через Composer (`sentry/sentry`).
- Инициализация на старте приложения с `dsn`, `environment` (prod/stage) и `release` (версия).
- Исключения и фатальные ошибки автоматически отправляются с трассировкой, URL, пользователем (по возможности без персональных данных).
- В Sentry группируются одинаковые ошибки, видно частоту и первый момент появления.
- Настроены уведомления о новых и частых ошибках.

**Grafana — «как ведёт себя система»:**

- Дашборды: время ответа страниц (среднее и 95-й перцентиль), нагрузка на CPU и память, количество ошибок 5xx.
- Очередь ad-hoc задач и возраст самой старой задачи (признак остановки cron).
- Метрики БД: медленные запросы, соединения.
- Время ответа Solr и поиска.
- Алерты на пороги.

**Типичный сценарий:**

1. Алерт: выросло время ответа страницы курса.
2. Определить, что изменилось: релиз, нагрузка, очередь.
3. По логам и профилированию найти тяжёлый SQL или лишний цикл.
4. Исправить (индекс, кеш, вынос в фон), выкатить.
5. Проверить по тем же графикам.

> [!question] Чем Sentry отличается от Grafana?
>
> **Ответ:** Sentry — про ошибки и исключения с контекстом и трассировкой. Grafana — про метрики во времени: нагрузка, скорость, очереди. Вместе они показывают и «что упало», и «как это влияет».

> [!question] Как выглядит разбор найденной проблемы
>
> **Ответ:** Схема: симптом (медленная страница) → как нашёл (график, Sentry) → причина (запрос в цикле, отсутствие индекса) → исправление → результат (время упало с X до Y).

---

# Часть B. Проект 2 — платформа для госсектора

## B0. Проект в одной минуте

> Обучающая платформа на Moodle для европейских институтов: обучение сотрудников и информирование граждан о политике и инициативах. Курсы строятся на официальных публикациях, есть анкеты самооценки и отчёты о прогрессе. Основные направления: собственные форматы курсов и блоки, модуль анкетирования с подплагинами типов вопросов, многошаговая форма самооценки с графиками на Chart.js, REST-функции для обмена с внешними сервисами, рефакторинг плагинов и перенос правок ядра в отдельные плагины. База данных — PostgreSQL.

---

## B1. Анализ требований и оценка

**Что входит в задачу:**

1. Чтение требований заказчика, выявление неясностей, вопросы.
2. Разбиение на технические задачи: какие плагины, какие таблицы, какие права, какие интеграции.
3. Оценка по частям с запасом на тестирование и риски.
4. Фиксация спецификации: что входит, что не входит, критерии приёмки.

**Мини-шаблон оценки:**

| Блок | Что учесть |
| --- | --- |
| Анализ | Вопросы к заказчику, прототип |
| Разработка | Плагин, схема БД, права, интерфейс |
| Тесты | Ручное и автоматическое тестирование |
| Интеграция | Внешние сервисы, согласования |
| Запас | Риски, правки по результатам показа |

> [!question] Как оценивать задачи?
>
> **Ответ:** Разбить на мелкие части, оценить каждую, добавить время на тесты и ревью, назвать допущения и риски. Если требований недостаточно — сначала уточнить, а оценку давать с диапазоном.

---

## B2. Собственные форматы курсов и блоки

**Контекст:** курсы о политике и инициативах должны выглядеть иначе, чем обычные: иначе структурированы и удобнее для поиска информации.

**Формат курса (`format_<имя>`):**

- Определяет, **как курс показывается**: секции, навигация, разделы.
- Базовый класс в современных версиях Moodle — `core_courseformat\base`.

```
course/format/policy/
├── version.php
├── lib.php                  # class format_policy extends core_courseformat\base
├── format.php               # точка входа (для старых версий)
├── classes/output/          # renderer, переопределения
├── templates/
├── styles.css
└── lang/en/format_policy.php
```

```php
class format_policy extends core_courseformat\base {
    public function uses_sections() {
        return true;
    }

    public function course_format_options($foreditform = false) {
        // свои настройки формата: например, показывать ли краткое описание политики
        return [
            'showsummary' => [
                'default' => 1,
                'type' => PARAM_INT,
            ],
        ];
    }
}
```

**Блок (`block_<имя>`):**

```php
class block_policyinfo extends block_base {
    public function init() {
        $this->title = get_string('pluginname', 'block_policyinfo');
    }

    public function get_content() {
        if ($this->content !== null) {
            return $this->content;
        }
        $this->content = new stdClass();
        $this->content->text = $this->page->get_renderer('block_policyinfo')->render_summary($this->page->course);
        return $this->content;
    }

    public function applicable_formats() {
        return ['course-view' => true, 'site' => true];
    }
}
```

**Зачем блоки:** быстрые ссылки на связанные публикации, краткая справка по инициативе, прогресс слушателя.

**Результат:** слушателю проще находить информацию об институтах и инициативах.

> [!question] Чем формат курса отличается от темы?
>
> **Ответ:** Формат определяет структуру и логику отображения курса (секции, навигацию). Тема — общий внешний вид сайта: цвета, шрифты, шаблоны страниц.

> [!question] Когда блок, а когда активность?
>
> **Ответ:** Блок — вспомогательный элемент на странице (справка, ссылки). Активность — учебный элемент с данными и оценками, который лежит в секции курса.

---

## B3. Элемент анкетирования с подплагинами типов вопросов

**Контекст:** нужны разные типы вопросов (шкала, выбор, текст, матрица), и их набор должен расти без переделки модуля.

**Идея:** модуль `mod_selfassess` определяет **тип подплагинов** (например, `selfassessitem`). Каждый тип вопроса — отдельный плагин в папке модуля. Так, например, устроены типы вопросов теста.

**Объявление типа подплагинов:**

```json
// mod/selfassess/db/subplugins.json (Moodle 4.3+)
{
  "plugintypes": {
    "selfassessitem": "mod/selfassess/item"
  }
}
```

В более ранних версиях использовался файл `db/subplugins.php`:

```php
$subplugins = ['selfassessitem' => 'mod/selfassess/item'];
```

**Структура:**

```
mod/selfassess/
├── classes/item/base.php           # абстрактный базовый класс типа вопроса
├── item/
│   ├── scale/                      # selfassessitem_scale
│   │   ├── version.php
│   │   ├── classes/item.php
│   │   └── lang/en/selfassessitem_scale.php
│   ├── choice/
│   └── text/
```

**Базовый класс (контракт для типов вопросов):**

```php
namespace mod_selfassess\item;

abstract class base {
    abstract public function add_form_elements(\MoodleQuickForm $mform, array $options): void;
    abstract public function validate(array $data): array;      // ошибки
    abstract public function score(array $answer): float;       // баллы
    abstract public function render_result(array $answer): string;
}
```

**Загрузка типов:** модуль получает список установленных подплагинов через `core_component::get_plugin_list('selfassessitem')` и создаёт объект нужного класса по типу вопроса.

**Что даёт такой подход:**

- Новый тип вопроса = новый плагин, без правки основного модуля.
- Каждый тип можно тестировать отдельно.
- Тип можно отключить или заменить.

**Хранение:** в основной таблице вопросов поле `type` (строка с именем типа) и поле с настройками (JSON). Ответы — в таблице ответов; формат зависит от типа.

> [!question] Зачем подплагины, если можно сделать `switch` по типу?
>
> **Ответ:** `switch` требует править основной код при каждом новом типе. С подплагинами новый тип — отдельный пакет. Это проще поддерживать и расширять, а ядро модуля остаётся стабильным.

> [!question] Как подплагины находятся и загружаются?
>
> **Ответ:** Тип объявлен в `subplugins.json`/`subplugins.php`. Moodle знает папки типа и регистрирует найденные плагины. Список получаем через `core_component`.

---

## B4. Многошаговая форма самооценки (Form API + Chart.js)

**Контекст:** слушатель отвечает на вопросы по шагам, сразу видит баллы и график результата.

**Как строится многошаговая форма в Moodle:** отдельного «мастера» в Form API нет, поэтому типовое решение:

1. На каждый шаг — своя форма (`moodleform`) или одна форма с параметром шага.
2. Номер шага — скрытое поле или параметр URL.
3. Ответы промежуточных шагов сохраняются как **черновик** в БД (таблица попыток) либо в сессии.
4. Последний шаг: подсчёт и сохранение результата.

```php
class step_form extends \moodleform {
    protected function definition() {
        $mform = $this->_form;
        $items = $this->_customdata['items'];

        foreach ($items as $item) {
            $item->add_form_elements($mform, $this->_customdata['options']);
        }

        $mform->addElement('hidden', 'step', $this->_customdata['step']);
        $mform->setType('step', PARAM_INT);

        $this->add_action_buttons(true, get_string('next'));
    }

    public function validation($data, $files) {
        $errors = parent::validation($data, $files);
        // проверка обязательных ответов
        return $errors;
    }
}
```

**Обработка:**

```php
$form = new step_form(null, $customdata);

if ($form->is_cancelled()) {
    redirect($returnurl);
} else if ($data = $form->get_data()) {
    attempt::save_step($attemptid, $data);          // сохраняем черновик
    if ($data->step < $laststep) {
        redirect(new moodle_url('/mod/selfassess/attempt.php', ['id' => $cmid, 'step' => $data->step + 1]));
    }
    $result = scorer::calculate($attemptid);        // подсчёт на сервере
    redirect(new moodle_url('/mod/selfassess/result.php', ['id' => $cmid, 'attempt' => $attemptid]));
}
```

**«Мгновенный» подсчёт:**

- Предварительный подсчёт на клиенте (JS) для отклика.
- Итоговый подсчёт — **только на сервере**, потому что клиенту доверять нельзя.
- Вариант: AJAX-вызов внешней функции Moodle для пересчёта без перезагрузки.

**График на Chart.js:**

1. PHP считает данные по разделам и передаёт в шаблон и JS.
2. AMD-модуль строит диаграмму на `<canvas>`.

```php
$PAGE->requires->js_call_amd('mod_selfassess/results', 'init', [[
    'labels' => $labels,
    'values' => $values,
]]);
```

```js
// amd/src/results.js
define(['core/chartjs'], function(Chart) {
    return {
        init: function(data) {
            const ctx = document.getElementById('selfassess-chart');
            new Chart(ctx, {
                type: 'radar',
                data: {
                    labels: data.labels,
                    datasets: [{label: 'Result', data: data.values}]
                }
            });
        }
    };
});
```

**В Moodle также есть PHP-обёртка для графиков:** `\core\chart_bar`, `\core\chart_line`, `\core\chart_pie`, выводятся через `$OUTPUT->render($chart)` — внутри это тоже Chart.js.

**Подводные камни:**

- Потеря ответов между шагами → сохранять черновик.
- Повторная отправка → проверка состояния попытки.
- Двойной подсчёт → один источник правды на сервере.

> [!question] Как хранить состояние между шагами?
>
> **Ответ:** Черновик в таблице попыток (статус `draft`), по шагам обновлять ответы. Так слушатель может вернуться и продолжить, а данные не пропадают при обновлении страницы.

> [!question] Почему итоговый балл считают на сервере?
>
> **Ответ:** JS можно изменить в браузере. Серверный подсчёт гарантирует корректный результат. Клиентский — только для мгновенного отклика.

---

## B5. Рефакторинг и перенос правок ядра в плагины

**Контекст:** в системе были правки прямо в коде Moodle. Они терялись или конфликтовали при обновлениях, усложняли поддержку.

**Что входит в задачу:**

1. **Инвентаризация:** сравнить код с чистой версией Moodle (`git diff` с эталонным релизом) и составить список правок.
2. **Классификация** каждой правки:
   - добавить поведение → событие или колбэк;
   - изменить вывод → шаблон или renderer в теме;
   - изменить данные → плагин с собственной логикой;
   - настройка → штатные настройки и capabilities.
3. **Перенос** в плагины и удаление правок из ядра.
4. **Тестирование** до и после (сценарии пользователей).
5. **Рефакторинг плагинов:** избавление от дублирования, переход на классы с автозагрузкой, вынос тяжёлых запросов, кеширование.

**Инструменты Moodle вместо правки ядра:**

| Потребность | Решение |
| --- | --- |
| Реакция на событие (курс создан, пользователь записан) | Наблюдатель событий — `db/events.php` + класс-обработчик |
| Дополнить навигацию, подвал, заголовок | Колбэки в `lib.php` (`local_xxx_extend_navigation`, `before_footer` и др.) |
| Новые версии, 4.3+ | **Hooks API** — `db/hooks.php` |
| Поменять вид страницы | Переопределение шаблона Mustache в теме |
| Изменить renderer | Переопределение в теме (`classes/output/core_renderer.php`) |
| Своя логика | Отдельный `local`-плагин |

**Пример наблюдателя событий:**

```php
// db/events.php
$observers = [[
    'eventname' => '\core\event\user_enrolment_created',
    'callback'  => '\local_policy\observer::user_enrolled',
]];
```

```php
namespace local_policy;

class observer {
    public static function user_enrolled(\core\event\user_enrolment_created $event) {
        // действие при записи на курс
    }
}
```

**Ускорение страниц:**

- Убрать запросы в циклах (заменить одним запросом `get_records_sql` со `JOIN`).
- Добавить индексы в собственные таблицы.
- Кешировать повторяющиеся данные через Cache API (`cache::make()`).
- Уменьшить объём данных на странице.

**Результат:** надёжнее обновления, быстрее страницы, правки собраны в плагинах и описаны.

> [!question] Как найти все правки ядра?
>
> **Ответ:** Сравнить проект с оригинальным релизом Moodle нужной версии (`diff`), получить список изменённых файлов и строк. Дальше разобрать каждую правку и решить, как заменить штатным механизмом.

> [!question] Что делать, если правку нельзя заменить плагином?
>
> **Ответ:** Сначала искать колбэк, событие или хук. Если нет — оформить как патч с документацией и отслеживать при обновлениях. Лучше предложить изменение в сообщество Moodle. Это последний вариант.

> [!question] Какие самые частые причины медленных страниц?
>
> **Ответ:** Запросы к БД в цикле, нет индексов, лишние данные в выборке, отсутствие кеширования, тяжёлые операции в запросе пользователя вместо фона.

---

## B6. REST веб-сервисы для госсервисов

**Контекст:** безопасный обмен данными между Moodle и внешними государственными сервисами.

Реализация по тому же принципу, что в **A9**. Особенности для госсектора:

- **Безопасность:** HTTPS, токены, ограничение по IP, минимальные права сервисного пользователя.
- **Персональные данные:** отдавать только необходимые поля.
- **Журналирование:** кто и когда обращался.
- **Стабильный контракт:** не менять формат ответа без новой версии функции.
- **Валидация:** строгие типы параметров (`PARAM_*`).

> [!question] Как обеспечить безопасность обмена с госсервисами?
>
> **Ответ:** HTTPS, отдельный сервисный пользователь с минимальными правами, токен с ограничениями, проверка capability и параметров в каждой функции, логирование обращений.

---

## B7. Тестирование, отладка, стабильность и безопасность

**Виды проверок:**

- **Ручное сквозное тестирование** (end-to-end): пройти пользовательский сценарий целиком (запись → курс → анкета → результат → отчёт).
- **PHPUnit** — тесты логики плагинов (подсчёт баллов, сохранение данных).
- **Behat** — автоматические сценарии в браузере.
- **Отладка:** режим разработчика Moodle (`debug = DEVELOPER`), логи, Xdebug.

**Запуск PHPUnit (кратко):**

```bash
php admin/tool/phpunit/cli/init.php
vendor/bin/phpunit --testsuite mod_selfassess_testsuite
```

**Безопасность в плагине — чек-лист:**

- [ ] `require_login()` на каждой странице.
- [ ] Проверка прав `require_capability()`.
- [ ] Параметры через `required_param()`/`optional_param()` с `PARAM_*`.
- [ ] SQL с плейсхолдерами.
- [ ] `require_sesskey()` для действий, меняющих данные.
- [ ] Вывод через шаблоны/`s()`/`format_string()` (защита от XSS).
- [ ] Загрузка файлов через File API.

> [!question] Как проверить, что плагин не сломал соседние функции?
>
> **Ответ:** Автотесты (PHPUnit/Behat) плюс сквозной ручной сценарий на тестовом стенде. Для изменений в общих местах — регрессионный набор ключевых сценариев.

---

# Часть C. Технологии: вопросы и ответы

> [!tip] Как пользоваться
> Для каждой технологии из CV: **что это → зачем → где в проектах → возможные вопросы**.

## C1. PHP

**Что и зачем:** язык бэкенда. Все плагины Moodle — PHP.

**Как применяется:** классы с пространствами имён и автозагрузкой (`classes/`), типизация, исключения, работа через Moodle API.

> [!question] Как в Moodle организован автозагрузчик?
>
> **Ответ:** Классы лежат в `classes/` плагина, имя класса и пространство имён должны совпадать с путём: `\local_deepl\deepl_client` → `local/deepl/classes/deepl_client.php`. Подключать вручную (`require`) не нужно.

> [!question] Чем `include` отличается от `require`, `require_once`?
>
> **Ответ:** `include` при ошибке даёт предупреждение, `require` — фатальную ошибку, `_once` — не подключает второй раз.

> [!question] Что такое Composer и автозагрузка PSR-4?
>
> **Ответ:** Composer — менеджер зависимостей. PSR-4 — стандарт соответствия пространств имён путям к файлам. Moodle использует свою схему, но она устроена по тому же принципу.

> [!question] Исключения: как ловить и бросать в Moodle?
>
> **Ответ:** Для ошибок плагина используют `moodle_exception` со строкой из языкового файла. Ловят там, где можно восстановиться (например, повторить запрос), иначе дают подняться — Moodle покажет страницу ошибки и запишет её в лог.

## C2. JavaScript

**Что и зачем:** клиентская логика: графики, динамика форм.

**Как применяется:** AMD-модули в `amd/src`, сборка Grunt в `amd/build`, подключение через `js_call_amd`.

> [!question] Как в Moodle подключают JS?
>
> **Ответ:** Пишется модуль в `amd/src/name.js`, собирается в `amd/build` (через Grunt), в PHP вызывается `$PAGE->requires->js_call_amd('component/name', 'init', [$args])`.

> [!question] Почему JS нужно собирать?
>
> **Ответ:** Moodle использует минифицированные AMD-модули из `amd/build`. Исходники в `amd/src` нужно пересобрать после изменений, иначе браузер получит старую версию.

## C3. Moodle (общие вопросы)

> [!question] Как устроена архитектура Moodle?
>
> **Ответ:** Ядро плюс плагины разных типов. Плагины подключаются через API и колбэки. Данные в БД, файлы в `moodledata`, фоновая работа через cron, кеши через Cache API.

> [!question] Что такое Frankenstyle?
>
> **Ответ:** Правило имён: `тип_имя` (`mod_selfassess`, `local_deepl`). Определяет префиксы функций, классов, таблиц, строк, capabilities.

> [!question] Какие бывают контексты?
>
> **Ответ:** Иерархия: система → категория курсов → курс → модуль (активность) → блок; отдельно пользовательский. Права и роли назначаются и проверяются в контексте.

> [!question] Что такое роли и capabilities?
>
> **Ответ:** Capability — конкретное право (`mod/x:submit`). Роль — набор прав. Роль назначается пользователю в контексте. Проверка в коде: `has_capability()`, `require_capability()`.

> [!question] Какие кеши есть в Moodle?
>
> **Ответ:** Cache API (приложение, сессия, запрос), кеш строк языка, кеш шаблонов Mustache, кеш тем (CSS), кеш JS. Сброс: Site administration → Development → Purge caches или `php admin/cli/purge_caches.php`.

> [!question] Как обновляется Moodle?
>
> **Ответ:** Заменить код на новый релиз, запустить обновление (веб или `php admin/cli/upgrade.php`). Moodle выполнит `upgrade.php` ядра и плагинов. Перед этим — резервная копия и проверка совместимости плагинов на тестовом стенде.

## C4. Plugin API

> [!question] Как Moodle находит плагин и узнаёт его версию?
>
> **Ответ:** По папке нужного типа и файлу `version.php`: там `$plugin->component`, `$plugin->version`, `$plugin->requires`, зависимости. При различии версий в БД и файлах Moodle запускает установку или обновление.

> [!question] Что такое колбэки и как плагин влияет на ядро?
>
> **Ответ:** Функции с заранее оговорённым именем в `lib.php` (`pluginname_function`) и классы-наблюдатели событий. Ядро их вызывает в нужные моменты. Новые версии добавляют ещё Hooks API.

## C5. Web Services API

> [!question] Чем отличаются AJAX-функции и обычные веб-сервисы?
>
> **Ответ:** Одна и та же внешняя функция может вызываться из JS страницы (`ajax => true`, с сессией пользователя) и снаружи через протокол (REST) с токеном.

> [!question] Какие протоколы поддерживает Moodle?
>
> **Ответ:** REST, SOAP, XML-RPC и др. Чаще всего используется REST с ответом JSON.

> [!question] Что такое `validate_parameters`, `validate_context`?
>
> **Ответ:** `validate_parameters` проверяет и очищает входные данные по описанию. `validate_context` проверяет, что контекст доступен текущему пользователю или токену.

## C6. DML API

> [!question] Как выбрать записи с JOIN?
>
> **Ответ:** `$DB->get_records_sql()` с плейсхолдерами:
>
> ```php
> $sql = "SELECT a.id, u.firstname, a.grade
>           FROM {clinicalcase_attempt} a
>           JOIN {user} u ON u.id = a.userid
>          WHERE a.clinicalcaseid = :ccid";
> $rows = $DB->get_records_sql($sql, ['ccid' => $ccid]);
> ```

> [!question] Зачем `{table}` в SQL?
>
> **Ответ:** Moodle подставит префикс таблиц (например `mdl_`), поэтому запросы переносимы между установками.

> [!question] Как писать запросы, одинаково работающие на MySQL и PostgreSQL?
>
> **Ответ:** Использовать функции Moodle: `$DB->sql_like()`, `sql_concat()`, `sql_equal()`, избегать диалектных особенностей. PostgreSQL строже: поля в `SELECT` должны быть в `GROUP BY`, нужна аккуратность с типами.

> [!question] Как избежать SQL-инъекций?
>
> **Ответ:** Плейсхолдеры (`:name` или `?`) вместо подстановки значений в строку.

> [!question] Что такое `$DB->get_in_or_equal()`?
>
> **Ответ:** Помощник для построения условия `IN (...)` с плейсхолдерами по массиву значений.

## C7. Form API

> [!question] Как проверить данные формы?
>
> **Ответ:** Типы через `$mform->setType()`, правила через `addRule()`, сложные проверки в методе `validation()`. Результат — `get_data()`.

> [!question] Как защищена форма от CSRF?
>
> **Ответ:** `moodleform` автоматически добавляет `sesskey` и проверяет его при отправке.

## C8. Task API

(см. A6)

> [!question] Как запустить задачу вручную для отладки?
>
> **Ответ:** `php admin/cli/scheduled_task.php --execute='\local_x\task\name'` для плановой, `php admin/cli/adhoc_task.php --execute` для ad-hoc очереди.

## C9. Global Search API

(см. A4)

> [!question] Что такое поисковая область?
>
> **Ответ:** Класс, который знает, какие записи индексировать, как превратить их в документы и как проверить доступ к результату.

## C10. Mustache и шаблоны

> [!question] Как передать данные в шаблон?
>
> **Ответ:** Через renderable-класс с методом `export_for_template()` и вызов `$OUTPUT->render($renderable)` или `$OUTPUT->render_from_template('component/name', $context)`.

> [!question] Как вывести строку перевода в шаблоне?
>
> **Ответ:** `{{#str}} key, component {{/str}}`.

> [!question] Чем `{{var}}` отличается от `{{{var}}}`?
>
> **Ответ:** Первый экранирует HTML, второй выводит как есть. Тройные скобки — только для заведомо безопасного HTML.

## C11. HTML / CSS / SCSS

> [!question] Как Moodle строит CSS темы?
>
> **Ответ:** Собирает SCSS из родительской и дочерней тем (с `pre`/`post` и колбэками), компилирует в CSS и кеширует. После правок сбросить кеши тем.

## C12. Chart.js

(см. B4)

> [!question] Как подготовить данные для графика?
>
> **Ответ:** PHP считает массивы подписей и значений, передаёт в JS-модуль, тот строит график на `<canvas>`.

## C13. MySQL и PostgreSQL

> [!question] Какие различия MySQL и PostgreSQL важны на практике?
>
> **Ответ:** PostgreSQL строже к `GROUP BY` и типам, регистрозависимый `LIKE` (в Moodle решается `sql_like()`), разное поведение функций дат и конкатенации. В CV проект 1 указан на MySQL, проект 2 на PostgreSQL; DML API делает плагины переносимыми между ними.

> [!question] Как ускорить медленный запрос?
>
> **Ответ:** `EXPLAIN`, добавить индекс по полям условий и соединений, убрать лишние выборки, избавиться от запросов в цикле.

## C14. Apache Solr

(см. A4)

> [!question] Что такое core в Solr?
>
> **Ответ:** Отдельный индекс со своей конфигурацией и схемой. Для Moodle создаётся своё ядро.

## C15. SAML

(см. A7)

> [!question] Чем SAML отличается от OAuth2/OpenID Connect?
>
> **Ответ:** SAML — XML-протокол единого входа, исторически корпоративный. OAuth2 — авторизация доступа, OIDC — слой аутентификации поверх OAuth2 с JSON-токенами. В организациях часто встречается SAML.

## C16. DeepL

(см. A3)

> [!question] Какие параметры DeepL API важны?
>
> **Ответ:** `text`, `target_lang`, `source_lang`, `tag_handling=html` для сохранения разметки, `formality` для тона.

## C17. Sentry и Grafana

(см. A10)

## C18. Docker, Docker Compose, Linux

**Что и зачем:** одинаковое окружение разработки и стендов, быстрый запуск.

**Типичный `docker-compose.yml` (схема):**

```yaml
services:
  web:
    build: ./docker/web          # PHP-FPM / Apache + расширения (intl, gd, pgsql/mysqli, solr)
    volumes:
      - ./moodle:/var/www/html
      - moodledata:/var/www/moodledata
    depends_on: [db, solr]
  db:
    image: mysql:8                # или postgres
    environment:
      MYSQL_DATABASE: moodle
    volumes:
      - dbdata:/var/lib/mysql
  solr:
    image: solr:8
    volumes:
      - solrdata:/var/solr
volumes:
  moodledata:
  dbdata:
  solrdata:
```

> [!question] Зачем Docker Compose?
>
> **Ответ:** Описывает все сервисы проекта в одном файле и запускает их одной командой. Окружение у всех одинаковое.

> [!question] Какие команды Linux нужны чаще всего?
>
> **Ответ:** `tail -f` для логов, `grep`, `ps`/`top`, `systemctl`, `crontab -e`, `chmod`/`chown` (права на `moodledata`), `ssh`.

> [!question] Как настроен cron для Moodle?
>
> **Ответ:** Запись `* * * * * php /path/to/moodle/admin/cli/cron.php`, запускается от пользователя веб-сервера каждую минуту.

## C19. Git, GitLab, BitBucket

> [!question] Какой процесс работы с ветками?
>
> **Ответ:** Ветка под задачу от основной, коммиты с понятными сообщениями, merge/pull request, ревью, слияние.

> [!question] Как разрешать конфликты?
>
> **Ответ:** Обновить ветку (`rebase` или `merge` основной), вручную выбрать нужные изменения, проверить работу и закоммитить.

> [!question] `merge` и `rebase`: в чём разница?
>
> **Ответ:** `merge` создаёт коммит слияния и сохраняет историю как есть. `rebase` переносит коммиты поверх новой базы и делает историю линейной. Опубликованные ветки не пересобирают.

## C20. Composer

> [!question] Как подключать сторонние библиотеки в плагин?
>
> **Ответ:** Через `composer require`, подключение автозагрузчика. Для плагинов Moodle сторонний код описывают в `thirdpartylibs.xml`, чтобы было видно, что это чужой код.
