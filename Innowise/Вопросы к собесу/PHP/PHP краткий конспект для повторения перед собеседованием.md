---
custom-width: 80
---
---

> [!info] Как пользоваться 
> Это не учебник, а список для быстрого повторения: прочитал пункт, вспомнил тему. Если что-то не вспоминается, открой подробный конспект `PHP.md` на нужном разделе. В конце есть блиц-вопросы для самопроверки.

---

## 1. Как работает PHP

- **Путь кода**: исходник → лексер → парсер (AST) → компилятор (опкоды) → Zend VM.
- **OPcache** хранит опкоды в разделяемой памяти, повторной компиляции нет. На проде обязателен.
- **shared nothing**: каждый запрос начинается с чистого состояния. Между запросами живут только OPcache, APCu, внешние хранилища.
- **SAPI**: CLI, FPM, mod_php. Стандарт: **Nginx + PHP-FPM** (FastCGI).
- **Долгоживущие рантаймы** (Swoole, RoadRunner, FrankenPHP): состояние живёт между запросами, нужно следить за утечками.
- Конфиг: `php.ini`, проверка `php --ini`, `php -m`, `php -l`.

## 2. Типы, переменные, константы

- Скаляры: `int`, `float`, `string`, `bool`. Составные: `array`, `object`, `callable`, `iterable`. Особые: `null`, `resource`.
- `int` 64 бита, при переполнении становится `float`. `0.1 + 0.2 != 0.3`, для денег: `int` в копейках, `bcmath` или `brick/money`.
- **Falsy**: `false`, `0`, `0.0`, `""`, `"0"`, `[]`, `null`. Истинны `"0.0"`, `" "`, `"false"`.
- `isset` — есть и не `null`. `empty` — нет или falsy. `array_key_exists` — ключ есть, даже если значение `null`.
- `const` — на этапе компиляции (в классах, нельзя в `if`). `define()` — в рантайме.
- `static` внутри функции хранит значение между вызовами. `global` и `$GLOBALS` — плохая практика.

## 3. Операторы и сравнение

- `===` по умолчанию. `==` приводит типы: `"0" == false`, `null == []`, `"1e3" == "1000"`.
- PHP 8: `0 == "abc"` теперь `false`. Magic hashes: `"0e123" == "0e456"` → `true`, поэтому хэши только через `hash_equals()`.
- `??` — «существует и не `null`», без notice. `?:` — «левое truthy».
- `<=>` возвращает -1, 0, 1. `??=` присваивает, если `null`.
- Приоритет: `and`/`or` ниже `=`, поэтому лучше `&&` и `||`.

## 4. Строки

- Строка — байты. UTF-8: кириллица 2 байта, поэтому `mb_strlen`, `mb_substr`, `mb_strtoupper`; в regex `/u`.
- `'...'` без интерполяции, `"..."` с ней. Heredoc с интерполяцией, nowdoc без.
- `strpos` возвращает `0` для начала строки, проверять `=== false`. В PHP 8 есть `str_contains`, `str_starts_with`, `str_ends_with`.
- Форматирование: `sprintf`, `number_format`. Сравнение секретов: `hash_equals`.

## 5. Массивы

- Массив — упорядоченная хэш-таблица, ключи `int` и `string`.
- `array_merge` перенумерует числовые ключи, `+` оставит левые. `array_filter` сохраняет ключи.
- `in_array` всегда с третьим `true`. Много проверок: `array_flip` и `isset` (O(1)).
- Основное: `array_map`, `array_filter`, `array_reduce`, `array_column`, `array_unique`, `array_slice`, `usort` с `<=>`, `array_key_first/last`, spread `...`.
- `array_unique` сохраняет ключи, сравнение нестрогое.

## 6. Функции, замыкания, генераторы

