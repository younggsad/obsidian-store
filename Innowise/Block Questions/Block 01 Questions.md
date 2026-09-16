---
custom-width: 80
---
**1. Question (EN):** How is DDEV different from bare Docker Compose, which you would write manually for a Drupal project?

**Перевод (RU):** Чем DDEV отличается от голого Docker Compose для проекта на Drupal?

**Answer (EN):** Docker Compose only starts containers — you write and maintain everything yourself. DDEV adds ready-made project presets, automatic HTTPS URLs (`*.ddev.site`), and simple commands (`ddev import-db`, `ddev xdebug`) so the whole team gets an identical environment with one config file.

**Перевод (RU):** Docker Compose лишь запускает контейнеры — всё остальное пишете сами. DDEV добавляет готовые пресеты проектов, автоматические HTTPS-адреса и простые команды, так что вся команда получает одинаковое окружение из одного конфиг-файла.

---

**2. Question (EN):** Why is Drupal deployed via `composer create-project` rather than downloading the core archive?

**Перевод (RU):** Почему Drupal разворачивают через `composer create-project`, а не скачиванием архива ядра?

**Answer (EN):** An archive is a static snapshot with no dependency tracking. Composer manages core, modules, and libraries as a proper dependency graph with a lock file, so the exact same build can be reproduced on any machine.

**Перевод (RU):** Архив — это статичный снимок без учёта зависимостей. Composer управляет ядром, модулями и библиотеками как графом зависимостей с lock-файлом, поэтому точно такую же сборку можно воспроизвести на любой машине.

---

**3. Question (EN):** What's the difference between `composer.json` and `composer.lock`, and why commit both?

**Перевод (RU):** В чём разница между `composer.json` и `composer.lock`, и почему коммитить оба?

**Answer (EN):** `composer.json` states what you _want_ (version ranges); `composer.lock` records exactly what was _installed_ (precise versions). Without the lock file, different machines could get different compatible versions — committing both makes installs identical everywhere.

**Перевод (RU):** `composer.json` описывает, что вы _хотите_ (диапазоны версий); `composer.lock` фиксирует, что _реально установлено_ (точные версии). Без lock-файла разные машины могли бы получить разные совместимые версии — коммит обоих делает установку одинаковой везде.

---

**4. Question (EN):** Why install with the Minimal profile instead of Standard?

**Перевод (RU):** Почему устанавливать с профилем Minimal, а не Standard?

**Answer (EN):** Standard pre-installs demo content types, views, and modules nobody asked for — extra noise to clean up later. Minimal starts clean, so everything in the config was deliberately built by the team.

**Перевод (RU):** Standard предустанавливает демо-типы контента, views и модули, которые никто не просил — лишний мусор для последующей чистки. Minimal даёт чистый старт, поэтому всё в конфигурации осознанно создано командой.

---

**5. Question (EN):** Why should `/web/core`, `/web/modules/contrib`, and `/vendor/` be in `.gitignore`?

**Перевод (RU):** Почему `/web/core`, `/web/modules/contrib` и `/vendor/` должны быть в `.gitignore`?

**Answer (EN):** They're third-party code fully reproducible from `composer.lock`. Committing them doubles storage, bloats the repo, and causes huge merge conflicts on every update.

**Перевод (RU):** Это сторонний код, полностью воспроизводимый из `composer.lock`. Коммит их удваивает хранение, раздувает репозиторий и вызывает огромные конфликты слияния при каждом обновлении.

---

**6. Question (EN):** Why keep `settings.local.php` out of git but commit `settings.php`?

**Перевод (RU):** Почему `settings.local.php` не в git, а `settings.php` коммитится?

**Answer (EN):** `settings.php` holds shared config, same everywhere. `settings.local.php` holds machine-specific values and secrets (DB passwords) — committing it would leak credentials or break other developers' setups.

**Перевод (RU):** `settings.php` содержит общую конфигурацию, одинаковую везде. `settings.local.php` — значения конкретной машины и секреты (пароли БД) — коммит утёк бы учётные данные или сломал бы настройки других разработчиков.

---

**7. Question (EN):** How does `debug` mode differ from `develop`/`coverage`/`profile`, and why is `debug` needed here?

**Перевод (RU):** Чем `debug` отличается от `develop`/`coverage`/`profile`, и почему нужен именно `debug`?

**Answer (EN):** `develop` improves error output, `coverage` measures test coverage, `profile` measures performance — none let you interact with running code. Only `debug` connects to the IDE for live breakpoints and step-through inspection.

**Перевод (RU):** `develop` улучшает вывод ошибок, `coverage` измеряет покрытие тестами, `profile` — производительность — ни один не даёт взаимодействовать с работающим кодом. Только `debug` подключается к IDE для точек останова и пошагового просмотра.

---

**8. Question (EN):** Why is path mapping needed in PhpStorm, and what breaks if it's wrong?

**Перевод (RU):** Зачем нужен path mapping в PhpStorm, и что ломается при неверной настройке?

