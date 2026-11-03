# 17 — Namespaces, Autoloading и Composer

## 1. Теория

### Пространства имён (Namespaces)
- Решают проблему коллизий имен классов, функций и констант в больших проектах.
- Объявляются в самом начале файла через `namespace Vendor\Package;`.
- Для обращения к классам используются:
  - Абсолютные пути (`\Vendor\Package\Class`).
  - Относительные пути (`SubPackage\Class`).
  - Псевдонимы через `use` (`use Vendor\Package\Class as MyClass;`).

### Автозагрузка (Autoloading) и Composer
- Вместо ручного подключения файлов через `include` / `require` используется стандарт PSR-4 (PHP Standard Recommendation).
- **Composer**: Стандартный пакетный менеджер для PHP. Управляет зависимостями, автозагрузкой классов (через `vendor/autoload.php` и карту классов/PSR-4) и версионированием.

---

## 2. Примеры кода

```json
// composer.json (PSR-4 autoloading configuration)
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

```php
<?php
declare(strict_types=1);

namespace App\Services;

class UserService {
    public function register(): void {
        echo "User registered.";
    }
}
```

---

## 3. Вопросы на собеседовании

1. **Что такое стандарт PSR-4 и как работает автозагрузка классов?**
   *Ответ*: PSR-4 связывает пространства имён с директориями файловой системы. Например, класс `App\Services\UserService` с префиксом `App\` в папке `src/` будет автоматически найден по пути `src/Services/UserService.php` без ручного подключения файлов.
2. **В чем разница между `require`, `include`, `require_once` и `include_once`?**
   *Ответ*: `include` выдает предупреждение (`E_WARNING`) при неудаче и продолжает выполнение, а `require` генерирует фатальную ошибку (`E_ERROR`) и останавливает скрипт. Суффикс `_once` проверяет, был ли файл уже подключен ранее, предотвращая повторное объявление классов/функций.
3. **Для чего нужны composer.json и composer.lock?**
   *Ответ*: `composer.json` описывает требования проекта и диапазоны версий пакетов. `composer.lock` фиксирует точные версии всех установленных пакетов (включая транзитивные зависимости), гарантируя одинаковое окружение у всех разработчиков и на production.