- Аргументы по значению (copy-on-write), объекты передаются как handle. Ссылка `&` нужна, чтобы подменить саму переменную.
- Замыкание берёт переменные через `use` (по значению, `&` по ссылке). Стрелочная `fn` захватывает автоматически, только по значению, одно выражение.
- Named arguments (8.0), variadic `...$args`, first-class callable `strlen(...)` (8.1).
- **Генератор** (`yield`): ленивая итерация, постоянная память. `yield from`, `send()`. Перемотать после старта нельзя.

## 7. Типизация

- `declare(strict_types=1)`: строгие скалярные типы в вызовах из этого файла (решает вызывающая сторона). Исключение: `int` → `float`.
- Типы: nullable `?T`, union `A|B`, intersection `A&B`, `mixed`, `void`, `never`, `static`, `iterable`.
- Типизированные свойства (7.4), constructor promotion (8.0).
- **Enum** (8.1): pure и backed (`from`, `tryFrom`), могут иметь методы и интерфейсы, нельзя `new`.
- `match`: строгое сравнение, выражение, бросает `UnhandledMatchError`.

## 8. ООП

- Столпы: инкапсуляция, наследование, полиморфизм, абстракция.
- Видимость: `public`, `protected`, `private`. Модификаторы: `final`, `abstract`, `static`, `readonly`.
- **Интерфейс** — контракт, класс реализует много. **Абстрактный класс** — частичная реализация, наследуется один.
- **Трейт**: горизонтальное переиспользование, конфликты через `insteadof` / `as`, статика у каждого класса своя.
- `self` — где написан код, `static` — класс вызова (late static binding), `new static()` для фабрик.
- `readonly`: один раз присвоить, но вложенный изменяемый объект менять можно.
- Объекты копируются по handle; `clone` поверхностный, глубокое копирование через `__clone`.

## 9. Продвинутое ООП

- Магические методы: `__construct`, `__destruct`, `__get`/`__set` (только для недоступных свойств), `__call`/`__callStatic`, `__toString`, `__invoke`, `__clone`, `__serialize`.
- `Stringable`, `ArrayAccess`, `Countable`, `IteratorAggregate`.
- Атрибуты `#[...]` читаются через Reflection. Анонимные классы, `instanceof`.
- Ковариантность (возврат) и контравариантность (параметры) — принцип Лисков.

## 10. Namespaces, Composer, PSR

