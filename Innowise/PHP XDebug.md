---
custom-width: 80
---
# 🐞 PHP Xdebug — полный конспект

> [!info] О конспекте 
> Собрано на основе официальной документации Xdebug (xdebug.org/docs) по состоянию на **4 сентября 2026 года**. Актуальная стабильная версия — **Xdebug 3.5.3** (разрабатываемая ветка — 3.6.0dev).

---

## 📖 Базовые понятия (глоссарий)

> [!abstract]+ Что такое Xdebug
> 
> - **Xdebug** — расширение PHP (написано на C), написанное Дериком Ретансом, дающее возможности отладки и анализа кода: пошаговая отладка, профилирование, трассировка функций, анализ покрытия кода, статистика сборщика мусора. Подключается к PHP как **zend_extension**.
> - **Модель "клиент-сервер наоборот" (reverse connection)** — ключевая особенность архитектуры: именно **PHP с Xdebug инициирует** соединение к отлаживающему клиенту (IDE), а не наоборот. IDE выступает TCP-сервером и слушает порт (по умолчанию **9003**), а Xdebug — TCP-клиентом, который подключается к нему при старте отладочной сессии.
> - **DBGp** — открытый протокол отладки (Debugger Protocol), на основе XML, которым Xdebug обменивается данными с IDE. Поддерживается почти всеми PHP IDE, включая VS Code и PhpStorm.
> - **IDE / Debugging client (отладочный клиент)** — программа, принимающая соединение от Xdebug и предоставляющая интерфейс отладки: PhpStorm, VS Code (с расширением `php-debug`), NetBeans, Eclipse (плагин), Sublime Text и др.
> - **Xdebug Cloud** — сервис "Proxy-as-a-Service" от команды Xdebug, решающий проблемы сетевой связности (например, когда PHP и IDE не могут напрямую соединиться из-за файрвола/NAT), выступая ретранслятором между Xdebug и IDE.
> - **`xdebug.mode`** — центральная настройка Xdebug 3, определяющая, какие именно функции расширения активны (в отличие от Xdebug 2, где для каждой функции был отдельный флаг включения). Позволяет ограничить накладные расходы только теми функциями, которые реально нужны.

> [!abstract]+ Режимы (`xdebug.mode`)
> 
> - **`off`** — ничего не включено, Xdebug почти не создаёт накладных расходов (используется на production, если расширение вообще оставлено загруженным).
> - **`develop`** — включает **Development Helpers**: улучшенный `var_dump()`, расширенные сообщения об ошибках со стек-трейсом, обнаружение бесконечной рекурсии.
> - **`debug`** — включает **Step Debugging** (пошаговую отладку).
> - **`coverage`** — включает **Code Coverage Analysis** — анализ покрытия кода тестами (используется, в частности, PHPUnit).
> - **`gcstats`** — включает **Garbage Collection Statistics** — статистику работы сборщика мусора PHP.
> - **`profile`** — включает **Profiling** — профилирование производительности (вывод в формате Cachegrind).
> - **`trace`** — включает **Function Trace** и **Flame Graphs** — построчную запись всех вызовов функций, присваиваний и возвращаемых значений.
> - Режимы можно комбинировать через запятую: `xdebug.mode=develop,trace`. Также можно задать через переменную окружения **`XDEBUG_MODE`**, которая имеет приоритет над `xdebug.mode` (но не изменяет само значение настройки).

> [!abstract]+ Активация (триггеры)
> 
> - **Trigger (триггер)** — механизм, позволяющий включать функциональность Xdebug выборочно, а не для каждого запроса подряд. Управляется настройкой **`xdebug.start_with_request`**.
> - **`xdebug.start_with_request`** — принимает `yes` (включать при каждом запросе), `no` (не включать автоматически), `trigger` (включать только при наличии триггера) или `default` (зависит от режима: для `debug`/`trace` по умолчанию `trigger`, для `profile` — `yes`).
> - **`XDEBUG_TRIGGER`** — универсальное имя триггера с Xdebug 3: переменная окружения, GET/POST-параметр или COOKIE с этим именем запускает соответствующую функциональность. Есть legacy-имена для конкретных функций: `XDEBUG_SESSION` (Step Debugging), `XDEBUG_PROFILE` (Profiling), `XDEBUG_TRACE` (Function Trace).
> - **`xdebug.trigger_value`** — если задано, триггер сработает только при точном совпадении значения (с Xdebug 3.1 можно указать несколько значений через запятую).
> - **Browser extension** — расширения браузера ("Xdebug Helper by JetBrains" для Firefox/Chrome/Edge, XDebugToggle для Safari), одним кликом выставляющие нужную cookie для запуска отладки прямо из браузера.

