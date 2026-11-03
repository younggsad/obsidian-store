---
custom-width: 80
---
---
## Оглавление

1. [[#1. Как работает PHP]]
2. [[#2. Типы, переменные, константы]]
3. [[#3. Операторы, сравнение, приведение типов]]
4. [[#4. Строки]]
5. [[#5. Массивы]]
6. [[#6. Функции, замыкания, генераторы]]
7. [[#7. Типизация]]
8. [[#8. ООП]]
9. [[#9. Namespace, Composer, PSR]]
10. [[#10. Ошибки и исключения]]
11. [[#11. Память: ссылки, copy-on-write, GC]]
12. [[#12. HTTP, сессии, cookies]]
13. [[#13. Файлы и include]]
14. [[#14. PDO, транзакции, SQL]]
15. [[#15. Безопасность]]
16. [[#16. Дата, regex, JSON]]
17. [[#17. SOLID, DI, паттерны]]
18. [[#18. SPL, итераторы, конкурентность]]
19. [[#19. Производительность и PHP-FPM]]
20. [[#20. Тестирование]]
21. [[#21. Версии PHP 7.x–8.x]]
22. [[#22. Фреймворки и CMS]]
23. [[#23. Задачи с кодом]]
24. [[#24. Что выведет код]]
25. [[#25. Сценарные вопросы]]
26. [[#26. Блиц-вопросы]]
27. [[#27. Шпаргалка]]

---

## 1. Как работает PHP

> [!abstract] Главное за 30 секунд
> 
> - PHP компилируется в **опкоды** и исполняется виртуальной машиной Zend. «Построчная интерпретация» — устаревшее представление.
> - **OPcache** хранит опкоды в разделяемой памяти: файл не компилируется заново на каждый запрос.
> - **Shared nothing**: каждый запрос стартует с чистого состояния.
> - В проде: Nginx + **PHP-FPM**, OPcache включён.

### Понятия

> [!abstract] Zend Engine
> 
> - **Что это:** ядро языка.
> - **Как работает:** исходник → лексер (токены) → парсер (AST) → компилятор (опкоды) → виртуальная машина исполняет опкоды.
> - **Где проявляется:** любой запуск PHP; от этого зависит скорость старта и смысл OPcache.

> [!abstract] OPcache
> 
> - **Что это:** расширение, кэширующее опкоды в разделяемой памяти.
> - **Как настраивается:** `opcache.enable`, `opcache.memory_consumption`, `opcache.validate_timestamps=0` (на проде, с перезапуском FPM при деплое), `opcache.preload`.
> - **Где применяется:** обязателен на продакшене.

> [!abstract] SAPI
> 
> - **Что это:** слой между PHP и внешним миром.
> - **Варианты:** `CLI` (консоль, воркеры), `FPM` (Nginx/Apache по FastCGI), `mod_php` (внутри Apache, устарел), `embed`.

> [!abstract] Shared nothing
> 
> - **Что это:** переменные, объекты и соединения не живут между запросами.
> - **Плюсы:** нет утечек состояния между пользователями, легко масштабировать горизонтально.
> - **Минусы:** приложение каждый раз поднимается заново; общее состояние выносят в БД, Redis, APCu.
> - **Исключение:** Swoole, RoadRunner, FrankenPHP держат приложение в памяти — там статика и синглтоны **живут между запросами**.

### Команды

```bash
php -v                      # версия
php -m                      # расширения
php --ini                   # какие php.ini загружены
php -S localhost:8000       # встроенный dev-сервер (только для разработки)
php -l file.php             # проверка синтаксиса
```

### Вопросы

> [!question] Чем mod_php, CGI и PHP-FPM отличаются?
> 
> |Вариант|Суть|Минус|
> |---|---|---|
> |`mod_php`|PHP внутри процесса Apache|каждый процесс тянет PHP, даже для статики|
> |CGI|новый процесс на запрос|очень медленно|
> |**PHP-FPM**|отдельный пул воркеров, общение по FastCGI|— (стандарт: пулы, изоляция, graceful reload)|

> [!question] Почему PHP «медленный на старте, быстрый после прогрева»?
> 
> Без OPcache каждый запрос проходит весь путь «лексер → парсер → компилятор → VM». С OPcache первые три шага пропускаются.
> 
> > [!warning] Подвох Не говорите «PHP интерпретируется построчно». Корректно: компилируется в опкоды, исполняется VM, OPcache убирает повторную компиляцию.

---

## 2. Типы, переменные, константы

> [!abstract] Главное за 30 секунд
> 
> - Скаляры: `int`, `float`, `string`, `bool`. Составные: `array`, `object`. Специальные: `null`, `resource`.
> - `float` неточен: `0.1 + 0.2 != 0.3`. Деньги — в копейках (`int`) или `bcmath`.
> - `isset` ≠ `empty` ≠ `is_null`; `empty("0") === true`.

### Понятия

> [!abstract] Константа: `const` и `define()`
> 
> - **Что это:** значение, которое нельзя изменить после объявления.
> - **`const`:** разрешается при компиляции, работает на верхнем уровне и в классах, учитывает namespace.
> - **`define()`:** вызов в рантайме, имя можно собрать динамически, можно внутри `if`.
> - **Где применяется:** настройки, коды статусов. В современном коде — `const`.

> [!abstract] Область видимости и `static`
> 
> - **Что это:** переменная внутри функции локальная. Глобальные — через `global` / `$GLOBALS` (плохая практика).
> - **`static` в функции:** значение сохраняется между вызовами.
> - **Где применяется:** счётчики, простое кэширование результата внутри функции.

### Falsy-значения и проверки

|Значение|В `bool`|
|---|:-:|
|`false`, `0`, `0.0`, `""`, `"0"`, `[]`, `null`|`false`|
|всё остальное, включая `"0.0"`, `" "`, `"false"`, пустой объект|`true`|

|Проверка|Когда `true`|
|---|---|
|`isset($x)`|существует и **не** `null`|
|`empty($x)`|не существует или falsy|
|`is_null($x)`|строго `null` (на несуществующей переменной — warning)|
|`array_key_exists('k', $a)`|ключ есть, **даже если значение `null`**|

### Код

```php
<?php
declare(strict_types=1);

function counter(): int
{
    static $n = 0;          // сохраняется между вызовами
    return ++$n;
}
counter(); counter();
echo counter();             // 3

var_dump(abs((0.1 + 0.2) - 0.3) < PHP_FLOAT_EPSILON);  // true: сравнение с допуском
var_dump(PHP_INT_MAX + 1);                              // float: переполнение int

$a = ['k' => null];
var_dump(isset($a['k']));                // false
var_dump(array_key_exists('k', $a));     // true
```

### Вопросы

> [!question] В чём разница между `isset`, `empty` и `is_null`?
> 
> - `isset` — переменная определена и не `null`; без предупреждений.
>     
> - `empty` — эквивалент `!isset($x) || !$x`.
>     
> - `is_null` — только проверка на `null`.
>     
> 
> > [!warning] Подвох `empty("0") === true`. Если пользователь введёт «0», `empty($_POST['qty'])` решит, что поле пустое. Для строк надёжнее `$v === ''`.

---

## 3. Операторы, сравнение, приведение типов

> [!abstract] Главное за 30 секунд
> 
> - По умолчанию `===`. В PHP 8 `0 == "abc"` → `false` (в PHP 7 было `true`).
> - `??` = «существует и не `null`»; `?:` = «истинно».
> - `match` строгий, без fall-through, бросает `UnhandledMatchError`.

### Понятия

> [!abstract] Оператор `??` и `??=`
> 
> - **Что это:** возвращает левый операнд, если он существует и не `null`, иначе правый.
> - **Где применяется:** значения по умолчанию для `$_GET['x'] ?? 1`, без notice.

> [!abstract] `match` (8.0)
> 
> - **Что это:** выражение с **строгим** сравнением, возвращает значение.
> - **Отличия от `switch`:** нет `break` и fall-through, нет нестрогого `==`, без совпадения — `UnhandledMatchError`.

> [!abstract] Null-safe `?->` (8.0)
> 
> - **Что это:** если слева `null`, вся цепочка вернёт `null`.
> - **Где применяется:** необязательные связи (`$user?->profile?->name`).

### Таблицы

|Выражение (PHP 8)|Результат|
|---|:-:|
|`0 == "abc"`|`false`|
|`"1" == "01"`|`true`|
|`"10" == "1e1"`|`true`|
|`null == false` / `null == 0` / `[] == false`|`true`|
|`"0" == false`|`true`|
|`"abc" <=> "abd"`|`-1`|

|Приведение|Результат|
|---|---|
|`(int) "12abc"`|`12` (числовой префикс)|
|`(int) "abc"`|`0`|
|`(int) 12.9` / `(int) -12.9`|`12` / `-12` (усечение к нулю)|
|`(bool) "false"`|`true`|
|`(string) false`|`""`|
|`(array) "a"`|`["a"]`|

### Управляющие конструкции

- `if / elseif / else`, `switch` (нестрогое сравнение), `match`.
- Циклы: `for`, `while`, `do-while`, `foreach` (по значению, `key => value`, по ссылке).
- `break 2` / `continue 2` — выход на N уровней вложенности.
- Альтернативный синтаксис для шаблонов: `if (...): ... endif;`, `foreach (...): ... endforeach;`.
- `exit` / `die` завершает скрипт, но выполняются shutdown-функции и деструкторы.

### Код

```php
$user = null;
echo $user?->profile?->name ?? 'Гость';      // Гость

$config['debug'] ??= false;                  // присвоит, если ключа нет

$text = match (true) {
    $status >= 500 => 'Server error',
    $status >= 400 => 'Client error',
    default        => 'OK',
};

usort($users, fn($a, $b) => [$a['age'], $a['name']] <=> [$b['age'], $b['name']]);

$x = true and false;     // $x === true: and/or ниже присваивания
echo -7 % 3;             // -1: знак берётся от делимого
echo intdiv(7, 2);       // 3
```

### Вопросы

> [!question] Чем `==` отличается от `===` и в чём опасность `==`?
> 
> - `==` приводит типы, `===` требует одинаковых типа и значения.
> - Ловушки: `"0" == false`, `null == []`, `"1e3" == "1000"`.
> - Уязвимость «magic hashes»: хэши `"0e123..."` и `"0e456..."` оба равны `0`. Секреты сравнивать через `hash_equals()`.

> [!question] Чем `??` отличается от `?:`?
> 
> - `??` проверяет «существует и не `null`» (как `isset`), не выдаёт notice.
> - `?:` проверяет истинность: `0`, `''`, `'0'` уйдут в правую часть.
> - `$_GET['page'] ?: 1` при отсутствии ключа даст предупреждение, при `page=0` вернёт 1.

---

## 4. Строки

> [!abstract] Главное за 30 секунд
> 
> - Строка — **последовательность байтов**. `strlen('привет') === 12`. Для UTF-8: `mb_*`.
> - `strpos` вернёт `0` для совпадения в начале: сравнивать только с `=== false`. В PHP 8 есть `str_contains`.
> - Экранировать на выводе: `htmlspecialchars`.

### Понятия

> [!abstract] Кавычки, heredoc, nowdoc
> 
> - **Одинарные:** без интерполяции. **Двойные:** интерполяция `"Hi $name"`, `"{$user['name']}"`, escape `\n`.
> - **Heredoc** `<<<EOT` ведёт себя как двойные, **nowdoc** `<<<'EOT'` как одинарные. С 7.3 закрывающий маркер можно отступать.
> - **Где применяется:** SQL, HTML-фрагменты, многострочные тексты.

> [!abstract] `mbstring`
> 
> - **Что это:** набор функций для многобайтных кодировок: `mb_strlen`, `mb_substr`, `mb_strtoupper`, `mb_str_split`.
> - **Где применяется:** любой текст не на латинице. Для regex по Unicode нужен модификатор `/u`.

### Функции по задачам

|Задача|Функции|
|---|---|
|Поиск|`strpos`, `str_contains`, `str_starts_with`, `str_ends_with` (8.0)|
|Разбиение и склейка|`explode`, `implode`, `str_split`, `trim` / `ltrim` / `rtrim`|
|Замена|`str_replace`, `substr_replace`, `strtr`, `preg_replace`|
|Форматирование|`sprintf`, `printf`, `number_format`, `str_pad`|
|Экранирование|`htmlspecialchars`, `urlencode` / `rawurlencode`, `http_build_query`, `escapeshellarg`|
|Сравнение|`strcmp`, `strnatcmp` (естественный порядок), `hash_equals` (секреты)|

### Код

```php
$name = 'Анна';
echo strlen($name);       // 8 (байты)
echo mb_strlen($name);    // 4 (символы)

if (strpos('abc', 'a') === false) { /* не найдено */ }   // 0 !== false
var_dump(str_contains('Hello', 'ell'));                  // true

printf("%05.1f|%-6s|%'*8d\n", 3.14159, 'ab', 42);

$s = 'abc'; echo $s[0], $s[-1];    // a c (отрицательные индексы с 7.1)

$text = <<<TXT
  Привет, {$name}!
  TXT;
```

### Вопросы

> [!question] Почему `strlen('привет')` не равно 6?
> 
> - Функции без `mb_` работают с байтами, кириллица в UTF-8 — 2 байта на символ.
> - `substr` на многобайтной строке может разрезать символ пополам.
> - `strtoupper('привет')` не изменит кириллицу, `mb_strtoupper` изменит.

---

## 5. Массивы

> [!abstract] Главное за 30 секунд
> 
> - Массив — **упорядоченная хеш-таблица**: один тип работает как список, словарь, стек, очередь, множество.
> - `+` сохраняет левые ключи; `array_merge` перенумеровывает числовые и перезаписывает строковые.
> - `in_array(..., true)`; `array_filter` сохраняет ключи; сортировка стабильна с 8.0.

### Понятия

> [!abstract] Ключи массива
> 
> - **Правила:** только `int` или `string`. `"5"` → `5`; `"05"` остаётся строкой; `float` усекается; `bool` → `0/1`; `null` → `""`.
> - **Порядок** элементов — порядок вставки.

> [!abstract] Copy-on-write
> 
> - **Что это:** `$b = $a` не копирует данные, пока один из массивов не изменён.
> - **Где проявляется:** передача больших массивов в функции дёшева.

### Функции по задачам

|Задача|Функции|Нюанс|
|---|---|---|
|Сортировка по значениям|`sort`, `rsort`|ключи сбрасываются|
|С сохранением ключей|`asort`, `arsort`, `ksort`||
|Своя функция|`usort`, `uasort`, `uksort`|используйте `<=>`|
|Поиск|`in_array($v, $a, true)`, `array_search`, `array_key_exists`|O(n)|
|Преобразование|`array_map`, `array_filter`, `array_reduce`, `array_column`|`array_filter` сохраняет ключи|
|Срез|`array_slice` (не меняет), `array_splice` (меняет)||
|Очередь / стек|`array_push` / `pop`, `array_shift` / `unshift`|`shift` — O(n)|
|Множества|`array_unique`, `array_diff`, `array_intersect`, `array_flip`||

Сложность: доступ и вставка по ключу — O(1), `in_array` — O(n).

### Код

```php
$users = [
    ['id' => 1, 'name' => 'Анна',  'age' => 30],
    ['id' => 2, 'name' => 'Борис', 'age' => 25],
];

$names  = array_column($users, 'name', 'id');                // [1 => 'Анна', 2 => 'Борис']
$adults = array_values(array_filter($users, fn($u) => $u['age'] > 26));
$total  = array_reduce($users, fn($c, $u) => $c + $u['age'], 0);

['id' => $id] = $users[0];
var_dump([...[1, 2], ...[3, 4]]);                            // [1,2,3,4]

var_dump(['a' => 1] + ['a' => 2, 'b' => 3]);                 // ['a'=>1,'b'=>3]
var_dump([1, 2] + [3, 4, 5]);                                // [1, 2, 5]
```

### Вопросы

> [!question] Чем `array_merge` отличается от `+`?
> 
> - `array_merge`: числовые ключи **перенумеровываются**, у строковых побеждает **правый** массив.
> - `+`: ключи сохраняются, побеждает **левый**, добавляются только отсутствующие ключи.
> - `[1, 2] + [3, 4, 5]` → `[1, 2, 5]`; `array_merge([1,2],[3,4,5])` → `[1,2,3,4,5]`.

---

## 6. Функции, замыкания, генераторы

> [!abstract] Главное за 30 секунд
> 
> - Аргументы по значению; объект передаётся как **handle** (идентификатор).
> - `function () use ($x)` — явный захват (можно по ссылке `&$x`), `fn` — автозахват **по значению**, одно выражение.
> - Генератор (`yield`) — ленивая последовательность, постоянный расход памяти.

### Понятия

> [!abstract] Замыкание (Closure)
> 
> - **Что это:** анонимная функция, запоминающая значения переменных из места создания.
> - **Как реализуется:** `function () use ($x) { ... }` — захват по значению; `use (&$x)` — по ссылке. Это объект класса `Closure`; есть `bind`, `bindTo`, `call`.
> - **Где применяется:** колбэки (`array_map`, `usort`), middleware, обработчики событий, фабрики функций, ленивые вычисления.

> [!abstract] Стрелочная функция `fn` (7.4)
> 
> - **Что это:** короткое замыкание из одного выражения.
> - **Как реализуется:** `fn($x) => $x * $k`, внешние переменные захватываются автоматически **по значению**.
> - **Где применяется:** короткие колбэки. Изменить внешнюю переменную из `fn` нельзя.

> [!abstract] Генератор
> 
> - **Что это:** функция с `yield`, возвращающая `Generator` (итератор); значения вычисляются по одному.
> - **Как реализуется:** `yield $v`, `yield $k => $v`, `yield from`, `send()`, `getReturn()`.
> - **Где применяется:** чтение файла построчно, выборка из БД порциями, конвейеры обработки, бесконечные последовательности.
> - **Ограничение:** после начала итерации генератор нельзя перемотать.

> [!abstract] Callable и first-class callable (8.1)
> 
> - **Что это:** всё, что можно вызвать: имя функции строкой, `[$obj, 'method']`, `'Class::method'`, `Closure`, объект с `__invoke`.
> - **First-class callable:** `strlen(...)`, `$obj->method(...)`, `Foo::bar(...)` — создаёт `Closure` из функции.

### Код

```php
function createCounter(): Closure
{
    $count = 0;
    return function () use (&$count): int {   // захват по ссылке
        return ++$count;
    };
}
$c = createCounter(); $c(); $c();
echo $c();                                    // 3

$k = 10;
$mul = fn(int $x): int => $x * $k;            // автозахват $k по значению
echo implode(',', array_map($mul, [1, 2, 3])); // 10,20,30

function sum(int ...$nums): int { return array_sum($nums); }   // variadic
echo sum(...[1, 2, 3]);                                          // 6

function connect(string $host, int $port = 3306, bool $ssl = false) {}
connect('db', ssl: true);                     // именованные аргументы (8.0)

$lengths = array_map(strlen(...), ['a', 'bb']);   // first-class callable

function readLines(string $file): Generator
{
    $h = fopen($file, 'r');
    try {
        while (($line = fgets($h)) !== false) {
            yield rtrim($line, "\r\n");       // ленивая выдача
        }
    } finally {
        fclose($h);
    }
}
foreach (readLines('big.log') as $line) { /* O(1) памяти */ }
```

### Вопросы

> [!question] Чем `function () use ($x)` отличается от `fn`?
> 
> - `fn` — одно выражение, автозахват по значению на момент создания.
>     
> - `function` — полное тело, явный `use`, возможен захват по ссылке.
>     
> 
> > [!warning] Подвох Из `fn` нельзя изменить внешнюю переменную. Для счётчика нужен `function () use (&$n)`.

---

## 7. Типизация

> [!abstract] Главное за 30 секунд
> 
> - `declare(strict_types=1);` — первая инструкция файла; строгая проверка касается **вызовов из этого файла**.
> - Типы: union `int|string` (8.0), intersection `A&B` (8.1), DNF `(A&B)|null` (8.2), `?T`, `void`, `never`, `mixed`.
> - Enum (8.1), атрибуты (8.0), типизированные свойства (7.4).

### Понятия

> [!abstract] `strict_types`
> 
> - **Что это:** режим, в котором скаляры не приводятся молча: `"5"` в параметр `int` → `TypeError`.
> - **Как работает:** решает **вызывающая** сторона. Исключение: `int` → `float` разрешён всегда.
> - **Где применяется:** почти везде в современном коде.
> - **Режим по умолчанию (coercive):** `"5"` молча станет `5`, нечисловая строка бросит `TypeError`.

> [!abstract] Enum (8.1)
> 
> - **Что это:** отдельный тип-перечисление (`enum Status: string`).
> - **Как реализуется:** `case Draft = 'draft';`, методы, интерфейсы, константы. `from()` бросает `ValueError`, `tryFrom()` возвращает `null`, `cases()` — все кейсы.
> - **Ограничения:** нельзя `new`, нельзя наследовать, нет изменяемого состояния.
> - **Где применяется:** статусы, роли, типы — вместо набора констант; хорошо работает в `match`.

> [!abstract] Атрибуты (8.0)
> 
> - **Что это:** машиночитаемые метаданные `#[Route('/')]`, замена аннотаций в докблоках.
> - **Как читаются:** через рефлексию `getAttributes()`.
> - **Где применяется:** роутинг, ORM-маппинг, валидация, плагины Drupal.

### Код

```php
<?php
declare(strict_types=1);

function area(int|float $w, int|float $h): float { return $w * $h; }
// area("3", 4);   // TypeError

enum Status: string
{
    case Draft     = 'draft';
    case Published = 'published';

    public function label(): string
    {
        return match ($this) {
            self::Draft     => 'Черновик',
            self::Published => 'Опубликовано',
        };
    }
}

Status::from('draft');        // Status::Draft (ValueError, если значения нет)
Status::tryFrom('xxx');       // null
Status::cases();              // все кейсы
```

### Вопросы

> [!question] Что делает `declare(strict_types=1)` и на что влияет?
> 
> - Включает строгую проверку скалярных типов в вызовах из текущего файла и для `return` в нём.
>     
> - Работает и для встроенных функций: `strlen(123)` → `TypeError`.
>     
> 
> > [!warning] Подвох Арифметика остаётся гибкой (`"5" + 3` работает). Режим касается только типов параметров и возвращаемых значений.

---

## 8. ООП

> [!abstract] Главное за 30 секунд
> 
> - Видимость: `public` / `protected` / `private`. Constructor promotion (8.0), `readonly` (8.1).
> - **Интерфейс** — контракт, **абстрактный класс** — база с состоянием, **трейт** — переиспользование кода без наследования.
> - `self` указывает на класс, где написан код; `static` — на класс, от имени которого вызван метод.
> - `clone` — поверхностный; объекты присваиваются по handle.

### Основы

|Модификатор|Внутри класса|В наследнике|Снаружи|
|---|:-:|:-:|:-:|
|`public`|✅|✅|✅|
|`protected`|✅|✅|❌|
|`private`|✅|❌|❌|

### Понятия

> [!abstract] Интерфейс
> 
> - **Что это:** контракт — набор методов без реализации и состояния.
> - **Как реализуется:** `interface Shape { ... }`, `class Circle implements Shape, Countable`.
> - **Где применяется:** DI, подмена реализаций в тестах, PSR-стандарты.

> [!abstract] Абстрактный класс
> 
> - **Что это:** класс, который нельзя инстанцировать; содержит общий код, состояние и абстрактные методы.
> - **Где применяется:** общая база семейства классов. Наследовать можно только один.

> [!abstract] Трейт (trait)
> 
> - **Что это:** способ повторно использовать код в несвязанных классах («горизонтальное подмешивание»).
> - **Как реализуется:** `trait Loggable { ... }`, `use Loggable;`. Конфликты имён: `insteadof`, `as`. Статические свойства у **каждого** использующего класса свои.
> - **Где применяется:** небольшие повторяющиеся куски (логирование, timestamps). Типом не является: `instanceof Trait` нельзя.

> [!abstract] Late Static Binding (позднее статическое связывание)
> 
> - **Что это:** `static::` разрешается в момент вызова в класс, **через который** вызван метод.
> - **Где применяется:** фабричные методы `new static()`, Active Record.
> - **Отличие:** `self::` — всегда класс, где написан код.

> [!abstract] `readonly` (8.1) и readonly-классы (8.2)
> 
> - **Что это:** свойство, которое можно записать один раз.
> - **Где применяется:** value object, DTO, неизменяемые сущности.
> - **Нюанс:** защищает от **переприсваивания**, но не от изменения вложенного изменяемого объекта.

### Магические методы

|Метод|Когда вызывается|Где применяется|
|---|---|---|
|`__get` / `__set` / `__isset` / `__unset`|доступ к недоступному свойству|прокси, ленивая загрузка, ORM|
|`__call` / `__callStatic`|вызов несуществующего метода|прокси, фасады, динамические API|
|`__toString`|приведение к строке (автоматически `Stringable`)|вывод, ключи|
|`__invoke`|объект вызывается как функция|команды, middleware, колбэки|
|`__clone`|после `clone`|глубокое копирование|
|`__serialize` / `__unserialize`|сериализация (предпочтительнее `__sleep`/`__wakeup`)|кэш, очереди|
|`__destruct`|уничтожение объекта|освобождение ресурсов (порядок нестабилен)|

### PHP 8.4

> [!abstract] Асимметричная видимость и property hooks
> 
> - **Асимметричная видимость:** `public private(set) int $id;` — читают все, меняет только класс.
> - **Property hooks:** `get` / `set` прямо в объявлении свойства — вычисляемые и валидируемые свойства без отдельных методов. Работают в интерфейсах и абстрактных классах.
> - **Ещё:** `new Foo()->bar()` без скобок, ленивые объекты.

> [!abstract] Рефлексия
> 
> - **Что это:** `ReflectionClass`, `ReflectionMethod`, `ReflectionProperty` — разбор классов во время выполнения.
> - **Где применяется:** DI-контейнеры (autowiring), ORM, сериализаторы, чтение атрибутов.

### Код

```php
<?php
declare(strict_types=1);

interface Shape { public function area(): float; }

abstract class BaseShape implements Shape
{
    public function __construct(protected readonly string $name) {}

    public function describe(): string
    {
        return sprintf('%s: %.2f', $this->name, $this->area());
    }
}

final class Circle extends BaseShape
{
    public function __construct(private float $r)
    {
        parent::__construct('Circle');
    }
    public function area(): float { return M_PI * $this->r ** 2; }
}

trait Loggable
{
    public function log(string $msg): void
    {
        echo '[' . static::class . '] ' . $msg . PHP_EOL;
    }
}

// Value object: иммутабельность
final class Money
{
    public function __construct(
        public readonly int $amount,          // в копейках
        public readonly string $currency,
    ) {}

    public function add(Money $o): static
    {
        if ($o->currency !== $this->currency) {
            throw new InvalidArgumentException('Currency mismatch');
        }
        return new static($this->amount + $o->amount, $this->currency);
    }
}

// Конфликт трейтов
class A { use T1, T2 { T1::hello insteadof T2; T2::hello as helloFromT2; } }

// Late Static Binding
class Parent_ { public static function create(): static { return new static; } }
class Child_ extends Parent_ {}
echo get_class(Child_::create());   // Child_
```

### Вопросы

> [!question] Абстрактный класс, интерфейс или трейт: что выбрать?
> 
> ||Интерфейс|Абстрактный класс|Трейт|
> |---|---|---|---|
> |Состояние|нет|да|да|
> |Реализация|нет|да|да|
> |Сколько можно|много|один|много|
> |Создаёт тип|да|да|нет|
> 
> **Практика:** зависимости объявлять через интерфейсы; абстрактный класс — когда есть общая база; трейты — для мелких повторяющихся кусков без бизнес-логики.

> [!question] Чем `self` отличается от `static`?
> 
> - `self` — класс, **где написан код** (разрешается при компиляции).
> - `static` — класс, **от имени которого вызван метод** (LSB).
> - В фабричном методе нужен `new static()`; `new self()` всегда создаст базовый класс.

---

## 9. Namespace, Composer, PSR

> [!abstract] Главное за 30 секунд
> 
> - **PSR-4**: префикс namespace → каталог; `App\Service\Mailer` → `src/Service/Mailer.php`.
> - `composer install` ставит версии из `composer.lock`, `composer update` пересчитывает и перезаписывает lock.
> - `^1.2.3` = `>=1.2.3 <2.0.0`, `~1.2.3` = `>=1.2.3 <1.3.0`.

### Понятия

> [!abstract] Namespace
> 
> - **Что это:** логическая группировка классов, функций, констант; защита от конфликтов имён.
> - **Как реализуется:** `namespace App\Service;`, импорт `use App\Foo as Bar;`, групповой `use App\{A, B};`, `use function`, `use const`. Полное имя: `\App\Service\Mailer`.
> - **Где применяется:** любой современный проект.

> [!abstract] Автозагрузка
> 
> - **Что это:** `spl_autoload_register()` подгружает файл, когда класс не найден.
> - **Как работает PSR-4:** Composer генерирует `vendor/autoload.php`; берётся самый длинный совпавший префикс, `\` заменяется на `/`, добавляется `.php`. Регистр имён важен на Linux.

> [!abstract] Composer
> 
> - **Что это:** менеджер зависимостей.
> - **Файлы:** `composer.json` — что нужно; `composer.lock` — какие версии установлены (для приложений коммитится).
> - **Команды:** `require`, `require --dev`, `install`, `update`, `dump-autoload -o`, `audit`, `outdated`.
> - **На проде:** `composer install --no-dev --optimize-autoloader`.

### Основные PSR

|Стандарт|О чём|
|---|---|
|PSR-1, PSR-12|стиль кода (PSR-12 заменён PER Coding Style)|
|PSR-3|`LoggerInterface`|
|PSR-4|автозагрузка|
|PSR-7, PSR-17|HTTP-сообщения и фабрики|
|PSR-15|HTTP middleware|
|PSR-18|HTTP-клиент|
|PSR-11|контейнер (`get`, `has`)|
|PSR-14|диспетчер событий|
|PSR-16|Simple Cache|

### Код

```json
{
  "name": "acme/app",
  "require": { "php": ">=8.2", "monolog/monolog": "^3.0" },
  "require-dev": { "phpunit/phpunit": "^11.0" },
  "autoload": { "psr-4": { "App\\": "src/" } },
  "autoload-dev": { "psr-4": { "App\\Tests\\": "tests/" } }
}
```

```php
<?php
declare(strict_types=1);

namespace App\Service;

use Psr\Log\LoggerInterface;

final class Mailer
{
    public function __construct(private LoggerInterface $logger) {}
}
```

---

## 10. Ошибки и исключения

> [!abstract] Главное за 30 секунд
> 
> - `Throwable` → `Error` (баги и ошибки движка) и `Exception` (прикладные).
> - `finally` выполняется всегда; `return` в нём перекрывает результат `try`.
> - Прод: `display_errors=Off`, `log_errors=On`.

### Иерархия

```text
Throwable (interface)
├── Error                        ← ошибки движка
│   ├── TypeError
│   │   └── ArgumentCountError
│   ├── ValueError
│   ├── ArithmeticError
│   │   └── DivisionByZeroError
│   └── UnhandledMatchError
└── Exception                    ← прикладные
    ├── LogicException           ← ошибка в логике кода
    │   └── InvalidArgumentException, DomainException ...
    ├── RuntimeException         ← ошибка при выполнении
    │   └── OutOfBoundsException, UnexpectedValueException ...
    └── JsonException
```

### Понятия

> [!abstract] `try / catch / finally`
> 
> - **Как работает:** `catch (A | B $e)` ловит несколько типов; `catch (Exception)` без переменной (8.0); `finally` выполняется всегда.
> - **Цепочка:** `new RuntimeException('msg', 0, $previous)`, `getPrevious()`.

> [!abstract] Notice / Warning / Deprecated
> 
> - **Что это:** старые «ошибки», не исключения. Управляются `error_reporting` и `set_error_handler`.
> - **Приём:** `set_error_handler` бросает `ErrorException` — warnings становятся исключениями (так делают фреймворки).

### Код

```php
final class InsufficientFundsException extends DomainException
{
    public function __construct(public readonly int $needed, ?Throwable $prev = null)
    {
        parent::__construct("Не хватает $needed", 0, $prev);
    }
}

try {
    $r = intdiv(1, 0);
} catch (DivisionByZeroError $e) {
    error_log($e->getMessage());
} catch (TypeError | ValueError $e) {
    throw new RuntimeException('Некорректные данные', previous: $e);
} finally {
    // освобождение ресурсов
}

set_error_handler(function (int $no, string $str, string $file, int $line): bool {
    throw new ErrorException($str, 0, $no, $file, $line);   // warning → исключение
});
```

### Вопросы

> [!question] В чём разница между `Error` и `Exception`?
> 
> - Оба реализуют `Throwable`.
> - `Error` — ошибки движка (неверный тип, несуществующий метод), обычно баг программиста.
> - `catch (Exception $e)` **не поймает** `Error`. Для всего — `catch (Throwable $e)`, но на верхнем уровне, а не «глуша» бизнес-код.

> [!question] Что вернёт функция, если `return` есть и в `try`, и в `finally`?
> 
> ```php
> function f() {
>     try { return 'try'; }
>     finally { return 'finally'; }
> }
> echo f();   // finally
> ```
> 
> `finally` перекрывает возврат из `try` и «проглатывает» неперехваченное исключение. `return` внутри `finally` — плохая практика.

---

## 11. Память: ссылки, copy-on-write, GC

> [!abstract] Главное за 30 секунд
> 
> - **Refcount**: при нулевом счётчике память освобождается сразу. Циклические ссылки ловит **сборщик циклов**.
> - **Copy-on-write**: присваивание не копирует данные до первой записи.
> - `foreach` по ссылке → после цикла `unset($v)`.

### Понятия

> [!abstract] Подсчёт ссылок и сборщик циклов
> 
> - **Как работает:** у каждого значения (zval) есть счётчик; при `0` память освобождается, деструктор вызывается сразу.
> - **Проблема:** цикл A → B → A имеет ненулевой счётчик. Их ловит cycle collector (`gc_collect_cycles()`, включён по умолчанию).
> - **Где важно:** долгие воркеры и CLI; в коротком веб-запросе редко.

> [!abstract] Ссылка `&`
> 
> - **Что это:** псевдоним переменной на то же значение. `$b = &$a;` делает обе переменные одним значением.
> - **Где применяется:** функции с выходным параметром, `foreach` по ссылке. Источник неочевидных багов.

### Код

```php
$arr = [1, 2, 3];
foreach ($arr as &$v) { $v *= 2; }
// unset($v);                       // ← без него баг ниже
foreach ($arr as $v) { }
print_r($arr);                      // [2, 4, 4]: последний элемент перезаписан

$a = 1; $b = &$a; $b++;
echo $a;                            // 2

class Node { public ?Node $other = null; }
$x = new Node; $y = new Node;
$x->other = $y; $y->other = $x;     // цикл
unset($x, $y);                      // refcount не 0 → ждёт GC
gc_collect_cycles();
```

---

## 12. HTTP, сессии, cookies

> [!abstract] Главное за 30 секунд
> 
> - Заголовки отправляются **до** любого вывода (иначе `headers already sent`).
> - Сессия: данные на сервере, в cookie только ID. После логина — `session_regenerate_id(true)`.
> - После POST делать редирект (**PRG**); JSON-тело читать из `php://input`.

### Суперглобальные массивы

|Массив|Что содержит|Заметка|
|---|---|---|
|`$_GET`|query string||
|`$_POST`|тело формы|JSON сюда **не попадает**|
|`$_REQUEST`|GET + POST + COOKIE|**не использовать**|
|`$_COOKIE`, `$_FILES`|cookies, загруженные файлы||
|`$_SESSION`|данные сессии|нужен `session_start()`|
|`$_SERVER`|заголовки, метод, путь, IP|`REQUEST_METHOD`|
|`$GLOBALS`|все глобальные|плохая практика|

### Понятия

> [!abstract] Сессия
> 
> - **Что это:** серверное хранилище данных пользователя между запросами.
> - **Как работает:** `session_start()` читает cookie `PHPSESSID` и загружает данные (по умолчанию файлы, на кластере — Redis/БД через `session.save_handler`).
> - **Безопасность:** `session.cookie_httponly=1`, `cookie_secure=1`, `cookie_samesite=Lax`, `use_strict_mode=1`, `use_only_cookies=1`.
> - **Блокировка:** файловые сессии блокируют файл на время запроса; лечится `session_write_close()`.

> [!abstract] Cookie
> 
> - `setcookie($name, $value, ['expires' => ..., 'path' => '/', 'secure' => true, 'httponly' => true, 'samesite' => 'Lax'])`.
> - **Где применяется:** идентификатор сессии, настройки интерфейса. Секреты в cookie не хранить.

> [!abstract] Паттерн PRG (Post / Redirect / Get)
> 
> - **Что это:** после POST сервер отвечает редиректом, и браузер делает GET.
> - **Зачем:** обновление страницы не отправляет форму повторно.

### Код

```php
<?php
declare(strict_types=1);

session_set_cookie_params([
    'lifetime' => 0, 'path' => '/',
    'secure' => true, 'httponly' => true, 'samesite' => 'Lax',
]);
session_start();

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $email = filter_var($_POST['email'] ?? '', FILTER_VALIDATE_EMAIL);
    if ($email === false) {
        http_response_code(422);
        exit('Некорректный email');
    }
    session_regenerate_id(true);               // защита от session fixation
    $_SESSION['user_email'] = $email;
    header('Location: /profile', true, 303);   // PRG
    exit;
}

// JSON API
$data = json_decode(file_get_contents('php://input'), true, 512, JSON_THROW_ON_ERROR);
header('Content-Type: application/json; charset=utf-8');
echo json_encode(['ok' => true], JSON_UNESCAPED_UNICODE);
```

### Вопросы

> [!question] Что такое session fixation и как защититься?
> 
> - Атакующий подсовывает жертве известный ему ID сессии; после логина жертва «авторизует» этот ID.
> - Защита: `session_regenerate_id(true)` после аутентификации, `use_strict_mode=1`, только cookie, флаги `HttpOnly` / `Secure` / `SameSite`.

> [!question] Откуда ошибка «headers already sent»?
> 
> - Любой вывод (даже пробел или BOM перед `<?php`) заставляет PHP отправить заголовки.
> - Решения: не выводить до `header()`, не ставить `?>` в чисто PHP-файлах, использовать `ob_start()`, во фреймворках — объект Response.

---

## 13. Файлы и include

> [!abstract] Главное за 30 секунд
> 
> - `require` — фатальная ошибка при отсутствии файла, `include` — warning; `_once` — один раз.
> - Пути строить от `__DIR__`; пользовательский ввод в пути не подставлять (LFI/RFI).
> - Загрузка файлов: тип по **содержимому**, своё имя, хранить вне webroot.

### Понятия

### Код

```php
// запись с блокировкой
file_put_contents(__DIR__ . '/log.txt', date('c') . " start\n", FILE_APPEND | LOCK_EX);

// большой CSV построчно
$f = new SplFileObject(__DIR__ . '/big.csv');
$f->setFlags(SplFileObject::READ_CSV | SplFileObject::SKIP_EMPTY | SplFileObject::READ_AHEAD);
foreach ($f as $row) { /* ... */ }

// безопасная загрузка файла
if (($_FILES['avatar']['error'] ?? UPLOAD_ERR_NO_FILE) === UPLOAD_ERR_OK) {
    $mime    = (new finfo(FILEINFO_MIME_TYPE))->file($_FILES['avatar']['tmp_name']);
    $allowed = ['image/jpeg' => 'jpg', 'image/png' => 'png'];
    if (!isset($allowed[$mime])) { exit('Недопустимый тип'); }
    $name = bin2hex(random_bytes(16)) . '.' . $allowed[$mime];      // своё имя
    move_uploaded_file($_FILES['avatar']['tmp_name'], __DIR__ . "/../storage/$name");
}
```

### Вопросы

> [!question] Как безопасно принимать загруженные файлы?
> 
> 1. Проверить `UPLOAD_ERR_OK` и размер.
> 2. Определять тип по **содержимому** (`finfo`), а не по расширению или MIME клиента.
> 3. Разрешать только белый список типов.
> 4. Генерировать **своё** имя файла.
> 5. Хранить вне публичного каталога или запретить исполнение скриптов в нём.
> 6. Изображения перекодировать (GD/Imagick).
> 7. Приватные файлы отдавать через контроллер с проверкой прав.

---

## 14. PDO, транзакции, SQL

> [!abstract] Главное за 30 секунд
> 
> - **Prepared statements** защищают от SQL-инъекций, но только **значения**; имена таблиц/колонок и `ORDER BY` — через белый список.
> - Настройки PDO: `ERRMODE_EXCEPTION`, `EMULATE_PREPARES=false`, `charset=utf8mb4`.
> - **N+1**, индексы, транзакции (ACID).

### Понятия

> [!abstract] PDO
> 
> - **Что это:** единый интерфейс доступа к разным СУБД (MySQL, PostgreSQL, SQLite). Альтернатива — `mysqli` (только MySQL).
> - **Методы выборки:** `fetch`, `fetchAll`, `fetchColumn`, `FETCH_KEY_PAIR`, `FETCH_CLASS`.
> - **`bindValue` vs `bindParam`:** значение сразу / переменная по ссылке (читается при `execute`).

> [!abstract] Prepared statement
> 
> - **Что это:** СУБД получает структуру SQL с плейсхолдерами, а данные отдельно, поэтому данные не становятся кодом.
> - **Плейсхолдеры:** именованные `:id` и позиционные `?`.

> [!abstract] Транзакция и ACID
> 
> - **Что это:** группа операций «всё или ничего». `beginTransaction()`, `commit()`, `rollBack()`.
> - **ACID:** Atomicity, Consistency, Isolation, Durability.
> - **Нюанс:** DDL в MySQL приводит к неявному commit.

> [!abstract] Проблема N+1
> 
> - **Что это:** 1 запрос на список + N запросов на связанные данные.
> - **Решения:** `JOIN`, один запрос `WHERE id IN (...)`, eager loading ORM (`with()` в Eloquent).

### Уровни изоляции

|Уровень|Dirty read|Non-repeatable|Phantom|
|---|:-:|:-:|:-:|
|`READ UNCOMMITTED`|да|да|да|
|`READ COMMITTED`|нет|да|да|
|`REPEATABLE READ` (по умолчанию в InnoDB)|нет|нет|по стандарту да, InnoDB частично защищает|
|`SERIALIZABLE`|нет|нет|нет|

### SQL-минимум

|Тема|Что знать|
|---|---|
|`JOIN`|`INNER` — только совпадения; `LEFT` — все из левой таблицы + совпадения|
|Индексы|B-tree; помогают `WHERE`, `JOIN`, `ORDER BY`; составной индекс работает слева направо|
|`EXPLAIN`|показывает план: используется ли индекс, сколько строк читается|
|Пагинация|большой `OFFSET` медленный; keyset: `WHERE id < :last ORDER BY id DESC LIMIT 50`|
|`GROUP BY` / `HAVING`|агрегаты; `HAVING` фильтрует группы, `WHERE` — строки до группировки|
|Нормализация|убирает дублирование данных; денормализация — осознанный компромисс ради скорости|

### Код

```php
<?php
declare(strict_types=1);

$pdo = new PDO(
    'mysql:host=localhost;dbname=app;charset=utf8mb4', 'user', 'pass',
    [
        PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES   => false,
    ]
);

$stmt = $pdo->prepare('SELECT id, name FROM users WHERE email = :email AND active = :a');
$stmt->execute(['email' => $email, 'a' => 1]);
$user = $stmt->fetch();                          // false, если не найдено

// IN (...) с динамическим числом значений
$ids = [1, 2, 3];
$in  = implode(',', array_fill(0, count($ids), '?'));
$stmt = $pdo->prepare("SELECT * FROM users WHERE id IN ($in)");
$stmt->execute($ids);

// ORDER BY — только из белого списка
$col = in_array($_GET['sort'] ?? '', ['name', 'created_at'], true) ? $_GET['sort'] : 'name';
$sql = "SELECT * FROM users ORDER BY `$col`";

// транзакция
try {
    $pdo->beginTransaction();
    $pdo->prepare('UPDATE accounts SET balance = balance - ? WHERE id = ?')->execute([100, 1]);
    $pdo->prepare('UPDATE accounts SET balance = balance + ? WHERE id = ?')->execute([100, 2]);
    $pdo->commit();
} catch (Throwable $e) {
    if ($pdo->inTransaction()) { $pdo->rollBack(); }
    throw $e;
}
```

### Вопросы

> [!question] Как prepared statements защищают от SQL-инъекций?
> 
> - СУБД получает структуру запроса и данные раздельно, данные никогда не интерпретируются как код: `' OR '1'='1` останется обычным значением.
>     
> - Защита только для **значений**. Имена таблиц, колонок, `ORDER BY` — белый список.
>     
> - `addslashes` и ручное экранирование ненадёжны (ошибки кодировки).
>     
> 
> > [!warning] Подвох В PDO MySQL по умолчанию `EMULATE_PREPARES=true`: подстановка выполняется на стороне PHP. При верной кодировке это безопасно, но рекомендуют отключать эмуляцию ради нативных prepared statements и корректных типов.

---

## 15. Безопасность

> [!abstract] Главное за 30 секунд
> 
> - SQLi → prepared statements; XSS → экранировать **на выводе**; CSRF → токен + `SameSite`.
> - Пароли — `password_hash`; токены — `random_bytes`; секреты сравнивать `hash_equals`.
> - Не `unserialize` чужие данные; не подставлять ввод в пути и shell-команды.

### Таблица угроз

|Угроза|Суть|Защита|
|---|---|---|
|**SQL-инъекция**|ввод становится частью SQL|prepared statements, белый список идентификаторов, минимальные права БД|
|**XSS**|чужой JS выполняется на вашей странице|экранирование на выводе по контексту, CSP, `HttpOnly`|
|**CSRF**|чужой сайт шлёт запрос от имени пользователя|токен + `hash_equals`, `SameSite`, проверка `Origin`|
|**Session fixation**|подмена ID сессии|`session_regenerate_id(true)`, `use_strict_mode`|
|**IDOR**|доступ к чужому объекту через `?id=`|проверка прав на уровне объекта|
|**LFI / RFI**|подключение чужого файла|не подставлять ввод в `include`, `allow_url_include=Off`|
|**Object Injection**|`unserialize` запускает магические методы|JSON или `unserialize($d, ['allowed_classes' => false])`|
|**Командная инъекция**|ввод попадает в shell|не вызывать `exec`, `escapeshellarg`, `proc_open([...])`|
|**SSRF**|сервер ходит по URL атакующего|проверять схему и хост, блокировать внутренние адреса|
|**Open redirect**|редирект на чужой сайт|белый список / относительные пути|
|**Mass assignment**|в модель попадают лишние поля|перечислять разрешённые поля|

**Экранирование по контексту:** HTML — `htmlspecialchars($s, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8')`; JS — `json_encode` с `JSON_HEX_*`; URL — `rawurlencode`; Twig/Blade `{{ }}` экранируют сами.

### Понятия

> [!abstract] Хранение паролей
> 
> - **Как реализуется:** `password_hash($pw, PASSWORD_DEFAULT)` (bcrypt; можно `PASSWORD_ARGON2ID`) — соль и параметры внутри результата; `password_verify`; `password_needs_rehash`.
> - **Нельзя:** `md5`, `sha1`, голый `sha256`, обратимое шифрование, логирование паролей.

> [!abstract] Случайность
> 
> - `random_bytes()`, `random_int()` — криптографически стойкие: токены, сброс пароля, CSRF.
> - `rand()`, `mt_rand()`, `uniqid()` — предсказуемы: только игры и тестовые данные.

> [!abstract] Аутентификация и авторизация
> 
> - **Аутентификация** — «кто вы» (логин); **авторизация** — «что вам можно» (права).
> - **Сессии:** состояние на сервере, cookie с ID. **JWT:** подписанный токен без состояния на сервере (отзывать сложнее). **OAuth2:** делегирование доступа без передачи пароля.

### Код

```php
echo htmlspecialchars($comment, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');   // вывод

// CSRF
session_start();
$_SESSION['csrf'] ??= bin2hex(random_bytes(32));
// в форме: <input type="hidden" name="csrf" value="<?= htmlspecialchars($_SESSION['csrf']) ?>">
if (!hash_equals($_SESSION['csrf'] ?? '', $_POST['csrf'] ?? '')) {
    http_response_code(403);
    exit('CSRF');
}

// пароли
$hash = password_hash($plain, PASSWORD_DEFAULT);
if (password_verify($input, $hash) && password_needs_rehash($hash, PASSWORD_DEFAULT)) {
    $hash = password_hash($input, PASSWORD_DEFAULT);      // обновить в БД
}
```

### Вопросы

> [!question] Чем XSS отличается от CSRF?
> 
> - **XSS:** атакующий внедряет JavaScript в вашу страницу, он выполняется в браузере жертвы с правами сайта. Защита: экранирование на выводе, CSP, `HttpOnly`.
>     
> - **CSRF:** чужой сайт заставляет браузер жертвы отправить запрос на ваш сайт с её cookie. Защита: токены, `SameSite`, проверка `Origin`, GET без побочных эффектов.
>     
> 
> > [!warning] Подвох Экранировать нужно на **выводе**, а не при сохранении. В базе лежат исходные данные.

---

## 16. Дата, regex, JSON

> [!abstract] Главное за 30 секунд
> 
> - `DateTimeImmutable` вместо `DateTime`; хранить и передавать в **UTC**, переводить в зону пользователя при выводе.
> - `preg_*` для Unicode — с флагом `/u`; остерегаться ReDoS.
> - JSON: `JSON_THROW_ON_ERROR`.

### Понятия

> [!abstract] `DateTimeImmutable`
> 
> - **Что это:** дата-время, которое **не изменяется**: каждый `modify()` возвращает новый объект.
> - **Где применяется:** value object, сущности. `DateTime` меняется на месте и приводит к багам при передаче в несколько мест.
> - **Нюансы:** `modify('+1 month')` от 31 января даёт март; для «последнего дня» — `last day of next month`. Для тестов — `ClockInterface` (PSR-20).

> [!abstract] Регулярные выражения (PCRE)
> 
> - **Функции:** `preg_match`, `preg_match_all`, `preg_replace`, `preg_replace_callback`, `preg_split`, `preg_quote`.
> - **Модификаторы:** `i` регистр, `m` многострочный, `s` точка = перевод строки, `x` комментарии, `u` Unicode.
> - **Приёмы:** именованные группы `(?<year>\d{4})`, ленивые `.*?`, lookahead `(?=...)`, lookbehind `(?<=...)`.
> - **ReDoS:** вложенные квантификаторы `(a+)+$` вызывают катастрофический откат.

> [!abstract] JSON
> 
> - `json_encode($d, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES | JSON_THROW_ON_ERROR)`; `json_decode($s, true, 512, JSON_THROW_ON_ERROR)`.
> - `JsonSerializable::jsonSerialize()` управляет видом объекта; `json_validate()` (8.3).

### Код

```php
$now  = new DateTimeImmutable('now', new DateTimeZone('UTC'));
$next = $now->modify('+7 days')->setTime(9, 0);
echo $next->setTimezone(new DateTimeZone('Europe/Berlin'))->format('Y-m-d H:i');

if (preg_match('/^(?<year>\d{4})-(?<month>\d{2})$/', '2024-05', $m)) {
    echo $m['year'], '/', $m['month'];
}
$clean = preg_replace('/\s+/u', ' ', $text);

try {
    $data = json_decode($raw, true, 512, JSON_THROW_ON_ERROR);
} catch (JsonException $e) {
    http_response_code(400);
}
```

---

## 17. SOLID, DI, паттерны

> [!abstract] Главное за 30 секунд
> 
> - SOLID и **DI через конструктор**; зависеть от интерфейсов, а не от конкретных классов.
> - Самые частые паттерны: Strategy, Decorator, Factory, Repository, Observer.
> - Singleton — антипаттерн; композиция лучше наследования.

### SOLID

|Буква|Принцип|Смысл|
|---|---|---|
|**S**|Single Responsibility|у класса одна причина для изменения|
|**O**|Open / Closed|открыт для расширения, закрыт для модификации (стратегии, полиморфизм)|
|**L**|Liskov Substitution|наследник заменяет родителя без поломки поведения|
|**I**|Interface Segregation|много маленьких интерфейсов лучше одного большого|
|**D**|Dependency Inversion|зависеть от абстракций, а не от конкретных классов|

### Понятия

> [!abstract] Внедрение зависимостей (DI)
> 
> - **Что это:** зависимости передаются снаружи (в конструктор), а не создаются через `new` внутри и не берутся из глобального состояния.
> - **Как реализуется:** конструктор + интерфейсы; **DI-контейнер** (PSR-11) собирает граф объектов, autowiring по типам через рефлексию.
> - **Где применяется:** все современные фреймворки. **Service Locator** (брать зависимости из контейнера внутри класса) — антипаттерн.
> - **Зачем:** слабая связанность, подмена в тестах (mock / stub), единое место конфигурации.

> [!abstract] Strategy
> 
> - **Что это:** семейство взаимозаменяемых алгоритмов за одним интерфейсом.
> - **Где применяется:** скидки, способы оплаты, сортировка — вместо цепочек `if/switch`.

> [!abstract] Decorator
> 
> - **Что это:** обёртка с тем же интерфейсом, добавляющая поведение.
> - **Где применяется:** логирование, кэширование, метрики вокруг сервиса.

> [!abstract] Singleton
> 
> - **Что это:** один глобальный экземпляр. **Почему антипаттерн:** скрытая глобальная зависимость, сложно тестировать, мешает в долгоживущих рантаймах. «Один экземпляр» решается shared-сервисом в DI-контейнере.

### Прочие принципы

- **DRY**, **KISS**, **YAGNI**; композиция вместо наследования.
- **Закон Деметры:** не цепочки вида `$a->getB()->getC()->do()`.
- **Active Record** (Eloquent) vs **Data Mapper** (Doctrine): модель знает про БД или нет.

### Код

```php
interface PaymentGateway { public function charge(int $amount): bool; }

final class StripeGateway implements PaymentGateway
{
    public function charge(int $amount): bool { /* ... */ return true; }
}

// DI + DIP: зависим от интерфейса
final class CheckoutService
{
    public function __construct(
        private PaymentGateway $gateway,
        private LoggerInterface $logger,
    ) {}

    public function pay(int $amount): void
    {
        if (!$this->gateway->charge($amount)) {
            $this->logger->error('Payment failed');
        }
    }
}

// Decorator
final class LoggingGateway implements PaymentGateway
{
    public function __construct(private PaymentGateway $inner, private LoggerInterface $log) {}

    public function charge(int $amount): bool
    {
        $this->log->info("charge $amount");
        return $this->inner->charge($amount);
    }
}

// Strategy
interface DiscountStrategy { public function apply(int $price): int; }

final class PercentDiscount implements DiscountStrategy
{
    public function __construct(private int $percent) {}
    public function apply(int $price): int { return (int) round($price * (100 - $this->percent) / 100); }
}
```

### Вопросы

> [!question] Что такое DI и чем оно отличается от IoC?
> 
> - **DI** — класс получает зависимости извне, а не создаёт сам.
> - **IoC** (инверсия управления) — шире: создание и вызов объектов передаются фреймворку или контейнеру. DI — одна из реализаций IoC.

---

## 18. SPL, итераторы, конкурентность

> [!abstract] Главное за 30 секунд
> 
> - SPL: `SplStack`, `SplQueue`, `SplMinHeap`, `SplPriorityQueue`, `SplFixedArray`.
> - `Iterator` (5 методов) и `IteratorAggregate` (`getIterator()`) делают объект пригодным для `foreach`.
> - PHP однопоточен: параллельность — **процессы и очереди**; Fibers — кооперативные.

### Понятия

> [!abstract] Итераторы
> 
> - **Что это:** объект, по которому можно пройти `foreach`.
> - **`Iterator`:** `current()`, `key()`, `next()`, `rewind()`, `valid()`. **`IteratorAggregate`:** один метод `getIterator()`.
> - **Где применяется:** коллекции, ленивые выборки (`LimitIterator`, `FilterIterator`, `RecursiveIteratorIterator`).

> [!abstract] Fibers (8.1)
> 
> - **Что это:** лёгкая корутина с собственным стеком, которую можно приостановить (`Fiber::suspend`) и продолжить (`resume`).
> - **Отличие от потоков:** **кооперативная** многозадачность в одном потоке, настоящей параллельности и гонок данных нет.
> - **Где применяется:** под капотом async-библиотек (Amp, ReactPHP).

> [!abstract] Очереди и фоновые задачи
> 
> - **Что это:** тяжёлое выносится из HTTP-запроса: сообщение уходит в очередь (Redis, RabbitMQ), воркеры (CLI под Supervisor) забирают и выполняют.
> - **Где применяется:** почта, отчёты, импорты. Retry с задержкой, failed-очередь.

### Код

```php
$stack = new SplStack();
$stack->push(1); $stack->push(2);
echo $stack->pop();                 // 2

$heap = new SplMinHeap();
foreach ([5, 1, 3] as $n) { $heap->insert($n); }
echo $heap->extract();              // 1

$pq = new SplPriorityQueue();
$pq->insert('low', 1); $pq->insert('high', 10);
echo $pq->extract();                // high
```

---

## 19. Производительность и PHP-FPM

> [!abstract] Главное за 30 секунд
> 
> - Сначала **измерить** (профайлер), потом оптимизировать.
> - OPcache + настройки FPM; кэши; тяжёлое — в очередь.
> - Типичные причины: N+1, нет индексов, большой `OFFSET`, вся выборка в памяти.

### Понятия

> [!abstract] OPcache и preloading
> 
> - **OPcache** хранит опкоды; на проде `validate_timestamps=0` + перезапуск FPM при релизе.
> - **Preloading** (`opcache.preload`): классы загружаются в память один раз при старте FPM. Быстрее, но любое изменение требует перезапуска.

> [!abstract] Кэширование
> 
> - **APCu:** память одного сервера. **Redis / Memcached:** общий кэш. **HTTP-кэш:** `Cache-Control`, `ETag`, `Last-Modified`, reverse proxy (Nginx, Varnish), CDN. Плюс кэш конфигурации и маршрутов фреймворка.

> [!abstract] Настройка PHP-FPM
> 
> - `pm = static` (предсказуемо) / `dynamic` / `ondemand` (экономит память).
> - `pm.max_children` ≈ доступная память под PHP / память на воркер.
> - `pm.max_requests` — перезапуск воркера после N запросов (борьба с утечками). `slowlog` — диагностика долгих запросов.

### Узкие места

|Причина|Что делать|
|---|---|
|N+1 запросы|`JOIN`, `WHERE id IN`, eager loading|
|Нет индексов, `SELECT *`|`EXPLAIN`, нужные индексы, только нужные колонки|
|Большой `OFFSET`|keyset-пагинация|
|Вся выборка в памяти|генераторы, `chunk`, потоковая обработка|
|Тяжёлое в запросе (почта, PDF, внешний HTTP)|очередь и воркеры|
|Автозагрузка без оптимизации|`composer dump-autoload -o`|

### Вопросы

> [!question] С чего начать оптимизацию медленного приложения?
> 
> 1. **Измерить:** Xdebug / Blackfire / XHProf, APM, логи.
> 2. **БД:** медленные запросы, `EXPLAIN`, индексы, N+1.
> 3. **Кэш:** тяжёлые вычисления, HTTP-кэш, OPcache.
> 4. **Очередь** для тяжёлого.
> 5. **Инфраструктура:** FPM, версия PHP (8.x быстрее 7.x), память.
> 6. Микрооптимизации — в самом конце.

---

## 20. Тестирование

> [!abstract] Главное за 30 секунд
> 
> - Виды: **unit** (быстро, без БД/сети), **integration**, **E2E**.
> - **PHPUnit**: `test*`, `setUp`, `assertSame`, `expectException`, data providers.
> - **PHPStan / Psalm** находят ошибки без запуска кода; в CI — линтер + статанализ + тесты.

### Понятия

> [!abstract] Тестовые двойники
> 
> - **Stub** — возвращает заранее заданные значения. **Mock** — ещё и проверяет вызовы. **Fake** — упрощённая рабочая реализация (in-memory репозиторий). **Spy** — записывает вызовы.
> - **Совет:** не мокать всё подряд — тесты, привязанные к реализации, ломаются при рефакторинге.

> [!abstract] Статический анализ
> 
> - **PHPStan / Psalm:** несуществующие методы, неверные типы, возможный `null`, недостижимый код. Уровни строгости, baseline для legacy.
> - **Rector:** автоматический рефакторинг и апгрейд версий. **PHP_CodeSniffer / PHP-CS-Fixer:** стиль.

### Код

```php
<?php
declare(strict_types=1);

namespace App\Tests;

use App\Money;
use PHPUnit\Framework\Attributes\DataProvider;
use PHPUnit\Framework\TestCase;

final class MoneyTest extends TestCase
{
    #[DataProvider('additions')]
    public function testAdd(int $a, int $b, int $expected): void
    {
        $result = (new Money($a, 'USD'))->add(new Money($b, 'USD'));
        self::assertSame($expected, $result->amount);
    }

    public static function additions(): iterable
    {
        yield 'простое сложение' => [100, 50, 150];
        yield 'с нулём'          => [100, 0, 100];
    }

    public function testDifferentCurrenciesThrow(): void
    {
        $this->expectException(\InvalidArgumentException::class);
        (new Money(1, 'USD'))->add(new Money(1, 'EUR'));
    }
}
```

---

## 21. Версии PHP 7.x–8.x

> [!abstract] Главное за 30 секунд
> 
> - **7.0** — типы и `??`; **7.4** — typed properties и `fn`; **8.0** — union, `match`, атрибуты, named args.
> - **8.1** — enum, readonly, fibers; **8.2–8.4** — readonly-классы, типизированные константы, property hooks.
> - На собеседовании называйте версию: «это появилось в 8.0».

|Версия|Ключевое|
|---|---|
|**7.0**|рост производительности (~2× к 5.6), скалярные типы и типы возврата, `??`, `<=>`, анонимные классы, `Throwable` / `Error`, `intdiv`|
|**7.1**|`?Type`, `void`, `iterable`, константы с видимостью, `catch (A \| B)`, отрицательные индексы строк|
|**7.3**|гибкий heredoc, `array_key_first/last`, `JSON_THROW_ON_ERROR`|
|**7.4**|**typed properties**, **стрелочные функции**, `??=`, spread в массивах, `WeakReference`, preloading|
|**8.0**|**named arguments**, **атрибуты**, **constructor promotion**, **union types**, **`match`**, **`?->`**, `mixed`, `str_contains`, `WeakMap`, **JIT**, строгое сравнение числа и строки, стабильная сортировка|
|**8.1**|**enum**, **`readonly`**, **first-class callable**, **Fibers**, `never`, `new` в инициализаторах, intersection types, `array_is_list`|
|**8.2**|**readonly-классы**, DNF types, `null` / `false` / `true` как типы, deprecation динамических свойств|
|**8.3**|типизированные константы класса, `#[\Override]`, `json_validate()`|
|**8.4**|**property hooks**, **асимметричная видимость**, `new Foo()->bar()`, `array_find` / `array_any` / `array_all`, ленивые объекты|
|**8.5**|pipe-оператор `\|>`, `array_first` / `array_last`, `#[\NoDiscard]`|

> [!warning] Проверьте актуальность Новшества 8.4 и 8.5 сверьте с changelog на php.net перед собеседованием. Поддерживаемые ветки: `php.net/supported-versions.php`.

> [!question] Какие самые важные изменения принёс PHP 8.0?
> 
> - **Меньше шаблонного кода:** union types, constructor promotion, named arguments, `match`, `?->`, атрибуты.
> - **Предсказуемость:** `TypeError` для внутренних функций вместо тихих warnings, строгое сравнение числа и строки.
> - **Скорость:** JIT.

---

## 22. Фреймворки и CMS

> [!abstract] Главное за 30 секунд
> 
> - **Laravel / Symfony:** DI-контейнер, роутинг, middleware, ORM (Eloquent / Doctrine), очереди, события.
> - **Drupal:** Symfony-компоненты, сервисы, плагины, хуки и события. **Moodle:** глобальные `$DB`, `$CFG`, `$USER`; `required_param`, `require_capability`.
> - Везде работают те же основы: ООП, PSR-4, безопасность.

### Понятия

> [!abstract] Жизненный цикл запроса во фреймворке
> 
> - **Как идёт:** `index.php` → автозагрузка → контейнер → роутер → middleware → контроллер → ответ.
> - **Где применяется:** Laravel, Symfony, Slim — одна схема, разные названия.

> [!abstract] Laravel
> 
> - **Eloquent** (Active Record), **Blade** (шаблоны с автоэкранированием), **Service Container**, **Queues**, **Artisan**, **Policies / Gates** для авторизации, **Form Request** для валидации.

> [!abstract] Symfony
> 
> - **Компоненты** (HttpFoundation, EventDispatcher, DependencyInjection), **Doctrine** (Data Mapper), **Twig**, **Messenger** (очереди), **Voters** для прав, автовайринг сервисов.

> [!abstract] Drupal (на Symfony-компонентах)
> 
> - **Namespaces и PSR-4:** `Drupal\mymodule\...` в `modules/custom/mymodule/src/`.
> - **Сервисы и DI:** `mymodule.services.yml`, `arguments: ['@database', '@current_user']`; `\Drupal::service()` — только в процедурном коде.
> - **Плагины:** блоки, поля, виджеты — классы с атрибутами / аннотациями.
> - **События** (`EventSubscriberInterface`), **хуки** (`mymodule_*()`, OOP-хуки через `#[Hook]`), **Render / Form arrays**, **Twig**, **Database API** поверх PDO.

> [!abstract] Moodle
> 
> - **Глобальные объекты:** `$DB`, `$CFG`, `$USER`, `$PAGE`, `$OUTPUT`; в файлах `defined('MOODLE_INTERNAL') || die();`.
> - **Безопасность:** `required_param('id', PARAM_INT)`, `optional_param`, `require_login()`, `require_capability()`, `sesskey`.
> - **БД:** `$DB->get_record()`, `$DB->get_records_sql($sql, ['courseid' => $id])` с плейсхолдерами, `start_delegated_transaction()`.
> - **Прочее:** классы в `classes/` с автозагрузкой, события, hooks API, adhoc / scheduled tasks (аналог очередей), Mustache-шаблоны.

---

## 23. Задачи с кодом

> [!tip] Как решать у доски Условие → граничные случаи (пустой ввод, регистр, Unicode) → алгоритм → сложность → код → проверка на примере.

```php
// FizzBuzz
for ($i = 1; $i <= 100; $i++) {
    echo match (true) {
        $i % 15 === 0 => 'FizzBuzz',
        $i % 3 === 0  => 'Fizz',
        $i % 5 === 0  => 'Buzz',
        default       => $i,
    }, PHP_EOL;
}

// Палиндром (Unicode, регистр)
function isPalindrome(string $s): bool
{
    $clean = preg_replace('/[^\p{L}\p{N}]+/u', '', mb_strtolower($s));
    return $clean === implode('', array_reverse(mb_str_split($clean)));
}

// Анаграммы
function isAnagram(string $a, string $b): bool
{
    $norm = static function (string $s): array {
        $c = mb_str_split(mb_strtolower($s)); sort($c); return $c;
    };
    return $norm($a) === $norm($b);
}

// Two Sum — O(n) через хеш-таблицу
function twoSum(array $nums, int $target): ?array
{
    $seen = [];                                // значение → индекс
    foreach ($nums as $i => $n) {
        if (isset($seen[$target - $n])) { return [$seen[$target - $n], $i]; }
        $seen[$n] = $i;
    }
    return null;
}

// Группировка по ключу
function groupBy(array $rows, string $key): array
{
    $result = [];
    foreach ($rows as $row) { $result[$row[$key]][] = $row; }
    return $result;
}

// Скобки (стек)
function validBrackets(string $s): bool
{
    $pairs = [')' => '(', ']' => '[', '}' => '{'];
    $stack = [];
    foreach (str_split($s) as $ch) {
        if (in_array($ch, $pairs, true)) { $stack[] = $ch; }
        elseif (isset($pairs[$ch]) && array_pop($stack) !== $pairs[$ch]) { return false; }
    }
    return $stack === [];
}

// Бинарный поиск
function binarySearch(array $a, int $x): int
{
    [$lo, $hi] = [0, count($a) - 1];
    while ($lo <= $hi) {
        $mid = intdiv($lo + $hi, 2);
        if ($a[$mid] === $x) { return $mid; }
        $a[$mid] < $x ? $lo = $mid + 1 : $hi = $mid - 1;
    }
    return -1;
}
```

---

## 24. Что выведет код

|#|Код|Результат|Почему|
|--:|---|---|---|
|1|`var_dump(0.1 + 0.2 == 0.3);`|`false`|двоичная погрешность float|
|2|`var_dump("abc" == 0);`|PHP 8: `false`; PHP 7: `true`|в 8 число сравнивается со строкой как строка|
|3|`var_dump("10" == "1e1");`|`true`|обе числовые: 10 и 10|
|4|`$i = 5; echo $i++ + ++$i;`|`12`|`5 + 7`|
|5|`var_dump(null ?? 'a' ?? 'b');`|`"a"`|`??` ассоциативен справа, пропускает только `null`|
|6|`var_dump(false ?: 'x');`|`"x"`|`?:` пропускает falsy|
|7|`var_dump((bool)"0.0", (bool)"0", (bool)[]);`|`true, false, false`|falsy строки: только `""` и `"0"`|
|8|`$a=['x'=>1]; $b=$a; $b['x']=2; echo $a['x'];`|`1`|copy-on-write|
|9|`var_dump(-7 % 3, 7 % -3, intdiv(-7, 2), 7 <=> 3);`|`-1, 1, -3, 1`|знак `%` от делимого; `intdiv` усекает к нулю|

> [!question] 10. Что выведет код?
> 
> ```php
> $a = [1, 2, 3];
> foreach ($a as &$v) {}
> foreach ($a as $v) {}
> print_r($a);
> ```
> 
> `[1, 2, 2]`. После первого цикла `$v` — ссылка на последний элемент; второй цикл записывает в него каждое значение.

> [!question] 11. Что вернёт функция?
> 
> ```php
> function f() {
>     try { throw new Exception('a'); }
>     catch (Exception $e) { return 'catch'; }
>     finally { echo 'finally '; }
> }
> echo f();
> ```
> 
> `finally catch`. `finally` выполняется до фактического возврата значения.

> [!question] 12. Что выведет код?
> 
> ```php
> $o1 = new stdClass; $o1->v = 1;
> $o2 = $o1;           $o2->v = 2;
> $o3 = clone $o1;     $o3->v = 3;
> echo $o1->v;
> ```
> 
> `2`. `$o2` указывает на тот же объект; `$o3` — независимая копия.

> [!question] 13. Какой класс вернёт `B::create()`?
> 
> ```php
> class A { public static function create() { return new static; } }
> class B extends A {}
> echo get_class(B::create());
> ```
> 
> `B` (позднее статическое связывание). С `new self` было бы `A`.

---

## 25. Сценарные вопросы

> [!question] Форма регистрации (email, пароль): как сделать безопасно?
> 
> 1. **Транспорт:** HTTPS, POST, PRG-редирект.
> 2. **CSRF:** токен в форме, проверка `hash_equals`.
> 3. **Валидация:** `filter_var(..., FILTER_VALIDATE_EMAIL)`, длина пароля; уникальность email — **уникальным индексом** (единственная защита от гонки).
> 4. **Пароль:** `password_hash`.
> 5. **БД:** prepared statement; ловить нарушение уникальности (SQLSTATE 23000).
> 6. **Сессия:** `session_regenerate_id(true)` при входе.
> 7. **Вывод:** `htmlspecialchars`.
> 8. **Подтверждение email:** токен `random_bytes` со сроком жизни, письмо через очередь.
> 9. **Перебор:** rate limiting, капча.

> [!question] Страница списка заказов отвечает 8 секунд. Что делать?
> 
> 10. Профилирование и лог SQL.
> 11. Скорее всего **N+1** → `JOIN` / `IN`.
> 12. `EXPLAIN` и индексы под фильтры и сортировку.
> 13. Keyset-пагинация вместо большого `OFFSET`.
> 14. Выбирать только нужные колонки, ограничить выборку.
> 15. Кэшировать агрегаты (Redis), HTTP-кэш.
> 16. Тяжёлые отчёты — в очередь.
> 17. Проверить OPcache, версию PHP, настройки FPM.

> [!question] CSV на 2 ГБ нужно импортировать в БД. Как не упасть по памяти?
> 
> - Читать **потоком** (`fgetcsv` / `SplFileObject`) через генератор.
> - Вставлять **пачками** по 500–1000 строк в транзакции.
> - Запускать как CLI-скрипт с возможностью **возобновления** (сохранять номер обработанной строки).
> - Ошибочные строки — в отдельный отчёт.
> - Очень большие объёмы — разбить на части и обрабатывать параллельными воркерами через очередь.

> [!question] Пользователь A видит данные пользователя B, поменяв `?id=` в URL. Что это?
> 
> **IDOR** — нет проверки авторизации на уровне объекта. Решение: `WHERE id = :id AND user_id = :current`, политики / voters / Gates, ответ 404 или 403. UUID вместо id — лишь дополнительная мера, не защита.

> [!question] Как отправлять email после регистрации, чтобы пользователь не ждал?
> 
> Положить сообщение в **очередь** (Redis / RabbitMQ) и сразу ответить; воркер под Supervisor отправляет письмо, при сбое делает retry с задержкой, неудачные попадают в failed-очередь. Для надёжности — **Transactional Outbox**: запись о письме создаётся в одной транзакции с пользователем.

---

## 26. Блиц-вопросы

|Вопрос|Короткий ответ|
|---|---|
|`echo` или `print`?|`echo` — конструкция, несколько аргументов, ничего не возвращает. `print` возвращает `1`, один аргумент|
|`include` или `require`?|`require` — фатальная ошибка при отсутствии файла, `include` — warning|
|`unset($x)` или `$x = null`?|`unset` удаляет переменную (или ключ массива); `null` оставляет её со значением `null`|
|`array_map` или `array_walk`?|`array_map` возвращает новый массив; `array_walk` меняет по ссылке и возвращает `bool`|
|`$this` или `self`?|`$this` — текущий объект, `self` — класс|
|Что делает `final`?|Запрещает наследование класса или переопределение метода|
|Как PHP ищет функцию в namespace?|Сначала в текущем namespace, потом в глобальном. Для классов — только текущий namespace: нужен `use` или `\`|
|`htmlspecialchars` или `addslashes`?|Первое — для вывода в HTML; второе почти не нужно|
|`mysqli` или PDO?|PDO — разные СУБД, именованные плейсхолдеры; `mysqli` — только MySQL|
|`callable` или `Closure`?|`callable` — тип «любой вызываемый»; `Closure` — конкретный класс анонимной функции|
|Когда вызывается `__destruct`?|При уничтожении объекта (refcount = 0) или в конце скрипта|
|`GET` или `POST`?|GET — безопасный, идемпотентный, без побочных эффектов; POST — изменяет состояние|
|Сессия или cookie?|Сессия — данные на сервере; cookie — на клиенте. Секреты в cookie не хранят|
|Можно ли наследовать от нескольких классов?|Нет, один `extends`; несколько интерфейсов и трейтов|
|Чем `DateTime` отличается от `DateTimeImmutable`?|Первый меняется на месте, второй возвращает новый объект|
|Что такое middleware?|Обработчик в цепочке запроса: аутентификация, CORS, логирование|
|Что такое идемпотентность?|Повторный запрос даёт тот же результат (GET, PUT, DELETE)|
|Зачем `composer.lock`?|Фиксирует точные версии для воспроизводимой установки|
|Как защититься от повторной отправки формы?|PRG-редирект и CSRF-токен|

---

## 27. Шпаргалка

### Карта «вопрос → раздел»

|Тема|Раздел|
|---|---|
|Как PHP исполняется, OPcache, FPM|1, 19|
|Сравнения, приведение типов, `??`|3|
|Массивы|5|
|Замыкания, генераторы|6|
|Типизация, enum, `strict_types`|7|
|ООП, трейты, LSB, readonly|8|
|Composer, PSR-4, namespaces|9|
|Исключения, `finally`|10|
|Ссылки, copy-on-write, GC|11|
|Сессии, cookies, заголовки|12|
|PDO, транзакции, N+1|14|
|XSS, CSRF, SQLi, пароли|15|
|SOLID, DI, паттерны|17|
|Тесты, PHPStan|20|
|Версии PHP|21|

### Топ ловушек PHP

> [!warning] Ловушки, о которых любят спрашивать
> 
> 1. `strpos` вернул `0` — это не «не найдено» (`=== false`).
> 2. `empty("0")` → `true`.
> 3. `foreach` по ссылке без `unset($v)`.
> 4. `"abc" == 0` — разный результат в PHP 7 и 8.
> 5. `array_merge` против `+`.
> 6. `array_filter` сохраняет ключи.
> 7. `in_array` без строгого режима.
> 8. `strlen` / `substr` на UTF-8 без `mb_*`.
> 9. `float` и деньги.
> 10. `return` в `finally`.
> 11. Объекты копируются по handle, `clone` поверхностный.
> 12. `self` против `static`.
> 13. `headers already sent`.
> 14. `isset` против `array_key_exists` для `null`.
> 15. `md5` для паролей, `rand()` для токенов, `unserialize` на чужих данных.

### Десять золотых правил

> [!important] Запомнить
> 
> 1. Всегда `declare(strict_types=1)` и типы параметров, возвратов, свойств.
> 2. Строгое сравнение `===`, `in_array(..., true)`, `match` вместо `switch`.
> 3. Данные в SQL — только prepared statements; идентификаторы — белый список.
> 4. Экранировать на выводе: `htmlspecialchars` или автоэкранирование шаблонизатора.
> 5. Пароли — `password_hash`, токены — `random_bytes`, секреты — `hash_equals`.
> 6. Зависимости через конструктор, интерфейсы вместо конкретных классов; без Singleton.
> 7. Composer + PSR-4 + PSR-12; `composer.lock` в репозитории; `composer audit`.
> 8. Исключения для исключительных ситуаций; на проде `display_errors=Off`.
> 9. Сначала измерить, потом оптимизировать: профайлер, `EXPLAIN`, N+1, кэш, OPcache.
> 10. Тяжёлое — в очередь.

### Как отвечать на собеседовании

> [!tip] Стратегия
> 
> - **Сначала принцип, потом деталь:** «`==` приводит типы, `===` нет; я использую `===`, потому что…».
> - **Честно обозначайте границы:** «точную сигнатуру сверил бы по php.net; принцип такой…».
> - **Пример из практики:** проблема → решение → что узнали.
> - **Безопасность и производительность** упоминайте сами: prepared statements и индексы, экранирование, `password_hash`, генераторы для больших данных.
> - **Называйте версии:** «в 8.0 появилось…».
> - **Рассуждайте вслух** на задачах.

### Вопросы интервьюеру

1. Какая версия PHP в продакшене, есть ли план обновления?
2. Используете ли PHPStan / Psalm, PHPCS и PHPUnit в CI?
3. Как устроены код-ревью и деплой: есть ли staging, как откатываетесь?
4. Какие фреймворки и библиотеки, есть ли legacy и как с ним работаете?
5. Как организованы очереди, кэш и мониторинг?
6. Как онбордится новый разработчик, какие задачи на испытательном сроке?