- Namespace — защита от конфликтов имён. `use` импортирует, `as` задаёт алиас.
- **PSR-4**: namespace-префикс → каталог, `\` → `/`, имя файла = имя класса (регистр важен).
- Composer: `composer.json` (ограничения версий, `^`, `~`), `composer.lock` (точные версии, в git). `install` по lock, `update` пересчитывает.
- Прод: `composer install --no-dev -o`. `composer audit` — уязвимости.
- PSR: 1/12 стиль, 3 логгер, 4 автозагрузка, 7 HTTP-сообщения, 11 контейнер, 15 middleware, 18 HTTP-клиент.

## 11. Ошибки и исключения

- `Throwable` = `Error` (баги движка, `TypeError`, `DivisionByZeroError`) + `Exception` (прикладные).
- `catch (Exception)` не ловит `Error`. `catch (Throwable)` — на верхнем уровне.
- `finally` выполняется всегда, `return` в `finally` перебивает остальные. Цепочка исключений через `previous`.
- `set_error_handler` + `ErrorException`: warnings превращаются в исключения. Прод: `display_errors=Off`, логирование включено.
- Свои исключения: наследование, осмысленные имена, не ловить без действия.

## 12. Память и ссылки

- Подсчёт ссылок, при нуле память освобождается. Циклы чистит cycle collector (`gc_collect_cycles()`), критично в воркерах.
- **Ловушка foreach по ссылке**: после цикла `$v` остаётся ссылкой на последний элемент, нужен `unset($v)`.
- Copy-on-write: копия создаётся при изменении, а не при присваивании.
- Профилирование памяти: `memory_get_usage()`, `memory_get_peak_usage()`; `WeakReference` / `WeakMap` не удерживают объект.

## 13. HTTP, сессии, cookies

- Суперглобальные: `$_GET`, `$_POST`, `$_SERVER`, `$_COOKIE`, `$_SESSION`, `$_FILES`, `$_ENV`. Данные из них недоверенные.
- **Сессия**: данные на сервере, клиенту cookie `PHPSESSID`. Кластер — общее хранилище (Redis).
- Защита сессии: `session_regenerate_id(true)` после логина, `use_strict_mode`, флаги `HttpOnly`, `Secure`, `SameSite`.
- **headers already sent**: любой вывод до `header()` / `setcookie()`. Лечится буферизацией или Response-объектом.
- PRG (Post/Redirect/Get) против повторной отправки формы.

## 14. REST и API

|Метод|Идемпотентный|Назначение|
|---|---|---|
|GET|да|получить|
|POST|нет|создать|
|PUT|да|заменить целиком|
|PATCH|зависит|изменить частично|
|DELETE|да|удалить|

- Коды: 200, 201, 204, 301/302, 304, 400, **401** (нет аутентификации), **403** (нет прав), 404, 409, 422, 429, 500, 502/504.
- REST: ресурсы в URL, stateless, версии `/v1`, пагинация (курсорная лучше OFFSET), единый формат ошибок.
- **JWT**: без состояния, сложно отозвать, payload не шифруется. Для обычного сайта проще сессия.
- **CORS**: preflight `OPTIONS`, заголовки `Access-Control-Allow-*`; защищает пользователя браузера, а не сервер.
- Rate limiting (Redis, `429`), идемпотентность POST через `Idempotency-Key`.

## 15. Файлы и include

- `require` — фатальная ошибка при отсутствии файла, `include` — warning. `_once` не подключает повторно.
- Файлы: `fopen`/`fgets`/`fgetcsv`, `file_get_contents`, `SplFileObject`. Большие файлы читать потоково (генераторы).
- Блокировка записи `flock`. Атомарная запись: во временный файл, затем `rename`.
- Загрузка файлов: проверять тип по содержимому (`finfo`), размер, переименовывать, хранить вне webroot. Не подставлять ввод в `include` (LFI/RFI).

## 16. PDO и транзакции

- **Prepared statements**: структура SQL и данные раздельно, защита от SQLi. Только для **значений**; имена колонок, таблиц, `ORDER BY` — белый список.
- PDO: `ERRMODE_EXCEPTION`, `FETCH_ASSOC`, `ATTR_EMULATE_PREPARES=false`, `charset=utf8mb4`.
- Транзакция: `beginTransaction`, `commit`, `rollBack` в `try/catch`. **ACID**: атомарность, согласованность, изолированность, долговечность.
- **N+1**: 1 запрос списка + N запросов связей. Лечится `JOIN`, `WHERE IN`, eager loading.
- Пагинация: keyset вместо большого OFFSET.

## 17. SQL и индексы

- `INNER JOIN` — только пары, `LEFT JOIN` — все слева (справа `NULL`). Условие по правой таблице в `WHERE` ломает `LEFT`, ставить в `ON`.
- Индекс (B-tree) ускоряет чтение, замедляет запись. Составной `(a,b,c)` работает по **левому префиксу**.
- Индекс не используется: функция над колонкой, `LIKE '%x'`, несовпадение типов, нарушен префикс, низкая селективность.
- `EXPLAIN`: тип `ALL` — полный скан, смотреть `key`, `rows`, `filesort`.
- Изоляция: READ UNCOMMITTED → READ COMMITTED → REPEATABLE READ (MySQL по умолчанию) → SERIALIZABLE.
- Дедлок: одинаковый порядок доступа, короткие транзакции, повтор при ошибке. `SELECT ... FOR UPDATE` блокирует строки.
- `WHERE` до группировки, `HAVING` после. `UNION ALL` быстрее `UNION`.

## 18. Безопасность

|Угроза|Защита|
|---|---|
|SQL injection|prepared statements, белый список имён|
|XSS|`htmlspecialchars` на выводе, автоэкранирование шаблонов, CSP|
|CSRF|токен в форме, `SameSite`, `hash_equals`|
|IDOR|проверка прав на уровне объекта (`WHERE id=? AND user_id=?`)|
|Session fixation|`session_regenerate_id(true)`|
|Unserialize|JSON вместо `unserialize`, `allowed_classes`|
|Загрузка файлов|проверка MIME, хранить вне webroot|
|Пароли|`password_hash` / `password_verify` (bcrypt, Argon2id)|

- Нельзя: `md5`/`sha1` для паролей, `rand()` для токенов (нужен `random_bytes`), `eval`, вывод ошибок на проде.
- Валидация на входе, экранирование на выходе, минимальные права, HTTPS, `composer audit`.

## 19. Дата, regex, JSON

- `DateTimeImmutable` вместо `DateTime` (без побочных эффектов). Хранить в **UTC**, зону пользователя применять при выводе.
- Не хранить смещение вместо зоны (летнее время).
- `preg_match`, `preg_match_all`, `preg_replace`, `preg_replace_callback`, `preg_split`. Модификатор `/u` для Unicode. Опасность катастрофического отката.
- `json_encode` / `json_decode` с `JSON_THROW_ON_ERROR`; `JSON_UNESCAPED_UNICODE`; `json_validate` (8.3).

## 20. Паттерны, SOLID, DI

- **SOLID**: SRP (одна причина изменения), OCP (расширяем, не правим), LSP (наследник подменяет предка), ISP (мелкие интерфейсы), DIP (зависеть от абстракций).
- **DI**: зависимости приходят в конструктор. DI-контейнер (PSR-11, autowiring). **Service Locator** — антипаттерн.
- **IoC**: управление созданием и вызовом отдаём фреймворку, DI — один из способов.
- Паттерны: Factory, Strategy, Observer, Decorator, Adapter, Repository, DTO, Value Object, Command.
- **Singleton** — антипаттерн (скрытая глобальная зависимость, плохо тестируется).
- Active Record (Eloquent) проще, Data Mapper (Doctrine) лучше для сложного домена.

## 21. SPL и асинхронность

- SPL: `ArrayObject`, `SplStack`, `SplQueue`, `SplObjectStorage`, `SplPriorityQueue`, `SplFixedArray`; интерфейсы `Iterator`, `IteratorAggregate`, `Countable`, `ArrayAccess`.
- Тяжёлое: **очередь + воркеры** (Supervisor). Параллельные HTTP: `curl_multi`, Guzzle Promises, Amp, ReactPHP.
- **Fiber** (8.1): кооперативная корутина в одном потоке, без настоящей параллельности.

## 22. Производительность

- Узкие места: N+1, нет индексов, `SELECT *`, большой OFFSET, весь набор в память, тяжёлое в запросе (почта, PDF, внешние API).
- **OPcache** + preloading (меняешь файлы → перезапуск FPM). `composer dump-autoload -o`.
- **PHP-FPM**: `pm = static / dynamic / ondemand`. Предупреждение `max_children reached` → 502/504, лечится профилированием, очередью, ростом лимита по памяти.
- Измерять до оптимизации: Xdebug, Blackfire, Tideways, Symfony Profiler, лог медленных запросов.
- Масштабирование: stateless приложение, сессии и файлы вне сервера, реплики БД, кэш, CDN.

## 23. Кэш, Redis, очереди

- Уровни: HTTP (`Cache-Control`, `ETag`), CDN, приложение (Redis, APCu), OPcache.
- **Cache-aside**: сначала кэш, промах → БД → положить в кэш. Инвалидация: TTL, по событию, по версии ключа.
- **Cache stampede**: массовый промах популярного ключа. Лечат mutex, jitter TTL, отдачей устаревшего значения.
- Redis: строки, хэши, списки, множества, sorted sets, streams; счётчики и rate limit (`INCR` + `EXPIRE`), блокировки.
- Очереди: at-least-once, поэтому обработчик **идемпотентный**. Retry с backoff, failed-очередь, мониторинг.
- APCu — кэш одного сервера, Redis — общий.

## 24. Тестирование и качество

- **Пирамида**: много unit, меньше интеграционных, мало e2e.
- PHPUnit: arrange / act / assert, data providers. Двойники: **stub** (возвращает данные), **mock** (проверяет вызовы), **fake**, **spy**.
- Тестируемый код = DI, без глобального состояния и `new` внутри бизнес-логики.
- **PHPStan / Psalm**: статический анализ без запуска (уровни 0–9, baseline для legacy).
- Стиль: PSR-12, `phpcs`, `php-cs-fixer`. Автоматизация: Rector, CI.

## 25. Версии PHP

|Версия|Главное|
|---|---|
|7.0|скалярные типы, `??`, `<=>`, анонимные классы, `Throwable`|
|7.4|типизированные свойства, `fn`, `??=`, spread в массивах|
|**8.0**|named args, атрибуты, constructor promotion, union types, `match`, `?->`, JIT|
|**8.1**|enum, readonly, first-class callable, Fibers, `never`, intersection types|
|8.2|readonly-классы, DNF types, deprecation динамических свойств|
|8.3|типизированные константы, `#[\Override]`, `json_validate`|
|8.4|property hooks, асимметричная видимость, `array_find/any/all`|
|8.5|pipe-оператор `\|>`, `array_first/last` (сверить с php.net)|