**Answer (EN):** PHP runs in a container with different file paths than your host disk. Path mapping tells the IDE how to translate one to the other. If it's wrong, Xdebug still connects, but breakpoints silently never trigger.

**Перевод (RU):** PHP работает в контейнере с путями, отличными от диска хоста. Path mapping говорит IDE, как переводить одно в другое. При ошибке Xdebug всё равно подключается, но точки останова молча никогда не срабатывают.

---

**9. Question (EN):** Why is step debugging better than `var_dump()`/logs for complex logic?

**Перевод (RU):** Почему пошаговая отладка лучше `var_dump()`/логов для сложной логики?

**Answer (EN):** Dumps require guessing where to look in advance and mean slow trial-and-error. Step debugging lets you pause anywhere, inspect any variable live, and follow the actual execution path — much faster for logic you don't yet understand.

**Перевод (RU):** Дампы требуют заранее угадать, куда смотреть, и означают медленные пробы и ошибки. Пошаговая отладка позволяет остановиться где угодно, осмотреть любую переменную вживую и проследить реальный путь выполнения — намного быстрее для непонятной логики.

---

**10. Question (EN):** How does Xdebug's connection trigger differ between a web request and a Drush command?

**Перевод (RU):** Чем отличается триггер подключения Xdebug для веб-запроса и команды Drush?

**Answer (EN):** A web request triggers via a cookie or URL parameter. A Drush command is a CLI process with no browser — the trigger must come from an environment variable (`XDEBUG_SESSION` or `XDEBUG_MODE`) set before running the command.

**Перевод (RU):** Веб-запрос триггерится через cookie или параметр URL. Команда Drush — это CLI-процесс без браузера — триггер должен приходить из переменной окружения, заданной перед запуском команды.

---

**11. Question (EN):** Why must debugging be toggled and not always on?

**Перевод (RU):** Почему отладка должна переключаться, а не быть всегда включена?

**Answer (EN):** Xdebug adds overhead to every request and would constantly try connecting to an IDE that isn't listening, slowing things down and cluttering logs. Toggling it keeps everyday work fast.

**Перевод (RU):** Xdebug добавляет накладные расходы к каждому запросу и постоянно пытался бы подключиться к неслушающей IDE, замедляя работу и захламляя логи. Переключение сохраняет повседневную работу быстрой.

---

**12. Question (EN):** How are PHPCS and PHPStan fundamentally different?

**Перевод (RU):** Чем принципиально различаются PHPCS и PHPStan?

**Answer (EN):** PHPCS checks **style** (formatting, naming) — it doesn't understand logic. PHPStan checks **correctness** (types, undefined methods) — it doesn't care about formatting. Together they cover form and substance; alone, each misses half the problems.

**Перевод (RU):** PHPCS проверяет **стиль** (форматирование, именование) — не понимает логику. PHPStan проверяет **корректность** (типы, несуществующие методы) — не заботится о форматировании. Вместе они покрывают форму и суть; по отдельности каждый упускает половину проблем.

---

**13. Question (EN):** What's the difference between `Drupal` and `DrupalPractice` standards?

**Перевод (RU):** В чём разница между стандартами `Drupal` и `DrupalPractice`?

**Answer (EN):** `Drupal` is the mandatory formatting standard. `DrupalPractice` is advisory — it flags discouraged patterns and likely mistakes that aren't strict style violations. Using both catches both bad formatting and bad practices.

**Перевод (RU):** `Drupal` — обязательный стандарт форматирования. `DrupalPractice` — рекомендательный: отмечает нежелательные паттерны и вероятные ошибки, не являющиеся строгими нарушениями стиля. Оба вместе ловят и плохое форматирование, и плохие практики.

---

**14. Question (EN):** Why does GrumPHP check only modified custom files, not the whole tree?

**Перевод (RU):** Почему GrumPHP проверяет только изменённые кастомные файлы, а не всё дерево?

**Answer (EN):** Core and contrib aren't the team's code — checking them is pointless noise. Scanning everything would also make every commit painfully slow.

**Перевод (RU):** Core и contrib — не код команды, проверять их бессмысленный шум. К тому же сканирование всего сделало бы каждый коммит мучительно медленным.

---

**15. Question (EN):** Why use GrumPHP instead of running PHPCS/PHPStan manually?

**Перевод (RU):** Зачем GrumPHP, если PHPCS/PHPStan можно запускать вручную?

**Answer (EN):** Manual runs easy to forget. GrumPHP makes checks automatic and mandatory for every commit, so nothing is accidentally skipped.

**Перевод (RU):** Ручной запуск легко забыть. GrumPHP делает проверки автоматическими и обязательными для каждого коммита, ничего случайно не пропускается.

---

**16. Question (EN):** What are the trade-offs of bypassing a hook with `--no-verify`?

**Перевод (RU):** Какие компромиссы у обхода хука через `--no-verify`?

**Answer (EN):** It's useful for genuine emergencies (hotfixes, false positives), but it can be abused to routinely skip quality checks. A server-side CI check is the backstop that catches anything bypassed locally.