---

## 🗺️ Карта конспекта

|#|Раздел|Что внутри|
|---|---|---|
|1|[[#🖥️ Установка]]|PIE, PECL, из исходников, настройка php.ini|
|2|[[#⚙️ Режимы и активация]]|`xdebug.mode`, триггеры, `XDEBUG_MODE`|
|3|[[#🔍 Step Debugging]]|Пошаговая отладка, подключение к IDE, активация|
|4|[[#📊 Profiling]]|Профилирование, Cachegrind, KCacheGrind|
|5|[[#📜 Function Trace и Flame Graphs]]|Трассировка вызовов функций|
|6|[[#🧪 Coverage и Development Helpers]]|Покрытие кода тестами, отладочные хелперы|
|7|[[#🐳 Xdebug и DDEV]]|Как Xdebug настроен и используется внутри DDEV|
|8|[[#🪵 Логирование и диагностика]]|`xdebug.log`, `xdebug_info()`, troubleshooting|
|9|[[#🔗 Полезные ссылки]]|Ссылки на официальную документацию|

---

## 🖥️ Установка

> [!quote] Определение 
> Способ установки зависит от системы. Актуальный официальный путь для Linux (без пакетного менеджера), macOS и Windows — **PIE**; для Linux с пакетным менеджером — системные пакеты; устаревший (legacy) способ — **PECL**.

### Через пакетный менеджер (Linux)

```bash
# Debian/Ubuntu
sudo apt-get install php-xdebug

# Ubuntu (Ondřej Surý PPA, с версией PHP)
sudo apt-get install php8.3-xdebug

# Fedora / CentOS (Remi Repo)
sudo dnf install php-xdebug
sudo yum install php83-php-xdebug3

# Arch Linux
sudo pacman -S xdebug

# Alpine Linux
sudo apk add php8-pecl-xdebug
```

> [!warning] Устаревшие версии в дистрибутивах 
> Дистрибутивы Linux иногда предоставляют старую и/или более не поддерживаемую версию Xdebug. Если пакетный менеджер ставит неподдерживаемую версию — используйте PIE или сборку из исходников.

### PIE — актуальный кроссплатформенный способ

**PIE** (PHP Installer for Extensions) — официальный инсталлятор PHP-расширений для Linux, macOS и Windows, построенный по образцу Composer и работающий поверх экосистемы Packagist. Заменил устаревший PECL.

```bash
# macOS — сначала установить PIE через Homebrew
brew install pie

# Затем в любой ОС:
pie install xdebug/xdebug
```

На Linux/macOS PIE скачивает исходники и сам конфигурирует, компилирует и устанавливает расширение. На Windows PIE скачивает готовые предсобранные бинарники. В обоих случаях PIE также создаёт нужный `.ini`-файл (например, `90-xdebug.ini`) или добавляет строку в `php.ini`.

### PECL (legacy)

```bash
pecl install xdebug
```

> [!warning] Не следуйте подсказке PECL добавить `extension=xdebug.so` 
> Xdebug — **zend_extension**, а не обычное расширение; использование `extension=` вместо `zend_extension=` вызовет проблемы.

### Из исходников

```bash
tar -xzf xdebug-3.5.3.tgz && cd xdebug-3.5.3
phpize
./configure --enable-xdebug
make
make install
```

### Настройка php.ini

```ini
zend_extension=xdebug
```

> [!warning] Порядок с OPcache 
> Если используете Xdebug вместе с **OPcache**, строка `zend_extension` для Xdebug должна идти **после** строки для OPcache (либо в файле с более высоким номером — например, `99-xdebug.ini` против `20-opcache.ini`), иначе они не будут работать корректно вместе.

Проверка установки:

```bash
php -v
# Xdebug должен появиться в выводе с номером версии
```

Либо вызвать функцию **`xdebug_info()`** на веб-странице — покажет полную диагностику (аналог `phpinfo()`).

### Несовместимые расширения

Другие модули, глубоко встраивающиеся во внутренности PHP, обычно несовместимы с Xdebug и должны быть отключены при его использовании: **Blackfire**, **Opcache** (с осторожностью — см. выше), **DBG**, **APD**, **ionCube**, **PHP8 JIT**.

---

## ⚙️ Режимы и активация

### `xdebug.mode` — центральная настройка

> [!warning] Где можно задать 
> `xdebug.mode` можно установить **только** в `php.ini` или подобных файлах, читаемых при старте PHP-процесса (напрямую или через php-fpm). Задать это значение в `.htaccess`, `.user.ini` или через `php_admin_value` (Apache VHost / PHP-FPM pool) **нельзя**.

|Режим|Что включает|
|---|---|
|`off`|Ничего; минимальные накладные расходы|
|`develop`|Development Helpers (улучшенный `var_dump()`, детальные ошибки)|
|`debug`|[[#🔍 Step Debugging]]|
|`coverage`|[[#🧪 Coverage и Development Helpers\|Code Coverage Analysis]]|
|`gcstats`|Garbage Collection Statistics|
|`profile`|[[#📊 Profiling]]|
|`trace`|[[#📜 Function Trace и Flame Graphs]]|

```ini
xdebug.mode=debug
; или несколько одновременно:
xdebug.mode=develop,trace
```

### Переменная окружения `XDEBUG_MODE`

Переопределяет `xdebug.mode` на время выполнения (например, для одной CLI-команды), но не изменяет само значение настройки:

```bash
XDEBUG_MODE=debug php script.php
```

> [!warning] PHP-FPM и переменные окружения 
> У PHP-FPM есть настройка `clear_env` (по умолчанию `on`), которая **очищает переменные окружения** перед передачей в PHP. Чтобы `XDEBUG_MODE` работал через PHP-FPM, нужно выставить `clear_env=no` или явно разрешить нужную переменную.

### `xdebug.start_with_request` — когда активироваться

|Значение|Поведение|
|---|---|
|`yes`|Функциональность стартует в начале каждого запроса, до выполнения PHP-кода|
|`no`|Не активируется автоматически (Step Debugging и Profiling вообще не запустятся; Function Trace/GC Stats можно запустить программно)|
|`trigger`|Активируется только при наличии триггера (`XDEBUG_TRIGGER` в `$_ENV`/`$_GET`/`$_POST`/`$_COOKIE`)|
|`default`|Зависит от режима: `debug`→`trigger`, `trace`→`trigger`, `profile`→`yes`, `gcstats`→`no`|

### Триггеры

- **`XDEBUG_TRIGGER`** — универсальное имя (с Xdebug 3). Значение не важно, если не задан `xdebug.trigger_value`.
- Legacy-имена для конкретных функций: **`XDEBUG_SESSION`** (Step Debugging), **`XDEBUG_PROFILE`** (Profiling), **`XDEBUG_TRACE`** (Function Trace).
- **`xdebug.trigger_value`** — при непустом значении триггер срабатывает только на совпадение; с Xdebug 3.1 можно перечислить несколько значений через запятую: `xdebug.trigger_value=StartDebuggerForMe,StartDebuggerForYou`.

### Другие способы запуска

- **`xdebug_break()`** — программно ставит точку останова в месте вызова; запускает отладочную сессию, если `xdebug.start_with_request=trigger` и сессия ещё не активна.
- **`xdebug.start_upon_error`** — если `yes`, отладочная сессия запускается автоматически при возникновении PHP Notice/Warning или при выбросе `Throwable` (Exception/Error) — независимо от значения `xdebug.start_with_request`.
- **`xdebug_connect_to_client()`** _(с версии 3.1)_ — заново пытается установить соединение с клиентом в рамках уже идущего долгоживущего процесса (например, PHP-воркер очереди), даже если в начале запроса клиент не слушал.

---

## 🔍 Step Debugging

> [!quote] Определение 
> Пошаговый отладчик Xdebug позволяет интерактивно проходить по коду для анализа потока управления и просмотра структур данных. Xdebug взаимодействует с IDE через открытый протокол **DBGp**.

### Настройка соединения

1. Установить `xdebug.mode=debug` в PHP ini
2. Если PHP/Xdebug и IDE на одной машине — этого достаточно
3. Если на разных машинах (или в Docker) — нужно явно указать IDE-хост:

|Настройка|Назначение|По умолчанию|
|---|---|---|
|`xdebug.client_host`|IP/хостнейм машины с IDE|`localhost`|
|`xdebug.client_port`|TCP-порт, на котором слушает IDE|`9003`|
|`xdebug.discover_client_host`|Автоматически определять хост IDE по HTTP-заголовкам входящего запроса|`false`|
|`xdebug.client_discovery_header`|Какой HTTP-заголовок использовать при `discover_client_host`|`HTTP_X_FORWARDED_FOR,REMOTE_ADDR`|
|`xdebug.connect_timeout_ms`|Сколько ждать подтверждения от IDE|`200`|

> [!tip] Специальные значения `xdebug.client_host`
> 
> - `xdebug://gateway` — использовать сетевой шлюз системы (только Linux)
> - `xdebug://nameserver` — использовать DNS-сервер приватной сети (только Linux, не работает с musl libc/Alpine)
> - `unix:///path/to/sock` — Unix domain socket (не-Windows платформы, поддерживается ограниченным числом клиентов)

> [!note] Xdebug Cloud 
> Если прямое соединение между Xdebug и IDE невозможно из-за сети или файрвола — **Xdebug Cloud** выступает ретранслятором между ними. Настраивается через `xdebug.cloud_id` с токеном из профиля на xdebug.cloud.

### Активация — Command Line

```bash
export XDEBUG_SESSION=1     # Unix
set XDEBUG_SESSION=1        # Windows
php myscript.php
vendor/bin/phpunit
```

### Активация — веб-приложение

|Способ|Как работает|
|---|---|
|**Browser extension**|"Xdebug Helper by JetBrains" (Firefox/Chrome/Edge), XDebugToggle (Safari) — одним кликом ставит нужную cookie; отладка стартует для каждого запроса, пока переключатель включён|
|**Ручной запуск (одиночный запрос)**|GET/POST-параметр `XDEBUG_SESSION=имя_сессии`|
|**Сессия на несколько запросов**|GET/POST-параметр `XDEBUG_SESSION_START=имя` — устанавливает cookie `XDEBUG_SESSION`, действующую (с Xdebug 3.1) без ограничения по времени, пока не отправлен `XDEBUG_SESSION_STOP`|
|**HTTP Cookie напрямую**|`Cookie: XDEBUG_SESSION=start`|

### Игнорирование триггера

Если нужно **предотвратить** запуск отладки, несмотря на присутствующий триггер (например, для XHR-запроса от debug-тулбара) — передать `XDEBUG_IGNORE` (cookie/GET/POST) с любым значением, кроме `no`/`0`.

### IDE-клиенты

PhpStorm (JetBrains, коммерческий), VS Code (плагин `php-debug`, open source), NetBeans, Eclipse (плагин), KDevelop, Komodo, PHP Tools for Visual Studio, SublimeTextXdebug, VIM (плагин vdebug). Также доступен простой **Command Line Debug Client** (`dbgpClient`) из состава проекта Xdebug — для отладки рекомендуется полноценная IDE.

---

## 📊 Profiling

> [!quote] Определение 
> Встроенный профилировщик Xdebug позволяет находить узкие места в скрипте и визуализировать их внешними инструментами вроде **KCacheGrind** или **QCacheGrind**. Также собирает данные об использовании памяти по функциям.

### Запуск

```ini
xdebug.mode=profile
xdebug.output_dir=/tmp
```

Xdebug пишет данные в файлы формата **Cachegrind**, имя которых начинается с `cachegrind.out.` и по умолчанию заканчивается PID процесса (настраивается через `xdebug.profiler_output_name`). HTTP-ответ профилируемого запроса содержит заголовок **`X-Xdebug-Profile-Filename`** с именем файла.

Выборочный запуск — через триггер:

```ini
xdebug.mode=profile
xdebug.start_with_request=trigger
```

### Анализ результатов

|Инструмент|Платформа|
|---|---|
|**KCacheGrind**|Linux (KDE), полнофункциональный|
|**QCacheGrind**|Windows, macOS (Homebrew) — тот же KCacheGrind без зависимости от KDE|
|**Webgrind**|Веб-интерфейс на PHP|

В KCacheGrind панель **"Flat Profile"** показывает все функции скрипта, отсортированные по времени; колонка **"Self"** — время внутри самой функции (без вложенных вызовов), **"Called"** — сколько раз вызвана.

### Ключевые настройки

|Настройка|Значение по умолч.|Назначение|
|---|---|---|
|`xdebug.output_dir`|`/tmp`|Куда пишутся файлы профилирования, трассировки, GC-статистики|
|`xdebug.profiler_output_name`|`cachegrind.out.%p`|Шаблон имени файла (спецификаторы, как у `sprintf`/`strftime`)|
|`xdebug.profiler_append`|`0`|При `1` — новые профили дописываются в существующий файл вместо перезаписи|
|`xdebug.use_compression`|`true`|GZip-сжатие файлов профилирования и трассировки (не поддерживается QCacheGrind — для него нужно выставить `false`)|

---

## 📜 Function Trace и Flame Graphs

> [!quote] Определение 
> **Function Trace** записывает в файл каждый вызов функции, включая аргументы, присваивания переменных и возвращаемые значения, произошедшие за время запроса. **Flame Graphs** используют эти данные для визуализации характеристик производительности.

```ini
xdebug.mode=trace
xdebug.start_with_request=yes   ; трассировать весь запрос целиком
```

Как и в профилировании, можно запускать выборочно через `xdebug.start_with_request=trigger` и `XDEBUG_TRACE`/`XDEBUG_TRIGGER`. Файлы также пишутся в `xdebug.output_dir`.

---

## 🧪 Coverage и Development Helpers

### Code Coverage Analysis (`xdebug.mode=coverage`)

Генерирует отчёты о покрытии кода тестами — используется в основном в связке с **PHPUnit** для оценки того, какая часть кодовой базы выполняется тестами.

### Development Helpers (`xdebug.mode=develop`)

- Перегруженная (улучшенная) версия **`var_dump()`** — с подсветкой типов и структурой
- Расширенные сообщения об ошибках со стек-трейсом вызовов
- Обнаружение бесконечной рекурсии

### Garbage Collection Statistics (`xdebug.mode=gcstats`)

Сбор статистики о работе механизма сборки мусора PHP — сколько циклов, сколько объектов собрано и т.д.

---

## 🐳 Xdebug и DDEV

> [!note] Связь с другими конспектами 
> Если вы используете [[DDEV — конспект|DDEV]] для локальной разработки (в том числе на [[Drupal — конспект|Drupal]]), Xdebug уже установлен в web-контейнере — не нужно ничего компилировать самостоятельно.

### Модель подключения в DDEV

Поскольку PHP выполняется **внутри Docker-контейнера**, а IDE — на хост-машине, Xdebug должен "дотянуться" до хоста через границу контейнера. DDEV настраивает Xdebug на подключение к специальному хосту **`host.docker.internal`**, который внутри контейнера резолвится в IP-адрес хост-машины. Порт по умолчанию — **9003** (TCP-сервер — IDE, TCP-клиент — Xdebug/PHP, как описано в [[#🔍 Step Debugging]]).

### Управление через `ddev xdebug`

```bash
ddev xdebug on        # включить Xdebug (действует до `ddev start`/`ddev restart`)
ddev xdebug off        # выключить (лучше для производительности, когда не отлаживаете)
ddev xdebug toggle      # переключить состояние
ddev xdebug status      # показать текущее состояние
```

> [!warning] Composer и Xdebug 
> **Composer отключает Xdebug**, даже если он включён в DDEV. Чтобы отладить сам Composer, установите переменную окружения `COMPOSER_ALLOW_XDEBUG=1`. Из-за перемещения классов (например, плагинов) во временные файлы во время выполнения точки останова в IDE могут срабатывать не всегда — в таких случаях используйте `xdebug_break()` прямо в коде.

### xdebugctl — управление на лету

DDEV включает утилиту **`xdebugctl`** для динамического запроса и изменения настроек Xdebug, переключения режимов (`debug`, `profile`, `trace`) без перезапуска контейнера:

```bash
ddev exec xdebugctl --help
```

### Диагностика в DDEV

```bash
ddev debug xdebug-diagnose            # проверка конфигурации, сети, настроек IDE
ddev debug xdebug-diagnose --interactive  # пошаговая интерактивная диагностика с проверкой реального отклика IDE
```

### Особые сетевые случаи

- **WSL2 (Mirrored networking mode)** — требует `hostAddressLoopback=true` в `.wslconfig` (см. также [[DDEV — конспект#🖥️ Установка и системные требования|конспект DDEV]])
- **IDE внутри WSL2 (WSLg) или через прокси (JetBrains Gateway)**: `ddev config global --xdebug-ide-location=wsl2`
- **IDE с прокси внутри web-контейнера**: `ddev config global --xdebug-ide-location=container`

### Настройка VS Code

Типовой `.vscode/tasks.json` для включения/выключения Xdebug прямо из VS Code:

```json
{
  "version": "2.0.0",
  "tasks": [
    { "label": "DDEV: Enable Xdebug", "type": "shell", "command": "ddev xdebug on" },
    { "label": "DDEV: Disable Xdebug", "type": "shell", "command": "ddev xdebug off" }
  ]
}
```

Эти задачи удобно подключить как `preLaunchTask`/`postDebugTask` в `.vscode/launch.json`, чтобы Xdebug включался и выключался автоматически вместе с сессией отладки.

---

## 🪵 Логирование и диагностика

### `xdebug.log`

```ini
xdebug.log=/tmp/xdebug.log
```

Логирует попытки соединения (Step Debugging), проблемы создания файлов (Profiling, Trace) и — при высоком уровне логирования — полный обмен DBGp-командами с IDE в формате XML. Файл открывается в режиме дозаписи (append) и не перезаписывается.

> [!warning] systemd private tmp 
> На многих дистрибутивах Linux с systemd используются приватные директории `/tmp` для сервисов — реальный путь к логу может выглядеть как `/tmp/systemd-private-<hash>-apache2.service-<rand>/xdebug.log`.

### `xdebug.log_level`

|Уровень|Название|Пример|
|---|---|---|
|0|Criticals|Ошибки конфигурации|
|1|Errors|Ошибки соединения|
|3|Warnings|Предупреждения о соединении|
|5|Communication|Сообщения протокола|
|**7** _(по умолч.)_|Information|Информация о процессе подключения|
|10|Debug|Информация о разрешении точек останова|

### `xdebug_info()`

Функция диагностики — вызванная без аргументов на веб-странице, выводит HTML-страницу (аналог `phpinfo()`) с активным режимом, всеми настройками и диагностическим логом ошибок/предупреждений.

```php
var_dump(xdebug_info('mode'));             // ['debug', 'develop', 'trace']
var_dump(xdebug_info('extension-flags'));  // ['compression', 'control-socket', 'tsc']
```

### `xdebug_is_debugger_active()`

Возвращает `true`, если отладочная сессия через DBGp сейчас активна и клиент подключён.

---

## 🔗 Полезные ссылки

- 📘 [Documentation Home](https://xdebug.org/docs/) — главная страница документации Xdebug
- 🖥️ [Installation](https://xdebug.org/docs/install) — установка (PIE, PECL, из исходников)
- 🔍 [Step Debugging](https://xdebug.org/docs/step_debug) — полная документация по пошаговой отладке
- 📊 [Profiling](https://xdebug.org/docs/profiler) — профилирование и Cachegrind
- 📜 [Function Trace](https://xdebug.org/docs/trace) · [Flame Graphs](https://xdebug.org/docs/flamegraphs)
- 🧪 [Code Coverage Analysis](https://xdebug.org/docs/code_coverage) · [Development Helpers](https://xdebug.org/docs/develop) · [Garbage Collection Statistics](https://xdebug.org/docs/garbage_collection)
- ⚙️ [All Settings](https://xdebug.org/docs/all_settings) — полный справочник всех настроек
- 🔀 [Upgrading from Xdebug 2 to 3](https://xdebug.org/docs/upgrade_guide) — гайд по миграции
- 🐛 [Description of Errors](https://xdebug.org/docs/errors) — расшифровка ошибок и предупреждений
- 🐳 [Step Debugging with DDEV](https://docs.ddev.com/en/stable/users/debugging-profiling/step-debugging/) · [Xdebug Profiling with DDEV](https://docs.ddev.com/en/stable/users/debugging-profiling/xdebug-profiling/)
- 💻 [Репозиторий на GitHub](https://github.com/xdebug/xdebug)

> [!success] Готово 
> Конспект охватывает установку (актуальный способ через PIE), центральную концепцию режимов `xdebug.mode` и триггеров активации, все основные функции (Step Debugging, Profiling, Function Trace, Coverage, Development Helpers), а также практическую интеграцию с DDEV — включая команды `ddev xdebug`, `xdebugctl` и особенности сетевого подключения через `host.docker.internal`. Смотрите также связанные конспекты [[DDEV — конспект]] и [[Drupal — конспект]].