## 26. Фреймворки

- Общее: роутинг, DI-контейнер, middleware, ORM, миграции, события, очереди, валидация, шаблонизатор.
- **Laravel**: Eloquent (Active Record), Blade, facades, service providers, Artisan, Queues/Horizon.
- **Symfony**: Doctrine (Data Mapper), Twig, autowiring, bundles, Messenger, компоненты (их использует Drupal).
- **Middleware**: слой вокруг контроллера (auth, CORS, лимиты), порядок важен.
- N+1 в ORM: `with()` (Eloquent), `JOIN FETCH` (Doctrine).

## 27. Инструменты

- **Git**: `merge` сохраняет историю, `rebase` делает линейной (не для общих веток), `revert` безопасно отменяет, `cherry-pick`, `stash`.
- **Docker**: образ → контейнер, `docker compose` (nginx + php-fpm + БД), volumes. Прод-образ: без Xdebug, с OPcache.
- **Xdebug**: отладка и профилирование, на проде выключен.
- **CI**: install → phpcs → phpstan → phpunit → сборка → деплой. Секреты в env, `.env` не в git, утёкший секрет ротировать.
- Логи: PSR-3 (Monolog), уровни, correlation id.

## 28. Drupal и Moodle

- **Drupal** (Symfony-компоненты): PSR-4 `Drupal\module\`, сервисы в `*.services.yml`, DI в конструктор, плагины через атрибуты/аннотации, события (EventSubscriber), хуки (`hook_*`, OOP-хуки `#[Hook]`), Render/Form arrays, Database API на PDO, Twig экранирует вывод.
- **Moodle**: `$DB`, `$CFG`, `$USER`, `$PAGE`, `$OUTPUT`; `required_param`/`optional_param` с `PARAM_*`; `require_login()`, `require_capability()`; sesskey против CSRF; `classes/` с автозагрузкой; adhoc/scheduled tasks вместо очередей; Mustache.

