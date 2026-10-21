---
custom-width: 80
---
# 🐳 DDEV — полный конспект

> [!info] О конспекте
>  Собрано на основе официальной документации DDEV (docs.ddev.com) и репозитория [ddev/ddev](https://github.com/ddev/ddev) по состоянию на **2 сентября 2026 года**. Актуальная стабильная версия — **DDEV v1.25.3**.

---

## 📖 Базовые понятия (глоссарий)

> [!abstract]+ Что такое DDEV
> 
> - **DDEV** — открытый (Apache License 2.0) инструмент для запуска локальных окружений веб-разработки, готовых за минуты. Поддерживает **PHP, Python и Node.js**. Написан на **Go**, распространяется как один бинарник `ddev`.
> - **Docker workflow без сложности** — DDEV даёт командам возможность использовать Docker в рабочем процессе без необходимости знать Docker и без кастомной настройки каждого проекта — конфигурации переносимы, версионируются и расшариваются между разработчиками.
> - **Docker provider (провайдер Docker)** — среда выполнения контейнеров, которую DDEV использует под капотом: **Docker Desktop**, **OrbStack**, **Lima**, **Colima**, **Rancher Desktop** (macOS/Linux) или **Docker CE** внутри WSL2 (Windows). DDEV сам Docker не устанавливает — провайдер нужен заранее.
> - **DDEV Foundation** — некоммерческая организация, сопровождающая разработку DDEV; проект существует на пожертвования спонсоров.
> - **Project (проект)** — единица работы DDEV: директория с кодом сайта плюс поддиректория `.ddev/`, описывающая его конфигурацию. У каждого проекта — свой набор контейнеров.
> - **Project type (тип проекта)** — предустановка для конкретной CMS/фреймворка (`drupal11`, `wordpress`, `laravel`, `craftcms` и т.д.) либо общий тип `php` для любого современного PHP- или статического HTML/JS-проекта без предположений о конфигурации.

> [!abstract]+ Контейнеры и сеть
> 
> - **`ddev-webserver`** — контейнер веб-сервера (один на проект), запускает **nginx** или **apache** и **php-fpm** — выполняет всю базовую работу PHP-интерпретирующего веб-сервера для одного сайта.
> - **`ddev-dbserver`** — контейнер базы данных (один на проект), управляет MariaDB/MySQL/PostgreSQL. Доступен из web-контейнера по хостнейму `db` либо явным именем `ddev-<projectname>-db`.
> - **`ddev-router`** — единственный на всю систему глобальный контейнер-**reverse proxy** (обратный прокси) на базе **Traefik**: принимает входящие HTTP/S-запросы, определяет по хостнейму нужный проект и направляет запрос в его `ddev-webserver`. Именованные URL вида `https://project.ddev.site` идут через router; `127.0.0.1`-адреса — напрямую в веб-контейнер.
> - **`ddev-ssh-agent`** — единственный глобальный контейнер, запускающий `ssh-agent` внутри Docker-сети, чтобы после `ddev auth ssh` все проекты могли использовать SSH-ключи хоста для исходящих запросов (приватный Composer-доступ, SCP на удалённый хост).
> - **Add-on service (сервис-надстройка)** — дополнительный контейнер для конкретного проекта: например, `phpmyadmin`, `solr`, `elasticsearch`, `memcached` — добавляется через **Add-ons** (см. ниже).
> - **`ddev.site`** — специальный домен, который DDEV использует для локальных URL проектов (`https://myproject.ddev.site`), автоматически резолвящийся через `ddev-router`.

> [!abstract]+ Конфигурация и файлы
> 
> - **`.ddev` директория (per-project)** — конфигурация конкретного проекта: `config.yaml` и связанные файлы. Живёт в корне проекта.
> - **`.ddev` директория (global)** — глобальная конфигурация DDEV, обычно `$HOME/.ddev` (может быть перенесена через `$DDEV_XDG_CONFIG_HOME`); хранит список всех проектов и настройки, действующие для всех.
> - **`config.yaml`** — главный конфигурационный файл проекта в формате YAML: тип проекта, docroot, версия PHP, тип и версия базы, дополнительные хостнеймы и т.д.
> - **`config.*.yaml`** — файлы **environmental overrides** — переопределения отдельных частей `config.yaml` для конкретного окружения.
> - **`docker-compose.*.yaml`** — пользовательские Docker Compose-файлы в `.ddev/`, добавляющие или переопределяющие сервисы проекта.
> - **Add-on (аддон / надстройка)** — устанавливаемый пакет, расширяющий проект: дополнительный сервис (например, Redis, Solr), набор кастомных команд или конфигурация под конкретную CMS. Устанавливается через `ddev add-on get`.
> - **Custom command (кастомная команда)** — пользовательская команда-скрипт, которая выполняется на хосте или внутри контейнера и доступна как `ddev <имя-команды>`.
> - **Hook (хук DDEV)** — команда, которая автоматически выполняется на определённом этапе жизненного цикла DDEV-команды (например, `post-start`, `pre-import-db`) — не путать с Drupal-хуками из отдельного конспекта.

> [!abstract]+ Ключевые команды
> 
> - **`ddev config`** — инициализация или изменение конфигурации проекта (создаёт/обновляет `.ddev/config.yaml`); DDEV пытается автоматически определить тип проекта и docroot.
> - **`ddev start`** / **`ddev stop`** — запуск/остановка контейнеров проекта.
> - **`ddev launch`** — открытие проекта в браузере.
> - **`ddev describe`** — вывод детальной информации о запущенном проекте (URL, статус контейнеров, версии и т.д.).
> - **`ddev exec`** — выполнение произвольной команды внутри контейнера (по умолчанию — web).
> - **`ddev ssh`** — интерактивный вход в Linux-окружение внутри контейнера.
> - **`ddev import-db`** / **`ddev import-files`** — импорт дампа базы данных / пользовательских файлов (например, `sites/default/files` у Drupal или `wp-content/uploads` у WordPress).
> - **`ddev snapshot`** — создание снапшота базы данных для быстрого отката.
> - **`ddev share`** — временный публичный доступ к локальному сайту для демонстрации другим.
> - **`ddev list`** — список всех запущенных проектов на машине.

---

## 🗺️ Карта конспекта

|#|Раздел|Что внутри|
|---|---|---|
|1|[[#🖥️ Установка и системные требования]]|Поддерживаемые ОС, Docker-провайдеры, WSL2-нюансы|
|2|[[#🚀 Старт проекта]]|`ddev config` → `start` → `launch`, CMS Quickstarts|
|3|[[#🏗️ Архитектура — контейнеры и файлы]]|Устройство контейнеров, структура `.ddev/`|
|4|[[#⚙️ Конфигурация проекта]]|`config.yaml`, ключевые опции, environmental overrides|
|5|[[#🧩 Расширение — Add-ons, Hooks, Custom Commands]]|Как расширять DDEV под свои нужды|
|6|[[#🛠️ Повседневные задачи]]|БД, отладка, Vite, устранение неполадок|
|7|[[#🌐 Хостинг и деплой]]|Интеграции с провайдерами хостинга|
|8|[[#🔗 Полезные ссылки]]|Ссылки на официальную документацию|

---

## 🖥️ Установка и системные требования

> [!quote] Определение 
> DDEV — открытый инструмент для запуска локальных окружений веб-разработки за минуты. Поддерживаемые окружения можно расширять, версионировать и расшаривать, получая преимущества Docker-workflow без опыта работы с Docker.

### Поддерживаемые платформы

|Платформа|Требования|Docker provider|
|---|---|---|
|**macOS**|Sonoma (14)+, ARM64 и AMD64, RAM 8GB, Storage 256GB|OrbStack / Lima / Docker Desktop / Rancher Desktop / Colima|
|**Windows WSL2**|RAM 8GB, Storage 256GB, рекомендуется Ubuntu-дистрибутив|Docker CE внутри WSL2 или Docker Desktop на стороне Windows|
|**Traditional Windows**|Любая современная редакция Windows Home/Pro|Docker Desktop с WSL2-backend|
|**Linux**|Большинство дистрибутивов, AMD64 и ARM64, RAM 8GB, Storage 256GB|Docker (нативно)|
|**GitHub Codespaces**|Только браузер и интернет — ничего устанавливать не нужно|встроен|

### Порядок установки

1. Проверить системные требования
2. Установить **Docker provider** (обязателен — DDEV работает поверх него)
3. Установить сам **DDEV**
4. Запустить первый проект

> [!warning] WSL2 — режимы сети 
> При использовании **"Mirrored" networking mode** в WSL2 нужно включить экспериментальную настройку `hostAddressLoopback=true` (через приложение "WSL Settings" или файл `C:\Users\<you>\.wslconfig`):
> 
> ```ini
> [wsl2]
> networkingMode=Mirrored
> 
> [experimental]
> hostAddressLoopback=true
> ```
> 
> Режим **"VirtioProxy"** поддерживается экспериментально и имеет известные ограничения (WSL2-дистрибутив может остаться без доступа в интернет — не будут работать Composer/npm). Если контейнеры не могут достучаться до хоста, IDE стоит запускать внутри WSL2 через WSLg: `ddev config global --xdebug-ide-location=wsl2`.

---
+
## 🚀 Старт проекта

> [!quote] Базовый workflow
> 
> 
> ```text
> cd <project-dir> → ddev config → ddev start → ddev launch
> ```

### Пошагово

1. Склонировать или создать код проекта
2. `cd` в директорию проекта и выполнить **`ddev config`** — инициализирует DDEV-проект (создаёт `.ddev/config.yaml`)
3. **`ddev start`** — поднимает контейнеры проекта
4. **`ddev launch`** — открывает проект в браузере

> [!tip] Интерактивный дашборд 
> Просто набрав `ddev` без аргументов, можно управлять всеми проектами через **интерактивный дашборд**.

DDEV пытается автоматически определить тип проекта и docroot. Если угадал неверно — используйте `ddev config` с флагами или отредактируйте `.ddev/config.yaml` напрямую. Проверить результат: `ddev describe`.

### Тип проекта `php`

Наиболее общий тип — подходит для любого современного PHP или статического HTML/JS-проекта без предположений о конфигурации; можно использовать и с CMS/фреймворком. Для подключения к базе данных из приложения: хост, пользователь, пароль и имя БД по умолчанию — все равны `db`.

### CMS Quickstarts — пример для Drupal

DDEV поддерживает готовые сценарии старта для десятков CMS/фреймворков (Drupal, WordPress, Laravel, Craft CMS, TYPO3, Magento, Symfony, Moodle и др.). Пример полного цикла для **Drupal 11**:

```bash
mkdir -p my-drupal11-site && cd my-drupal11-site
ddev config --project-type=drupal11 --docroot=web
ddev start
ddev composer create-project drupal/recommended-project
ddev composer require drush/drush
ddev drush site:install --account-name=admin --account-pass=admin -y
ddev launch
# автоматический вход:
ddev launch $(ddev drush uli)
```

Аналогичные quickstart-сценарии есть для **Drupal CMS** (официальный дистрибутив с рецептами: `ddev composer create-project drupal/cms` + `ddev composer drupal:recipe-unpack`), **Drupal 10**, **Drupal 6/7** и даже экспериментального **Drupal 12 (HEAD)**.

> [!note] Подробнее о Drupal 
> Все понятия — content type, bundle, hooks, Configuration Management и т.д. — разобраны в отдельном конспекте [[Drupal — конспект]].

---

## 🏗️ Архитектура — контейнеры и файлы

> [!quote] Как это работает 
> DDEV — Go-приложение, хранящее конфигурацию в файлах на рабочей станции. По этим "чертежам" оно монтирует файлы проекта в Docker-контейнеры, обеспечивающие работу локального окружения. DDEV сам пишет и использует **docker-compose**-файлы — деталь, о которой можно не задумываться, если вы не собираетесь определять собственные сервисы.

### Модель контейнеров

Проще всего представить DDEV как набор маленьких "сетевых компьютеров" (Docker-контейнеров), находящихся в отдельной от рабочей станции сети, но доступных из неё.

|Контейнер|Количество|Роль|
|---|---|---|
|**`ddev-webserver`**|один на проект|nginx/apache + php-fpm — обслуживает конкретный сайт|
|**`ddev-dbserver`**|один на проект|MariaDB/MySQL/PostgreSQL, доступен по хостнейму `db`|
|**`ddev-router`**|один глобально|reverse proxy на Traefik — маршрутизирует по хостнейму к нужному проекту|
|**`ddev-ssh-agent`**|один глобально|ssh-agent внутри Docker-сети для всех проектов после `ddev auth ssh`|
|**Add-on-сервисы**|по требованию, на проект|`phpmyadmin`, `solr`, `elasticsearch`, `memcached` и др.|

> [!tip] Как проекты общаются между собой 
> Хотя это не самый частый сценарий, разные DDEV-проекты **могут обмениваться данными друг с другом** (например, один сайт обращается к API другого локального сайта) — детали в официальном FAQ.

### Структура `.ddev/` проекта (основное)

| Файл/директория         | Назначение                                                                                         |
| ----------------------- | -------------------------------------------------------------------------------------------------- |
| `config.yaml`           | Главный конфигурационный файл проекта                                                              |
| `config.*.yaml`         | Environmental overrides — частичные переопределения `config.yaml`                                  |
| `commands/`             | Кастомные shell-команды проекта (host или container)                                               |
| `docker-compose.*.yaml` | Пользовательские compose-файлы, добавляющие/переопределяющие сервисы                               |
| `web-build/`            | Кастомный Dockerfile для web-контейнера                                                            |
| `db-build/`             | Кастомный Dockerfile для db-контейнера                                                             |
| `homeadditions/`        | Файлы, копируемые в домашнюю директорию контейнера при старте (`.bashrc`, `.ssh` и т.д.)           |
| `web-entrypoint.d/`     | Кастомные `*.sh`-скрипты, выполняемые при старте web-контейнера до запуска php-fpm                 |
| `db_snapshots/`         | Снапшоты БД от `ddev snapshot`                                                                     |
| `traefik/`              | Конфигурация `ddev-router` (Traefik)                                                               |
| `addon-metadata/`       | Метаданные установленных add-on-сервисов (для `ddev add-on list/remove`)                           |
| `mutagen/`              | `mutagen.yml` — переопределение конфигурации Mutagen (для производительности синхронизации файлов) |
|                         |                                                                                                    |

> [!warning] Скрытые файлы (`.` в начале имени) 
> Файлы, начинающиеся с точки (`.dbimageBuild`, `.ddev-docker-compose-base.yaml`, `.gitignore` и др.), **регенерируются при каждом `ddev start`** — их не стоит редактировать вручную.

### Глобальная директория `.ddev`

Одна на систему, по умолчанию `$HOME/.ddev` (порядок переопределения: `$DDEV_XDG_CONFIG_HOME/ddev` → `$HOME/.ddev` → `$HOME/.config/ddev` на Linux/WSL2). Содержит:

- **`global_config.yaml`** — глобальная конфигурация, применяемая ко всем проектам
- **`project_list.yaml`** — список всех известных DDEV-проектов на машине
- **`bin/`** — приватные исполняемые бинарники (`mutagen`, `docker-buildx`)
- **`commands/`** — глобальные кастомные команды, доступные во всех проектах (`db`, `host`, `web` — по подпапкам)
- **`homeadditions/`** — файлы, копируемые в домашнюю директорию **каждого** web-контейнера

---

## ⚙️ Конфигурация проекта

### Ключевые опции `config.yaml` (через `ddev config`)

|Опция|Назначение|Пример|
|---|---|---|
|`--project-type`|Тип проекта (CMS/фреймворк)|`drupal11`, `wordpress`, `laravel`, `php`|
|`--docroot`|Корневая директория для веб-сервера|`web`, `public`, `webroot`|
|`--php-version`|Версия PHP|`8.3`|
|`--database`|Тип и версия БД|`mysql:8.0`, `mariadb:10.11`|
|`--webserver-type`|Тип веб-сервера|`nginx-fpm` (по умолч.), `apache-fpm`, `generic`|
|`--omit-containers`|Не создавать указанные контейнеры|`db` (например, для SQLite-проектов)|
|`--upload-dirs`|Директории для пользовательских файлов (для `import-files`)|`sites/default/files` (Drupal), `wp-content/uploads` (WordPress)|
|`--additional-hostnames`|Дополнительные хосты, которые должен обслуживать проект|—|
|`--disable-settings-management`|Отключить автоматическую генерацию файлов настроек CMS|—|

### `webserver_type: generic`

Позволяет определить собственные веб-процессы и порты для проектов, не использующих стандартную связку `nginx-fpm`/`apache-fpm` — например, для Node.js-приложений (SvelteKit, Express) через `web_extra_daemons` и `web_extra_exposed_ports` в `.ddev/config.*.yaml`.

### Environmental overrides

Файлы `config.*.yaml` (например, `config.local.yaml`) позволяют переопределить часть настроек `config.yaml` для конкретного окружения без изменения основного файла — удобно для CI или личных настроек, не идущих в общий репозиторий.

---

## 🧩 Расширение — Add-ons, Hooks, Custom Commands

### Add-ons (аддоны)

Устанавливаемые пакеты, расширяющие проект дополнительными сервисами, командами или конфигурацией:

```bash
ddev add-on get ddev/ddev-opensearch            # добавить OpenSearch как сервис
ddev add-on get backdrop-ops/ddev-backdrop-bee  # добавить CLI-инструмент как набор команд
ddev add-on list --installed                    # список установленных
ddev add-on remove <name>                       # удалить
```

Каталог доступных аддонов — **Add-on Registry** (addons.ddev.com). Метаданные установленных аддонов хранятся в `.ddev/addon-metadata/`.

### Hooks (хуки DDEV)

Команды, автоматически выполняемые на определённых этапах жизненного цикла DDEV-команд, задаются в `.ddev/config.yaml`:

```yaml
hooks:
  post-start:
    - exec: composer install
  pre-import-db:
    - exec-host: echo "About to import database"
```

Частые точки: `pre-start`, `post-start`, `pre-import-db`, `post-import-db`, `pre-composer`, `post-composer` и др.

### Custom Commands (кастомные команды)

Собственные shell-скрипты, доступные как `ddev <имя>`. Размещаются в `.ddev/commands/<контекст>/` (например, `.ddev/commands/web/mycommand`) — контекст определяет, где выполняется команда: `db`, `host` или `web`. Глобальные кастомные команды — в `$HOME/.ddev/commands/`.

### Кастомизация Docker-образов

- **`web-build/Dockerfile`** и **`db-build/Dockerfile`** — надстройка над стандартными образами DDEV для web/db контейнеров (установка дополнительных пакетов, расширений PHP и т.д.)
- **Custom Docker Compose Services** — полное определение собственных сервисов через compose-файлы в `.ddev/`

---

## 🛠️ Повседневные задачи

### Работа с базой данных

```bash
ddev import-db --file=/path/to/db.sql.gz     # импорт дампа БД
ddev export-db --file=dump.sql.gz            # экспорт БД
ddev snapshot                                # снапшот для быстрого отката
ddev snapshot restore <name>                 # восстановление снапшота
```

Доступ через любой локальный клиент БД (TablePlus, Sequel Ace, DBeaver, HeidiSQL) — DDEV сообщает порт через `ddev describe`.

### Отладка и профилирование

- **Xdebug** — пошаговая отладка (step debugging), настраивается через IDE (PhpStorm, VS Code)
- **Blackfire**, **xhprof**, **Xdebug Profiling** — варианты профилирования производительности

### Vite Integration

Встроенная поддержка **Vite** для фронтенд-сборки (актуально, например, для Laravel-проектов с Vite как asset-бандлером по умолчанию с версии Laravel 9.19).

### Устранение неполадок

Официальная страница **Troubleshooting** покрывает типовые проблемы: занятые порты 80/443 другим локальным ПО, конфликты сети, проблемы с сертификатами. Диагностика: `ddev debug`, `ddev logs`.

### Работа офлайн

DDEV поддерживает режим работы **без подключения к интернету** (Using DDEV Offline) — полезно в самолёте, на закрытых объектах и т.п., при условии что образы уже скачаны локально.

---

## 🌐 Хостинг и деплой

### Sharing — временный доступ к сайту

```bash
ddev share
```

Открывает временный публичный URL к локальному проекту — удобно для демонстрации клиенту или тестирования вебхуков без деплоя.

### Интеграции с хостинг-провайдерами

DDEV предоставляет готовые интеграции (**providers**) для получения данных (БД, файлов) с продакшн/staging окружений напрямую в локальный DDEV-проект через `ddev pull`:

|Провайдер|Особенность|
|---|---|
|**Acquia**|Хостинг для Drupal|
|**Pantheon**|Хостинг для Drupal/WordPress|
|**Lagoon**|Kubernetes-based хостинг (amazee.io)|
|**Upsun Fixed / Platform.sh**|PaaS-хостинг|
|**Upsun Flex**|Новое поколение Upsun-платформы|

### Remote Docker Environments

DDEV может работать не только с локальным Docker, но и с **удалёнными Docker-окружениями** — полезно при ограниченных ресурсах локальной машины.

---

## 🔗 Полезные ссылки

- 📘 [Get Started with DDEV](https://docs.ddev.com/en/stable/) — официальная стартовая страница документации
- 🚀 [Starting a Project](https://docs.ddev.com/en/stable/users/project/) — базовый workflow
- 🎯 [CMS Quickstarts](https://docs.ddev.com/en/stable/users/quickstart/) — готовые сценарии для 30+ CMS/фреймворков
- 🏗️ [How DDEV Works](https://docs.ddev.com/en/stable/users/usage/architecture/) — архитектура контейнеров и файлов
- ⚙️ [Config Options](https://docs.ddev.com/en/stable/users/configuration/config/) — полный справочник опций `config.yaml`
- 📋 [Commands Reference](https://docs.ddev.com/en/stable/users/usage/commands/) — все команды `ddev`
- 🧩 [Using DDEV Add-ons](https://docs.ddev.com/en/stable/users/extend/using-add-ons/) · [Add-on Registry](https://addons.ddev.com)
- 🪝 [Hooks](https://docs.ddev.com/en/stable/users/configuration/hooks/)
- 🐛 [Step Debugging with Xdebug](https://docs.ddev.com/en/stable/users/debugging-profiling/step-debugging/)
- 🌐 [Hosting Provider Integrations](https://docs.ddev.com/en/stable/users/providers/)
- 💻 [Репозиторий на GitHub](https://github.com/ddev/ddev)
- 💬 [Discord-сообщество](https://ddev.com/s/discord)

> [!success] Готово 
> Конспект охватывает установку и системные требования, старт проекта (включая quickstart для Drupal), архитектуру контейнеров (`ddev-webserver`, `ddev-dbserver`, `ddev-router`, `ddev-ssh-agent`), конфигурацию `.ddev/`, расширение через Add-ons/Hooks/Custom Commands, повседневные задачи и интеграции с хостингом. Смотрите также связанный конспект [[Drupal — конспект]] — DDEV часто используется именно для локальной разработки на Drupal.