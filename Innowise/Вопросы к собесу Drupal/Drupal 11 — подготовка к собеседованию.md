---
custom-width: 80
---
---
## Карта: что за что отвечает

|Тема|За что отвечает|Ключевые понятия|
|---|---|---|
|Multilingual|языки интерфейса, контента, конфигурации|language negotiation, translatable fields, `t()`|
|REST / JSON:API|доступ к данным по HTTP|REST Resource, JSON:API, serialization, CORS|
|Configuration Management|перенос настроек между окружениями|`cex`/`cim`, sync storage, config schema, Config Split|
|Services & DI|переиспользуемая логика и зависимости|service container, `services.yml`, `create()`|
|Performance|кеш, запросы, масштабирование|cache contexts/tags/max-age, N+1, BigPipe|
|Testing|проверка поведения и доступа|Unit, Kernel, Functional, `BrowserTestBase`|
|Security|доступ, CSRF, XSS, SQL|Entity Access, CSRF token, Twig escaping|
|Multisite|несколько сайтов на одной кодовой базе|`sites/`, `settings.php`, `sites.php`|
|Events API|реакция на события без жёстких связей|`event_dispatcher`, subscribers|
|Entity API|CRUD данных, bundles, query|storage, Entity Query, `accessCheck`, revisions|
|Plugin API|взаимозаменяемые компоненты|Plugin Manager, annotations/attributes, derivatives|
|Batch / Cron / Queue|долгие и фоновые операции|`$sandbox`, `hook_cron`, QueueWorker|
|Migration API|перенос данных из внешних источников|source/process/destination, ID map|
|Field API|структура данных сущностей|field type/storage/config, widget, formatter|
|Authentication API|кто делает запрос|Authentication Provider, Basic, OAuth, CSRF|

---

## Блок 1. Мультиязычность (Multilingual)

> [!abstract] Кратко
> 
> В Drupal мультиязычность состоит из трёх независимых слоёв: язык интерфейса (системные строки и сообщения), язык контента (переводы сущностей и их полей) и язык конфигурации (метки полей, названия типов материалов).
> 
> Язык текущего запроса определяет механизм language negotiation: по префиксу URL, домену, предпочтениям пользователя или заголовку браузера Accept-Language. Переводится не обязательно вся сущность целиком: часть полей может быть общей для всех переводов.
> 
> И последнее, что важно помнить: кеш должен учитывать язык, иначе пользователь получит страницу на чужом языке.

### 1.1. Общая концепция

Drupal поддерживает сайты, на которых интерфейс, содержимое и конфигурация существуют на нескольких языках. Например, у сайта русский и английский интерфейс, а одна статья имеет русскую и английскую версии. Не весь контент обязан иметь перевод.

Три аспекта мультиязычности:

- **Interface language** — язык интерфейса: кнопок, сообщений, системных надписей.
- **Content language** — язык содержимого: материалов, пользовательских сущностей, переводимых полей.
- **Configuration language** — язык переводимых значений конфигурации, например меток и описаний полей.

Модули ядра:

|Модуль|Назначение|
|---|---|
|`Language`|управление языками, определение языкового контекста|
|`Interface Translation` (`locale`)|перевод интерфейсных строк|
|`Content Translation`|перевод содержимого|
|`Configuration Translation`|перевод поддерживаемых значений конфигурации|

Модули работают совместно, но решают разные задачи.

Минимальная настройка мультиязычного сайта:

1. Включить четыре модуля выше.
2. Добавить языки (_Configuration → Regional and language → Languages_).
3. Настроить language negotiation.
4. Включить перевод для нужных entity types и bundles (_Content language and translation_) и отметить переводимые поля.

### 1.2. Language negotiation

**Language negotiation** — механизм определения языка текущего запроса.

Язык можно определять по:

- языковому префиксу URL: `/en/about`;
- домену или поддомену: `en.example.com`;
- предпочтениям пользователя;
- заголовку браузера `Accept-Language`;
- языку по умолчанию.

Методы и их приоритет настраиваются в административном интерфейсе отдельно для каждого типа языка. Язык интерфейса и язык содержимого могут различаться: пользователь видит английские системные сообщения, но открывает русскую статью.

Сервис — `language_manager`:

```php
$language_manager = \Drupal::languageManager();

$interface_language = $language_manager
  ->getCurrentLanguage(\Drupal\Core\Language\LanguageInterface::TYPE_INTERFACE);

$content_language = $language_manager
  ->getCurrentLanguage(\Drupal\Core\Language\LanguageInterface::TYPE_CONTENT);

$interface_code = $interface_language->getId(); // 'ru', 'en', ...
$content_code = $content_language->getId();
```

В классах приложения `language_manager` предпочтительно получать через Dependency Injection, а не через статический `\Drupal::languageManager()`.

### 1.3. Перевод сущностей

Перевод существует на уровне сущности и отдельных полей. Например, у `Article`:

- `title`, `body` — переводимые;
- `field_publish_date`, `field_external_id` — общие для всех переводов.

Переводимость поля настраивается отдельно (`translatable`). Если поле непереводимое, его значение разделяют все переводы сущности.

```php
$node = \Drupal\node\Entity\Node::load($nid);

if ($node && $node->hasTranslation('en')) {
  $english_node = $node->getTranslation('en');

  $title = $english_node->label();
  $body = $english_node->get('body')->value;
}
```

Создание перевода:

```php
if ($node && !$node->hasTranslation('en')) {
  $node->addTranslation('en', [
    'title' => 'Implement task workflow',
  ]);
  $node->save();
}
```

Основные методы:

- `isTranslatable()` — поддерживает ли сущность переводы;
- `isDefaultTranslation()` — является ли перевод основным;
- `hasTranslation($langcode)` — есть ли перевод;
- `getTranslation($langcode)` — получает существующий перевод;
- `addTranslation($langcode, $values)` — создаёт перевод.

`getTranslation()` **не создаёт** перевод и бросает исключение, если его нет, — всегда проверяйте `hasTranslation()`.

Переводы используют общую сущность и её ID. Это не отдельные независимые материалы, хотя значения переводимых полей могут различаться. Путь (alias) тоже может быть разным для каждого языка.

### 1.4. Перевод интерфейса

Строки интерфейса должны быть переводимыми. В классах обычно используется `$this->t()` (трейт `StringTranslationTrait`):

```php
$message = $this->t('Task has been saved.');

$message = $this->t('Task @title has been saved.', ['@title' => $title]);
```

Маркеры подстановки:

- `@value` — текст с экранированием;
- `%value` — экранирование + визуальное выделение (`<em>`);
- `:value` — URL (для атрибутов `href`/`src`), обрабатывается по правилам URL.

Для множественного числа — `formatPlural()`:

```php
$text = $this->formatPlural($count, '1 task', '@count tasks');
```

В Twig:

```twig
{{ 'Task saved'|t }}
{% trans %}Task {{ title }} saved{% endtrans %}
```

Не собирайте фразу из кусочков и не склеивайте переводимые строки: переводчик должен иметь возможность менять порядок слов. Функция перевода не заменяет экранирование — контекст вывода по-прежнему важен.

### 1.5. Перевод конфигурации

Конфигурация хранит определения полей, типы материалов, Views, настройки модулей. Часть значений переводима: например, **label** поля переводится, а **machine name** — нет. Что именно переводимо, определяет config schema (см. Блок 3).

Не путайте:

- **Configuration Management** — перенос настроек между окружениями;
- **Configuration Translation** — перевод значений настроек.

Экспорт конфигурации не является переводом, а перевод не заменяет деплой.

### 1.6. Language fallback

**Language fallback** — выбор альтернативного значения, когда нужного перевода нет. Поведение зависит от типа данных и используемого API; нельзя считать, что отсутствующий перевод всегда заменится значением языка по умолчанию. Отдельно учитывайте отсутствие перевода сущности и отсутствие перевода конкретного поля.

### 1.7. Мультиязычность и кеширование

Язык влияет на результат рендеринга, поэтому кеш должен учитывать языковой контекст. Используются cache contexts `languages:language_interface` и `languages:language_content`. Если компонент выводит заголовок переведённой статьи, кеш не должен отдавать английский вариант русскому запросу. Контекст добавляется в `#cache` render array либо «всплывает» (bubbling) из вложенных элементов (см. Блок 5).

> [!example] Практика
> 
> На практике мультиязычный сайт начинается с включения модулей Language, Interface Translation, Content Translation и Configuration Translation. Затем настраивают language negotiation и отмечают переводимые bundles и поля.
> 
> В коде все видимые пользователю строки проходят через t(), а компонентам, результат которых зависит от языка, задают cache contexts languages:language_interface или languages:language_content. Значения вроде статусов или кодов хранят в машинном виде, а подписи переводят. Например, в DrupalJira статусы задач хранятся как in_progress и done, а подписи выводятся через перевод.

> [!warning] Типичные ошибки
> 
> - Смешивать язык интерфейса и язык содержимого.
> - Не проверять наличие перевода перед `getTranslation()`.
> - Ожидать отдельные значения для непереводимого поля.
> - Оставлять пользовательские сообщения без `t()`.
> - Забывать языковые cache contexts.
> - Переводить machine names полей и конфигурационных объектов.
> - Предполагать одинаковое поведение fallback для всех типов данных.

> [!question] Частые вопросы на собеседовании
> 
> **В чём разница между interface language и content language?** Interface language — язык системных строк и элементов UI. Content language — язык отображаемого содержимого сущностей. Они определяются отдельно и могут различаться.
> 
> **Что будет, если вызвать getTranslation() для несуществующего перевода?** Бросается исключение, перевод автоматически не создаётся. Сначала проверяют hasTranslation(), а для создания используют addTranslation().
> 
> **Что такое переводимое и непереводимое поле?** Переводимость настраивается на уровне поля. Значение непереводимого поля общее для всех переводов сущности (например, внешний ID или дата).
> 
> **Чем Configuration Translation отличается от Configuration Management?** Configuration Translation переводит значения настроек (например, метки полей). Configuration Management переносит настройки между окружениями. Это разные механизмы.

> [!tip] Ответ на собеседовании (EN)
> 
> In Drupal, multilingual support has three separate layers: interface language, content language and configuration language. The language of each request is resolved by language negotiation, which can use the URL prefix, the domain, the user's preference or the browser's Accept-Language header.
> 
> Entities and fields can be translatable. A translation is not a separate node: it lives on the same entity with the same ID, and non-translatable fields are shared between all translations. In code I always check hasTranslation() before getTranslation(), because getTranslation() does not create a translation.
> 
> All user-facing strings go through t() with placeholders, so translators can reorder words. Finally, render caches must vary by language through cache contexts, otherwise one user can see another language's cached output.

---

## Блок 2. REST API

> [!abstract] Кратко
> 
> REST в Drupal — это не просто «вернуть JSON». Есть три основных пути: REST-модуль ядра с плагинами ресурсов, JSON:API для стандартных операций с сущностями и собственные маршруты с контроллерами.
> 
> Какой бы путь ни выбрали, нужны аутентификация, авторизация, валидация входных данных, корректные HTTP-коды и метаданные кеша.

### 2.1. Что такое REST

REST (Representational State Transfer) — архитектурный стиль, при котором клиент работает с ресурсами через HTTP. Например, React-фронтенд запрашивает список задач из Drupal и отправляет запросы на смену статуса.

Важны структура ресурсов, HTTP-методы, коды ответов, аутентификация, проверка доступа, валидация и обработка ошибок.

### 2.2. Ресурсы и HTTP-методы

**Ресурс** — объект или представление данных (задача, коллекция задач). **URI** идентифицирует ресурс или коллекцию.

|Метод|Назначение|
|---|---|
|`GET`|получить представление ресурса|
|`POST`|создать ресурс или выполнить операцию|
|`PATCH`|частично изменить ресурс|
|`PUT`|создать или полностью заменить ресурс (по контракту API)|
|`DELETE`|удалить ресурс|

Пример запроса к JSON:API (`node` — entity type, `task` — bundle):

```http
GET /jsonapi/node/task
Accept: application/vnd.api+json
```

С фильтром, связью и пагинацией:

```http
GET /jsonapi/node/task?filter[field_status]=in_progress&include=field_project&page[limit]=20
```

Основные HTTP-коды:

|Код|Значение|
|---|---|
|`200 OK`|успешно|
|`201 Created`|ресурс создан|
|`204 No Content`|выполнено без тела ответа|
|`400 Bad Request`|запрос некорректен|
|`401 Unauthorized`|нет подходящей аутентификации|
|`403 Forbidden`|доступ запрещён|
|`404 Not Found`|ресурс не найден или не раскрывается клиенту|
|`405 Method Not Allowed`|метод не поддерживается|
|`422 Unprocessable Entity`|не прошла валидация сущности|

Конкретный ответ зависит от реализации endpoint и настроек безопасности.

### 2.3. REST в Drupal

Способы построить API:

- **REST-модуль ядра** (REST Resource plugins);
- **JSON:API** (модуль ядра);
- **собственные маршруты и контроллеры** (`JsonResponse` / `CacheableJsonResponse`);
- **Views REST export** — список в JSON/XML из Views;
- contrib: **GraphQL** и др.

Инфраструктура REST:

- **REST Resource plugins** — реализация операций;
- **Serialization API** — normalization и encoding;
- **Authentication API** — определение пользователя;
- **Access API** — проверка разрешения операции.

JSON:API — отдельная реализация, следующая спецификации JSON:API. Не путайте её с REST Resource plugins.

### 2.4. REST Resource plugins

Собственный ресурс — плагин, наследующий `ResourceBase`.

```php
<?php

namespace Drupal\example\Plugin\rest\resource;

use Drupal\rest\Plugin\ResourceBase;
use Drupal\rest\ResourceResponse;

/**
 * Provides a simple tasks resource.
 *
 * @RestResource(
 *   id = "example_tasks",
 *   label = @Translation("Example tasks"),
 *   uri_paths = {
 *     "canonical" = "/api/tasks"
 *   }
 * )
 */
final class TasksResource extends ResourceBase {

  public function get(): ResourceResponse {
    return new ResourceResponse([
      'data' => [
        ['id' => 1, 'title' => 'Prepare notes'],
      ],
    ]);
  }

}
```

Это упрощённая структура. В production нужны контракт API, проверки доступа, сериализация, ошибки и cache-метаданные (`ResourceResponse` — кешируемый ответ; добавляйте `addCacheableDependency()`). В новых версиях Drupal аннотация заменяется PHP-атрибутом `#[RestResource]`.

Ресурс нужно **включить и настроить** — наличие класса не открывает endpoint. Настройка — конфигурационная сущность `rest.resource.*`:

```yaml
id: example_tasks
plugin_id: example_tasks
granularity: resource
configuration:
  methods: [GET]
  formats: [json]
  authentication: [cookie]
```

Доступ к методам выдаётся permissions вида `restful get example_tasks`.

### 2.5. REST Resource и JSON:API

|REST Resource|JSON:API|
|---|---|
|собственные REST-плагины|стандартизированный API сущностей|
|структуру ответа определяет разработчик|структура соответствует спецификации JSON:API|
|для специализированных операций|для стандартных операций с сущностями|
|нужна собственная логика|типовые операции уже реализованы|

Если нужны стандартные данные сущностей — сначала оцените JSON:API. Для специальной бизнес-операции — REST Resource или собственный маршрут. JSON:API по умолчанию допускает запись; для read-only API его настраивают соответствующим образом.

### 2.6. Serialization API

Serialization API преобразует данные между PHP-представлением и внешним форматом.

- **Normalization** — объект → структурированное представление (массив).
- **Encoding** — нормализованные данные → формат (JSON, XML).

Обработка API-запроса (упрощённо):

1. Drupal получает HTTP-запрос.
2. Определяет маршрут и обработчик.
3. Выполняет аутентификацию и проверку доступа.
4. Обрабатывает запрос, получает данные.
5. Готовит ответ.
6. Сериализует данные.
7. Возвращает HTTP-ответ.

Порядок и распределение обязанностей зависят от реализации endpoint.

### 2.7. Authentication, Authorization и CSRF

- **Authentication** — кто выполняет запрос.
- **Authorization** — разрешена ли операция этому субъекту.
- **CSRF protection** — защита от подделки запросов через браузерную сессию.

Аутентифицированный пользователь не обязательно вправе менять конкретную сущность; проверка выполняется на сервере. При cookie-аутентификации действует CSRF-защита (токен из `/session/token` передаётся в заголовке `X-CSRF-Token`). HTTPS защищает канал, но не заменяет авторизацию и CSRF. Подробнее — Блоки 7 и 15.

### 2.8. Валидация и обработка ошибок

Проверяются:

- формат и типы значений;
- обязательные поля;
- допустимые значения перечислений;
- существование связанных сущностей;
- права пользователя;
- бизнес-ограничения.

Недостаточно проверить, что новый статус — строка: он должен быть разрешённым, а переход — допустимым. Ошибки возвращаются в согласованном формате и не раскрывают stack traces, секреты и чувствительные данные.

### 2.9. Кеширование API и CORS