---

## Топ ловушек (повтори перед входом)

> [!warning] Ловушки PHP
> 
> 1. `strpos` вернул `0` — это не «не найдено», сравнивать `=== false`.
> 2. `empty("0")` → `true`.
> 3. `foreach` по ссылке без `unset($v)`.
> 4. `==` между числом и строкой.
> 5. `array_merge` против `+`; `array_filter` сохраняет ключи.
> 6. `in_array` без третьего `true`.
> 7. `strlen` / `substr` на UTF-8 без `mb_*`.
> 8. `float` для денег.
> 9. `return` в `finally`.
> 10. Объекты копируются по handle, `clone` поверхностный.
> 11. `self` против `static`.
> 12. `headers already sent`.
> 13. `isset` против `array_key_exists` на `null`.
> 14. `md5` для паролей, `rand()` для токенов, `unserialize` на чужих данных.

## Блиц-вопросы для самопроверки

> [!question] Почему PHP «быстрый после прогрева»?
> 
> OPcache хранит опкоды в разделяемой памяти, лексер, парсер и компилятор пропускаются.

---

> [!question] `isset` / `empty` / `array_key_exists`?
> 
> `isset`: есть и не `null`. `empty`: нет или falsy. `array_key_exists`: ключ есть, даже если `null`.