**Перевод (RU):** Полезно для настоящих экстренных случаев (хотфиксы, ложные срабатывания), но может использоваться для рутинного обхода проверок. CI-проверка на сервере — страховка, ловящая всё пропущенное локально.

---

**17. Question (EN):** Why must the tooling config be reproducible via `composer install`, not set up manually?

**Перевод (RU):** Почему конфигурация инструментов должна воспроизводиться через `composer install`, а не настраиваться вручную?

**Answer (EN):** A manual local setup is invisible to new developers and CI — they'd have no way to know the exact rules or versions. Everything coming from committed files guarantees the same checks run identically everywhere.

**Перевод (RU):** Ручная локальная настройка невидима для новых разработчиков и CI — они не узнают точные правила или версии. Всё, что исходит из закоммиченных файлов, гарантирует одинаковые проверки везде.

---

**18. Question (EN):** Why is Drupal config synced via files (`cex`/`cim`) instead of a direct database copy?

**Перевод (RU):** Почему конфигурация Drupal синхронизируется через файлы, а не прямым переносом БД?

**Answer (EN):** A DB copy would overwrite content along with config — usually unwanted. Files are diffable and reviewable in git, like normal code changes, and don't require direct DB access between environments.

**Перевод (RU):** Копия БД перезаписала бы контент вместе с конфигурацией — обычно нежелательно. Файлы диффятся и рецензируются в git, как обычные изменения кода, и не требуют прямого доступа к БД между окружениями.

---

**19. Question (EN):** How is configuration different from content in Drupal, and why does that matter?

**Перевод (RU):** Чем конфигурация отличается от контента в Drupal, и почему это важно?

**Answer (EN):** Configuration is the site's structure (same everywhere); content is actual data (unique per environment). This split ensures `cim` can safely rewrite structure without ever touching or deleting real content.

**Перевод (RU):** Конфигурация — это структура сайта (одинаковая везде); контент — реальные данные (уникальные в каждом окружении). Это разделение гарантирует, что `cim` может безопасно переписать структуру, никогда не трогая реальный контент.

---

**20. Question (EN):** What does "no differences" on the configuration sync page mean?

**Перевод (RU):** Что значит «нет различий» на странице синхронизации конфигурации?

**Answer (EN):** It means the database's active configuration exactly matches the YAML files in the sync folder — the import fully succeeded, nothing is left unsynced.

**Перевод (RU):** Это значит, что активная конфигурация в базе данных точно совпадает с YAML-файлами папки синхронизации — импорт полностью удался, ничего не осталось несинхронизированным.

---

**21. Question (EN):** Why shouldn't the config sync folder be inside `sites/default/files`?

**Перевод (RU):** Почему папка синхронизации конфигурации не должна быть в `sites/default/files`?

**Answer (EN):** That folder is publicly accessible over the web. The config folder reveals the site's entire structure — anyone could download it and use it for reconnaissance.

**Перевод (RU):** Эта папка публично доступна по вебу. Папка конфигурации раскрывает всю структуру сайта — кто угодно мог бы скачать её для разведки.

---

**22. Question (EN):** What problem needs Config Split/Config Ignore, and why hasn't it appeared here?

**Перевод (RU):** Какая проблема требует Config Split/Config Ignore, и почему она пока не возникла?

**Answer (EN):** They're needed when config must legitimately differ per environment, or part of it is edited live in production. This task is still a simple single-environment pipeline, so that need hasn't arisen yet.

**Перевод (RU):** Они нужны, когда конфигурация должна законно отличаться по окружениям, или часть её редактируется прямо на проде. Эта задача — пока простой пайплайн одного окружения, поэтому такая необходимость ещё не возникла.

---

**23. Question (EN):** Why `git pull` then `cim`, not edit UI first then commit without exporting?

**Перевод (RU):** Почему сначала `git pull`, потом `cim`, а не сначала UI, потом коммит без экспорта?

**Answer (EN):** Git is the source of truth — pull brings the intended state, `cim` applies it. UI changes only exist in the database until `cex` runs, so committing without exporting saves nothing new at all.

**Перевод (RU):** Git — источник истины: pull приносит нужное состояние, `cim` его применяет. Изменения UI существуют только в БД, пока не выполнен `cex`, поэтому коммит без экспорта не сохранит ничего нового.

---

**24. Question (EN):** What happens if a developer forgets to run `config:export` before committing?

**Перевод (RU):** Что будет, если разработчик забудет выполнить `config:export` перед коммитом?

**Answer (EN):** The change stays only in that local database, invisible to git and the team. Worse, the next `cim` will silently overwrite and lose it, since the database is forced to match the (outdated) YAML files.

**Перевод (RU):** Изменение остаётся только в локальной базе, невидимое для git и команды. Хуже того, следующий `cim` молча перезапишет и потеряет его, так как база принудительно приводится в соответствие (устаревшим) YAML-файлам.