Ответы можно кешировать, если это безопасно. Учитывайте query parameters, заголовки запроса, язык, права доступа, аутентификацию, персональные данные. Персонализированные ответы нельзя без контекстов помещать в общий кеш. Для собственных маршрутов используйте `CacheableJsonResponse` с cache metadata.

Если фронтенд находится на другом домене, нужен **CORS**: параметр `cors.config` в `services.yml` (`enabled`, `allowedOrigins`, `allowedMethods`, `allowedHeaders`, `supportsCredentials`).

> [!example] Практика
> 
> Для стандартных данных сущностей чаще всего берут JSON:API: он есть в ядре, соответствует спецификации и поддерживает фильтры, include и пагинацию. Для специальной операции пишут REST Resource или собственный маршрут. Списки без кода можно отдать через Views REST export.
> 
> Для отдельного фронтенда обязательны аутентификация, проверка доступа к каждой сущности, CORS и CSRF для cookie-запросов. Например, список задач DrupalJira для React-клиента — это запрос /jsonapi/node/task с фильтром.

> [!warning] Типичные ошибки
> 
> - Разрешать операции без полноценной проверки доступа.
> - Считать аутентификацию достаточной для изменения данных.
> - Игнорировать CSRF при cookie-аутентификации.
> - Не валидировать вход.
> - Путать REST Resource и JSON:API.
> - Возвращать внутренние ошибки и чувствительные сведения.
> - Использовать неподходящие HTTP-методы (например, `GET` для изменения).
> - Кешировать персональные ответы без контекстов.

> [!question] Частые вопросы на собеседовании
> 
> **Чем JSON:API отличается от REST-модуля ядра?** JSON:API — готовый стандартизированный API сущностей. REST-модуль строится на плагинах ресурсов, где структуру ответа и логику определяет разработчик.
> 
> **Что нужно, чтобы собственный REST Resource заработал?** Плагин-класс, включённый ресурс в конфигурации rest.resource.* (методы, форматы, аутентификация) и выданные permissions. Одного класса недостаточно.
> 
> **Когда 401, когда 403 и когда 404?** 401 — нет подходящей аутентификации, 403 — пользователь известен, но доступ запрещён, 404 — ресурса нет или он не раскрывается клиенту.
> 
> **Как защитить запросы с cookie-аутентификацией от CSRF?** Клиент получает токен на /session/token и передаёт его в заголовке X-CSRF-Token. Проверка прав доступа при этом всё равно обязательна.

> [!tip] Ответ на собеседовании (EN)
> 
> For standard entity data I would start with JSON:API, because it is in core, follows a specification and already supports filtering, includes and pagination. For a custom business operation I would write a REST Resource plugin or a custom route with a controller.
> 
> In both cases authentication only tells me who the caller is. I still check route and entity access, validate the input, return correct status codes such as 403 or 422, and avoid leaking internal errors.
> 
> I also think about caching: responses need cacheability metadata, personalised responses must not go to a shared cache, and if the frontend is on another domain I configure CORS.

---

## Блок 3. Configuration Management

> [!abstract] Кратко
> 
> Configuration Management хранит настройки и устройство сайта в YAML-файлах и позволяет переносить их между окружениями через Git. В конфигурацию входят типы материалов, поля, Views, роли и разрешения. Контент (сами задачи, проекты, записи времени) в конфигурацию не входит.
> 
> Идея в том, что код и конфигурация разворачиваются вместе, а на каждом окружении отдельно задаются только секреты и параметры инфраструктуры.

### 3.1. Назначение

Разработчик создаёт тип материала `Task`, поля, Views и permissions. Эти изменения нужно перенести на тестовый сервер и затем в production. Вместо ручного повторения конфигурация экспортируется в YAML, коммитится в Git и импортируется на целевом окружении.

**Конфигурация** — настройки и устройство сайта:

- типы материалов; определения полей;
- Views; настройки модулей;
- роли и разрешения;
- настройки workflow;
- настройки отображения форм и полей.

**Контент** — рабочие данные: статья, задача DrupalJira, проект, запись времени.

Не каждый объект в базе данных — контент: классификация зависит от типа сущности и её назначения. Сравнение хранилищ:

|Что|Где/как|Переносится через CM?|
|---|---|---|
|Config|active config в БД + YAML|да|
|Content|таблицы сущностей|нет (миграции, `default_content`)|
|State|`state` service, служебные значения окружения (например, время последнего cron)|нет|
|Settings|`settings.php`, секреты, пути|нет (задаются на окружении)|

### 3.2. Active configuration и synchronization storage

- **Active configuration** — конфигурация работающего сайта, обычно в БД.
- **Synchronization storage** — каталог экспортированных файлов для переноса. Путь задаётся в `settings.php` (`$settings['config_sync_directory']`).

Рабочий процесс:

1. Разработчик меняет настройки.
2. Экспортирует конфигурацию.
3. Проверяет YAML.
4. Коммитит в Git.
5. Разворачивает код и конфигурацию на другом окружении.
6. Импортирует конфигурацию.

### 3.3. Команды Drush

```bash
# Экспорт активной конфигурации в sync-каталог
drush config:export     # drush cex

# Импорт из sync-каталога
drush config:import     # drush cim

# Различия между active и sync
drush config:status     # drush cst

# Просмотр и изменение одного объекта
drush config:get system.site
drush config:set system.site name 'DrupalJira'
```

Типичная последовательность деплоя: `drush updb` → `drush cim` → `drush cr` (или `drush deploy`, который выполняет эти шаги). Перед импортом проверьте изменения и готовность окружения: импорт создаёт, изменяет и **удаляет** конфигурационные объекты — это небезопасно на production без проверки.

### 3.4. YAML, machine names и UUID

Пример `node.type.task.yml`:

```yaml
langcode: en
status: true
name: Task
type: task
description: 'Work item for a project.'
new_revision: true
```

Это иллюстративный фрагмент; реальная конфигурация содержит больше ключей и зависимостей.

- **Machine name** — технический идентификатор.
- **Label** — отображаемое название.
- **UUID** — уникальный идентификатор объекта.

`field_status` — machine name, `Status` — label. Machine names используются в коде и зависимостях, поэтому их смена требует анализа связанных объектов. UUID **сайта** в sync-каталоге должен совпадать с UUID сайта-цели — иначе полный импорт будет отклонён (сайты должны происходить от одной установки).

### 3.5. Config schema

Config schema описывает структуру и типы значений конфигурации. Она помогает Drupal интерпретировать данные, валидировать структуру и определять переводимые значения.

```yaml
# config/install/example.settings.yml
api_endpoint: 'https://api.example.com'
timeout: 30
enabled: true
```

```yaml
# config/schema/example.schema.yml
example.settings:
  type: config_object
  label: 'Example settings'
  mapping:
    api_endpoint:
      type: uri
      label: 'API endpoint'
    timeout:
      type: integer
      label: 'Timeout'
    enabled:
      type: boolean
      label: 'Enabled'
```

Схема должна соответствовать реальной структуре. Её отсутствие или неверные типы осложняют валидацию, перевод и сопровождение (в тестах это даёт ошибки схемы).

### 3.6. `config/install` и `config/optional`

- `config/install` — конфигурация, ставящаяся при установке модуля, если выполнены условия.
- `config/optional` — конфигурация, ставящаяся при наличии нужных зависимостей.

Добавление файла в `config/install` **не обновляет** уже существующие сайты. Для них используют управляемый импорт конфигурации или update hooks.

### 3.7. Dependencies

Конфигурационные объекты зависят друг от друга: View ссылается на тип материала, поля, форматтеры, модули. Если зависимость отсутствует, импорт может завершиться ошибкой. Проверяйте:

- включены ли нужные модули;
- существуют ли нужные объекты;
- корректны ли machine names;
- совместимы ли схемы;
- не удаляется ли объект, на который ссылаются другие;
- соответствуют ли UUID и зависимости ожидаемому состоянию.

### 3.8. Configuration Management и update hooks

CM — декларативное описание и перенос настроек. Update hooks — программное преобразование состояния существующей установки.

```php
/**
 * Adds a new configuration value.
 */
function example_update_10001(): void {
  $config = \Drupal::configFactory()->getEditable('example.settings');

  $config
    ->set('enabled', TRUE)
    ->save();
}
```

Реальная логика должна учитывать существующие значения. Update hooks не должны без нужды дублировать обычный export/import. Подробнее о batch в update hook — Блок 12.

### 3.9. Окружения, overrides и Config Split

Окружения должны использовать согласованную конфигурацию, но могут отличаться настройками: секреты, пароли, адреса БД. Их не хранят в публичном репозитории: используют `settings.php`, переменные окружения, secret management.

Разделяйте:

- настройки приложения → конфигурация;
- секреты и параметры инфраструктуры → окружение.

Инструменты:

- **Config overrides** в `settings.php`: `$config['system.site']['name'] = 'Dev';` — значение подменяется «на лету» и не попадает в экспорт.
- **Config Split** (contrib) — разные наборы конфигурации для окружений (например, dev-модули включены только локально).
- **Config Ignore** (contrib) — исключает объекты из импорта/экспорта.

> [!example] Практика
> 
> Типичный процесс: разработчик меняет настройки локально, экспортирует их командой drush cex, коммитит YAML в Git, а на другом окружении выполняет деплой кода, updb, cim и cr. Перед импортом смотрят различия через drush cst, потому что импорт может и удалять объекты.
> 
> Для различий между окружениями используют config overrides в settings.php, Config Split или Config Ignore, а секреты хранят вне репозитория. В DrupalJira в Git попадают типы Project и Task, их поля, роли и Views, а сами задачи — это контент и в экспорт не входят.

> [!warning] Типичные ошибки
> 
> - Менять конфигурацию на production без переноса в Git.
> - Считать, что `config/install` обновляет существующие сайты.
> - Игнорировать зависимости YAML.
> - Удалять конфигурационные объекты без проверки ссылок.
> - Путать контент с конфигурацией.
> - Заменять экспорт/импорт update hook'ами без причины.
> - Импортировать без просмотра изменений (`drush cst`).
> - Хранить секреты в репозитории.

> [!question] Частые вопросы на собеседовании
> 
> **Чем конфигурация отличается от контента?** Конфигурация описывает устройство и настройки сайта и переносится через YAML. Контент — рабочие данные (материалы, пользователи), он через Configuration Management не переносится.
> 
> **Что произойдёт, если UUID сайта в sync-каталоге не совпадает с целевым?** Полный импорт будет отклонён: сайты должны происходить от одной установки.
> 
> **Как задать разные настройки для окружений?** Через $config overrides и секреты в settings.php, а также Config Split для наборов конфигурации (например, dev-модули только локально).
> 
> **Обновит ли файл в config/install уже существующие сайты?** Нет, он применяется только при установке модуля. Для существующих сайтов нужен импорт конфигурации или update hook.

> [!tip] Ответ на собеседовании (EN)
> 
> Configuration Management lets me export site configuration, such as content types, fields, views and roles, to YAML files, commit them to Git and import them on other environments. Content is not configuration, so nodes and custom entities are not transferred this way.
> 
> A typical deployment is: update the code, run database updates, import configuration and clear caches. Before importing I review the differences with drush config:status, because an import can also delete configuration objects.
> 
> Secrets and environment-specific values stay out of the repository and are set in settings.php or environment variables. When existing data has to change, I use update hooks, because config import only changes configuration.

---

## Блок 4. Services и Dependency Injection

> [!abstract] Кратко
> 
> Service — это объект с определённой функциональностью, который зарегистрирован в service container и может использоваться разными частями Drupal. Dependency Injection — принцип, при котором класс получает зависимости извне, чаще всего через конструктор, а не создаёт их сам.
> 
> Это делает код переиспользуемым, слабо связанным и легко тестируемым: в тесте зависимость можно заменить.

### 4.1. Что такое Service

Service — объект, предоставляющий функциональность разным компонентам. Вместо того чтобы реализовывать отправку письма, загрузку сущности или логирование в каждом классе, Drupal предоставляет сервисы.

|Сервис|Назначение|
|---|---|
|`entity_type.manager`|типы сущностей и их storage|
|`current_user`|текущий пользователь и права|
|`logger.factory`|логгеры|
|`database`|соединение с БД|
|`cache.default`|кеш-хранилище|
|`language_manager`|языки|
|`config.factory`|конфигурация|
|`http_client`|HTTP-запросы наружу|
|`request_stack`|текущий запрос|
|`event_dispatcher`|отправка событий|
|`queue`|очереди|

Сервисы регистрируются в контейнере и получают уникальные идентификаторы.

### 4.2. Service Container

Service Container хранит определения сервисов и отвечает за создание объектов и подстановку зависимостей. Drupal использует Symfony DependencyInjection.

Контроллеру нужны менеджер сущностей и логгер — он получает их извне, а не создаёт сам.

Преимущества:

- повторное использование;
- централизованное управление зависимостями;
- простое тестирование;
- меньшая связанность;
- возможность подмены реализаций.

Обычно сервис создаётся один раз за жизненный цикл контейнера, но не каждый сервис — глобальный singleton на весь процесс PHP: поведение зависит от определения сервиса и окружения.

### 4.3. Dependency Injection

**DI** — объект получает зависимости извне, а не создаёт их сам.

Без DI:

```php
$storage = \Drupal::entityTypeManager()->getStorage('node');
```

Работает, но жёстко привязывает класс к статическому API.

С DI:

```php
<?php

namespace Drupal\example;

use Drupal\Core\Entity\EntityTypeManagerInterface;
use Drupal\node\NodeInterface;

final class TaskManager {

  public function __construct(
    private readonly EntityTypeManagerInterface $entityTypeManager,
  ) {}

  public function loadTask(int $id): ?NodeInterface {
    $entity = $this->entityTypeManager
      ->getStorage('node')
      ->load($id);

    return $entity instanceof NodeInterface && $entity->bundle() === 'task'
      ? $entity
      : NULL;
  }

}
```

DI делает зависимости явными и позволяет подменять интерфейс в тестах. Статический `\Drupal::service()` допустим там, где DI недоступен (процедурный код, `.module`, hooks в старом стиле).

### 4.4. Регистрация собственного сервиса

Файл `example.services.yml`:

```yaml
services:
  example.task_manager:
    class: Drupal\example\TaskManager
    arguments:
      - '@entity_type.manager'
```

- `example.task_manager` — идентификатор;
- `class` — реализующий класс;
- `arguments` — зависимости конструктора;
- `@entity_type.manager` — ссылка на существующий сервис.

После регистрации в обычном коде сервис внедряют в потребляющий класс, а не берут из контейнера. После изменения `services.yml` нужен `drush cr`.

Дополнительные возможности определения сервиса:

- **`tags`** — помечают сервис для обнаружения подсистемой (`event_subscriber`, `access_check`, `normalizer`, `authentication_provider`, `cache.bin` и др.);
- **decorators** — подмена/расширение сервиса без правки оригинала;
- **`parent`**, **`lazy: true`**, **`public: false`** и др. — тонкая настройка.

```yaml
services:
  example.task_manager.logged:
    class: Drupal\example\LoggedTaskManager
    decorates: example.task_manager
    arguments: ['@example.task_manager.logged.inner', '@logger.factory']
```

### 4.5. DI в контроллерах

Контроллеры получают сервисы через статический метод `create()` (`ContainerInjectionInterface`; `ControllerBase` его уже реализует).

```php
<?php

namespace Drupal\example\Controller;

use Drupal\Core\Controller\ControllerBase;
use Drupal\Core\Entity\EntityTypeManagerInterface;
use Symfony\Component\DependencyInjection\ContainerInterface;

final class TaskController extends ControllerBase {

  public function __construct(
    private readonly EntityTypeManagerInterface $entityTypeManager,
  ) {}

  public static function create(ContainerInterface $container): static {
    return new static(
      $container->get('entity_type.manager'),
    );
  }

  public function view(int $node): array {
    $task = $this->entityTypeManager->getStorage('node')->load($node);

    if (!$task || $task->bundle() !== 'task') {
      return ['#markup' => $this->t('Task not found.')];
    }

    return [
      '#type' => 'item',
      '#title' => $task->label(),
    ];
  }

}
```

Это демонстрация DI, а не production-контроллер. В реальном маршруте используют требования доступа и параметрический конвертер сущностей (`{node}` в маршруте), а не ручную загрузку (см. Приложение A). Тот же паттерн `create()` применяется в формах.

### 4.6. DI в плагинах

Плагины создаёт **Plugin Manager**, поэтому внедрение отличается: используется `ContainerFactoryPluginInterface` и метод `create()` с аргументами `$configuration`, `$plugin_id`, `$plugin_definition`. Нельзя просто добавить параметр в конструктор плагина — менеджер сам его не передаст. Полный пример — Блок 11.

### 4.7. Autowiring и явная регистрация