---

> [!question] `==` против `===`?
> 
> `==` приводит типы, `===` сравнивает тип и значение. Секреты: `hash_equals`.

---

> [!question] `self` против `static`?
> 
> `self` — класс, где написан код; `static` — класс, на котором вызвали (late static binding).

---

> [!question] Интерфейс против абстрактного класса?
> 
> Интерфейс — контракт без состояния, реализуется многими; абстрактный класс — частичная реализация, наследуется один.

---

> [!question] Как защититься от SQL-инъекции?
> 
> Prepared statements для значений, белый список для имён таблиц и колонок.

---

> [!question] Как хранить пароли?
> 
> `password_hash()` (bcrypt / Argon2id), проверка `password_verify()`, перехэширование `password_needs_rehash`.

---

> [!question] Что такое N+1 и как лечить?
> 
> 1 запрос списка + N запросов связей. `JOIN`, `WHERE IN`, eager loading.

---

> [!question] `Error` против `Exception`?
> 
> Оба `Throwable`. `Error` — ошибки движка, баги; `Exception` — прикладные. `catch (Exception)` не поймает `Error`.

---

> [!question] Зачем `strict_types`?
> 
> Строгая проверка скалярных типов в вызовах из файла, нет тихого приведения.

---

> [!question] Что такое DI?
> 
> Зависимости приходят снаружи (в конструктор), а не создаются внутри; проще тестировать и подменять.

---

> [!question] Сессия против JWT?
> 
> Сессия: состояние на сервере, легко отозвать. JWT: без состояния, сложно отозвать, подходит для API.

---

> [!question] `401` против `403`?
> 
> `401` — не аутентифицирован, `403` — аутентифицирован, но прав нет.

---

> [!question] Составной индекс `(a,b,c)`: когда работает?
> 
> По левому префиксу: `a`; `a,b`; `a,b,c`. Один `b` не использует индекс.

---

> [!question] Как не упасть по памяти на огромном CSV?
> 
> Читать потоком (`fgetcsv`, генератор), вставлять пачками в транзакции, уметь возобновлять.

---

> [!question] Как сделать так, чтобы пользователь не ждал письма?
> 
> Положить задачу в очередь, ответить сразу, воркер отправит; retry, failed-очередь, идемпотентность.

---

> [!question] Что нового в PHP 8.0 и 8.1?
> 
> 8.0: named args, атрибуты, promotion, union, `match`, `?->`, JIT. 8.1: enum, readonly, first-class callable, Fibers, `never`.

---

## Что сказать, если опыта мало

> [!tip] Схема ответа
> 
> 1. Принцип в одном предложении.
> 2. Пример из своего проекта (модуль Drupal, лабораторная).
> 3. Безопасность и производительность: prepared statements, экранирование, индексы, очередь.
> 4. Назвать версию PHP, где появилась возможность.
> 5. Если не помнишь точную сигнатуру: «сверил бы по php.net, принцип такой...».

> [!tip] Вопросы интервьюеру
> 
> - Какая версия PHP на проде?
> - Есть ли PHPStan, PHPCS, PHPUnit в CI?
> - Как устроены code review и деплой?
> - Какие фреймворки, есть ли legacy?
> - Как организованы очереди, кэш и мониторинг?