В Symfony-проектах распространён autowiring — зависимости определяются по типам аргументов. В Drupal его поддержка зависит от версии и от способа регистрации сервиса (нужны alias'ы интерфейсов). Явная регистрация через `services.yml` делает зависимости очевидными, и именно ей стоит отдавать предпочтение. Не рассчитывайте, что любая зависимость разрешится только потому, что указана в type declaration конструктора.

### 4.8. Почему DI улучшает тестируемость

Если класс сам обращается к глобальному контейнеру, его сложно тестировать изолированно. При передаче через конструктор в тест можно подставить mock/stub (например, подменный `EntityTypeManagerInterface`). Но DI не означает, что всё нужно мокать: для проверки реального взаимодействия с Drupal API лучше Kernel/Functional tests (Блок 6).

### 4.9. Когда не нужно создавать сервис

Сервис оправдан, когда логика:

- используется в нескольких местах;
- имеет самостоятельную ответственность;
- требует DI;
- должна тестироваться изолированно;
- представляет отдельную бизнес-операцию.

Для небольшой локальной операции сервис лишь усложняет проект.

> [!example] Практика
> 
> Свои сервисы описывают в module.services.yml: класс, аргументы, при необходимости теги (event_subscriber, access_check и др.) и декораторы. Потребители получают зависимости через конструктор: контроллеры и формы через create(), плагины через ContainerFactoryPluginInterface.
> 
> Статический \Drupal::service() допустим в процедурном коде, но в классах от него отказываются ради тестируемости. Пример из DrupalJira: TaskStatService со статистикой по задачам, который используют блок, контроллер и update hook.

> [!warning] Типичные ошибки
> 
> - Использовать `\Drupal::service()` повсюду вместо DI.
> - Создавать «божественные» сервисы с множеством обязанностей.
> - Передавать зависимости через сеттеры без необходимости.
> - Забыть зарегистрировать сервис или сбросить кеш.
> - Неправильно передавать зависимости в конструкторы плагинов.
> - Зависеть от конкретных классов там, где достаточно интерфейса.
> - Считать, что все объекты Drupal создаются одинаково.

> [!question] Частые вопросы на собеседовании
> 
> **Что такое service container?** Компонент Symfony DependencyInjection, который хранит определения сервисов, создаёт объекты и подставляет зависимости.
> 
> **Почему лучше DI, чем \Drupal::service()?** DI делает зависимости явными, снижает связанность и позволяет подменять зависимости в тестах.
> 
> **Как внедрить сервис в плагин?** Реализовать ContainerFactoryPluginInterface и метод create(), потому что плагины создаёт Plugin Manager, а не контейнер напрямую.
> 
> **Что такое service tags и decorators?** Тег помечает сервис для обнаружения подсистемой (например, event_subscriber). Декоратор подменяет или расширяет существующий сервис без изменения его кода.

> [!tip] Ответ на собеседовании (EN)
> 
> A service is a reusable object registered in the service container, and dependency injection means that a class receives its dependencies through the constructor instead of creating them or calling \Drupal::service() directly. It keeps classes loosely coupled and makes them easy to test.
> 
> I register my own services in the module's services.yml with their arguments. Controllers and forms get services through the create() method, and plugins use ContainerFactoryPluginInterface, because a plugin manager creates them, not the container directly.
> 
> I do not create a service for every small function. A service makes sense when the logic has its own responsibility, is used in several places or needs to be tested in isolation.

---

## Блок 5. Производительность и масштабирование (Performance & Scalability)

> [!abstract] Кратко
> 
> Performance — это скорость при определённой нагрузке, scalability — способность расти вместе с нагрузкой. Основа производительности Drupal — правильное кеширование через три механизма: cache contexts (от чего зависит результат), cache tags (от каких данных) и max-age (сколько живёт).
> 
> Вторая группа проблем — лишние запросы к БД, в первую очередь N+1, и тяжёлая работа в HTTP-запросе. Любую оптимизацию начинают с измерений, а не с предположений.

### 5.1. Производительность и масштабирование

Сайт может быть быстрым при малом числе посетителей и медленным при одновременных запросах — поэтому понятия связаны, но не тождественны.

Источники проблем:

- избыточные запросы к БД;
- повторная загрузка одних и тех же сущностей;
- неправильное кеширование;
- тяжёлые операции в каждом HTTP-запросе;
- большие render arrays и сложный рендеринг;
- вызовы внешних API;
- неэффективные индексы;
- длительные фоновые операции в синхронном режиме.

Начинать нужно с **измерений**, а не с предположений. Инструменты: Webprofiler (Devel), Xdebug/профайлеры (Blackfire, XHProf), логирование медленных SQL-запросов, APM (New Relic и др.).

### 5.2. Cache API и уровни кеширования

- **Internal Page Cache** — полные ответы страниц для подходящих анонимных запросов.
- **Dynamic Page Cache** — страницы и динамические части с учётом контекстов; полезен и для аутентифицированных пользователей.
- **Render Cache** — результаты рендеринга отдельных компонентов.

Механизмы дополняют друг друга. Кеширование уменьшает повторные вычисления, но не исправляет архитектурные проблемы. Для данных вне render-системы используют Cache API (`cache.default`, `cache.data`, собственные cache bins) с теми же tags и max-age.

### 5.3. Cache contexts

**Cache context** описывает, от какого контекста зависит результат (кеш «варьируется»). Если блок показывает разный текст в зависимости от роли, кеш должен учитывать роль.

|Context|Что означает|
|---|---|
|`user`|текущий пользователь|
|`user.roles`|роли пользователя|
|`languages:language_interface`|язык интерфейса|
|`languages:language_content`|язык содержимого|
|`url.path`|путь URL|
|`url.query_args`|параметры запроса|

```php
return [
  '#markup' => $message,
  '#cache' => [
    'contexts' => [
      'user.roles',
      'languages:language_interface',
    ],
  ],
];
```

Не добавляйте контексты без необходимости: много вариантов кеша снижает его эффективность.

### 5.4. Cache tags

Cache tags описывают, **от каких данных** зависит результат. Если задача изменилась, связанный кеш должен инвалидироваться.

```php
return [
  '#markup' => $task->label(),
  '#cache' => [
    'tags' => $task->getCacheTags(),   // например, node:123
  ],
];
```

При изменении или удалении сущности Drupal инвалидирует кеш по её тегам. Теги отвечают за зависимость от данных, а не за вариации по пользователю или языку. Для списков используют теги списков (`node_list`) или собственные.

### 5.5. Cache max-age

`max-age` — максимальное время жизни в секундах.

```php
return [
  '#markup' => $message,
  '#cache' => ['max-age' => 3600],
];
```

- `0` — не кешировать;
- `Cache::PERMANENT` — без временного ограничения (инвалидация по тегам остаётся);
- кеш может быть инвалидирован раньше, если изменились данные.

Осторожно: `max-age: 0` в одном вложенном элементе «всплывает» (bubbling) и отключает кеширование всей страницы.

### 5.6. Cacheability и cache bubbling

**Cacheability** — contexts, tags, max-age объекта. **Cache bubbling** — распространение этих свойств от вложенных элементов к родительским при построении результата. Если блок включает данные задачи, зависящие от языка и имеющие теги, итоговая страница должна их учитывать.

Компонент без корректных метаданных может выдать устаревшие или чужие персонализированные данные. Объекты доступа тоже несут cacheability metadata (`AccessResult::allowedIf(...)->addCacheContexts([...])`).

### 5.7. Entity loading и N+1

**N+1** — сначала список сущностей, затем отдельный запрос для каждой. Например, 100 задач и отдельная загрузка проекта для каждой.

Способы смягчить:

- Entity Query для получения ID;
- `loadMultiple()` вместо цикла `load()`;
- предзагрузка связанных сущностей;
- Views и другие подходящие механизмы;
- кеширование повторно используемых данных;
- агрегирующие SQL-запросы для статистики (суммы часов).

```php
$storage = $this->entityTypeManager->getStorage('node');
$tasks = $storage->loadMultiple($ids);   // один пакетный вызов
```

Измеряйте реальные SQL-запросы: не каждая последовательность обращений — серьёзная проблема.

### 5.8. Database indexes

Индексы позволяют находить строки без полного сканирования. Нужны для частых условий фильтрации, сортировки, соединений. Но:

- занимают место;
- замедляют запись;
- требуют правильного выбора колонок и их порядка;
- не гарантируют ускорение любого запроса.

Анализируйте SQL и план выполнения (`EXPLAIN`), а не добавляйте индексы наугад.

### 5.9. BigPipe, placeholders и lazy builders

**BigPipe** отправляет часть HTML раньше, а динамические участки доставляет позже. **Placeholders** откладывают рендеринг отдельных элементов. Для персонализированных частей используют **lazy builders**:

```php
$build['summary'] = [
  '#lazy_builder' => ['drupaljira_timelog.task_stat:buildSummary', [$task_id]],
  '#create_placeholder' => TRUE,
];
```

Callback должен быть доверенным (`TrustedCallbackInterface`). Эти механизмы не ускоряют сами операции и не заменяют кеширование и оптимизацию запросов.

### 5.10. OPcache, Redis, CDN и reverse proxy

- **OPcache** — кеш байткода PHP.
- **Redis** (или Memcache) — внешнее кеш-хранилище.
- **CDN** — статические ресурсы и, при настройке, кешируемые ответы ближе к посетителям.
- **Reverse proxy** (Varnish, Nginx) — кеш HTTP-ответов перед PHP.

Это разные уровни. Само их наличие не ускоряет сайт: нужны корректная настройка кеша, инвалидации (tags → purge) и безопасности. На production также включают агрегацию CSS/JS, оптимизируют изображения (image styles) и отключают dev-модули (Devel, Webprofiler).

### 5.11. Масштабирование

**Горизонтальное масштабирование** — несколько экземпляров приложения. Для Drupal учитывают:

- общую или согласованную БД;
- доступность файлов (shared storage / S3);
- общие кеши и locks;
- согласованную конфигурацию;
- выполнение cron и фоновых задач (один раз, а не на каждом узле);
- хранение сессий;
- ограничения внешних API и БД.

Если два экземпляра независимо обрабатывают одну задачу, возможны дубли и конфликты.

> [!example] Практика
> 
> Практика кеширования в Drupal — это правильные метаданные в render arrays: contexts для вариаций (роль, язык, URL), tags для инвалидации при изменении данных и max-age для ограничения по времени. Для медленных персонализированных частей используют lazy builders и BigPipe.
> 
> Для запросов — loadMultiple вместо загрузки в цикле, агрегирующие запросы и индексы по результатам EXPLAIN, а тяжёлые операции выносят в очередь. Например, страница проекта с итогами по часам в DrupalJira должна считать суммы одним запросом, а не загружать записи по одной.

> [!warning] Типичные ошибки
> 
> - Отключать кеш вместо исправления его настройки.
> - Забывать cache tags и contexts.
> - Использовать `max-age` как единственный механизм инвалидации.
> - Создавать N+1 при работе с сущностями.
> - Добавлять индексы без анализа SQL.
> - Выполнять тяжёлую обработку в HTTP-запросе.
> - Масштабировать PHP, игнорируя ограничения БД.
> - Кешировать персональные данные без контекстов и правил доступа.

> [!question] Частые вопросы на собеседовании
> 
> **Из чего состоит cacheability?** Из cache contexts (от чего зависит результат), cache tags (от каких данных) и max-age (время жизни).
> 
> **Что такое cache bubbling?** Распространение метаданных кеша от вложенных элементов к родительским при построении страницы. Из-за него max-age: 0 в одном элементе отключает кеш всей страницы.
> 
> **Как избежать проблемы N+1?** Загружать сущности пакетно (loadMultiple), получать связанные данные одним запросом или Views, использовать агрегирующие запросы.
> 
> **Чем Internal Page Cache отличается от Dynamic Page Cache?** Page Cache кеширует целые ответы для анонимных запросов. Dynamic Page Cache кеширует страницы с динамическими частями с учётом контекстов, в том числе для авторизованных пользователей.

> [!tip] Ответ на собеседовании (EN)
> 
> I start with measuring: profiling, query logs and checking what is actually slow. In Drupal the most important tool is correct cacheability metadata. Cache contexts describe variations, cache tags describe dependencies for invalidation and max-age sets a time limit.
> 
> I avoid the N+1 problem by loading entities with loadMultiple or by using a single query instead of loading in a loop, and I add database indexes only after looking at the query plan. Heavy work such as sending emails goes into a queue.
> 
> Infrastructure helps on top of that: OPcache, Redis, a CDN or a reverse proxy, and BigPipe for personalised parts of a page. For scaling horizontally I have to think about the shared database, files, cache and running cron once.

---

## Блок 6. Тестирование (Testing)

> [!abstract] Кратко
> 
> Тесты проверяют поведение кода и защищают от регрессий. В Drupal четыре уровня: Unit-тесты для изолированной логики, Kernel-тесты, которые поднимают ядро и тестовую БД, Functional-тесты через HTTP и FunctionalJavascript, когда нужен браузер с JS.
> 
> Правило простое: проверять функциональность на самом простом уровне, который подтверждает её корректность. И всегда тестировать права доступа, причём не только успешный сценарий.

### 6.1. Зачем нужны тесты

Модули тесно взаимодействуют с Entity API, сервисами, маршрутизацией, конфигурацией, доступом и рендерингом. Тесты помогают находить:

- ошибки бизнес-логики;
- нарушения прав доступа;
- проблемы интеграции с БД;
- ошибки конфигурации;
- неверное поведение маршрутов и форм;
- регрессии после изменений.

### 6.2. Unit tests

Проверяют небольшую часть кода изолированно от установки Drupal; зависимости заменяются mocks/stubs. Наследуются от `UnitTestCase`.

```php
<?php

namespace Drupal\Tests\example\Unit;

use Drupal\Tests\UnitTestCase;

final class TaskTitleTest extends UnitTestCase {

  public function testTitleIsFormatted(): void {
    $title = 'Prepare interview notes';

    self::assertSame(
      'Task: Prepare interview notes',
      'Task: ' . $title,
    );
  }

}
```

Пример не загружает Drupal и не проверяет Entity API. Unit tests быстрые и подходят для вычислений и логики, отделимой от инфраструктуры.

### 6.3. Kernel tests

Запускают значительную часть инфраструктуры Drupal: контейнер, систему сущностей, тестовую БД. Наследуются от `KernelTestBase`. Применяются для:

- создания и загрузки сущностей;
- работы с сервисами;
- проверки hooks и событий;
- проверки конфигурации;
- интеграции с БД.

Нужно явно подключить модули и установить схемы/конфигурацию:

```php
<?php

namespace Drupal\Tests\drupaljira_timelog\Kernel;

use Drupal\KernelTests\KernelTestBase;

/**
 * @group drupaljira_timelog
 */
final class TaskStatServiceTest extends KernelTestBase {

  protected static $modules = ['system', 'user', 'node', 'field', 'text', 'drupaljira_timelog'];

  protected function setUp(): void {
    parent::setUp();
    $this->installEntitySchema('user');
    $this->installEntitySchema('node');
    $this->installConfig(['node', 'drupaljira_timelog']);
  }

  public function testLoggedHours(): void {
    $service = $this->container->get('drupaljira_timelog.task_stat');
    // Создать Project, Task и записи time_log, затем проверить расчёт.
    self::assertNotNull($service);
  }

}
```

### 6.4. Functional tests

Проверяют поведение сайта через HTTP и браузероподобный механизм. Наследуются от `BrowserTestBase`. Позволяют создавать пользователей и роли, входить в систему, открывать страницы, отправлять формы, проверять редиректы, сообщения и доступ к маршрутам.

```php
final class TaskAccessTest extends BrowserTestBase {

  protected static $modules = ['node', 'drupaljira_timelog'];
  protected $defaultTheme = 'stark';

  public function testAnonymousCannotEdit(): void {
    // ... создать задачу
    $this->drupalGet('/node/1/edit');
    $this->assertSession()->statusCodeEquals(403);
  }

}
```

Полезны для сценариев, затрагивающих несколько уровней приложения.

### 6.5. FunctionalJavascript tests

Нужны, если поведение зависит от клиентского JavaScript, динамического взаимодействия, браузера. Сложнее и медленнее обычных functional tests; для серверной логики не нужны.

### 6.6. Пирамида тестирования

- много быстрых unit tests;
- достаточно kernel tests для интеграции;
- functional tests для важных пользовательских сценариев;
- ограниченное число тяжёлых browser-based tests.

Это не жёсткое требование: соотношение зависит от архитектуры. Главная цель — проверять функциональность на самом простом уровне, который действительно подтверждает корректность.

### 6.7. Mocks и stubs

- **Mock** — объект, позволяющий задавать ожидаемые вызовы и проверять взаимодействие.
- **Stub** — объект с заранее заданными ответами.

Сервис уведомлений подменяют, чтобы тест не отправлял письма. Но избыток моков даёт тесты деталей реализации, а не поведения. Для проверки интеграции с Drupal API лучше kernel/functional test.

### 6.8. Тестирование доступа

Проверка доступа — одна из важнейших областей. Недостаточно проверить, что администратор может выполнить операцию; нужно проверить и пользователя с ограниченными правами. Для DrupalJira:

- администратор редактирует задачу;
- менеджер проекта выполняет разрешённые операции;
- обычный пользователь не меняет чужие задачи;
- анонимный пользователь не получает закрытые данные;
- недоступная сущность не раскрывается через API.

Учитывайте route access и entity access, если применимы оба.

### 6.9. Fixtures и тестовые данные

В Drupal тестовые сущности обычно создают прямо в тесте, чтобы не зависеть от содержимого БД. Данные должны быть:

- детерминированными;
- независимыми от случайного состояния окружения;
- минимальными, но достаточными;
- понятными по назначению;
- изолированными между тестами.

Для теста workflow задач создавайте задачи с известными статусами.

### 6.10. Regression testing

Проверяет, что существующая функциональность работает после изменений. Изменение доступа к TimeLog не должно случайно запретить менеджеру создавать записи времени, если это разрешено требованиями. Особенно полезны при изменениях: permissions и access handlers, workflow, entity definitions, конфигурации, API endpoints, миграций, кеширования. Тесты запускают в CI перед слиянием.

### 6.11. Запуск и статический анализ

```bash
# Запуск PHPUnit-тестов модуля (из корня проекта)
./vendor/bin/phpunit -c web/core web/modules/custom/drupaljira_timelog/tests

# Один тест
./vendor/bin/phpunit -c web/core --filter testLoggedHours web/modules/custom/drupaljira_timelog
```

Нужны настроенные `phpunit.xml` и переменные `SIMPLETEST_DB`, `SIMPLETEST_BASE_URL` для Kernel/Functional. Дополнительно используют:

- **PHPCS/PHPCBF** со стандартом `Drupal`/`DrupalPractice` — кодстайл;
- **PHPStan** (`phpstan-drupal`) — статический анализ;
- **Nightwatch/Playwright** — для front-end сценариев (по необходимости).

> [!example] Практика
> 
> Подход: unit-тесты для логики без Drupal, Kernel-тесты для сервисов и сущностей на реальной тестовой БД, Functional-тесты для маршрутов, форм и прав, FunctionalJavascript только когда нужен JS. Тестовые данные создают в самом тесте.
> 
> Для кодстайла используют PHPCS с правилами Drupal, для статического анализа — PHPStan с phpstan-drupal, всё это запускают в CI. Для сервиса статистики в DrupalJira подходит Kernel-тест, а для проверки доступа к записям времени — Functional.

> [!warning] Типичные ошибки
> 
> - Проверять только успешный сценарий.
> - Слишком много mocks без проверки интеграции.
> - Тесты, зависящие от существующих данных сайта.
> - Не проверять права доступа.
> - Functional test на каждую мелкую функцию.
> - Игнорировать нестабильные тесты.
> - Проверять детали реализации вместо поведения.
> - Не запускать тесты после изменения конфигурации или схемы сущностей.

> [!question] Частые вопросы на собеседовании
> 
> **Какие типы тестов есть в Drupal?** Unit, Kernel, Functional (BrowserTestBase) и FunctionalJavascript. Выбирают самый простой уровень, который подтверждает поведение.
> 
> **Чем Kernel-тест отличается от Functional?** Kernel поднимает ядро и БД без браузера, нужные модули и схемы подключают вручную. Functional работает через HTTP с полноценной установкой сайта.
> 
> **Как тестировать доступ?** Проверять и разрешённого, и запрещённого пользователя: роли с разными правами, анонима, доступ к чужой сущности. Учитывать route access и entity access.
> 
> **Зачем $modules и $defaultTheme в тесте?** $modules — список включаемых для теста модулей, $defaultTheme обязателен для BrowserTestBase и задаёт тему для запросов.

> [!tip] Ответ на собеседовании (EN)
> 
> I test at the cheapest level that proves the behavior. Unit tests are for pure logic without Drupal, kernel tests are for services and entities against a real test database, and functional tests are for routes, forms and permissions in a simulated browser.
> 
> I avoid overusing mocks, because tests that only check implementation details are fragile. I always test both an allowed and a forbidden user, since access control is where Drupal bugs are most expensive.
> 
> Test data is created inside the test, so it does not depend on the state of a site. I run the tests in CI together with coding standards checks and static analysis like PHPCS and PHPStan.

---

## Блок 7. Безопасность Drupal (Security)

> [!abstract] Кратко
> 
> Безопасность в Drupal многоуровневая: аутентификация, авторизация, контроль доступа, валидация входных данных, экранирование вывода, CSRF-защита и защита от SQL-инъекций. Главное правило — не доверять пользовательским данным и проверять доступ на сервере, а не в интерфейсе.
> 
> Наличие permission ещё не значит, что пользователь может работать с любой сущностью: доступ к конкретной задаче проверяется отдельно.

### 7.1. Основные принципы

Нельзя доверять данным, даже если они пришли из формы Drupal или от авторизованного пользователя.

- **Authentication** — проверка личности.
- **Authorization** — разрешено ли действие.
- **Access control** — управление доступом к сущностям, маршрутам, операциям.
- **Input validation** — проверка входных данных.
- **Output escaping** — безопасный вывод (против XSS).
- **CSRF protection** — защита от подделки межсайтовых запросов.
- **SQL injection protection** — защита от внедрения SQL.

### 7.2. Permissions и Roles

**Permission** — какое действие разрешено. **Role** — набор разрешений. В DrupalJira:

- Project Manager — создание и редактирование проектов и задач;
- Authenticated User — просмотр доступных проектов, работа с собственными задачами;
- Administrator — управление сайтом.

Разрешения — в UI или в `*.permissions.yml`:

```yaml
# drupaljira_timelog.permissions.yml
manage project tasks:
  title: 'Manage project tasks'
  description: 'Create and manage tasks in projects.'
  restrict access: true
```

Проверка в коде:

```php
$current_user = \Drupal::currentUser();

if ($current_user->hasPermission('manage project tasks')) {
  // Пользователю разрешено управлять задачами.
}
```

В классе предпочтительнее DI (`current_user`). Наличие permission не означает доступа ко всем сущностям: пользователь может иметь право редактировать задачи, но не конкретную задачу из чужого проекта.

### 7.3. Access Control

**Доступ к маршрутам** — через `requirements`:

```yaml
drupaljira_timelog.task_list:
  path: '/tasks'
  defaults:
    _controller: '\Drupal\drupaljira_timelog\Controller\TaskController::list'
    _title: 'Tasks'
  requirements:
    _permission: 'access content'
```

Drupal проверяет разрешение до вызова контроллера. Для сложных проверок — `_custom_access` или собственный access checker (сервис с тегом `access_check`). Для сущностей — `_entity_access: 'node.update'`.

**Доступ к сущностям** — Entity Access API:

```php
$access = $node->access('update', NULL, TRUE);

if ($access->isAllowed()) {
  // Пользователю разрешено редактировать материал.
}
```

Третий аргумент `TRUE` возвращает `AccessResultInterface` вместо bool; результат содержит cacheability — это важно, потому что доступ зависит от пользователя, ролей, состояния сущности.

**Нельзя полагаться на интерфейс.** Скрытая кнопка не защита: можно отправить HTTP-запрос напрямую. Проверка — на сервере: в маршруте, на сущности, в бизнес-логике. Для запросов к данным (Entity Query, Views) учитывайте `accessCheck`.

### 7.4. CSRF Protection

**CSRF** — атака, заставляющая браузер авторизованного пользователя отправить нежелательный запрос.

Form API защищает формы автоматически (form token). Для собственного маршрута, изменяющего данные:

```yaml
drupaljira_timelog.update_status:
  path: '/task/{node}/status'
  defaults:
    _controller: '\Drupal\drupaljira_timelog\Controller\TaskController::updateStatus'
  requirements:
    _entity_access: 'node.update'
    _csrf_token: 'TRUE'
  methods: [POST]
  options:
    parameters:
      node:
        type: entity:node
```

Для маршрутов с `_csrf_token` токен ожидается в параметре `token` запроса; для API-запросов с cookie-аутентификацией (REST, JSON:API) токен получают на `/session/token` и передают в заголовке `X-CSRF-Token`. Для AJAX- и REST-подобных маршрутов отдельно проверьте подходящий механизм.

CSRF-токен **не заменяет** проверку разрешений: нужно убедиться и что запрос легитимен, и что у пользователя есть право на действие.

### 7.5. XSS Protection

**XSS** — когда пользовательские данные выводятся как исполняемый HTML/JS. Например, ввод `<script>alert('XSS')</script>`.

В Twig работает автоэкранирование:

```twig
<div class="task-title">
  {{ task_title }}
</div>
```

Не отключайте его без необходимости:

```twig
{{ task_title|raw }}   {# только если безопасность гарантирована отдельно #}
```

В PHP:

```php
use Drupal\Component\Utility\Html;

$safe_title = Html::escape($title);
```

В Render API предпочтительно использовать корректные render arrays: `#plain_text` (всё экранируется), `#markup` (фильтруется по белому списку тегов `Xss::filterAdmin`), а непроверенный HTML не передаётся как доверенный. Для пользовательского HTML применяют **text formats** с фильтрами.

### 7.6. SQL Injection

**SQL injection** — внедрение SQL через данные, подставляемые в запрос.

```php
// Не делайте так.
$query = "SELECT uid FROM users_field_data WHERE name = '$username'";
```

Безопасно — Database API с параметрами:

```php
$connection = \Drupal::database();

$result = $connection->select('users_field_data', 'u')
  ->fields('u', ['uid'])
  ->condition('name', $username)
  ->execute()
  ->fetchField();
```

Значение передаётся как параметр, а не вставляется в SQL. Для контента и пользователей лучше Entity API, а не прямые обращения к таблицам. Для «сырого» SQL — `$connection->query('... WHERE name = :name', [':name' => $username])`.

### 7.7. Security Best Practices

- Принцип наименьших привилегий.
- Проверять доступ к каждой сущности и операции.
- Не доверять значениям из URL, форм, API, JavaScript.
- Использовать Entity API и Database API вместо ручного SQL.
- Не выводить пользовательский ввод как доверенный HTML.
- Использовать HTTPS; настроить `trusted_host_patterns`.
- Не хранить пароли открытым текстом; хранить `hash_salt` и секреты вне репозитория.
- Не оставлять отладочные маршруты и диагностические данные в production.
- Регулярно обновлять core и contrib; следить за security advisories (`composer audit`).
- Для загрузки файлов — ограничивать расширения и размер.
- Использовать flood control для входа и чувствительных операций.
- Добавлять тесты на разрешённые и запрещённые операции.

### 7.8. Пример: смена статуса чужой задачи

Обычный пользователь пытается изменить статус чужой задачи через `/task/123/status`. Безопасная реализация проверяет:

1. HTTP-метод (например, `POST`).
2. CSRF-токен.
3. Что пользователь авторизован.
4. Что он вправе менять **именно эту** задачу.
5. Что новый статус допустим для текущего workflow.
6. Что изменение сохраняется через корректный API Drupal.

Нельзя считать запрос безопасным лишь потому, что кнопка видна только менеджеру.

> [!example] Практика
> 
> Практика безопасности в Drupal — это использование штатных API: Twig с автоэкранированием, Render API, Entity и Database API с параметрами, Form API с токенами, route requirements и Entity Access API. Скрытая кнопка в интерфейсе ничего не защищает.
> 
> Отдельно следят за обновлениями core и contrib (security advisories, composer audit), ограничивают права по принципу наименьших привилегий и тестируют запрещённые сценарии. Например, смена статуса чужой задачи в DrupalJira должна проходить через CSRF, entity access и проверку workflow.

> [!warning] Типичные ошибки
> 
> - Скрывать кнопку в UI вместо серверной проверки.
> - Проверять только permission, но не доступ к конкретной сущности.
> - Использовать `|raw` или непроверенный `#markup` для пользовательского ввода.
> - Конкатенировать пользовательские значения в SQL.
> - Игнорировать CSRF для изменяющих маршрутов.
> - Выдавать лишние права «для удобства».
> - Оставлять секреты и debug-вывод в production.

> [!question] Частые вопросы на собеседовании
> 
> **Как Drupal защищает от XSS?** Twig экранирует вывод автоматически, #plain_text экранируется полностью, #markup фильтруется по белому списку тегов. Фильтр raw используют только для доверенного HTML.
> 
> **Как безопасно работать с БД?** Через Entity API или Database API с параметрами (condition(), placeholders), без подстановки пользовательских значений в строку SQL.
> 
> **Как защитить собственный маршрут от CSRF?** Добавить _csrf_token: 'TRUE' и ограничить методы. При этом проверку прав (_permission, _entity_access) CSRF-токен не заменяет.
> 
> **Чем permission отличается от entity access?** Permission — общее право на тип действия. Entity access решает, можно ли выполнить действие над конкретной сущностью, и зависит от владельца, состояния и других условий.

> [!tip] Ответ на собеседовании (EN)
> 
> I never trust input and I never rely on the UI for security. Access is checked on the server at route level and at entity level, using permissions, access checkers and the Entity Access API.
> 
> For XSS I rely on Twig auto-escaping and render arrays, and I avoid the raw filter on user data. For SQL injection I use the Entity API or the Database API with placeholders, never string concatenation.
> 
> Forms are protected by Form API tokens, and custom routes that change data get CSRF protection too. I also follow least privilege, keep core and contrib updated, and cover allowed and forbidden cases with tests.

---

## Блок 8. Multisite

> [!abstract] Кратко
> 
> Multisite — это несколько Drupal-сайтов, работающих на одной кодовой базе: общие core, модули и темы. У каждого сайта свой каталог в sites/, свой settings.php, обычно своя база данных и своя конфигурация.
> 
> Он удобен для похожих сайтов, но ошибка или обновление в общем коде затрагивает все сайты сразу.

### 8.1. Что такое Multisite

Компания может иметь `company.example.com`, `docs.example.com`, `support.example.com`. Сайты используют общие core и модули, но имеют разные настройки, контент, пользователей и БД. Multisite **не** означает общую БД или общих пользователей автоматически — это зависит от конфигурации.

### 8.2. Структура

```text
web/
  core/
  modules/
  themes/
  sites/
    sites.php                  # (необязательно) сопоставление доменов и каталогов
    default/
      settings.php
    company.example.com/
      settings.php
    docs.example.com/
      settings.php
    support.example.com/
      settings.php
```

У каждого сайта свой `settings.php`, подключение к БД и специфичные параметры. Drupal определяет сайт по запросу (домен/путь) и загружает соответствующие настройки. `docs.example.com` может использовать `docs_db`, а `support.example.com` — `support_db`, при общих файлах core. Если имя каталога не совпадает с доменом, соответствие задают в `sites/sites.php`:

```php
$sites['docs.localhost'] = 'docs.example.com';
```

### 8.3. Что общее, что отдельное

|Компонент|Совместное использование|
|---|---|
|Drupal core|обычно общий|
|Модули (файлы)|могут быть общими|
|Темы (файлы)|могут быть общими|
|Конфигурация сайта|обычно отдельная|
|Контент|обычно отдельный при разных БД|
|Пользователи|зависит от хранения и подключения|
|Файлы и загрузки|зависит от настроек (`sites/<site>/files`)|
|Кеш|зависит от инфраструктуры|

Общий код упрощает обновление, но повышает требования к тестированию: изменение общего модуля затрагивает все сайты.

### 8.4. Настройка

1. Установить общую кодовую базу.
2. Создать каталоги сайтов в `sites/`.
3. Настроить `settings.php` каждого сайта.
4. Указать подключения к БД.
5. Направить домены на общий document root веб-сервера.
6. Установить каждый сайт, проверить модули, темы, конфигурацию.

```php
$databases['default']['default'] = [
  'database' => 'docs_db',
  'username' => 'drupal_user',
  'password' => 'secret',
  'host' => 'database',
  'driver' => 'mysql',
  'prefix' => '',
];
```

Это пример структуры; реальные секреты в публичный репозиторий не кладут.

Drush работает с конкретным сайтом через `--uri`:

```bash
drush --uri=docs.example.com status
drush --uri=docs.example.com cr
drush --uri=docs.example.com cim
```

Cron, очереди и обновления БД (`updb`) выполняются **отдельно для каждого сайта**.

### 8.5. Преимущества

- общая кодовая база;
- меньше дублирования модулей и тем;
- централизованное обновление core;
- удобство для группы похожих сайтов.

### 8.6. Недостатки и риски

- обновление общего кода затрагивает все сайты;
- нужно тщательно тестировать совместимость модулей;
- возможны конфликты требований разных сайтов;
- отдельно планируются бэкапы и восстановление;
- ошибка в общем коде имеет большой радиус поражения;
- изоляция данных зависит от конфигурации и инфраструктуры.

Если сайты имеют разные жизненные циклы, зависимости и требования безопасности, отдельные установки часто проще.

### 8.7. Multisite и Configuration Management

У каждого сайта свой активный набор конфигурации и, как правило, **свой sync-каталог** (`$settings['config_sync_directory']` в `settings.php` сайта). Экспорт одного сайта автоматически не подходит остальным. Перед импортом учитывайте:

- наличие нужных модулей;
- UUID сайта и конфигурационных объектов;
- зависимости между объектами;
- различия между окружениями и сайтами.

### 8.8. Когда выбирать Multisite

Когда несколько сайтов используют общую кодовую базу и имеют похожие требования к поддержке. Если сайты должны независимо обновляться, использовать несовместимые версии модулей или иметь разные процессы разработки — отдельные установки удобнее. Альтернативы: отдельные установки с общим Composer-шаблоном, один сайт с contrib-модулем **Domain** (несколько доменов — один сайт и одна БД).

> [!example] Практика
> 
> Multisite используют, когда нужно несколько похожих сайтов на одной кодовой базе. Для каждого создают каталог в sites/ с settings.php и, при необходимости, запись в sites.php, а Drush вызывают с --uri.
> 
> Cron, updb и импорт конфигурации выполняются отдельно для каждого сайта, и у каждого свой sync-каталог. Если сайтам нужны независимые обновления или разные версии модулей, лучше отдельные установки.

> [!warning] Типичные ошибки
> 
> - Считать, что у всех сайтов общая БД/пользователи.
> - Обновлять общий код без тестирования на каждом сайте.
> - Забывать выполнять `updb`/cron для каждого сайта.
> - Использовать один sync-каталог для разных сайтов.
> - Хранить секреты `settings.php` в репозитории.
> - Выбирать multisite для сайтов с разными жизненными циклами.

> [!question] Частые вопросы на собеседовании
> 
> **Как Drupal определяет, какой сайт обслуживать?** По домену и пути запроса сопоставляет их с каталогом в sites/ (имя каталога или запись в sites/sites.php) и загружает его settings.php.
> 
> **Что в multisite общее, а что отдельное?** Обычно общие core, модули и темы. Отдельные settings.php, БД, конфигурация и файлы.
> 
> **Каковы главные риски?** Ошибка или обновление в общем коде влияет на все сайты, нужно тестировать каждый из них и выполнять updb и cron по отдельности.
> 
> **Как выполнить команду Drush для конкретного сайта?** Через --uri, например drush --uri=docs.example.com cr.

> [!tip] Ответ на собеседовании (EN)
> 
> Multisite runs several Drupal sites from one codebase. Each site has its own folder under sites/ with its own settings.php, and usually its own database and configuration.
> 
> The main benefit is easier maintenance for similar sites: one core and one set of modules. The main risk is the blast radius, because a change in shared code affects all sites, so every update has to be tested for each of them.
> 
> Database updates, cron and configuration sync are done per site, for example with drush --uri. If sites have different life cycles or security requirements, I would prefer separate installations.

---

## Блок 9. Events API

> [!abstract] Кратко
> 
> Events API позволяет одному компоненту сообщить о событии, а другим — отреагировать на него, не зная друг о друге. Событие — это объект, который отправляется через сервис event_dispatcher, а обработчики — event subscribers, то есть сервисы с тегом event_subscriber.
> 
> Подписчик объявляет события и приоритеты в getSubscribedEvents(). Это уменьшает связанность кода.

### 9.1. Что такое Events API

Механизм, позволяющий одному компоненту сообщить о событии, а другим — отреагировать. Например:

- сущность загружена;
- пользователь вошёл в систему;
- произошла операция, на которую подписан обработчик;
- система выполнила действие.

Отправитель не обязан знать, кто на событие реагирует — это уменьшает связанность.

### 9.2. Основные понятия

- **Event** — объект с информацией о событии.
- **Event dispatcher** — отправляет событие обработчикам.
- **Event subscriber** — класс, подписанный на одно или несколько событий.
- **Event listener** — обработчик конкретного события.
- **Event name** — идентификатор события (строка, обычно константа).

Обработчики в Drupal чаще всего реализуются как сервисы.

Примеры событий ядра: `KernelEvents::REQUEST`, `KernelEvents::RESPONSE`, `KernelEvents::EXCEPTION` (Symfony HttpKernel), `ConfigEvents::SAVE`, `RoutingEvents::ALTER`. Список событий и подписчиков: `drush debug:event`.

### 9.3. Создание Event Subscriber

`web/modules/custom/drupaljira_timelog/src/EventSubscriber/ExampleSubscriber.php`:

```php
<?php

namespace Drupal\drupaljira_timelog\EventSubscriber;

use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\HttpKernel\Event\RequestEvent;
use Symfony\Component\HttpKernel\KernelEvents;

final class ExampleSubscriber implements EventSubscriberInterface {

  public static function getSubscribedEvents(): array {
    return [
      // [метод, приоритет]: больший приоритет — раньше.
      KernelEvents::REQUEST => ['onRequest', 30],
    ];
  }

  public function onRequest(RequestEvent $event): void {
    // Выполнить нужную логику.
  }

}
```

`getSubscribedEvents()` сообщает диспетчеру, на какие события подписан класс и какие методы вызвать; значением может быть имя метода, пара `[метод, приоритет]` или список таких пар. Сигнатура обработчика соответствует объекту события, который отправляет код-источник. Событие можно остановить: `$event->stopPropagation()`.

### 9.4. Регистрация подписчика как сервиса

`drupaljira_timelog.services.yml`:

```yaml
services:
  drupaljira_timelog.example_subscriber:
    class: Drupal\drupaljira_timelog\EventSubscriber\ExampleSubscriber
    arguments: ['@current_user']        # если нужны зависимости
    tags:
      - { name: event_subscriber }
```

Тег `event_subscriber` позволяет Drupal найти подписчика при сборке контейнера. После добавления сервиса нужен `drush cr`.

### 9.5. Собственное событие

Чтобы подписчик сработал, событие должен кто-то отправить — сервис `event_dispatcher`.

Класс с именами событий и класс самого события:

```php
namespace Drupal\drupaljira_timelog\Event;

final class TaskEvents {
  public const STATUS_CHANGED = 'drupaljira_timelog.task_status_changed';
}
```

```php
namespace Drupal\drupaljira_timelog\Event;

use Drupal\node\NodeInterface;
use Symfony\Contracts\EventDispatcher\Event;

final class TaskStatusChangedEvent extends Event {

  public function __construct(
    private readonly NodeInterface $task,
    private readonly string $oldStatus,
    private readonly string $newStatus,
  ) {}

  public function getTask(): NodeInterface {
    return $this->task;
  }

  public function getOldStatus(): string {
    return $this->oldStatus;
  }

  public function getNewStatus(): string {
    return $this->newStatus;
  }

}
```

Отправка (через DI):

```php
use Symfony\Contracts\EventDispatcher\EventDispatcherInterface;

final class TaskStatusUpdater {

  public function __construct(
    private readonly EventDispatcherInterface $eventDispatcher,
  ) {}

  public function changeStatus($task, string $new): void {
    $old = $task->get('field_status')->value;
    $task->set('field_status', $new)->save();

    // Событие — после реального сохранения.
    $this->eventDispatcher->dispatch(
      new TaskStatusChangedEvent($task, $old, $new),
      TaskEvents::STATUS_CHANGED,
    );
  }

}
```

Простейший вариант без DI — `\Drupal::service('event_dispatcher')->dispatch($event, 'example.event')`, но в модуле лучше DI. Важен момент отправки: событие об изменении статуса не отправляют до того, как изменение действительно произошло.

### 9.6. Events API и Hooks

|Events API|Hooks|
|---|---|
|диспетчеризация событий|система хуков Drupal|
|сервисы и подписчики|функции `hook_*` (или OOP-реализации с атрибутом `#[Hook]` в Drupal 11.1+)|
|подходит для событийной архитектуры|широко используется в API Drupal|
|событие — объект|данные определяются сигнатурой хука|

```php
function example_entity_presave(\Drupal\Core\Entity\EntityInterface $entity): void {
  // Проверка или подготовка данных перед сохранением.
}
```

Не заменяйте все хуки событиями — используйте механизм, который Drupal предоставляет в нужной точке расширения.

### 9.7. Events API и Queue API

Events API — реакция в ходе выполнения приложения; Queue API — отложенная обработка. Сценарий:

1. Сохранить новый статус.
2. Отправить событие об изменении.
3. Подписчик ставит задание уведомления в очередь.
4. Worker отправляет уведомление позже.

Основной запрос не выполняет тяжёлую работу синхронно.

### 9.8. Когда применять

Когда несколько компонентов должны реагировать на одно действие, а основной код не должен зависеть от них напрямую: уведомления, аудит, интеграция с внешней системой. Не создавайте событие для каждого простого вызова — оно полезно, когда создаёт понятную точку расширения и снижает связанность.

> [!example] Практика
> 
> Events API применяют, когда нужна точка расширения без жёсткой связи: компонент отправляет событие через event_dispatcher, а подписчики реагируют на него. Подписчик — сервис с тегом event_subscriber, который в getSubscribedEvents() объявляет события и приоритеты.
> 
> Часто используют события ядра (KernelEvents::REQUEST, RESPONSE, EXCEPTION), для собственной логики создают класс события с данными. Например, в DrupalJira событие смены статуса задачи может запускать аудит и постановку уведомления в очередь.

> [!warning] Типичные ошибки
> 
> - Забыть тег `event_subscriber` или сбросить кеш.
> - Не соответствовать сигнатуре события.
> - Отправлять событие до сохранения изменений.
> - Делать тяжёлую работу прямо в подписчике (лучше Queue).
> - Заменять событиями все хуки.
> - Создавать событие без реальной точки расширения.
> - Путать приоритеты: больший приоритет выполняется раньше.

> [!question] Частые вопросы на собеседовании
> 
> **Чем события отличаются от хуков?** События — объекты, которые обрабатывают сервисы-подписчики. Хуки — именованные точки расширения Drupal, реализуемые функциями или OOP-методами. Используют то, что предоставлено в нужной точке.
> 
> **Как создать подписчика?** Класс с EventSubscriberInterface и getSubscribedEvents(), зарегистрированный как сервис с тегом event_subscriber, затем сбросить кеш.
> 
> **Что означает приоритет подписчика?** Чем он больше, тем раньше вызывается обработчик. Событие можно остановить через stopPropagation().
> 
> **Как отправить собственное событие?** Создать класс события и вызвать dispatch() у сервиса event_dispatcher (через DI), после того как изменение реально произошло.

> [!tip] Ответ на собеседовании (EN)
> 
> The Events API lets one part of the system announce that something happened, and other parts react without being called directly. I dispatch an event object through the event_dispatcher service and handle it in subscribers, which are services tagged event_subscriber.
> 
> A subscriber returns its events and priorities from getSubscribedEvents(). I create custom events when I want a clear extension point, for example when a task status changes, and I dispatch the event only after the change has really been saved.
> 
> Events are different from hooks, and I use whichever Drupal provides at that point. Heavy work in a subscriber should be moved to a queue.

---

## Блок 10. Entity API

> [!abstract] Кратко
> 
> Entity — это единица данных Drupal. Entity type (node, user, taxonomy_term) задаёт общее поведение, а bundle (task, project) — подтип со своим набором полей. Сущности бывают контентными и конфигурационными: первые хранят рабочие данные, вторые — настройки сайта.
> 
> Entity API — стандартный способ создавать, загружать, искать и удалять данные и проверять доступ, без прямого SQL.

### 10.1. Что такое Entity API

Единый способ создавать, загружать, изменять, сохранять и удалять сущности. Примеры:

- Node — материал (проект, задача);
- User — пользователь;
- Taxonomy term — термин;
- Media — медиаобъект;
- File — файл;
- Custom content entity — собственная сущность (например, `time_log`).

В DrupalJira проект и задача — nodes разных bundles; оба используют систему сущностей Node.

### 10.2. Entity Type и Bundle

**Entity Type** — общий вид сущности и доступные операции: `node`, `user`, `taxonomy_term`, `media`.

**Bundle** — подтип, определяющий структуру: у `node` это `article`, `page`, `project`, `task`. У проекта может быть поле типа проекта, у задачи — статус, исполнитель, оценка.

### 10.3. Content Entity и Configuration Entity

**Content Entity** — содержательные данные, создаваемые в ходе работы: Node, User, Media, Taxonomy term. Имеют поля, права доступа, часто revisions.

**Configuration Entity** — конфигурация сайта: типы материалов, определения полей, Views, текстовые форматы, роли. Экспортируются в YAML (Блок 3).

Content Entity хранит рабочие данные, Configuration Entity описывает, как сайт должен работать.

### 10.4. Загрузка сущностей

```php
$node = \Drupal::entityTypeManager()
  ->getStorage('node')
  ->load(123);

if ($node !== NULL) {
  $title = $node->label();
}
```

- `entityTypeManager()` — менеджер типов сущностей;
- `getStorage('node')` — хранилище материалов;
- `load(123)` — загрузка по ID (`NULL`, если не найдена);
- `label()` — отображаемое название.

Другие методы storage: `loadMultiple($ids)`, `loadByProperties([...])`. В сервисах и контроллерах внедряйте `entity_type.manager` через конструктор.

### 10.5. Создание и сохранение

```php
$node = \Drupal::entityTypeManager()
  ->getStorage('node')
  ->create([
    'type' => 'task',
    'title' => 'Implement task workflow',
    'status' => 1,
  ]);

$node->save();
```

Поле `status` здесь — статус **публикации**, а не пользовательское поле workflow. В DrupalJira `field_status` и состояние moderation workflow — отдельные механизмы, их нельзя путать с публикацией. Перед сохранением проверьте обязательные поля, права и бизнес-правила; для валидации по ограничениям — `$node->validate()`. Удаление: `$node->delete()`.

### 10.6. Работа с полями

```php
$status = $node->get('field_status')->value;

$node->set('field_status', 'in_progress');
$node->save();
```

Значение должно соответствовать настройке поля и допустимым значениям списка. Для Entity Reference:

```php
$project = $node->get('field_project')->entity;
```

Если ссылка не заполнена или цель недоступна, результат — `NULL`. Поля с несколькими значениями — см. Блок 14.

### 10.7. Entity Query

```php
$ids = \Drupal::entityQuery('node')
  ->condition('type', 'task')
  ->condition('field_status', 'in_progress')
  ->accessCheck(TRUE)
  ->execute();
```

Результат — массив ID. `accessCheck(TRUE)` включает проверку доступа. В Drupal 10 и 11 режим нужно указывать явно; `accessCheck(FALSE)` допустим только когда обход обоснован и безопасность обеспечена иначе. Query возвращает ID, а не объекты — загружайте через storage (`loadMultiple`).

### 10.8. Entity Access API

```php
if ($node->access('view')) {
  // Пользователь может просматривать материал.
}

$access = $node->access('update', NULL, TRUE);
if ($access->isAllowed()) {
  // Можно редактировать.
}
```

Второй вариант возвращает `AccessResultInterface` с cacheability metadata. Особенно важно для собственных контроллеров, endpoints и операций изменения. Доступ определяет **access control handler** сущности; доступ к отдельным полям — `hook_entity_field_access`/field access.

### 10.9. Revisions

Revisions — отдельные версии объекта: история изменений и возврат к прежним версиям. Например, при изменении задачи создаётся новая revision с новым статусом.

```php
$node->setNewRevision(TRUE);
$node->setRevisionLogMessage('Status changed to in_progress');
$node->save();
```

Не все сущности поддерживают revisions одинаково: учитывайте entity type, конфигурацию и модули (Content Moderation).

### 10.10. Entity API и Render API

Entity API — данные, Render API — отображение. Контроллер может загрузить материал и отдать его на рендеринг:

```php
return [
  '#type' => 'markup',
  '#markup' => $node->label(),
];
```

Для реального вывода используйте view builders и режимы отображения: `$viewBuilder->view($node, 'teaser')`. Не вставляйте непроверенные пользовательские данные в HTML.

### 10.11. Собственная Content Entity

Когда стандартных nodes недостаточно (структура данных своя, много записей, нужна отдельная таблица), создают **custom content entity**. Именно так устроена `time_log` в DrupalJira. Составные части:

- **класс сущности** (`ContentEntityBase`) с описанием типа (аннотация `@ContentEntityType` или атрибут в новых версиях): `id`, `label`, `base_table`, `entity_keys`, `handlers`, `links`, `admin_permission`;
- **`baseFieldDefinitions()`** — базовые поля, объявленные в коде (например, `task` как entity_reference на node, `uid`, `hours` как decimal, `logged_date`, `comment`);
- **handlers**: `storage`, `access`, `list_builder`, `form`, `views_data`;
- **routing и permissions** (через `route_provider` и `permission_provider` либо вручную);
- **схема БД** создаётся при установке модуля; изменения структуры существующей сущности требуют update hook (`EntityDefinitionUpdateManager`).

Различие:

| Критерий              | Base fields                                      | Configurable fields (Field UI)                      |
| --------------------- | ------------------------------------------------ | --------------------------------------------------- |
| Где объявлены         | В коде сущности                                  | В конфигурации (`field.storage.*`, `field.field.*`) |
| Доступность в bundles | Обычно доступны всем bundles данного entity type | Настраиваются отдельно для конкретных bundles       |
| Как изменяются        | Через код и update hooks                         | Через UI и Configuration Management                 |

### 10.12. Когда применять Entity API

- создавать и изменять материалы;
- загружать пользователей и медиа;
- получать значения полей;
- работать со ссылками между сущностями;
- искать материалы по условиям;
- проверять доступ к объектам;
- управлять revisions.

Главное преимущество — работа с данными через стандартную абстракцию Drupal вместо прямого взаимодействия с таблицами БД.

> [!example] Практика
> 
> Практический минимум по Entity API: загрузка и сохранение через storage, поиск через Entity Query с явным accessCheck, проверка доступа через access(), работа с полями и revisions. SQL к таблицам сущностей не пишут.
> 
> Когда node не подходит, создают собственную content entity с базовыми полями, handlers и правами. Например, в DrupalJira project и task — bundles node, а записи времени — отдельная сущность time_log.

> [!warning] Типичные ошибки
> 
> - Путать entity type и bundle.
> - Путать `status` (публикация), `field_status` и moderation state.
> - Забывать `accessCheck()` в Entity Query.
> - Не проверять `NULL` после `load()` и у ссылок.
> - Загружать сущности в цикле по одной (N+1).
> - Писать SQL к таблицам вместо Entity API.
> - Менять схему custom entity без update hook.

> [!question] Частые вопросы на собеседовании
> 
> **Чем entity type отличается от bundle?** Entity type задаёт общий вид сущности (node, user). Bundle — подтип со своим набором полей (article, page).
> 
> **Чем content entity отличается от configuration entity?** Content entity хранит рабочие данные сайта, configuration entity описывает настройки и экспортируется в YAML.
> 
> **Зачем нужен accessCheck в Entity Query?** Он определяет, учитывать ли права доступа при выборке. В Drupal 10 и 11 его нужно указывать явно, а FALSE допустим только осознанно.
> 
> **Чем base fields отличаются от configurable fields?** Base fields объявляются в коде сущности и есть у всех bundles, configurable fields создаются через UI или конфигурацию для конкретных bundles.

> [!tip] Ответ на собеседовании (EN)
> 
> An entity type defines a kind of data, such as node or user, and a bundle defines a subtype with its own fields, such as task or project. Content entities store site data, and configuration entities describe how the site works.
> 
> I work with entities through the storage API and entity queries, and I always set accessCheck explicitly. I use loadMultiple to avoid N+1 and I check for NULL after load or when following an entity reference.
> 
> When nodes do not fit, I create a custom content entity, like our time_log. Its base fields are defined in code, and schema changes for existing sites need update hooks.

---

## Блок 11. Plugin API

> [!abstract] Кратко
> 
> Plugin — это взаимозаменяемая реализация известного контракта: блок, виджет поля, queue worker, process-плагин миграции. Plugin Manager находит плагины (по аннотациям или атрибутам), хранит их определения и создаёт экземпляры.
> 
> Плагин нужен для интеграции с подсистемой Drupal, а бизнес-логику лучше выносить в сервис.

### 11.1. Что такое Plugin API

Плагин реализует общий контракт, а Drupal обнаруживает его, создаёт экземпляр и использует в нужном контексте. Плагины применяются для:

- блоков;
- способов аутентификации;
- источников миграции;
- виджетов и форматтеров полей;
- обработчиков очередей;
- действий (Actions), условий, фильтров текста, эффектов изображений и др.

Система работает с общим интерфейсом, а конкретные реализации различаются.

### 11.2. Plugin Manager

Plugin Manager обнаруживает плагины одного типа и управляет ими. Менеджер блоков знает доступные block plugins и предоставляет их для размещения на странице. Плагины одного типа имеют общий интерфейс, но различаются поведением и конфигурацией.

Разработчик редко создаёт экземпляры вручную: этим занимается менеджер (сервис вида `plugin.manager.block`):

```php
$plugin = \Drupal::service('plugin.manager.block')->createInstance('task_summary', []);
```

### 11.3. Plugin ID и Plugin Definition

- **Plugin ID** — уникальный идентификатор плагина в рамках типа.
- **Plugin Definition** — описание: ID, название, категория, класс и другие метаданные.

Например, блок с ID `task_summary` и названием «Task Summary». Определение помогает Drupal обнаружить плагин и показать его в UI.

### 11.4. Discovery: как находятся плагины

- **Annotation discovery** — аннотации в docblock (`@Block(...)`), каталог `src/Plugin/<Type>/`.
- **Attribute discovery** — PHP-атрибуты (`#[Block(...)]`), поддерживаются в современных версиях Drupal (10.2/10.3+, предпочтительны в Drupal 11).
- **YAML discovery** — плагины в `*.links.menu.yml`, `*.migrate_drupal.yml` и др.
- **Hook discovery** — через `hook_*_info`.

После добавления плагина нужен `drush cr` (кеш определений).

### 11.5. Собственный Block Plugin

`web/modules/custom/drupaljira_timelog/src/Plugin/Block/TaskSummaryBlock.php`:

```php
<?php

namespace Drupal\drupaljira_timelog\Plugin\Block;

use Drupal\Core\Block\BlockBase;

/**
 * Provides a task summary block.
 *
 * @Block(
 *   id = "task_summary",
 *   admin_label = @Translation("Task Summary"),
 *   category = @Translation("DrupalJira")
 * )
 */
final class TaskSummaryBlock extends BlockBase {

  public function build(): array {
    return [
      '#markup' => $this->t('Task summary'),
    ];
  }

}
```

Тот же плагин через атрибут (современные версии):

```php
use Drupal\Core\Block\Attribute\Block;
use Drupal\Core\StringTranslation\TranslatableMarkup;

#[Block(
  id: 'task_summary',
  admin_label: new TranslatableMarkup('Task Summary'),
  category: new TranslatableMarkup('DrupalJira'),
)]
final class TaskSummaryBlock extends BlockBase { /* ... */ }
```

- `id` — идентификатор плагина;
- `admin_label` — название в административном интерфейсе;
- `category` — категория блока;
- `build()` — возвращает render array.

### 11.6. Почему Render Array

`build()` возвращает render array, а не HTML-строку. Это позволяет Drupal корректно обработать:

- рендеринг и вложенные элементы;
- библиотеки CSS/JS;
- cache contexts, tags и max-age;
- доступ к отображаемым данным.

```php
public function build(): array {
  return [
    '#theme' => 'item_list',
    '#items' => [
      $this->t('Backlog'),
      $this->t('In progress'),
      $this->t('Done'),
    ],
    '#cache' => ['tags' => ['node_list:task']],
  ];
}
```

Реальную статистику берите из сервиса или Entity Query, а не из статических значений.

### 11.7. Dependency Injection в плагинах

Плагинам нужны сервисы (Entity Type Manager, Current User). Нельзя просто добавить параметр в конструктор: система должна знать, как создать экземпляр. Реализуйте `ContainerFactoryPluginInterface` и `create()`:

```php
use Drupal\Core\Block\BlockBase;
use Drupal\Core\Entity\EntityTypeManagerInterface;
use Drupal\Core\Plugin\ContainerFactoryPluginInterface;
use Symfony\Component\DependencyInjection\ContainerInterface;

final class TaskSummaryBlock extends BlockBase implements ContainerFactoryPluginInterface {

  public function __construct(
    array $configuration,
    $plugin_id,
    $plugin_definition,
    private readonly EntityTypeManagerInterface $entityTypeManager,
  ) {
    parent::__construct($configuration, $plugin_id, $plugin_definition);
  }

  public static function create(
    ContainerInterface $container,
    array $configuration,
    $plugin_id,
    $plugin_definition,
  ): static {
    return new static(
      $configuration,
      $plugin_id,
      $plugin_definition,
      $container->get('entity_type.manager'),
    );
  }

  public function build(): array {
    return ['#markup' => $this->t('Task summary')];
  }

}
```

Drupal передаёт зависимости через контейнер. Проверяйте соответствие сигнатур интерфейсам вашей версии Drupal (в `BlockBase` базовые аргументы конструктора могут отличаться по версиям).

### 11.8. Plugin API и Hooks

Hooks позволяют вмешаться в определённые точки расширения. Plugins создают отдельные реализации известного типа компонента, которыми управляет Plugin Manager.

- `hook_entity_presave()` — реакция на сохранение сущности;
- Block Plugin — отдельный блок;
- Migration Plugin — источник или обработчик миграции.

Не всякое расширение должно быть плагином.

### 11.9. Плагины с конфигурацией

Блок может иметь настройку (например, сколько задач показывать): переопределяются `defaultConfiguration()`, `blockForm()`, `blockSubmit()`. Различайте:

- **определение** плагина — описывает доступный тип;
- **конфигурация экземпляра** — параметры конкретного использования.

### 11.10. Свой тип плагинов (кратко)

Нужен, когда вы хотите дать другим модулям расширять ваш функционал. Составные части:

1. интерфейс и базовый класс плагина;
2. класс аннотации/атрибута (определение);
3. Plugin Manager (наследник `DefaultPluginManager`), зарегистрированный как сервис `plugin.manager.<type>`;
4. каталог discovery (`Plugin/<Type>`) и cache tag для определений.

Для плагинов, порождающих набор вариантов из данных, используют **derivatives** (например, один блок на каждый проект).

### 11.11. Когда применять

Когда нужен компонент, который Drupal сможет обнаруживать, конфигурировать и использовать как одну из реализаций. Блок статистики задач логично оформить как Block Plugin, а расчёт вынести в сервис: плагин отвечает за интеграцию с системой блоков, сервис — за бизнес-логику.

> [!example] Практика
> 
> Плагины применяют там, где Drupal ожидает взаимозаменяемые реализации: блоки, виджеты и форматтеры, queue workers, process-плагины миграций, действия. Плагин лежит в src/Plugin/<Type>, описывается аннотацией или атрибутом и обнаруживается после сброса кеша.

Хорошая практика — тонкий плагин, который интегрируется с подсистемой, и сервис с бизнес-логикой. Например, блок статистики в DrupalJira только строит render array, а считает данные сервис.

> [!warning] Типичные ошибки
> 
> - Добавлять аргументы в конструктор плагина без `ContainerFactoryPluginInterface`.
> - Не сбросить кеш после добавления плагина.
> - Класс не в правильном namespace/каталоге — плагин не находится.
> - Класть бизнес-логику в плагин вместо сервиса.
> - Возвращать HTML-строку вместо render array.
> - Путать определение плагина и конфигурацию экземпляра.
> - Использовать плагин там, где достаточно hook или сервиса.

> [!question] Частые вопросы на собеседовании
> 
> **Что такое Plugin Manager?** Сервис, который находит плагины своего типа, хранит их определения и создаёт экземпляры.
> 
> **Чем annotation discovery отличается от attribute discovery?** Определение в docblock-аннотации или в PHP-атрибуте. В новых версиях Drupal (10.2/10.3+ и 11) предпочтительны атрибуты.
> 
> **Как получить сервисы в плагине?** Через ContainerFactoryPluginInterface и create(), а не добавлением аргументов в обычный конструктор.
> 
> **Когда плагин, а когда hook?** Плагин — когда нужна отдельная, конфигурируемая реализация известного типа компонента. Hook — когда нужно вмешаться в существующую точку расширения.

> [!tip] Ответ на собеседовании (EN)
> 
> A plugin is a swappable implementation of a contract, found by a plugin manager through annotations or attributes in a known directory. Blocks, field widgets, queue workers and migration process plugins are all plugins.
> 
> A plugin has an ID, a definition with metadata and, optionally, instance configuration. Because plugins are created by their manager, I inject services through ContainerFactoryPluginInterface and the create() method.
> 
> I keep the plugin thin: it integrates with the subsystem, for example it returns a render array with cache metadata, and the real logic lives in a service. After adding a plugin I clear caches so its definition is discovered.

---

## Блок 12. Batch API, Cron API и Queue API

> [!abstract] Кратко
> 
> Долгие операции нельзя выполнять в одном HTTP-запросе. Batch API делит работу на шаги с индикатором прогресса и подходит для действий, запускаемых пользователем. Cron API выполняет периодические задачи по расписанию. Queue API ставит отдельные задания в очередь, которые обрабатывают воркеры.
> 
> Эти механизмы часто комбинируют: например, batch импортирует данные, в очередь ставятся уведомления, а cron запускает воркеры.

### 12.1. Зачем нужны фоновые операции

Не всё должно выполняться в одном запросе:

- импорт тысяч записей;
- обработка большого набора материалов;
- рассылка уведомлений;
- удаление устаревших данных;
- синхронизация с внешней системой.

Иначе возможны превышение лимитов PHP, тайм-ауты и перегрузка сервера.

### 12.2. Batch API

Разбивает операцию на небольшие шаги, выполняемые последовательно, с отображением прогресса. Применяется для массовых обновлений, обработки большого числа сущностей, административных операций, импорта с прогресс-баром.

```php
use Drupal\Core\Batch\BatchBuilder;

$builder = (new BatchBuilder())
  ->setTitle(t('Updating tasks'))
  ->setFinishCallback(['\Drupal\drupaljira_timelog\Batch\TaskBatch', 'finished'])
  ->addOperation(['\Drupal\drupaljira_timelog\Batch\TaskBatch', 'process'], [$ids]);

batch_set($builder->toArray());
```

Старый вариант — массив `$batch = ['title' => ..., 'operations' => [...], 'finished' => ...]; batch_set($batch);`.

Операция обрабатывает ограниченную порцию и обновляет контекст:

```php
public static function process(array $ids, array &$context): void {
  if (!isset($context['sandbox']['progress'])) {
    $context['sandbox']['progress'] = 0;
    $context['sandbox']['max'] = count($ids);
  }

  $chunk = array_slice($ids, $context['sandbox']['progress'], 50);
  // ... обработать $chunk ...
  $context['sandbox']['progress'] += count($chunk);

  $context['finished'] = $context['sandbox']['max'] > 0
    ? $context['sandbox']['progress'] / $context['sandbox']['max']
    : 1;
}
```

- `$context['sandbox']` — состояние между итерациями;
- `$context['finished']` — доля выполнения от 0 до 1 (операция повторяется, пока не достигнет 1);
- `finished` callback — итоговое сообщение, ошибки, финальные действия.

В **update hook** тот же принцип: `function hook_update_N(&$sandbox)` и `$sandbox['#finished']` (0…1) для пошаговой обработки больших объёмов. Batch API не делает тяжёлую операцию безопасной автоматически: обработчики сами отвечают за данные, права и ошибки.

### 12.3. Cron API

Cron — механизм периодических задач: очистка временных данных, обработка очередей, обслуживание модулей, удаление просроченного, синхронизация.

Drupal запускает cron через UI (Automated Cron), Drush или системный планировщик.

```php
/**
 * Implements hook_cron().
 */
function drupaljira_timelog_cron(): void {
  // Выполнить периодические действия.
}
```

Не размещайте в `hook_cron()` тяжёлую обработку тысяч записей без ограничений: используйте его для запуска небольших задач и постановки заданий в очередь.

Cron в Drupal — логика приложения; системный cron (или другой планировщик) — то, что запускает её по расписанию:

```bash
drush cron     # alias: drush core:cron
```

Запускайте из корректного каталога проекта и окружения. Если планировщик не настроен, периодические задачи выполняются нерегулярно. Время последнего запуска собственных задач удобно хранить в `State`.

### 12.4. Queue API

Складывает задачи в очередь и обрабатывает отдельно. После создания задачи нужно уведомить десять пользователей — вместо отправки в запросе ставим задания в очередь.

- **Queue** — очередь;
- **Queue item** — элемент;
- **Queue worker** — обработчик;
- **Claim** — получение элемента для обработки;
- **Delete** — удаление успешно обработанного элемента.

Добавление:

```php
$queue = \Drupal::queue('drupaljira_notifications');

$queue->createItem([
  'task_id' => 123,
  'recipient_uid' => 45,
]);
```

Пример только кладёт данные в очередь, не отправляя уведомление. В рабочем коде получайте очередь через сервис `queue` (`QueueFactory`) и DI.

Queue Worker — плагин:

```php
namespace Drupal\drupaljira_timelog\Plugin\QueueWorker;

use Drupal\Core\Queue\QueueWorkerBase;

/**
 * Processes task notifications.
 *
 * @QueueWorker(
 *   id = "drupaljira_notifications",
 *   title = @Translation("Task notifications"),
 *   cron = {"time" = 30}
 * )
 */
final class TaskNotificationWorker extends QueueWorkerBase {

  public function processItem($data): void {
    // Проверить данные, найти задачу и получателя, отправить уведомление.
  }

}
```

`cron = {"time" = 30}` — воркер участвует в cron-обработке с бюджетом 30 секунд. Нужно учитывать, что задача или пользователь могли быть удалены. Из `processItem()` можно бросить `RequeueException` (повторить позже) или `SuspendQueueException` (остановить обработку очереди, например, при недоступности внешней системы). Ручной запуск: `drush queue:run drupaljira_notifications`; список — `drush queue:list`.

### 12.5. Надёжность Queue API

Если внешняя почтовая система временно недоступна, задание нельзя бездумно удалять. Помните:

- повторная обработка возможна;
- воркер должен быть устойчив к повторному запуску;
- данные задания валидируются;
- временные сбои обрабатываются предсказуемо;
- ошибки логируются;
- чувствительные данные в очередь без необходимости не кладут.

Если повторная обработка приводит к дублям (повторному списанию, множеству дубликатов писем), архитектуру нужно менять. Для критичных операций нужна **идемпотентность** — безопасное повторное выполнение.

### 12.6. Сравнение

|API|Основное назначение|Типичный сценарий|
|---|---|---|
|Batch API|длительная операция шагами, с прогрессом|админ обновляет тысячи задач|
|Cron API|периодический запуск обслуживания|регулярная очистка старых данных|
|Queue API|отложенная обработка отдельных заданий|уведомления после изменения задачи|

Комбинация:

1. Batch импортирует набор задач.
2. Для каждой создаётся задание уведомления в Queue.
3. Cron периодически запускает обработчики очереди.

### 12.7. Как выбрать

- **Batch** — пользователь сам запускает длительную операцию и нужен прогресс.
- **Cron** — регулярные задачи по расписанию.
- **Queue** — работу нужно отложить, разбить на задания или обрабатывать независимо от запроса.

Не используйте очередь для каждого простого действия: короткую операцию лучше выполнить сразу.

> [!example] Практика
> 
> Выбор механизма зависит от сценария: Batch API — пользователь запускает долгую операцию и ему нужен прогресс; Cron — регулярное обслуживание; Queue — отложенные независимые задания. В update hook большие объёмы обрабатывают порциями через $sandbox.
> 
> Воркеры очередей пишут идемпотентными, ошибки обрабатывают через RequeueException или SuspendQueueException, а системный cron настраивают отдельно. В DrupalJira update hook 10001 обрабатывает существующие задачи порциями, а уведомления логично отправлять через очередь.

> [!warning] Типичные ошибки
> 
> - Тяжёлая обработка прямо в `hook_cron()`.
> - Batch-операция без ограничения порции и без `$context['finished']`.
> - Воркер, не рассчитанный на повторный запуск (нет идемпотентности).
> - Удалять элемент очереди при временной ошибке.
> - Не настроить системный cron.
> - Класть в очередь чувствительные данные и огромные объекты.
> - Использовать очередь для коротких мгновенных операций.

> [!question] Частые вопросы на собеседовании
> 
> **Когда Batch, когда Queue и когда Cron?** Batch — запуск пользователем с прогрессом, Queue — отложенные отдельные задания, Cron — расписание. Их часто комбинируют.
> 
> **Что такое идемпотентность воркера и зачем она нужна?** Повторная обработка того же элемента не должна давать дублей. Это важно, потому что элементы очереди могут выполняться повторно.
> 
> **Как работать с большим объёмом данных в hook_update_N()?** Использовать &$sandbox, обрабатывать порцию за вызов и выставлять $sandbox['#finished'] от 0 до 1.
> 
> **Как вручную запустить очередь?** Командой drush queue:run <имя очереди>, список очередей — drush queue:list.

> [!tip] Ответ на собеседовании (EN)
> 
> Batch API splits a long operation into chunks with a progress bar, and it is the right tool when a user starts a heavy action. Cron runs scheduled maintenance, and hook_cron should only start small tasks or put jobs into a queue.
> 
> The Queue API stores individual jobs that a queue worker processes later, usually on cron or with drush queue:run. Workers must be idempotent, because an item can be retried, and they should handle temporary failures without deleting the item.
> 
> I do not put a queue everywhere: a short operation that must finish immediately is better done directly. For long operations I never rely on a single web request.

---

## Блок 13. Migration API

> [!abstract] Кратко
> 
> Migration API переносит данные из источника в Drupal по схеме Source, Process, Destination. Source читает строки, process-плагины преобразуют значения, destination создаёт сущности. Соответствие старых и новых записей хранится в ID map, поэтому миграции можно перезапускать, обновлять и откатывать.
> 
> Связи между миграциями (проекты, затем задачи) задаются зависимостями и плагином migration_lookup.

### 13.1. Что такое Migration API

Механизм переноса данных из одного источника в другой:

- из старой версии Drupal в новую;
- из CSV;
- из XML или JSON;
- из другой CMS;
- из внешней БД;
- из устаревшей структуры полей в новую.

Нужно преобразовывать данные, сопоставлять поля, создавать связи, обрабатывать ошибки. Обычно используется Migrate (core) и расширения **Migrate Plus** и **Migrate Tools**.

### 13.2. Этапы миграции

1. **Source** — чтение исходных данных.
2. **Process** — преобразование и подготовка.
3. **Destination** — создание или обновление данных в Drupal.

Старая система хранит статус `working`, новая Drupal — `in_progress`: Source получает `working`, Process преобразует в `in_progress`, Destination сохраняет в поле.

### 13.3. Source, Process, Destination

**Source** — откуда брать данные: SQL-таблица, CSV, XML, JSON API, данные Drupal. Источник отдаёт строки (rows) с уникальными `ids`.

**Process** — как преобразовать: переименовать поле, изменить формат даты, преобразовать статус, найти сущность, создать ссылку, объединить значения. Используются process plugins.

**Destination** — куда записывать: материалы, пользователи, термины, поддерживаемые конфигурационные сущности. Для материалов — `entity:node`.

### 13.4. Пример YAML-миграции

Таблица `legacy_tasks`: `id`, `title`, `old_status`. Нужны материалы типа `task`.

```yaml
id: legacy_tasks
label: 'Import legacy tasks'
migration_group: legacy

source:
  plugin: embedded_data
  data_rows:
    - id: 1
      title: 'Fix login'
      old_status: working
    - id: 2
      title: 'Review design'
      old_status: todo
  ids:
    id:
      type: integer

process:
  type:
    plugin: default_value
    default_value: task

  title: title

  field_status:
    plugin: static_map
    source: old_status
    map:
      todo: backlog
      working: in_progress
      qa: review
      completed: done

destination:
  plugin: 'entity:node'
```

Пример использует встроенные данные для демонстрации; в реальном проекте источник — таблица, CSV и т.д. Файлы миграций лежат в `migrations/` модуля (или как config entity при Migrate Plus).

- `source` — исходные данные;
- `process` — сопоставление и преобразование;
- `destination` — результат: материалы Drupal;
- `static_map` — преобразование старых статусов в новые.

Если значения нет в карте `static_map`, поведение определяется настройками плагина (`default_value`, `bypass`): нельзя полагаться, что неизвестные значения станут корректными.

### 13.5. Process plugins

|Плагин|Назначение|
|---|---|
|`get`|получить исходное значение|
|`default_value`|значение по умолчанию|
|`static_map`|преобразование по карте|
|`concat`|объединение значений|
|`skip_on_empty`|пропуск при пустом значении|
|`migration_lookup`|результат другой миграции|
|`entity_lookup`|поиск сущности по критериям (Migrate Plus)|
|`callback`, `format_date`, `explode`, `sub_process`|вспомогательные|

Доступность плагинов зависит от модулей и версии; некоторые даёт Migrate Plus. Собственные преобразования — custom process plugin (`transform()`).

### 13.6. Migration lookup и зависимости

Сначала мигрируются проекты, затем задачи; задача ссылается на проект через Entity Reference:

```yaml
migration_dependencies:
  required:
    - legacy_projects

process:
  field_project:
    plugin: migration_lookup
    migration: legacy_projects
    source: legacy_project_id
```

- `legacy_project_id` — ID проекта в старой системе;
- `legacy_projects` — ID миграции проектов;
- `migration_lookup` — ищет сущность, созданную миграцией проектов.

Миграция проектов должна быть выполнена, а её map-данные доступны. Если проект не найден, решите: пропускать задачу, импортировать без связи или завершать ошибкой — зависит от требований.

### 13.7. ID map

Drupal хранит соответствие исходных записей и созданных сущностей (**migration map**): исходная задача `42` → материал `137`. Это нужно для:

- повторного запуска;
- обновления импортированных данных;
- поиска связей между миграциями;
- учёта обработанных записей.

Map не заменяет резервное копирование. Перед опасными операциями понимайте последствия для сущностей и map.

### 13.8. Запуск и управление

Drush и Migrate Tools:

```bash
drush migrate:status
drush migrate:import legacy_tasks
drush migrate:import legacy_tasks --update     # обновить уже импортированные
drush migrate:import legacy_tasks --limit=10   # пробный запуск
drush migrate:messages legacy_tasks
drush migrate:rollback legacy_tasks
```

- `migrate:status` — состояние;
- `migrate:import` — импорт;
- `migrate:messages` — сообщения и ошибки;
- `migrate:rollback` — откат (может удалять созданные сущности!).

Доступность команд зависит от набора инструментов. Перед повторным запуском выясните, какие записи обработаны, и прочитайте сообщения об ошибках. Для инкрементальных импортов используют `track_changes` и `high_water_property`.

### 13.9. Изменение существующей миграции

Изменение YAML не всегда обновляет уже импортированные записи: Drupal хранит состояние и map. Определите:

- какие записи нужно обработать повторно;
- можно ли обновлять существующие сущности (`--update`);
- нужен ли откат;
- нужно ли обновить определение миграции (в config entity — импорт конфигурации);
- как избежать потери данных.

Сначала проверьте сценарий на тестовой БД.

### 13.10. Миграции и update hooks

Миграция — перенос данных **из внешнего источника**. Update hook — преобразование **существующей** установки Drupal при обновлении кода. Преобразование внутри уже работающего сайта (как инициализация `field_project_type` у существующих задач) обычно делается update hook'ом, а не миграцией.

> [!example] Практика
> 
> Миграции описывают в YAML (source, process, destination), запускают через Migrate Tools и Drush, а зависимости между ними задают в migration_dependencies. Для связей используют migration_lookup, для преобразований — static_map и другие process-плагины.
> 
> Перед запуском на реальных данных миграцию проверяют на копии с --limit и смотрят migrate:messages. В DrupalJira старые статусы задач переводятся в новые, при этом нужно согласовать и состояние moderation workflow.

> [!warning] Типичные ошибки
> 
> - Не указать `migration_dependencies` / запускать задачи раньше проектов.
> - Менять YAML и ожидать автоматического обновления импортированных данных.
> - Выполнять `rollback` без понимания последствий.
> - Не обработать значения вне `static_map`.
> - Мигрировать только `field_status`, забыв moderation state и публикацию.
> - Не проверить миграцию на тестовой копии.
> - Использовать Migrate Plus-плагины, не установив модуль.

> [!question] Частые вопросы на собеседовании
> 
> **Из чего состоит миграция?** Из source (чтение), process (преобразование) и destination (сохранение в Drupal).
> 
> **Что такое ID map?** Таблица соответствия исходных записей и созданных сущностей. Она позволяет перезапускать, обновлять и откатывать миграцию.
> 
> **Как связать две миграции?** Указать migration_dependencies и использовать process-плагин migration_lookup.
> 
> **Какие команды Drush используются?** migrate:status, migrate:import (с --update и --limit), migrate:messages и migrate:rollback (откат может удалять сущности).

> [!tip] Ответ на собеседовании (EN)
> 
> A migration has a source, a process pipeline and a destination. Source plugins read rows, process plugins transform the values, and the destination, for example entity:node, saves them. The ID map remembers which source row became which entity, so I can re-run, update and roll back.
> 
> If tasks reference projects, I run the projects migration first, declare migration dependencies and use migration_lookup. For value conversions I use static_map and think about what happens with unknown values.
> 
> I first run a migration on a copy of the data and check messages with drush migrate:messages. And I make sure related state, like moderation state and publication status, stays consistent with the migrated fields.

---

## Блок 14. Field API

> [!abstract] Кратко
> 
> Field API добавляет сущностям структурированные данные. Нужно различать три понятия: Field Type (тип данных), Field Storage (как и где хранится, включая cardinality) и Field Configuration (использование поля на конкретном bundle). Ввод значения обеспечивает widget, вывод — formatter.
> 
> Поля бывают базовыми (объявлены в коде сущности) и конфигурируемыми (создаются через UI и хранятся в конфигурации).

### 14.1. Что такое Field API

Механизм добавления полей к сущностям и управления их значениями. Хранит:

- текст;
- числа;
- даты;
- ссылки;
- изображения;
- файлы;
- ссылки на другие сущности;
- значения списков.

У задачи DrupalJira: `field_status`, `field_assignee`, `field_estimate`, `field_project`. Каждое поле имеет тип данных и настройки.

### 14.2. Field Type, Field Storage, Field Configuration

**Field Type** — тип данных и поведение:

|Тип|Назначение|
|---|---|
|`string`|короткий текст|
|`text_long`|длинный текст|
|`integer`|целое число|
|`decimal`|десятичное число|
|`boolean`|логическое значение|
|`entity_reference`|ссылка на сущность|
|`datetime`|дата и время|
|`list_string`|список допустимых значений (подходит для статусов)|

**Field Storage** — как хранятся данные и базовые характеристики (тип, cardinality). `field_estimate` может иметь тип `decimal` и одно значение на материал. Storage связан с entity type и machine name поля.

**Field Configuration (instance)** — как поле используется на конкретном bundle: `field_estimate` добавлено к `task`, но отсутствует у `project`. Конфигурация хранится в `field.field.*`, storage — в `field.storage.*`.

### 14.3. Base fields и configurable fields

- **Base fields** объявляются в коде сущности (`baseFieldDefinitions()`), есть у всех bundles (`title`, `uid`, `status` у node).
- **Configurable fields** создаются через UI/конфигурацию для конкретных bundles (`field_status` у `task`).

Для custom entity вроде `time_log` поля обычно объявляют как base fields (см. Блок 10).

### 14.4. Создание поля через интерфейс

1. Открыть управление структурой сайта.
2. Выбрать тип материала.
3. Перейти к управлению полями.
4. Добавить поле.
5. Выбрать тип данных.
6. Настроить количество значений, обязательность и т.д.
7. Сохранить.

Например, `field_estimate` типа Decimal для оценки трудозатрат. Затем проверьте форму редактирования, отображение, права доступа и **экспорт конфигурации**.

### 14.5. Создание поля через конфигурацию

Определения полей — YAML. Упрощённый field storage:

```yaml
langcode: en
status: true
dependencies:
  module:
    - node
id: node.field_estimate
field_name: field_estimate
entity_type: node
type: decimal
settings:
  precision: 10
  scale: 2
module: core
locked: false
cardinality: 1
translatable: false
```

Это иллюстрация структуры; полная конфигурация зависит от типа поля, версии и модулей. Привязка к bundle `task` хранится отдельно (`field.field.node.task.field_estimate`). Не пишите YAML наугад — экспортируйте рабочую конфигурацию и используйте её как основу.

### 14.6. Чтение и изменение значений

```php
$estimate = $node->get('field_estimate')->value;

$node->set('field_estimate', '5.50');
$node->save();
```

Поле должно существовать у bundle (`$node->hasField('field_estimate')`) и содержать значение. Для `decimal` значение обычно строка — учитывайте точность. Для Entity Reference:

```php
$assignee = $node->get('field_assignee')->entity;   // может быть NULL
```

Поле с несколькими значениями:

```php
foreach ($node->get('field_attachments') as $item) {
  $target_id = $item->target_id;
}

$node->get('field_x')->isEmpty();
$node->get('field_x')->getValue();          // массив значений
$node->get('field_project')->referencedEntities();
```

Одиночное `->value` читает только первый элемент.

### 14.7. Cardinality

- `1` — одно значение;
- `-1` — неограниченное;
- другое положительное число — максимум.

Оценка задачи — `1`; вложения — несколько; несколько ссылок на пользователей — несколько. Cardinality влияет на форму, хранение и API доступа к значениям.

### 14.8. Entity Reference

Позволяет сущности ссылаться на другую: у задачи `field_project`.

```php
$project = $task->get('field_project')->entity;
```

Вместо дублирования названия проекта в каждой задаче храним ссылку и получаем название через связанную сущность. Учитывайте:

- существование целевой сущности;
- её доступность;
- допустимые целевые типы;
- правила удаления и обновления ссылок.

### 14.9. Field Widget и Field Formatter

**Widget** — как пользователь вводит значение: текстовое поле, список выбора, выбор даты, Entity Reference Autocomplete.

**Formatter** — как значение отображается: обычный текст, ссылка, изображение, дата в формате, ссылка на сущность.

Один тип поля может иметь несколько виджетов и форматтеров. Они настраиваются в _Manage form display_ и _Manage display_. Это плагины (Блок 11): собственный тип поля состоит из трёх плагинов — **FieldType**, **FieldWidget**, **FieldFormatter**.

### 14.10. Field API и Views

Views выводит данные сущностей и полей без ручных запросов и HTML. Представление задач может показывать название, статус, исполнителя, оценку и проект. Для нестандартной логики Views расширяют плагинами и handlers.

### 14.11. Поля и Configuration Management

Определения полей хранятся в конфигурации. При переносе учитывайте зависимости:

- модуль, предоставляющий тип поля;
- entity type и bundle;
- целевые entity types для Entity Reference;
- виджеты и форматтеры;
- связанные display-конфигурации.

`field.field.node.task.field_project` зависит от storage и типа `task`. Если модуль или тип сущности отсутствует, импорт может упасть.

### 14.12. Поля и schema updates

При разработке собственных типов полей или изменении структуры хранения учитывайте схему БД. Новая схема **не** применяется к существующей БД автоматически: нужны update hooks, обновление конфигурации или миграция данных. Особенно осторожно менять тип поля и структуру хранения, если в поле уже есть данные.

### 14.13. Когда применять

- добавить структурированные данные к сущности;
- настроить тип поля и cardinality;
- хранить ссылки на другие сущности;
- создать собственный тип поля;
- настроить виджеты и форматтеры;
- управлять конфигурацией полей;
- отображать поля через Views и Render API.

Преимущество — единый механизм хранения, редактирования и отображения данных.

> [!example] Практика
> 
> Практика с полями: создать через UI или конфигурацию, проверить виджет и форматтер, экспортировать конфигурацию, а значения читать через Entity API (get('field')->value, итерация по многозначным полям, entity для ссылок). Для существующих данных любые изменения storage требуют update hook или миграции.
> 
> Для собственной сущности поля обычно объявляют как base fields, для nodes — как configurable. Например, в DrupalJira у задачи есть список статусов, decimal-оценка и ссылка на проект.

> [!warning] Типичные ошибки
> 
> - Путать field storage и field configuration.
> - Читать только `->value` у многозначного поля.
> - Не проверять `NULL` у Entity Reference.
> - Менять тип/storage поля с данными без update hook или миграции.
> - Писать YAML полей вручную без экспорта.
> - Забывать экспортировать конфигурацию после создания поля через UI.
> - Менять `field_status` и забывать про moderation state.

> [!question] Частые вопросы на собеседовании
> 
> **Чем field storage отличается от field configuration?** Storage описывает тип, cardinality и хранение поля для entity type. Field configuration (field.field.*) описывает использование поля на конкретном bundle.
> 
> **Чем widget отличается от formatter?** Widget отвечает за ввод значения в форме, formatter — за отображение значения.
> 
> **Как прочитать многозначное поле?** Итерацией по элементам (foreach ($node->get('field') as $item)) или через getValue(). Простое ->value возвращает только первый элемент.
> 
> **Что делать, если нужно изменить storage поля с данными?** Использовать update hook или миграцию данных: изменение схемы не применяется к существующей БД автоматически.

> [!tip] Ответ на собеседовании (EN)
> 
> A field has a type that defines the kind of data, a storage that defines how it is stored, such as the entity type, field name and cardinality, and a field configuration for each bundle. Widgets handle editing and formatters handle display.
> 
> I read single values with get('field')->value and iterate over multi-value fields, and I always handle NULL when following entity references. Base fields are defined in code, and configurable fields are created through the UI and exported to configuration.
> 
> Because field definitions are configuration, they move with config sync, but changing the data of existing content or the storage of a field with data needs update hooks or migrations.

---

## Блок 15. Authentication API

> [!abstract] Кратко
> 
> Аутентификация отвечает на вопрос «кто делает запрос», авторизация — «что ему разрешено». За аутентификацию отвечают authentication providers: cookie-сессия, HTTP Basic, токены и OAuth из contrib-модулей.
> 
> После того как пользователь определён, Drupal всё равно проверяет доступ к маршруту, к конкретной сущности и бизнес-правила.

### 15.1. Что такое Authentication API

Механизм, с помощью которого Drupal определяет, кто выполняет запрос. Важен для REST API, интеграций с внешними системами, автоматизированных клиентов, пользовательских механизмов входа, токенов.

Пользователь может успешно войти, но не иметь права редактировать задачи.

### 15.2. Сессия и аутентификация

После входа через форму Drupal использует **сессию**, связанную с cookie. Браузер отправляет cookie при каждом запросе, и Drupal определяет пользователя. Удобно для веб-интерфейса, но не всегда подходит для внешних клиентов: они могут не использовать браузерные сессии, а передавать токен или иной способ подтверждения.

### 15.3. Authentication Provider

Компонент, определяющий, может ли данный механизм аутентифицировать запрос. Один использует сессию, другой — токен или HTTP Basic. Drupal выбирает провайдеры по типу запроса, модулям и конфигурации маршрута (`options: _auth: [...]`). Провайдер должен быть включён и настроен для маршрута.

### 15.4. Аутентификация и REST API

Для REST необходимо настроить:

- REST resource;
- HTTP-методы;
- форматы данных;
- механизм аутентификации (`authentication: [cookie, basic_auth]` в `rest.resource.*`);
- разрешения и доступ к сущностям.

Даже если токен подтверждает личность, Drupal проверяет право на операцию.

### 15.5. HTTP Basic Authentication

Имя и пароль передаются в заголовке `Authorization`, закодированные в Base64:

```http
Authorization: Basic dXNlcjpwYXNzd29yZA==
```

Base64 — **кодирование, а не шифрование**, поэтому используйте только через HTTPS. Модуль ядра — `basic_auth`. Подходит для отдельных интеграций, но не стоит выбирать его автоматически для всех API. Учитывайте безопасность передачи, хранение секретов в клиенте, ограничения доступа, доступность модуля, требования интеграции.

### 15.6. Cookie-аутентификация и CSRF

Браузер автоматически отправляет cookie, что создаёт риск CSRF для изменяющих операций. Изменение статуса через пользовательский маршрут не должно разрешаться лишь потому, что есть cookie сессии. Проверяйте:

- HTTP-метод;
- CSRF-токен (для cookie-запросов к REST/JSON:API — заголовок `X-CSRF-Token` со значением из `/session/token`);
- разрешения пользователя;
- доступ к сущности;
- допустимость операции.

### 15.7. Аутентификация через токены

**Токен** — значение, которое клиент предъявляет для подтверждения личности или полномочий:

- сессионные токены;
- персональные API-токены;
- OAuth access tokens (contrib: `simple_oauth`);
- JWT, key-auth и т.п. (contrib).

Не все механизмы входят в core. Учитывайте срок действия, отзыв, безопасное хранение, область полномочий (scopes), передачу по HTTPS, защиту от утечки через логи и URL. Не передавайте секретные токены в query string, если есть безопасная передача через заголовок.

### 15.8. Authentication и Authorization

Последовательность:

1. Drupal получает HTTP-запрос.
2. Механизм аутентификации определяет пользователя.
3. Проверяется доступ к маршруту.
4. Проверяется доступ к сущности.
5. Приложение проверяет бизнес-правила.
6. Данные изменяются.
7. Формируется ответ.

Успешная аутентификация сама по себе не даёт права на любые операции.

### 15.9. Собственный Authentication Provider

Нужен, когда стандартных механизмов недостаточно (например, собственная система идентификации и специальный токен). Реализуется как сервис с тегом `authentication_provider` и интерфейсом `AuthenticationProviderInterface`:

```yaml
services:
  authentication.example_token:
    class: Drupal\example\Authentication\Provider\TokenAuth
    arguments: ['@entity_type.manager']
    tags:
      - { name: authentication_provider, provider_id: 'example_token', priority: 100 }
```

```php
use Drupal\Core\Authentication\AuthenticationProviderInterface;
use Symfony\Component\HttpFoundation\Request;

final class TokenAuth implements AuthenticationProviderInterface {

  public function applies(Request $request): bool {
    return $request->headers->has('X-Api-Token');
  }

  public function authenticate(Request $request) {
    // Проверить токен; вернуть учётную запись или NULL.
  }

}
```

Метод `applies()` определяет, подходит ли провайдер к запросу, `authenticate()` возвращает пользователя либо `NULL`. Требуется понимание контрактов API, заголовков HTTP, жизненного цикла запроса, обработки невалидных данных, безопасности токенов и тестирования позитивных и негативных сценариев. Не создавайте собственный механизм ради защиты одного маршрута: для стандартных сценариев берите существующие механизмы Drupal или поддерживаемые модули.

### 15.10. Безопасность API

- использовать HTTPS;
- минимально необходимые права;
- проверять доступ к каждой сущности;
- валидировать входные данные;
- ограничивать HTTP-методы;
- не раскрывать секреты в логах;
- правильно обрабатывать ошибки;
- тестировать запросы без аутентификации и с недостаточными правами;
- следить за сроком действия и отзывом токенов;
- применять flood control для попыток входа.

Особенно важно, чтобы пользователь не получал чужую задачу простой сменой ID в URL.

### 15.11. Authentication vs Authorization

|Authentication|Authorization|
|---|---|
|определяет личность|определяет доступные действия|
|проверяет учётные данные или токен|проверяет permissions и access rules|
|сессия или токен|роли, разрешения, проверки доступа|
|«Кто ты?»|«Что тебе разрешено?»|

Оба механизма необходимы.

> [!example] Практика
> 
> На практике для внешней интеграции выбирают провайдер аутентификации: cookie для браузерных клиентов, Basic через HTTPS для простых интеграций, OAuth или токены из contrib-модулей для API. Его указывают в конфигурации REST-ресурса или в опции маршрута _auth.
> 
> После аутентификации проверяют permissions и entity access, а технической учётной записи выдают минимальные права. Например, система, читающая задачи DrupalJira, получает отдельного пользователя только с правом чтения.

> [!warning] Типичные ошибки
> 
> - Путать аутентификацию и авторизацию.
> - Считать, что токен/сессия дают право на любые операции.
> - Использовать Basic Auth без HTTPS.
> - Передавать токены в URL или логах.
> - Не включить провайдер для маршрута (`_auth`/`authentication` в REST-конфигурации).
> - Игнорировать CSRF для cookie-запросов.
> - Писать собственный провайдер там, где хватает готового.
> - Выдавать технической учётной записи права администратора.

> [!question] Частые вопросы на собеседовании
> 
> **Чем authentication отличается от authorization?** Authentication определяет, кто делает запрос. Authorization определяет, что этому пользователю разрешено делать.
> 
> **Что такое Authentication Provider?** Сервис с тегом authentication_provider, реализующий AuthenticationProviderInterface: applies() определяет, подходит ли он к запросу, authenticate() возвращает пользователя.
> 
> **Как включить Basic Auth для REST-ресурса?** Включить модуль basic_auth и добавить basic_auth в список authentication в конфигурации rest.resource.*.
> 
> **Почему Basic Auth нельзя использовать без HTTPS?** Base64 — это кодирование, а не шифрование: без HTTPS логин и пароль можно прочитать при перехвате.

> [!tip] Ответ на собеседовании (EN)
> 
> Authentication answers who is making the request, and authorization answers what that user is allowed to do. In Drupal an authentication provider identifies the user, for example from a session cookie, Basic auth or a token, and after that route access, entity access and business rules are checked.
> 
> Browsers send cookies automatically, so for cookie authentication on changing requests I also need CSRF protection. Basic auth must only be used over HTTPS, because Base64 is encoding, not encryption.
> 
> For integrations I use a dedicated account with least privilege and a token mechanism with expiry and revocation. I write a custom authentication provider only when existing mechanisms are not enough.

---

## Приложение. Базовые темы, на которые опираются блоки 1–15

> [!info] Зачем приложение
> 
> Без этих тем ответы на блоки неполны: на собеседовании их спрашивают рядом с основными. Каждая тема — минимум для уверенного ответа.

### A. Routing и Controllers

Маршрут связывает URL с обработчиком (`*.routing.yml`):

```yaml
drupaljira_timelog.task_view:
  path: '/task/{node}'
  defaults:
    _controller: '\Drupal\drupaljira_timelog\Controller\TaskController::view'
    _title: 'Task'
  requirements:
    _permission: 'access content'
    node: \d+
  options:
    parameters:
      node:
        type: entity:node
```

- `{node}` + `type: entity:node` — **параметрический конвертер** (param upcasting): контроллер получает объект `NodeInterface`, а не ID;
- `requirements` — доступ (`_permission`, `_entity_access`, `_custom_access`, `_csrf_token`, `_role`);
- `methods: [POST]` — ограничение HTTP-методов;
- контроллер возвращает render array или `Response`; наследуется от `ControllerBase`, зависимости — через `create()` (Блок 4).

### B. Form API

Формы наследуются от `FormBase` (обычные) или `ConfigFormBase` (настройки модуля). Методы:

- `getFormId()` — уникальный ID;
- `buildForm(array $form, FormStateInterface $form_state)` — массив элементов (`#type => 'textfield'`);
- `validateForm()` — проверка ввода (`$form_state->setErrorByName()`);
- `submitForm()` — обработка отправки.

Form API автоматически добавляет CSRF-токен (form token) и обрабатывает AJAX. Сущностные формы (`ContentEntityForm`) строятся из настроек _Manage form display_ (виджеты полей). Альтер чужих форм — `hook_form_alter()` / `hook_form_FORM_ID_alter()`.

### C. Render API, Twig и Theme layer

Вывод строится из **render arrays** (`#theme`, `#markup`, `#type`, `#cache`, `#attached`). Drupal превращает их в HTML, собирая метаданные кеша (bubbling) и библиотеки. Шаблоны — **Twig** (`*.html.twig`), переменные готовятся в preprocess-функциях (`hook_preprocess_HOOK`), темы регистрируются через `hook_theme()`. Подсказки имён шаблонов — Twig debug (`twig.config` в `development.services.yml`). Библиотеки CSS/JS — `*.libraries.yml` и `#attached['library']`. Автоэкранирование Twig — основа защиты от XSS (Блок 7).

### D. Hooks: процедурные и OOP

Хуки — именованные точки расширения: `hook_entity_presave()`, `hook_form_alter()`, `hook_cron()`, `hook_theme()` и др. Традиционно это функции в `.module`; в Drupal 11.1+ поддерживаются **OOP-хуки** — методы классов с атрибутом `#[Hook('entity_presave')]` (с DI через конструктор).

Отдельная группа — хуки установки и обновления в `.install`:

- `hook_install()` / `hook_uninstall()`;
- `hook_schema()` — таблицы БД;
- `hook_update_N()` — изменение схемы/данных у существующего сайта (номер, например `10001`, определяет порядок);
- `hook_post_update_NAME()` — действия после обновления схемы, когда доступны все сервисы и сущности.

Запуск — `drush updb`. Различие с событиями — Блок 9, с Configuration Management — Блок 3.

### E. Views

Конфигурируемый конструктор запросов и выводов: поля, фильтры, сортировки, relationships (JOIN), contextual filters, режимы отображения, форматы (таблица, список, JSON через REST export). Views — конфигурационные сущности (экспортируются в YAML), расширяются плагинами и `hook_views_data()` (для собственных сущностей, например `time_log`). Views кешируются по tags/time.

### F. Content Moderation и Workflows

Модули `workflows` и `content_moderation` добавляют состояния (draft, published, …) и **переходы**, учитывают роли и revisions. **Moderation state** хранится отдельно от поля публикации `status` и от пользовательского `field_status`. Поэтому при миграции и update hook'ах (`10001`) все три нужно согласовывать (Блоки 10, 13, 14).

### G. Инструменты разработки: Composer, Drush, логи

- **Composer** — зависимости, обновления (`composer require drupal/module`, `composer update`, `composer audit`).
- **Drush**: `cr` (сброс кеша), `updb`, `cex`/`cim`, `cron`, `queue:run`, `migrate:*`, `watchdog:show` (логи), `sql:cli`, `php:eval`, `uli` (одноразовая ссылка входа).
- **Логирование**: сервис `logger.factory` (`$this->getLogger('drupaljira_timelog')->error(...)`), модули `dblog`/`syslog`.
- **Окружение**: DDEV/Lando, `settings.local.php`, `development.services.yml` (отключение кеша для разработки).

---

## Итоговая шпаргалка: «что за что отвечает»

|Нужно…|Используй|
|---|---|
|Хранить настройки и переносить их|Configuration Management|
|Логику с зависимостями, переиспользуемую|Service + DI|
|Реакция на действие без жёстких связей|Event + Subscriber (или hook)|
|Данные: создать, найти, изменить|Entity API (storage, Entity Query)|
|Структура данных сущности|Field API|
|Заменяемый компонент (блок, виджет, worker)|Plugin API|
|Долгая операция с прогрессом|Batch API|
|Регулярное обслуживание|Cron|
|Отложенные независимые задания|Queue|
|Перенос данных из внешнего источника|Migration API|
|Преобразование существующего сайта при обновлении|update hook|
|API для фронтенда|JSON:API / REST Resource / маршрут|
|Кто делает запрос|Authentication API|
|Что ему разрешено|Permissions + Access API|
|Не отдать чужие/устаревшие данные из кеша|Cache contexts, tags, max-age|
|Несколько языков|Language, Content/Interface/Config Translation|
|Несколько сайтов на одном коде|Multisite|
|Проверка корректности|Unit → Kernel → Functional тесты|