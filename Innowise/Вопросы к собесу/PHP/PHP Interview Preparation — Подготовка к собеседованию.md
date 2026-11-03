---
custom-width: 80
---
## 1. PHP Fundamentals

### 1.1. Что такое PHP

PHP (PHP: Hypertext Preprocessor) — серверный интерпретируемый язык программирования общего назначения, преимущественно используемый для Backend-разработки и создания динамических веб-приложений.

Основные характеристики:

- Dynamically typed — тип переменной определяется во время выполнения.
    
- Weakly typed by default — в некоторых контекстах PHP выполняет автоматическое преобразование типов.
    
- Multi-paradigm — поддерживает процедурное, объектно-ориентированное и функциональное программирование.
    
- Garbage collected — управляет памятью автоматически.
    
- Request-oriented — традиционно веб-запрос обрабатывается отдельным жизненным циклом выполнения PHP.
    
- Extensible — функциональность расширяется через extensions.
    

PHP применяется для:

- REST API и GraphQL API.
    
- Backend-приложений.
    
- CMS: Drupal, WordPress.
    
- E-commerce.
    
- Микросервисов.
    
- CLI-инструментов и фоновых обработчиков.
    
- Интеграции с внешними сервисами.
    

### 1.2. Как работает PHP

В стандартной конфигурации PHP-FPM + Nginx запрос проходит следующие этапы:

1. Клиент отправляет HTTP request.
    
2. Nginx принимает запрос.
    
3. Nginx передаёт PHP-запрос в PHP-FPM через FastCGI.
    
4. PHP-FPM выбирает свободный worker.
    
5. PHP выполняет скрипт.
    
6. Zend Engine преобразует PHP-код в инструкции для выполнения.
    
7. Результат возвращается через PHP-FPM в Nginx.
    
8. Nginx отправляет HTTP response клиенту.
    

Основные компоненты:

|Компонент|Назначение|
|---|---|
|Zend Engine|Исполняет PHP-код|
|PHP Parser|Разбирает исходный код|
|Opcache|Кэширует скомпилированный bytecode|
|PHP-FPM|Управляет PHP-процессами|
|Extensions|Добавляют функциональность|
|SAPI|Интерфейс взаимодействия PHP с окружением|

PHP может работать через разные SAPI:

- CLI — командная строка.
    
- FPM/FastCGI — обработка веб-запросов.
    
- Apache module — интеграция с Apache.
    
- Другие специализированные интерфейсы.
    

### 1.3. Синтаксис PHP

```
<?php

declare(strict_types=1);

$name = 'John';
$age = 30;

echo "Name: {$name}";
echo PHP_EOL;

$result = $age + 10;
```

`<?php` начинает PHP-код.

`declare(strict_types=1)` включает строгую проверку скалярных типов для соответствующего файла в определённых контекстах вызова.

В современных PHP-приложениях рекомендуется использовать `strict_types=1` в исходных файлах, где это возможно.

### 1.4. Variables

Переменные в PHP начинаются с `$`.

```
$name = 'John';
$age = 30;
$price = 19.99;
$isActive = true;
$items = ['Apple', 'Orange'];
```

PHP не требует предварительного объявления переменной или указания её типа.

Переменные имеют динамический тип:

```
$value = 100;
$value = 'Hello';
$value = false;
```

Тип определяется текущим значением.

### 1.5. Constants

Константы — значения, которые нельзя переопределить после объявления.

```
define('APP_NAME', 'My Application');

const API_VERSION = 'v1';

echo APP_NAME;
echo API_VERSION;
```

Важные особенности:

- `define()` позволяет объявлять константы во время выполнения.
    
- `const` объявляется на этапе компиляции и имеет ограничения на контекст использования.
    
- Константы не используют `$`.
    
- Константы класса объявляются через `const`.
    

```
class User
{
    public const ROLE_ADMIN = 'admin';
}
```

### 1.6. Data Types

PHP поддерживает следующие основные типы:

|Категория|Типы|
|---|---|
|Scalar|int, float, string, bool|
|Compound|array, object, callable, iterable|
|Special|null, resource|
|Advanced type declarations|mixed, never, void, static, self, parent|

Пример:

```
$integer = 10;
$float = 10.5;
$string = 'PHP';
$boolean = true;
$array = [1, 2, 3];
$object = new stdClass();
$null = null;
```

#### Integer

Целые числа.

```
$number = 100;
$negative = -50;
$hex = 0xFF;
$binary = 0b1010;
```

Размер `int` зависит от платформы. На современных 64-битных системах обычно используется 64-битный integer.

#### Float

Числа с плавающей точкой.

```
$price = 19.99;
$rate = 0.15;
```

Float использует двоичное представление, поэтому некоторые десятичные значения не могут быть представлены точно.

```
var_dump(0.1 + 0.2);
```

Для финансовых вычислений не следует полагаться на обычный `float`.

Варианты:

- Integer в минимальных денежных единицах, например центах.
    
- Decimal в базе данных.
    
- Специализированные decimal-библиотеки.
    

#### String

```
$name = 'John';

$message = "Hello, {$name}";

$multiline = <<<TEXT
Hello
World
TEXT;
```

Основные функции:

- `strlen()`
    
- `mb_strlen()`
    
- `substr()`
    
- `mb_substr()`
    
- `strpos()`
    
- `str_contains()`
    
- `str_starts_with()`
    
- `str_ends_with()`
    
- `explode()`
    
- `implode()`
    
- `trim()`
    
- `str_replace()`
    
- `sprintf()`
    

Для Unicode необходимо учитывать разницу между байтовой длиной строки и количеством символов.

```
$text = 'Привет';

echo strlen($text);
echo mb_strlen($text);
```

`strlen()` возвращает количество байтов, `mb_strlen()` умеет работать с многобайтовыми кодировками.

#### Boolean

```
$isActive = true;

if ($isActive) {
    echo 'Active';
}
```

Falsy-значения в PHP:

- `false`
    
- `0`
    
- `0.0`
    
- `''`
    
- `'0'`
    
- `[]`
    
- `null`
    

Важно: строка `'0'` является falsy, несмотря на то что это непустая строка.

### 1.7. Type Juggling и Type Casting

Type Juggling — автоматическое преобразование типов.

```
$result = '10' + 5;

var_dump($result); // int(15)
```

Type Casting — явное преобразование.

```
$value = '100';

$integer = (int) $value;
$float = (float) $value;
$string = (string) $integer;
$boolean = (bool) $value;
```

В PHP существуют функции:

- `intval()`
    
- `floatval()`
    
- `strval()`
    
- `boolval()`
    
- `settype()`
    

Для проверки типа:

```
is_int($value);
is_string($value);
is_array($value);
is_object($value);
is_null($value);
```

Для проверки значения:

```
isset($value);
empty($value);
is_null($value);
```

Разница:

|Функция|Поведение|
|---|---|
|isset()|false для неопределённой переменной и null|
|empty()|true для falsy-значений|
|is_null()|Проверяет null|
|array_key_exists()|Проверяет существование ключа массива, даже если значение null|

```
$data = [
    'name' => null,
];

isset($data['name']);             // false
array_key_exists('name', $data); // true
```

### 1.8. Operators

#### Arithmetic

```
$a + $b;
$a - $b;
$a * $b;
$a / $b;
$a % $b;
$a ** $b;
```

#### Comparison

```
$a == $b;
$a === $b;
$a != $b;
$a !== $b;
$a > $b;
$a < $b;
$a >= $b;
$a <= $b;
```

Ключевое различие:

- `==` сравнивает значения с возможным преобразованием типов.
    
- `===` сравнивает значения и типы.
    

```
var_dump(10 == '10');  // true
var_dump(10 === '10'); // false
```

В бизнес-логике обычно предпочтительнее строгое сравнение.

#### Logical

```
&&
||
!
and
or
xor
```

`&&` и `and` имеют разный приоритет операторов.

#### Null coalescing

```
$name = $input['name'] ?? 'Guest';
```

Эквивалентно проверке существования значения на null, но не эквивалентно `isset()` в каждом контексте с массивами и выражениями.

#### Nullsafe operator

```
$city = $user?->getAddress()?->getCity();
```

Если один из объектов в цепочке равен `null`, результатом будет `null`, а дальнейшее обращение по цепочке не выполняется.

#### Spaceship operator

```
$result = $a <=> $b;
```

Возвращает:

- `-1`, если `$a < $b`.
    
- `0`, если значения равны.
    
- `1`, если `$a > $b`.
    

Используется, например, при сортировке.

### 1.9. Control Structures

#### if / elseif / else

```
if ($age >= 18) {
    echo 'Adult';
} elseif ($age >= 13) {
    echo 'Teenager';
} else {
    echo 'Child';
}
```

#### match

`match` — выражение, которое возвращает значение и использует строгое сравнение.

```
$status = match ($code) {
    200 => 'OK',
    404 => 'Not Found',
    500 => 'Server Error',
    default => 'Unknown',
};
```

Отличия от `switch`:

- Строгое сравнение.
    
- Возвращает значение.
    
- Не требует `break`.
    
- Не допускает fall-through.
    
- Необходимо обработать все варианты либо определить `default`.
    

#### Loops

```
for ($i = 0; $i < 5; $i++) {
    echo $i;
}

while ($condition) {
    // ...
}

foreach ($items as $item) {
    echo $item;
}

foreach ($items as $key => $value) {
    echo "{$key}: {$value}";
}
```

При изменении элементов массива по ссылке:

```
foreach ($items as &$item) {
    $item *= 2;
}

unset($item);
```

После `foreach` необходимо удалить ссылочную переменную через `unset()`, иначе она может продолжать ссылаться на последний элемент массива.

### 1.10. Functions

Функция — переиспользуемый блок кода, который может принимать аргументы и возвращать значение.

```
function calculateTotal(float $price, int $quantity): float
{
    return $price * $quantity;
}
```

Аргументы могут иметь:

- Тип.
    
- Значение по умолчанию.
    
- Named arguments.
    
- Variadic arguments.
    
- Ссылочную передачу.
    

```
function createUser(
    string $name,
    int $age = 18,
): array {
    return [
        'name' => $name,
        'age' => $age,
    ];
}
```

Named arguments:

```
createUser(
    age: 25,
    name: 'John',
);
```

Variadic arguments:

```
function sum(int ...$numbers): int
{
    return array_sum($numbers);
}

echo sum(1, 2, 3, 4);
```

Возвращаемый тип:

```
function getName(): string
{
    return 'John';
}
```

Функция может возвращать `void`, если не должна возвращать значение.

```
function logMessage(string $message): void
{
    echo $message;
}
```

### 1.11. Scope

Область видимости определяет, где доступна переменная.

Основные варианты:

- Local scope.
    
- Global scope.
    
- Static local variables.
    
- Function parameters.
    
- Object properties.
    

```
$name = 'Global';

function test(): void
{
    $name = 'Local';

    echo $name;
}
```

Для доступа к глобальной переменной:

```
$name = 'Global';

function test(): void
{
    global $name;

    echo $name;
}
```

Другой вариант:

```
$name = 'Global';

function test(): void
{
    echo $GLOBALS['name'];
}
```

`global` и `$GLOBALS` обычно не рекомендуются для бизнес-логики, поскольку создают скрытые зависимости.

Static local variable:

```
function counter(): int
{
    static $count = 0;

    return ++$count;
}
```

Значение сохраняется между вызовами функции в рамках текущего выполнения скрипта.

### 1.12. References

Reference позволяет нескольким переменным обращаться к одному значению через ссылочную семантику PHP.

```
$a = 10;
$b = &$a;

$b = 20;

echo $a; // 20
```

Передача аргумента по ссылке:

```
function increment(int &$number): void
{
    $number++;
}

$value = 10;

increment($value);

echo $value; // 11
```

References не следует путать с указателями в C/C++.

В PHP reference — механизм управления доступом к значению, а не обычный способ передачи объекта.

Для объектов отдельная модель: переменные объекта содержат идентификатор объекта, и копирование такой переменной обычно не клонирует сам объект.

---

## 2. PHP Data Structures

### 2.1. Arrays

В PHP массив — это ordered map, то есть упорядоченная структура ключ-значение.

Он может работать как:

- Indexed array.
    
- Associative array.
    
- Stack.
    
- Queue.
    
- Hash map.
    
- Multidimensional array.
    

```
$numbers = [10, 20, 30];

$user = [
    'name' => 'John',
    'age' => 30,
];
```

Массивы PHP не являются обычными непрерывными массивами, как в C. Они поддерживают ключи разных типов с определёнными правилами преобразования.

```
$data = [
    'name' => 'John',
    10 => 'value',
    true => 'boolean key',
];
```

Некоторые ключи автоматически преобразуются:

- Integer-like string может преобразоваться в integer при определённых условиях.
    
- Boolean преобразуется в integer.
    
- Float преобразуется в integer.
    
- Null преобразуется в пустую строку.
    

### 2.2. Array Operations

```
$items[] = 'New item';

array_push($items, 'A', 'B');

array_pop($items);

array_shift($items);

array_unshift($items, 'First');
```

Объединение массивов:

```
$a = ['name' => 'John', 'age' => 30];
$b = ['age' => 35, 'city' => 'Berlin'];

$result = $a + $b;
```

Оператор `+` сохраняет значения из левого массива при совпадении ключей.

Для полного объединения с перезаписью:

```
$result = array_merge($a, $b);
```

### 2.3. Array Functions

Часто используемые функции:

|Функция|Назначение|
|---|---|
|array_map()|Преобразование элементов|
|array_filter()|Фильтрация|
|array_reduce()|Агрегация|
|array_walk()|Обход с callback|
|array_column()|Получение колонки|
|array_keys()|Получение ключей|
|array_values()|Получение значений|
|array_key_exists()|Проверка ключа|
|in_array()|Поиск значения|
|array_search()|Поиск с возвратом ключа|
|array_unique()|Удаление дубликатов|
|array_slice()|Получение части массива|
|array_splice()|Изменение части массива|
|array_combine()|Создание массива из двух массивов|
|array_chunk()|Разбиение на части|

Пример:

```
$users = [
    ['name' => 'John', 'active' => true],
    ['name' => 'Anna', 'active' => false],
    ['name' => 'Mike', 'active' => true],
];

$activeUsers = array_filter(
    $users,
    fn(array $user): bool => $user['active'],
);

$names = array_map(
    fn(array $user): string => $user['name'],
    $activeUsers,
);
```

### 2.4. Array Reduce

`array_reduce()` позволяет свести массив к одному значению.

```
$prices = [100, 200, 300];

$total = array_reduce(
    $prices,
    fn(int $carry, int $price): int => $carry + $price,
    0,
);
```

Результат — `600`.

### 2.5. Destructuring

```
$user = ['John', 30];

[$name, $age] = $user;
```

Для ассоциативного массива:

```
$user = [
    'name' => 'John',
    'age' => 30,
];

[
    'name' => $name,
    'age' => $age,
] = $user;
```

### 2.6. Spread Operator

```
$first = [1, 2];
$second = [3, 4];

$result = [...$first, ...$second];
```

Для ассоциативных массивов важно учитывать правила объединения ключей и поведение одинаковых строковых ключей.

### 2.7. Sorting

```
sort($items);
rsort($items);

asort($items);
arsort($items);

ksort($items);
krsort($items);

usort($items, $callback);
uasort($items, $callback);
uksort($items, $callback);
```

Различия:

- `sort()` сортирует значения и переиндексирует массив.
    
- `asort()` сохраняет связь ключей со значениями.
    
- `ksort()` сортирует по ключам.
    
- `usort()` использует пользовательский callback.
    

```
usort(
    $users,
    fn(array $a, array $b): int => $a['age'] <=> $b['age'],
);
```

### 2.8. Array Performance

Для больших объёмов данных важно учитывать:

- Размер элементов.
    
- Количество копирований.
    
- Использование `array_map()` и `array_filter()`.
    
- Необходимость хранения всего набора данных в памяти.
    
- Использование генераторов для потоковой обработки.
    
- Выбор правильной структуры данных.
    

PHP arrays удобны, но могут потреблять значительно больше памяти, чем специализированные структуры.

---

## 3. Advanced PHP Language Features

### 3.1. include, require, include_once, require_once

Эти конструкции подключают PHP-файлы.

```
require_once __DIR__ . '/config.php';

include __DIR__ . '/helpers.php';
```

Разница:

|Конструкция|Повторное подключение|Ошибка при отсутствии|
|---|---|---|
|include|Допускается|Warning|
|require|Допускается|Error|
|include_once|Защищает от повторного подключения|Warning|
|require_once|Защищает от повторного подключения|Error|

Для современных приложений рекомендуется использовать Composer autoloading вместо ручного подключения большого количества классов.

### 3.2. Namespaces

Namespaces позволяют группировать классы, интерфейсы, функции и константы, избегая конфликтов имён.

```
<?php

namespace App\Service;

class UserService
{
}
```

Использование:

```
use App\Service\UserService;

$service = new UserService();
```

Alias:

```
use App\Repository\UserRepository as Repository;
```

Глобальное пространство имён обозначается через `\`.

```
$now = \time();
```

### 3.3. Anonymous Functions

Анонимная функция не имеет имени.

```
$greet = function (string $name): string {
    return "Hello, {$name}";
};

echo $greet('John');
```

### 3.4. Closures

Closure — анонимная функция, которая может захватывать переменные из внешнего контекста.

```
$message = 'Hello';

$greet = function (string $name) use ($message): string {
    return "{$message}, {$name}";
};
```

`use ($message)` захватывает значение переменной на момент создания closure.

`use (&$message)` захватывает переменную по ссылке.

### 3.5. Arrow Functions

Arrow functions — компактный синтаксис для коротких callback-функций.

```
$multiplier = 2;

$calculate = fn(int $value): int => $value * $multiplier;
```

Особенности:

- Автоматически захватывают внешние переменные по значению.
    
- Не используют `use`.
    
- Содержат одно выражение.
    
- Не подходят для сложной многооператорной логики.
    

### 3.6. Callable и Callbacks

Callable — значение, которое может быть вызвано как функция.

```
function process(callable $callback): void
{
    $callback();
}

process(function (): void {
    echo 'Executed';
});
```

Допустимые callable:

- Имя функции.
    
- Closure.
    
- Массив `[object, method]`.
    
- Массив `[ClassName::class, method]`.
    
- Invokable object.
    

Invokable object:

```
class Logger
{
    public function __invoke(string $message): void
    {
        echo $message;
    }
}

$logger = new Logger();

$logger('Hello');
```

### 3.7. Generators

Generator позволяет создавать значения последовательно, не загружая весь набор данных в память.

Используется `yield`.

```
function generateNumbers(int $max): Generator
{
    for ($i = 1; $i <= $max; $i++) {
        yield $i;
    }
}

foreach (generateNumbers(1000000) as $number) {
    // Process number
}
```

Обычный массив:

```
function getNumbers(int $max): array
{
    $result = [];

    for ($i = 1; $i <= $max; $i++) {
        $result[] = $i;
    }

    return $result;
}
```

Generator:

- Не хранит все результаты одновременно.
    
- Возвращает значения по мере обхода.
    
- Удобен для обработки больших файлов и потоков.
    
- Не делает сам по себе тяжёлые вычисления быстрее.
    
- Не заменяет полноценные очереди или асинхронную обработку.
    

### 3.8. Iterators

Iterator — интерфейс для объектов, которые можно обходить.

```
class NumberIterator implements Iterator
{
    private int $position = 0;

    private array $items = [10, 20, 30];

    public function current(): mixed
    {
        return $this->items[$this->position];
    }

    public function key(): int
    {
        return $this->position;
    }

    public function next(): void
    {
        $this->position++;
    }

    public function rewind(): void
    {
        $this->position = 0;
    }

    public function valid(): bool
    {
        return isset($this->items[$this->position]);
    }
}
```

`IteratorAggregate` позволяет вернуть другой iterator:

```
class UserCollection implements IteratorAggregate
{
    public function __construct(
        private array $users,
    ) {
    }

    public function getIterator(): Traversable
    {
        return new ArrayIterator($this->users);
    }
}
```

### 3.9. Fibers

Fibers появились в PHP 8.1.

Они позволяют приостанавливать и возобновлять выполнение кода.

```
$fiber = new Fiber(function (): void {
    echo "Start\n";

    Fiber::suspend('Paused');

    echo "Resumed\n";
});

$value = $fiber->start();

$fiber->resume();
```

Fiber — механизм cooperative multitasking, а не полноценная параллельная обработка.

Он не создаёт автоматически отдельный поток CPU.

Практическое применение зависит от event loop и библиотек, которые управляют переключением выполнения.

### 3.10. Enums

Enums появились в PHP 8.1.

Они позволяют определить ограниченный набор допустимых значений.

```
enum OrderStatus: string
{
    case Pending = 'pending';
    case Paid = 'paid';
    case Cancelled = 'cancelled';
}
```

Использование:

```
function processOrder(OrderStatus $status): void
{
    if ($status === OrderStatus::Paid) {
        // ...
    }
}
```

Backed enums:

- `int`
    
- `string`
    

Pure enums не имеют scalar value.

Enums могут реализовывать интерфейсы и содержать методы.

```
enum OrderStatus: string
{
    case Pending = 'pending';
    case Paid = 'paid';

    public function isFinal(): bool
    {
        return $this === self::Paid;
    }
}
```

### 3.11. Attributes

Attributes появились в PHP 8.0.

Они позволяют добавлять структурированные метаданные к классам, методам, свойствам, параметрам и другим элементам.

```
#[Attribute]
class Route
{
    public function __construct(
        public string $path,
    ) {
    }
}
```

Использование:

```
#[Route('/users')]
class UserController
{
}
```

Получение через Reflection:

```
$reflection = new ReflectionClass(UserController::class);

$attributes = $reflection->getAttributes(Route::class);

$route = $attributes[0]->newInstance();
```

Attributes сами по себе не выполняют бизнес-логику. Framework или application code должен прочитать их и определить поведение.

### 3.12. Reflection API

Reflection позволяет исследовать структуру PHP-классов во время выполнения.

```
$reflection = new ReflectionClass(UserService::class);

foreach ($reflection->getMethods() as $method) {
    echo $method->getName();
}
```

Применение:

- Dependency Injection containers.
    
- ORM.
    
- Serialization.
    
- Attribute processing.
    
- Testing frameworks.
    
- Framework metadata discovery.
    

Недостатки:

- Дополнительная сложность.
    
- Runtime overhead.
    
- Сложность статического анализа.
    
- Риск чрезмерной динамичности.
    

Для повторяющихся операций Reflection metadata обычно кэшируют.

### 3.13. Magic Methods

Основные magic methods:

|Метод|Назначение|
|---|---|
|`__construct()`|Создание объекта|
|`__destruct()`|Уничтожение объекта|
|`__get()`|Чтение недоступного свойства|
|`__set()`|Запись недоступного свойства|
|`__isset()`|Проверка через isset|
|`__unset()`|Удаление недоступного свойства|
|`__call()`|Вызов недоступного метода|
|`__callStatic()`|Статический вызов недоступного метода|
|`__toString()`|Преобразование в строку|
|`__invoke()`|Вызов объекта как функции|
|`__clone()`|Клонирование|
|`__serialize()`|Пользовательская сериализация|
|`__unserialize()`|Восстановление объекта|

Magic methods могут скрывать реальное поведение объекта. Их следует использовать осознанно.

### 3.14. Serialization

```
$data = serialize($object);

$object = unserialize($data);
```

JSON:

```
$json = json_encode($data, JSON_THROW_ON_ERROR);

$data = json_decode(
    $json,
    true,
    512,
    JSON_THROW_ON_ERROR,
);
```

Различия:

|PHP Serialization|JSON|
|---|---|
|Может сохранять структуру PHP-объектов|Стандарт обмена данными|
|PHP-специфичен|Языконезависим|
|Может вызывать magic methods|Не восстанавливает произвольные классы|
|Опасен при работе с недоверенными данными|Безопаснее для обычного обмена, но требует валидации|

Никогда не вызывайте `unserialize()` на недоверенных данных без строгого обоснования и безопасных ограничений.

---

## 4. Object-Oriented Programming

### 4.1. Основные принципы OOP

ООП строится вокруг объектов, которые объединяют состояние и поведение.

Основные принципы:

1. Encapsulation.
    
2. Abstraction.
    
3. Inheritance.
    
4. Polymorphism.
    

### 4.2. Classes and Objects

Класс описывает структуру и поведение объекта.

Объект — экземпляр класса.

```
class User
{
    public function __construct(
        private string $name,
        private string $email,
    ) {
    }

    public function getName(): string
    {
        return $this->name;
    }
}

$user = new User('John', 'john@example.com');
```

### 4.3. Visibility

|Modifier|Доступ|
|---|---|
|public|Из любого контекста|
|protected|Внутри класса и наследников|
|private|Только внутри объявившего класса|

По умолчанию свойства лучше делать `private` или `protected`, предоставляя контролируемый API.

### 4.4. Encapsulation

Инкапсуляция скрывает внутреннюю реализацию и предоставляет контролируемый интерфейс.

```
class BankAccount
{
    public function __construct(
        private int $balance,
    ) {
        if ($balance < 0) {
            throw new InvalidArgumentException();
        }
    }

    public function deposit(int $amount): void
    {
        if ($amount <= 0) {
            throw new InvalidArgumentException();
        }

        $this->balance += $amount;
    }

    public function getBalance(): int
    {
        return $this->balance;
    }
}
```

Внешний код не может напрямую присвоить произвольное значение приватному свойству.

### 4.5. Inheritance

Наследование позволяет создавать класс на основе другого класса.

```
class Animal
{
    public function eat(): void
    {
        echo 'Eating';
    }
}

class Dog extends Animal
{
    public function bark(): void
    {
        echo 'Woof';
    }
}
```

Наследование полезно при наличии реального отношения is-a.

Не следует использовать наследование только ради повторного использования кода.

### 4.6. Polymorphism

Полиморфизм позволяет работать с разными реализациями через общий контракт.

```
interface PaymentGateway
{
    public function pay(int $amount): void;
}

class StripeGateway implements PaymentGateway
{
    public function pay(int $amount): void
    {
        // Stripe integration
    }
}

class PaypalGateway implements PaymentGateway
{
    public function pay(int $amount): void
    {
        // PayPal integration
    }
}
```

Клиентский код зависит от интерфейса:

```
class PaymentService
{
    public function __construct(
        private PaymentGateway $gateway,
    ) {
    }

    public function process(int $amount): void
    {
        $this->gateway->pay($amount);
    }
}
```

### 4.7. Abstraction

Абстракция позволяет определить существенные характеристики объекта без раскрытия деталей реализации.

Используются:

- Interfaces.
    
- Abstract classes.
    
- Contracts.
    
- Domain abstractions.
    

### 4.8. Abstract Classes

Абстрактный класс нельзя создать напрямую.

```
abstract class Report
{
    abstract public function generate(): string;

    public function save(): void
    {
        // Common logic
    }
}
```

Класс-наследник обязан реализовать абстрактные методы.

### 4.9. Interfaces

Интерфейс определяет контракт поведения.

```
interface LoggerInterface
{
    public function log(string $message): void;
}
```

Класс может реализовывать несколько интерфейсов.

```
class FileLogger implements LoggerInterface
{
    public function log(string $message): void
    {
        // Write to file
    }
}
```

### 4.10. Abstract Class vs Interface

|Abstract Class|Interface|
|---|---|
|Может содержать состояние|Не предназначен для хранения состояния объекта|
|Может содержать реализацию методов|Определяет контракт методов и допускает некоторые константы|
|Только одно наследование класса|Можно реализовать несколько интерфейсов|
|Подходит для общей базовой реализации|Подходит для определения контракта|

Выбор зависит от архитектурной задачи.

### 4.11. Traits

Traits позволяют переиспользовать методы между классами без классического наследования.

```
trait Timestampable
{
    public function getTimestamp(): int
    {
        return time();
    }
}

class Order
{
    use Timestampable;
}
```

Traits не являются полноценной заменой композиции.

При конфликте методов можно использовать:

- `insteadof`
    
- `as`
    

```
class Example
{
    use FirstTrait, SecondTrait {
        FirstTrait::run insteadof SecondTrait;
        SecondTrait::run as runSecond;
    }
}
```

### 4.12. Composition vs Inheritance

Composition — объект использует другие объекты для выполнения задач.

```
class OrderService
{
    public function __construct(
        private PaymentGateway $gateway,
    ) {
    }
}
```

Преимущества композиции:

- Меньшая связанность.
    
- Простая замена реализаций.
    
- Удобное тестирование.
    
- Более гибкая архитектура.
    

В современных приложениях композиция обычно предпочтительнее глубоких иерархий наследования.

### 4.13. Static

Static properties и methods принадлежат классу, а не отдельному экземпляру.

```
class Counter
{
    private static int $count = 0;

    public static function increment(): void
    {
        self::$count++;
    }

    public static function getCount(): int
    {
        return self::$count;
    }
}
```

Чрезмерное использование static усложняет тестирование и управление зависимостями.

### 4.14. self, static, parent

- `self` — класс, в котором объявлен метод.
    
- `static` — late static binding, фактический класс вызова.
    
- `parent` — родительский класс.
    

```
class ParentClass
{
    public static function getName(): string
    {
        return self::class;
    }

    public static function getLateName(): string
    {
        return static::class;
    }
}

class ChildClass extends ParentClass
{
}

echo ChildClass::getName();
echo ChildClass::getLateName();
```

`self::class` возвращает имя класса, где метод был объявлен. `static::class` учитывает наследника.

### 4.15. Final

`final` запрещает переопределение метода или наследование класса.

```
final class SecurityService
{
}
```

```
class UserService
{
    final public function validate(): bool
    {
        return true;
    }
}
```

Используется для защиты инвариантов и ограничения расширения API.

### 4.16. Readonly

Readonly properties появились в PHP 8.1, readonly classes — в PHP 8.2.

```
class UserDto
{
    public function __construct(
        public readonly string $name,
        public readonly int $age,
    ) {
    }
}
```

Readonly property нельзя повторно присвоить после инициализации.

Readonly не означает глубокую неизменяемость объекта: если свойство содержит объект, внутреннее состояние этого объекта может изменяться.

### 4.17. Constructor Property Promotion

```
class User
{
    public function __construct(
        private string $name,
        private string $email,
    ) {
    }
}
```

Позволяет объявить и инициализировать свойства непосредственно в конструкторе.

### 4.18. Cloning

```
$copy = clone $original;
```

Обычное клонирование создаёт поверхностную копию.

Вложенные объекты остаются теми же объектами, если явно не клонировать их в `__clone()`.

### 4.19. Object Identity

Два объекта могут иметь одинаковые значения свойств, но быть разными экземплярами.

```
$a = new stdClass();
$b = new stdClass();

var_dump($a === $b); // false
```

`===` для объектов проверяет идентичность экземпляров.

`==` сравнивает объекты по классу и значениям свойств с учётом правил PHP.

### 4.20. Property Hooks — PHP 8.4

Property Hooks позволяют определить поведение при чтении и записи свойства.

```
class User
{
    public string $name {
        get => strtolower($this->name);
        set => trim($value);
    }
}
```

Property Hooks могут использоваться для:

- Валидации.
    
- Нормализации.
    
- Вычисляемых значений.
    
- Контролируемого доступа к свойствам.
    

Важно: это не то же самое, что обычные getter/setter-методы.

---

## 5. PHP Type System

### 5.1. Type Declarations

PHP поддерживает декларации типов для:

- Параметров функций.
    
- Возвращаемых значений.
    
- Свойств.
    
- Констант класса.
    
- Конструкторов и других членов класса.
    

```
function calculate(int $a, int $b): int
{
    return $a + $b;
}
```

### 5.2. Scalar Types

- `int`
    
- `float`
    
- `string`
    
- `bool`
    

### 5.3. Nullable Types

```
function findUser(int $id): ?User
{
    return null;
}
```

`?User` эквивалентен `User|null`.

### 5.4. Union Types

Union type допускает несколько типов.

```
function process(int|string $value): int|string
{
    return $value;
}
```

### 5.5. Intersection Types

Intersection type требует, чтобы значение соответствовало нескольким типам одновременно.

```
interface Logger
{
    public function log(string $message): void;
}

interface Formatter
{
    public function format(string $message): string;
}

function process(Logger&Formatter $service): void
{
}
```

### 5.6. DNF Types

Disjunctive Normal Form позволяет комбинировать union и intersection types.

```
function process((Logger&Formatter)|null $service): void
{
}
```

### 5.7. mixed

`mixed` означает, что значение может иметь любой тип.

```
function process(mixed $value): void
{
}
```

Не следует использовать `mixed` повсеместно. Это снижает возможности статического анализа.

### 5.8. never

`never` означает, что функция никогда не возвращает управление обычным образом.

```
function terminate(): never
{
    throw new RuntimeException();
}
```

Также `never` может использоваться для функций, которые завершают выполнение через `exit`.

### 5.9. void

Функция не возвращает полезного значения.

```
function logMessage(string $message): void
{
    echo $message;
}
```

### 5.10. static, self, parent

Эти типы полезны для деклараций, связанных с классами и наследованием.

```
class Factory
{
    public static function create(): static
    {
        return new static();
    }
}
```

### 5.11. strict_types

```
declare(strict_types=1);
```

Строгая типизация влияет на проверку скалярных типов при вызовах функций.

Важный нюанс: режим проверки аргументов определяется файлом вызывающего кода, а проверка возвращаемого значения — контекстом файла, в котором объявлена функция.

`strict_types` не заменяет полноценную статическую типизацию и не отключает все преобразования PHP.

### 5.12. Covariance and Contravariance

Covariance позволяет возвращать более конкретный тип, чем определён в родительском контракте.

Contravariance позволяет принимать более общий тип аргумента.

Принцип:

- Return type может сужаться.
    
- Parameter type может расширяться.
    

Это связано с принципом подстановки Лисков (LSP).

### 5.13. PHPDoc

PHPDoc используется для дополнительного описания типов, generic-подобных структур и контрактов.

```
/**
 * @param array<int, User> $users
 * @return array<string, User>
 */
function indexUsers(array $users): array
{
    $result = [];

    foreach ($users as $user) {
        $result[$user->getEmail()] = $user;
    }

    return $result;
}
```

PHPDoc особенно важен для PHPStan и Psalm.

### 5.14. Generics в PHP

Встроенной полноценной системы generics в PHP пока нет.

Но статические анализаторы поддерживают generics через PHPDoc.

```
/**
 * @template T
 */
class Collection
{
    /**
     * @param list<T> $items
     */
    public function __construct(
        private array $items,
    ) {
    }

    /**
     * @return list<T>
     */
    public function all(): array
    {
        return $this->items;
    }
}
```

### 5.15. Static Analysis

Инструменты:

- PHPStan.
    
- Psalm.
    

Они позволяют находить ошибки до запуска программы.

Проверяют:

- Типы аргументов.
    
- Возвращаемые типы.
    
- Возможные null.
    
- Недоступные методы.
    
- Неправильное использование массивов.
    
- Нарушения контрактов.
    
- Часть логических ошибок.
    

Для Senior-разработчика статический анализ — важная часть качества проекта.

---

## 6. Error Handling and Exceptions

### 6.1. Error vs Exception

В PHP есть общий интерфейс `Throwable`.

```
Throwable
├── Error
│   ├── TypeError
│   ├── ParseError
│   ├── ArgumentCountError
│   └── ...
└── Exception
    ├── RuntimeException
    ├── LogicException
    └── ...
```

`Exception` и `Error` — разные ветви.

Обе могут быть пойманы через `catch (Throwable $e)`.

### 6.2. Exceptions

```
try {
    $service->process();
} catch (RuntimeException $exception) {
    // Handle expected runtime failure
} finally {
    // Cleanup
}
```

`finally` обычно выполняется при выходе из `try` или `catch`, в том числе при исключении.

Однако принудительное завершение процесса и некоторые аварийные ситуации могут не позволить выполнить cleanup.

### 6.3. Custom Exceptions

```
class PaymentFailedException extends RuntimeException
{
}
```

Использование:

```
if (!$paymentSucceeded) {
    throw new PaymentFailedException(
        'Payment processing failed',
    );
}
```

Преимущества:

- Чёткие бизнес-сценарии ошибок.
    
- Упрощение обработки.
    
- Разделение технических и бизнес-ошибок.
    
- Более понятные API-контракты.
    

### 6.4. Exception Hierarchy

Рекомендуется проектировать иерархию исключений по смыслу.

Пример:

```
DomainException
├── OrderException
│   ├── InvalidOrderException
│   └── OrderAlreadyPaidException
└── PaymentException
    ├── PaymentDeclinedException
    └── PaymentProviderException
```

Не нужно создавать отдельный exception для каждого метода без необходимости.

### 6.5. Error Handling

PHP поддерживает:

- `error_reporting()`
    
- `set_error_handler()`
    
- `set_exception_handler()`
    
- `trigger_error()`
    
- `error_log()`
    

`set_error_handler()` может преобразовывать обрабатываемые PHP diagnostics в exceptions.

Не все ошибки могут быть перехвачены пользовательским error handler.

### 6.6. Production Error Handling

В production:

- Не показывать пользователю stack trace.
    
- Не выводить секреты и внутренние пути.
    
- Логировать ошибки.
    
- Использовать correlation/request ID.
    
- Возвращать корректный HTTP status code.
    
- Разделять публичное сообщение и внутренние детали.
    

Пример:

```
try {
    $orderService->create($data);
} catch (InvalidOrderException $exception) {
    $logger->warning(
        'Invalid order',
        ['exception' => $exception],
    );

    return new Response(
        'Invalid order',
        422,
    );
} catch (Throwable $exception) {
    $logger->error(
        'Unexpected error',
        ['exception' => $exception],
    );

    return new Response(
        'Internal Server Error',
        500,
    );
}
```

---

## 7. PHP Runtime and Memory Management

### 7.1. Zend Engine

Zend Engine — runtime engine PHP, отвечающий за выполнение PHP-кода.

Упрощённый процесс:

1. Lexing.
    
2. Parsing.
    
3. AST generation.
    
4. Compilation to opcodes.
    
5. Execution by Zend VM.
    

В реальности детали зависят от версии PHP и конфигурации.

### 7.2. Opcache

Opcache хранит скомпилированный bytecode в shared memory.

Без Opcache PHP обычно должен повторно разбирать и компилировать файлы при новых запросах.

С Opcache:

- Снижается повторная работа компилятора.
    
- Уменьшается CPU overhead.
    
- Улучшается производительность.
    
- Bytecode может использоваться несколькими PHP workers.
    

В production обычно рекомендуется включать Opcache.

### 7.3. JIT

JIT появился в PHP 8.0.

Он может компилировать некоторые участки bytecode в машинный код.

Однако JIT не гарантирует ускорение обычных веб-приложений.

Для типичных CRUD/API-систем часто важнее:

- SQL-запросы.
    
- Кэширование.
    
- Алгоритмы.
    
- Сетевые задержки.
    
- Сериализация.
    
- Профилирование.
    

### 7.4. Memory Management

PHP использует автоматическое управление памятью.

Основные механизмы:

- Reference counting.
    
- Copy-on-write.
    
- Garbage collection.
    

### 7.5. Reference Counting

Zend Engine отслеживает количество ссылок на значение.

Когда значение больше не используется, память может быть освобождена.

### 7.6. Copy-on-Write

При копировании массива PHP обычно не копирует все данные немедленно.

```
$a = [1, 2, 3];

$b = $a;
```

Пока массивы не изменяются, PHP может использовать общие данные.

При изменении одного из массивов выполняется копирование необходимых данных.

Это называется Copy-on-Write.

### 7.7. Garbage Collection

Reference counting не решает все случаи циклических ссылок.

```
$a = new stdClass();
$b = new stdClass();

$a->other = $b;
$b->other = $a;
```

Garbage Collector обнаруживает циклические структуры, которые больше не доступны из программы.

### 7.8. Memory Optimization

Для оптимизации:

- Не хранить большие результаты без необходимости.
    
- Использовать генераторы.
    
- Обрабатывать файлы потоково.
    
- Избегать ненужного копирования.
    
- Ограничивать размер batch processing.
    
- Не создавать лишние объекты.
    
- Использовать профилирование, а не предположения.
    

```
$handle = fopen('large.csv', 'rb');

while (($row = fgetcsv($handle)) !== false) {
    // Process one row
}

fclose($handle);
```

### 7.9. PHP-FPM

PHP-FPM управляет пулом PHP workers.

Основные параметры:

- `pm`
    
- `pm.max_children`
    
- `pm.start_servers`
    
- `pm.min_spare_servers`
    
- `pm.max_spare_servers`
    
- `pm.max_requests`
    

Режимы:

|Режим|Назначение|
|---|---|
|static|Фиксированное количество workers|
|dynamic|Количество workers изменяется в пределах настроек|
|ondemand|Workers запускаются по необходимости|

`pm.max_children` ограничивает максимальное количество одновременно работающих workers.

Слишком большое значение может привести к нехватке памяти.

Упрощённая оценка:

`max_children ≈ доступная память для PHP / среднее потребление памяти одним worker`

На практике нужно учитывать пики памяти, системные процессы и другие сервисы.

### 7.10. Request Lifecycle

В традиционной модели PHP-FPM:

- Worker принимает запрос.
    
- Выполняет PHP-код.
    
- Завершает request lifecycle.
    
- Освобождает request-scoped состояние.
    
- Worker может обработать следующий запрос.
    

Нельзя считать, что каждый запрос обязательно запускает новый системный процесс.

Долгоживущие процессы и приложения на event loop имеют другие требования к управлению состоянием.

### 7.11. Long-running PHP Processes

Примеры:

- Queue workers.
    
- RoadRunner.
    
- Swoole.
    
- FrankenPHP.
    
- CLI daemons.
    

В таких приложениях необходимо особенно внимательно относиться к:

- Утечкам памяти.
    
- Статическим свойствам.
    
- Глобальному состоянию.
    
- Повторному использованию объектов.
    
- Очистке request-scoped данных.
    
- Исключениям и восстановлению worker.
    

---

## 8. Composer and PHP Standards

### 8.1. Composer

Composer — стандартный dependency manager PHP.

Основные задачи:

- Установка библиотек.
    
- Управление версиями.
    
- Dependency resolution.
    
- Autoloading.
    
- Работа с пакетами.
    

### 8.2. composer.json

```
{
    "name": "app/project",
    "type": "project",
    "require": {
        "php": "^8.4",
        "guzzlehttp/guzzle": "^7.9"
    },
    "require-dev": {
        "phpunit/phpunit": "^11.0"
    },
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

### 8.3. composer.lock

`composer.lock` фиксирует конкретные версии установленных зависимостей.

Зачем он нужен:

- Повторяемые установки.
    
- Одинаковые версии в разных окружениях.
    
- Предсказуемые deployments.
    
- Контроль изменений зависимостей.
    

Для приложений lock-файл обычно коммитится в Git.

### 8.4. Composer Commands

```
composer install
composer update
composer require vendor/package
composer remove vendor/package
composer dump-autoload
composer validate
composer show
composer audit
```

Разница:

- `install` устанавливает версии из lock-файла.
    
- `update` разрешает зависимости заново и изменяет lock-файл.
    

В CI/CD обычно используется `composer install`, а не безусловный `composer update`.

### 8.5. Semantic Versioning

Формат:

`MAJOR.MINOR.PATCH`

- MAJOR — breaking changes.
    
- MINOR — новые возможности с обратной совместимостью.
    
- PATCH — исправления с обратной совместимостью.
    

Примеры Composer constraints:

- `^1.2` — совместимые обновления в пределах major 1.
    
- `~1.2.3` — обновления до следующего minor.
    
- `1.2.3` — точная версия.
    
- `*` — слишком широкое ограничение для многих production-сценариев.
    

### 8.6. Autoloading

Composer поддерживает:

- PSR-4.
    
- PSR-0.
    
- Classmap.
    
- Files autoload.
    

PSR-4 — основной современный стандарт autoloading.

```
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

Класс `App\Service\UserService` будет находиться в `src/Service/UserService.php` при соблюдении соглашения о регистре имён и каталогов.

### 8.7. PSR Standards

|Standard|Назначение|
|---|---|
|PSR-1|Basic Coding Standard|
|PSR-3|Logger Interface|
|PSR-4|Autoloading|
|PSR-7|HTTP Message Interfaces|
|PSR-11|Container Interface|
|PSR-12|Extended Coding Style|
|PSR-15|HTTP Server Request Handlers|
|PSR-17|HTTP Factories|
|PSR-18|HTTP Client|
|PSR-20|Clock|

PSR — PHP Standards Recommendations.

Они повышают совместимость библиотек и стандартизируют взаимодействие компонентов.

---

## 9. Database and PHP

### 9.1. Database Fundamentals

Backend-приложения обычно используют реляционные базы данных:

- MySQL.
    
- MariaDB.
    
- PostgreSQL.
    

Основные понятия:

- Tables.
    
- Rows.
    
- Columns.
    
- Primary keys.
    
- Foreign keys.
    
- Indexes.
    
- Constraints.
    
- Transactions.
    
- Views.
    
- Joins.
    

### 9.2. PDO

PDO — PHP Data Objects.

Предоставляет единый интерфейс работы с разными базами данных через соответствующие drivers.

```
$pdo = new PDO(
    'mysql:host=localhost;dbname=app;charset=utf8mb4',
    'user',
    'password',
    [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    ],
);
```

### 9.3. Prepared Statements

Prepared statements разделяют SQL-код и пользовательские значения.

```
$stmt = $pdo->prepare(
    'SELECT * FROM users WHERE email = :email',
);

$stmt->execute([
    'email' => $email,
]);

$user = $stmt->fetch();
```

Это основной способ защиты от SQL Injection при работе с динамическими значениями.

Важно: placeholders нельзя использовать для имён таблиц, колонок и SQL keywords. Такие динамические части нужно выбирать из заранее разрешённого списка.

### 9.4. SQL Injection

Опасный вариант:

```
$sql = "SELECT * FROM users WHERE email = '$email'";
```

Если `$email` содержит вредоносный SQL, запрос может изменить своё поведение.

Безопасный вариант — prepared statement.

### 9.5. Transactions

Транзакция объединяет несколько операций в единое логическое действие.

```
$pdo->beginTransaction();

try {
    $pdo->prepare(
        'UPDATE accounts SET balance = balance - ? WHERE id = ?',
    )->execute([100, 1]);

    $pdo->prepare(
        'UPDATE accounts SET balance = balance + ? WHERE id = ?',
    )->execute([100, 2]);

    $pdo->commit();
} catch (Throwable $exception) {
    if ($pdo->inTransaction()) {
        $pdo->rollBack();
    }

    throw $exception;
}
```

### 9.6. ACID

|Property|Значение|
|---|---|
|Atomicity|Все операции выполняются целиком или откатываются|
|Consistency|Данные сохраняют заданные ограничения|
|Isolation|Параллельные транзакции контролируют взаимное влияние|
|Durability|Подтверждённые изменения сохраняются|

### 9.7. Isolation Levels

Основные уровни:

- Read Uncommitted.
    
- Read Committed.
    
- Repeatable Read.
    
- Serializable.
    

Аномалии:

- Dirty Read.
    
- Non-repeatable Read.
    
- Phantom Read.
    
- Lost Update.
    

Конкретное поведение зависит от СУБД и её реализации MVCC/locking.

### 9.8. Indexes

Индексы ускоряют поиск и сортировку, но имеют стоимость:

- Дополнительная память и место на диске.
    
- Более дорогие INSERT/UPDATE/DELETE.
    
- Необходимость обслуживания.
    
- Возможность выбора неоптимального плана.
    

Основные типы:

- B-tree.
    
- Hash.
    
- Composite.
    
- Unique.
    
- Full-text.
    
- Partial indexes — поддерживаются не всеми СУБД одинаково.
    

### 9.9. N+1 Problem

N+1 возникает, когда приложение выполняет один запрос для получения списка, а затем по одному запросу на каждый элемент.

Например:

1. Получить 100 заказов.
    
2. Для каждого заказа получить пользователя.
    
3. Итого 101 запрос.
    

Решения:

- JOIN.
    
- Eager loading.
    
- Batch queries.
    
- DataLoader-подход.
    
- Корректное проектирование ORM-запросов.
    

### 9.10. Repository Pattern

Repository инкапсулирует получение и сохранение данных.

```
interface UserRepositoryInterface
{
    public function findById(int $id): ?User;

    public function save(User $user): void;
}
```

Реализация:

```
class PdoUserRepository implements UserRepositoryInterface
{
    public function __construct(
        private PDO $pdo,
    ) {
    }

    public function findById(int $id): ?User
    {
        $stmt = $this->pdo->prepare(
            'SELECT id, name FROM users WHERE id = ?',
        );

        $stmt->execute([$id]);

        $row = $stmt->fetch();

        if ($row === false) {
            return null;
        }

        return new User(
            id: (int) $row['id'],
            name: $row['name'],
        );
    }

    public function save(User $user): void
    {
        // Persistence implementation
    }
}
```

Repository полезен, когда он выражает понятную границу доступа к данным. Не нужно создавать бессмысленный repository для каждого SQL-запроса.

---

## 10. HTTP and Web Development

### 10.1. HTTP

HTTP — протокол взаимодействия клиента и сервера.

Основные части HTTP request:

- Method.
    
- URL.
    
- Headers.
    
- Body.
    

Основные части HTTP response:

- Status code.
    
- Headers.
    
- Body.
    

### 10.2. HTTP Methods

|Method|Назначение|
|---|---|
|GET|Получение ресурса|
|POST|Создание или выполнение операции|
|PUT|Полная замена ресурса|
|PATCH|Частичное обновление|
|DELETE|Удаление|
|HEAD|Получение заголовков|
|OPTIONS|Получение доступных возможностей|

Идемпотентность:

- GET — идемпотентен.
    
- PUT — идемпотентен по семантике.
    
- DELETE — идемпотентен по семантике.
    
- POST — обычно не идемпотентен.
    
- PATCH — зависит от операции.
    

### 10.3. HTTP Status Codes

|Code|Значение|
|---|---|
|200|OK|
|201|Created|
|202|Accepted|
|204|No Content|
|301|Moved Permanently|
|302|Found|
|304|Not Modified|
|400|Bad Request|
|401|Unauthorized|
|403|Forbidden|
|404|Not Found|
|409|Conflict|
|422|Unprocessable Content|
|429|Too Many Requests|
|500|Internal Server Error|
|502|Bad Gateway|
|503|Service Unavailable|
|504|Gateway Timeout|

401 означает отсутствие действительной аутентификации. 403 означает, что сервер отказывает в доступе, даже если запрос распознан.

### 10.4. REST API

REST — архитектурный стиль взаимодействия с ресурсами через стандартные HTTP-механизмы.

Пример:

```
GET    /api/users
GET    /api/users/10
POST   /api/users
PATCH  /api/users/10
DELETE /api/users/10
```

Основные принципы:

- Resource-oriented URLs.
    
- Stateless communication.
    
- Standard HTTP methods.
    
- Consistent representations.
    
- Cacheability.
    
- Layered architecture.
    

REST — не просто JSON поверх HTTP.

### 10.5. Request Data

PHP предоставляет:

```
$_GET;
$_POST;
$_COOKIE;
$_FILES;
$_SERVER;
$_SESSION;
```

Для JSON body:

```
$data = json_decode(
    file_get_contents('php://input'),
    true,
    512,
    JSON_THROW_ON_ERROR,
);
```

Полученные данные нужно валидировать до использования.

### 10.6. Cookies

Cookie хранится на стороне клиента и отправляется серверу в HTTP-запросах согласно правилам браузера.

Настройки:

- `Secure`
    
- `HttpOnly`
    
- `SameSite`
    
- `Path`
    
- `Domain`
    
- `Expires`
    
- `Max-Age`
    

### 10.7. Sessions

Session позволяет хранить серверное состояние, связанное с клиентом.

```
session_start();

$_SESSION['user_id'] = 10;
```

В production необходимо:

- Использовать HTTPS.
    
- Включать `HttpOnly`.
    
- Включать `Secure`.
    
- Настраивать `SameSite`.
    
- Регенерировать session ID после login.
    
- Настраивать expiration.
    
- Не хранить лишние чувствительные данные.
    

### 10.8. Authentication vs Authorization

Authentication — кто пользователь?

Authorization — что пользователю разрешено делать?

Пример:

- Authentication: пользователь вошёл в систему.
    
- Authorization: пользователь имеет право редактировать конкретный заказ.
    

Проверка доступа должна учитывать не только роль, но и объект, контекст и бизнес-правила.

### 10.9. JWT

JWT — JSON Web Token.

Структура:

```
Header.Payload.Signature
```

JWT обычно содержит claims.

Важно:

- Payload не является зашифрованным по умолчанию.
    
- Подпись подтверждает целостность.
    
- Необходимо проверять алгоритм.
    
- Проверять issuer, audience и expiration.
    
- Использовать безопасное управление ключами.
    
- Продумывать отзыв и ротацию токенов.
    

JWT не является автоматически более безопасным, чем server-side sessions.

### 10.10. CORS

CORS — механизм браузерной политики доступа к ресурсам между origins.

Основные заголовки:

- `Access-Control-Allow-Origin`
    
- `Access-Control-Allow-Methods`
    
- `Access-Control-Allow-Headers`
    
- `Access-Control-Allow-Credentials`
    

CORS не заменяет authentication и authorization.

### 10.11. CSRF

CSRF — атака, при которой браузер пользователя отправляет нежелательный запрос на сайт, где пользователь уже аутентифицирован.

Защита:

- CSRF tokens.
    
- SameSite cookies.
    
- Проверка Origin/Referer в подходящих сценариях.
    
- Не использовать GET для изменения состояния.
    
- Дополнительная проверка критичных операций.
    

### 10.12. cURL and HTTP Clients

PHP может выполнять HTTP-запросы через cURL.

```
$curl = curl_init('https://api.example.com/users');

curl_setopt_array($curl, [
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_TIMEOUT => 10,
    CURLOPT_CONNECTTIMEOUT => 3,
]);

$response = curl_exec($curl);

if ($response === false) {
    throw new RuntimeException(curl_error($curl));
}

$statusCode = curl_getinfo(
    $curl,
    CURLINFO_RESPONSE_CODE,
);

curl_close($curl);
```

Для приложений обычно удобнее использовать HTTP client library, например Guzzle.

Необходимо настраивать:

- Connection timeout.
    
- Request timeout.
    
- Retry policy.
    
- TLS verification.
    
- Logging.
    
- Circuit breaker для критичных интеграций.
    

---

## 11. PHP Security

### 11.1. Security Principles

Основные принципы:

- Least Privilege.
    
- Defense in Depth.
    
- Secure by Default.
    
- Input Validation.
    
- Output Encoding.
    
- Secrets Management.
    
- Fail Securely.
    

### 11.2. SQL Injection

Защита:

- Prepared statements.
    
- Parameter binding.
    
- Allowlist для динамических SQL identifiers.
    
- Минимальные права DB-пользователя.
    

### 11.3. XSS

XSS позволяет внедрять вредоносный JavaScript в страницы приложения.

Типы:

- Stored XSS.
    
- Reflected XSS.
    
- DOM-based XSS.
    

Защита зависит от контекста вывода:

```
echo htmlspecialchars(
    $userInput,
    ENT_QUOTES | ENT_SUBSTITUTE,
    'UTF-8',
);
```

Для HTML, JavaScript, CSS и URL используются разные правила encoding.

Нельзя считать одну функцию универсальной защитой для любого контекста.

### 11.4. CSRF

Защита через токены и настройки cookie.

Важно: CSRF актуален прежде всего там, где браузер автоматически прикрепляет credentials.

### 11.5. SSRF

SSRF позволяет злоумышленнику заставить сервер обращаться к нежелательным ресурсам.

Защита:

- Allowlist доменов.
    
- Проверка IP-адресов.
    
- Блокировка private/internal ranges.
    
- Контроль redirects.
    
- Ограничение протоколов.
    
- Egress network policies.
    

Особенно опасно разрешать произвольные URL в HTTP clients.

### 11.6. Path Traversal

Пример опасного сценария:

```
$file = $_GET['file'];

readfile('/var/app/files/' . $file);
```

Злоумышленник может попытаться обратиться к файлам за пределами разрешённого каталога.

Защита:

- Не принимать произвольные пути.
    
- Использовать идентификаторы файлов.
    
- Проверять разрешённую директорию.
    
- Использовать allowlist.
    
- Учитывать symlinks.
    

### 11.7. Command Injection

Опасный пример:

```
exec('convert ' . $filename);
```

Ввод пользователя может изменить команду.

Лучше избегать shell execution, если задача может быть решена PHP-библиотекой.

Если shell необходим:

- Использовать `escapeshellarg()` для отдельных аргументов.
    
- Не формировать произвольные команды из пользовательского ввода.
    
- Применять allowlist.
    
- Ограничивать права процесса.
    

### 11.8. Password Hashing

Нельзя хранить пароли в plaintext или использовать MD5/SHA1.

```
$hash = password_hash(
    $password,
    PASSWORD_DEFAULT,
);
```

Проверка:

```
if (password_verify($password, $hash)) {
    // Authenticated
}
```

Обновление хеша:

```
if (password_needs_rehash($hash, PASSWORD_DEFAULT)) {
    $hash = password_hash(
        $password,
        PASSWORD_DEFAULT,
    );
}
```

### 11.9. File Upload Security

Проверки:

- MIME type по содержимому.
    
- Размер файла.
    
- Разрешённые расширения.
    
- Безопасное имя.
    
- Права доступа.
    
- Запрет исполнения загруженных файлов.
    
- Хранение вне web root, если возможно.
    
- Проверка пользовательского доступа к скачиванию.
    

Нельзя доверять `$_FILES['type']` как единственному источнику MIME type.

### 11.10. Cryptography

Для случайных значений:

```
$token = bin2hex(random_bytes(32));
```

Для проверки подписи:

```
$signature = hash_hmac(
    'sha256',
    $payload,
    $secret,
);
```

Для сравнения секретных значений:

```
hash_equals($expected, $actual);
```

Не следует самостоятельно реализовывать криптографические алгоритмы. Используйте проверенные библиотеки и стандартные API.

---

## 12. Architecture and Design Patterns

### 12.1. SOLID

#### S — Single Responsibility Principle

Класс должен иметь одну основную ответственность и одну осмысленную причину для изменения.

Плохой пример: `UserService` одновременно выполняет SQL, отправляет email, формирует HTML и управляет HTTP response.

Хороший вариант — разделить обязанности между application service, repository, mailer и presentation layer.

#### O — Open/Closed Principle

Компоненты должны быть открыты для расширения и закрыты для модификации.

Например, новая платёжная система добавляется через новую реализацию `PaymentGateway`, а не через изменение большого `if/else`.

#### L — Liskov Substitution Principle

Наследник должен корректно заменять базовый тип, не нарушая его контракт.

Нельзя усиливать предусловия или ослаблять постусловия так, чтобы клиентский код ломался.

#### I — Interface Segregation Principle

Лучше несколько небольших интерфейсов, чем один большой интерфейс с методами, которые не нужны всем реализациям.

#### D — Dependency Inversion Principle

Высокоуровневые модули должны зависеть от абстракций, а не от конкретных реализаций.

### 12.2. DRY, KISS, YAGNI

- DRY — не дублировать знания и правила.
    
- KISS — выбирать простые решения.
    
- YAGNI — не реализовывать функциональность до появления реальной необходимости.
    

DRY не означает, что любой похожий код нужно немедленно объединять. Иногда похожие участки имеют разные причины для изменения.

### 12.3. Dependency Injection

Dependency Injection — передача зависимостей объекту извне.

```
class OrderService
{
    public function __construct(
        private OrderRepository $repository,
        private PaymentGateway $gateway,
    ) {
    }
}
```

Преимущества:

- Слабая связанность.
    
- Тестируемость.
    
- Явные зависимости.
    
- Возможность замены реализации.
    
- Упрощение управления жизненным циклом объектов.
    

### 12.4. Service Container

Service Container — объект, который управляет созданием и предоставлением сервисов.

Он может поддерживать:

- Autowiring.
    
- Constructor injection.
    
- Service definitions.
    
- Aliases.
    
- Factories.
    
- Scopes/lifetimes.
    

DI — принцип, Container — один из инструментов реализации.

### 12.5. Factory Pattern

Factory инкапсулирует создание объектов.

```
interface PaymentGateway
{
    public function pay(int $amount): void;
}

class PaymentGatewayFactory
{
    public function create(string $provider): PaymentGateway
    {
        return match ($provider) {
            'stripe' => new StripeGateway(),
            'paypal' => new PaypalGateway(),
            default => throw new InvalidArgumentException(),
        };
    }
}
```

В реальных проектах создание объектов с зависимостями часто делегируют DI container.

### 12.6. Strategy Pattern

Strategy инкапсулирует взаимозаменяемые алгоритмы.

```
interface DiscountStrategy
{
    public function calculate(int $price): int;
}

class RegularDiscount implements DiscountStrategy
{
    public function calculate(int $price): int
    {
        return $price;
    }
}

class PremiumDiscount implements DiscountStrategy
{
    public function calculate(int $price): int
    {
        return (int) ($price * 0.9);
    }
}
```

### 12.7. Observer Pattern

Observer позволяет подписчикам реагировать на событие.

Примеры:

- Отправка email после регистрации.
    
- Запись audit log после изменения заказа.
    
- Обновление статистики.
    
- Запуск фоновой задачи.
    

### 12.8. Adapter Pattern

Adapter позволяет использовать несовместимый внешний интерфейс через собственный контракт.

Например, интеграция нескольких payment providers через единый `PaymentGateway`.

### 12.9. Decorator Pattern

Decorator добавляет поведение объекту без изменения его исходного класса.

Пример: обернуть repository в caching decorator.

### 12.10. Repository Pattern

Инкапсулирует доступ к данным и отделяет application/domain logic от persistence implementation.

### 12.11. Unit of Work

Unit of Work отслеживает изменения объектов и объединяет их в одну операцию сохранения.

Часто используется ORM для управления транзакциями и изменёнными сущностями.

### 12.12. MVC

MVC разделяет приложение на:

- Model.
    
- View.
    
- Controller.
    

Controller принимает запрос и координирует выполнение операции.

Model представляет данные и бизнес-логику в зависимости от конкретной реализации архитектуры.

View отвечает за представление данных.

MVC — паттерн организации приложения, а не готовая архитектура всей системы.

### 12.13. Layered Architecture

Типичное разделение:

```
Presentation Layer
        |
Application Layer
        |
Domain Layer
        |
Infrastructure Layer
```

Presentation:

- Controllers.
    
- HTTP requests/responses.
    
- Serializers.
    

Application:

- Use cases.
    
- Application services.
    
- Command handlers.
    

Domain:

- Entities.
    
- Value Objects.
    
- Domain Services.
    
- Domain Events.
    

Infrastructure:

- Database.
    
- Redis.
    
- Message brokers.
    
- External APIs.
    
- File storage.
    

### 12.14. Clean Architecture

Основная идея — бизнес-правила не должны зависеть от инфраструктуры.

Зависимости направляются внутрь:

- Domain не зависит от Framework.
    
- Application зависит от Domain.
    
- Infrastructure реализует необходимые интерфейсы.
    
- Presentation вызывает application use cases.
    

Это помогает тестировать бизнес-логику отдельно от базы данных и HTTP.

### 12.15. Domain-Driven Design

DDD помогает моделировать сложную предметную область.

Основные понятия:

- Ubiquitous Language.
    
- Bounded Context.
    
- Entity.
    
- Value Object.
    
- Aggregate.
    
- Aggregate Root.
    
- Repository.
    
- Domain Service.
    
- Domain Event.
    

#### Entity

Объект с идентичностью.

Два объекта могут иметь одинаковые свойства, но оставаться разными сущностями из-за разных ID.

#### Value Object

Объект, определяемый значением, а не идентичностью.

Пример: Money, EmailAddress, DateRange.

#### Aggregate

Группа связанных объектов, изменения которой контролируются через Aggregate Root.

Aggregate Root обеспечивает соблюдение бизнес-инвариантов.

Не следует делать агрегат чрезмерно большим.

---

## 13. Testing and Code Quality

### 13.1. Types of Testing

|Type|Назначение|
|---|---|
|Unit|Проверка небольшого изолированного компонента|
|Integration|Проверка взаимодействия компонентов|
|Functional|Проверка пользовательского сценария|
|End-to-end|Проверка полного потока через систему|
|Performance|Проверка скорости и нагрузки|
|Security|Проверка уязвимостей|

### 13.2. PHPUnit

PHPUnit — основной инструмент тестирования PHP.

```
use PHPUnit\Framework\TestCase;

class CalculatorTest extends TestCase
{
    public function testAddition(): void
    {
        $calculator = new Calculator();

        self::assertSame(
            5,
            $calculator->add(2, 3),
        );
    }
}
```

### 13.3. Test Doubles

Основные виды:

- Dummy.
    
- Stub.
    
- Mock.
    
- Spy.
    
- Fake.
    

Stub предоставляет заранее заданные ответы.

Mock проверяет ожидаемые взаимодействия.

Fake — упрощённая рабочая реализация, например in-memory repository.

### 13.4. Unit Testing

Unit test должен проверять конкретное поведение изолированного компонента.

Хороший unit test:

- Независим.
    
- Детерминирован.
    
- Быстро выполняется.
    
- Не требует реальной внешней инфраструктуры.
    
- Проверяет поведение, а не внутреннюю реализацию без необходимости.
    

### 13.5. Integration Testing

Проверяет реальное взаимодействие компонентов.

Например:

- Repository + Database.
    
- HTTP client + test server.
    
- Queue producer + broker.
    
- Application service + persistence.
    

### 13.6. TDD

Test-Driven Development:

1. Red — написать падающий тест.
    
2. Green — реализовать минимальный код.
    
3. Refactor — улучшить реализацию без изменения поведения.
    

TDD не означает, что каждый технический шаг обязательно должен сопровождаться отдельным тестом. Важно, чтобы тесты направляли разработку поведения.

### 13.7. Code Coverage

Coverage показывает, какая часть кода выполнялась во время тестов.

Высокий coverage не гарантирует качество тестирования.

Важно проверять:

- Критичные бизнес-сценарии.
    
- Граничные значения.
    
- Ошибочные сценарии.
    
- Конкурентные операции.
    
- Права доступа.
    
- Интеграции.
    

### 13.8. Static Analysis

Инструменты:

- PHPStan.
    
- Psalm.
    

Они анализируют код без выполнения приложения.

### 13.9. Coding Standards

Инструменты:

- PHP_CodeSniffer.
    
- PHP-CS-Fixer.
    
- PHPStan.
    
- Psalm.
    
- Rector.
    

PHPCS проверяет соответствие стандартам.

PHP-CS-Fixer автоматически форматирует код.

Rector помогает выполнять автоматические преобразования и миграции.

### 13.10. Debugging

Xdebug поддерживает:

- Breakpoints.
    
- Step debugging.
    
- Stack traces.
    
- Variable inspection.
    
- Profiling.
    
- Code coverage.
    

Для production debugging лучше использовать структурированные логи, tracing и metrics, а не подключать интерактивный debugger к пользовательским запросам.

### 13.11. Observability

Три основных направления:

- Logs.
    
- Metrics.
    
- Traces.
    

Примеры:

- Logs: ошибка обработки заказа.
    
- Metrics: количество запросов в секунду.
    
- Traces: путь одного запроса через несколько сервисов.
    

Полезные метрики:

- Latency.
    
- Throughput.
    
- Error rate.
    
- CPU usage.
    
- Memory usage.
    
- Queue depth.
    
- Database query duration.
    

---

## 14. Advanced Backend Development

### 14.1. Caching

Кэширование сохраняет результаты вычислений или запросов, чтобы повторно не выполнять дорогие операции.

Варианты:

- Application cache.
    
- Database query cache.
    
- HTTP cache.
    
- Object cache.
    
- Distributed cache.
    
- CDN cache.
    

### 14.2. Cache Aside

Типичная схема:

1. Приложение проверяет кэш.
    
2. Если значение есть — возвращает его.
    
3. Если значения нет — обращается к БД.
    
4. Сохраняет результат в кэш.
    
5. Возвращает результат.
    

```
$value = $cache->get($key);

if ($value === null) {
    $value = $repository->find($id);

    $cache->set($key, $value, 300);
}
```

### 14.3. Cache Invalidation

Основная сложность кэширования — корректная инвалидация.

Стратегии:

- TTL.
    
- Explicit invalidation.
    
- Tag-based invalidation.
    
- Versioned keys.
    
- Event-driven invalidation.
    

Важно избегать ситуации, когда устаревшие данные продолжают возвращаться после изменения источника.

### 14.4. Redis

Redis — in-memory data store.

Применение:

- Cache.
    
- Sessions.
    
- Queues.
    
- Rate limiting.
    
- Distributed locks.
    
- Counters.
    
- Temporary data.
    

Структуры:

- Strings.
    
- Lists.
    
- Sets.
    
- Sorted Sets.
    
- Hashes.
    
- Streams.
    

Redis не должен автоматически становиться основной базой данных для всех сценариев.

### 14.5. Message Queues

Очереди позволяют отделить обработку задачи от основного HTTP request.

Пример:

1. Пользователь создаёт заказ.
    
2. Приложение сохраняет заказ.
    
3. Создаёт сообщение в очереди.
    
4. Worker получает сообщение.
    
5. Worker отправляет email или выполняет интеграцию.
    

Преимущества:

- Снижение времени HTTP response.
    
- Независимое масштабирование workers.
    
- Повторная обработка.
    
- Буферизация нагрузки.
    

### 14.6. RabbitMQ

RabbitMQ — message broker, поддерживающий AMQP.

Основные компоненты:

- Producer.
    
- Exchange.
    
- Queue.
    
- Binding.
    
- Consumer.
    
- Acknowledgement.
    

Типы exchanges:

- Direct.
    
- Fanout.
    
- Topic.
    
- Headers.
    

### 14.7. Delivery Semantics

Основные модели:

- At-most-once.
    
- At-least-once.
    
- Exactly-once.
    

Exactly-once processing трудно гарантировать в распределённых системах.

На практике часто используют at-least-once delivery и идемпотентную обработку.

### 14.8. Idempotency

Операция идемпотентна, если повторное выполнение не приводит к дополнительному изменению результата после первого успешного выполнения.

Пример:

```
POST /payments
Idempotency-Key: 7d9c...
```

Сервер хранит ключ и результат операции.

При повторном запросе возвращает ранее сохранённый результат, не создавая второй платёж.

### 14.9. Transactional Outbox

Проблема:

1. Записать заказ в БД.
    
2. Отправить событие в брокер.
    
3. Процесс падает между операциями.
    

Возможна ситуация, когда заказ сохранён, но событие не отправлено.

Transactional Outbox решает это:

- Заказ и outbox event записываются в одной DB transaction.
    
- Отдельный publisher читает outbox.
    
- Отправляет сообщения в брокер.
    
- Помечает события обработанными.
    

Это уменьшает риск потери событий между базой данных и брокером.

### 14.10. Retry and Dead Letter Queue

Retry используется для повторения временно неудачных операций.

Рекомендуется:

- Exponential backoff.
    
- Jitter.
    
- Ограниченное количество попыток.
    
- Разделение временных и постоянных ошибок.
    
- Dead Letter Queue для неисправимых сообщений.
    

Нельзя бесконечно повторять операцию без анализа причины ошибки.

### 14.11. Cron and Background Workers

Cron запускает периодические задачи.

Примеры:

- Очистка старых данных.
    
- Отправка отчётов.
    
- Проверка подписок.
    
- Обновление статистики.
    

Queue worker обрабатывает задачи асинхронно.

Cron подходит для расписания, queue worker — для обработки потока задач.

### 14.12. Event-Driven Architecture

Компоненты взаимодействуют через события.

Событие описывает факт, который уже произошёл.

Пример:

```
OrderPlaced
    |
    +--> SendConfirmationEmail
    |
    +--> UpdateAnalytics
    |
    +--> ReserveInventory
```

Важно разделять:

- Domain Events.
    
- Integration Events.
    
- Application Events.
    

События должны иметь стабильный контракт и продуманную схему версионирования.

### 14.13. WebSockets

WebSocket обеспечивает двустороннее соединение между клиентом и сервером.

Применение:

- Чаты.
    
- Live notifications.
    
- Real-time dashboards.
    
- Multiplayer applications.
    

В отличие от обычного HTTP request-response, соединение остаётся открытым.

### 14.14. Rate Limiting

Rate limiting ограничивает количество запросов.

Алгоритмы:

- Fixed Window.
    
- Sliding Window.
    
- Token Bucket.
    
- Leaky Bucket.
    

Используется для:

- Защиты API.
    
- Ограничения brute force.
    
- Защиты внешних интеграций.
    
- Контроля потребления ресурсов.
    

### 14.15. Pagination

Основные варианты:

- Offset pagination.
    
- Cursor pagination.
    

Offset:

```
SELECT *
FROM users
ORDER BY id
LIMIT 20 OFFSET 40;
```

Cursor:

```
SELECT *
FROM users
WHERE id > :last_id
ORDER BY id
LIMIT 20;
```

Cursor pagination обычно лучше масштабируется для больших и изменяющихся наборов данных, если используется подходящий стабильный индекс и порядок сортировки.

### 14.16. Database Locking

Варианты:

- Optimistic locking.
    
- Pessimistic locking.
    

Optimistic locking использует version или updated timestamp.

Pessimistic locking блокирует запись на время операции.

```
SELECT *
FROM accounts
WHERE id = 10
FOR UPDATE;
```

`FOR UPDATE` применяется внутри транзакции и зависит от возможностей СУБД.

### 14.17. Race Conditions

Race condition возникает, когда результат зависит от порядка выполнения параллельных операций.

Пример:

1. Два запроса одновременно читают баланс 100.
    
2. Оба пытаются списать 80.
    
3. Оба считают операцию допустимой.
    
4. Возникает некорректный результат.
    

Решения:

- Transactions.
    
- Row-level locking.
    
- Atomic SQL updates.
    
- Optimistic locking.
    
- Idempotency.
    
- Unique constraints.
    

### 14.18. Distributed Systems

Основные проблемы:

- Network failures.
    
- Partial failures.
    
- Latency.
    
- Retries.
    
- Duplicate messages.
    
- Data consistency.
    
- Clock differences.
    
- Service discovery.
    
- Cascading failures.
    

Важные паттерны:

- Circuit Breaker.
    
- Retry.
    
- Timeout.
    
- Bulkhead.
    
- Saga.
    
- Outbox.
    
- Idempotent Consumer.
    

### 14.19. Circuit Breaker

Circuit Breaker временно прекращает обращения к неисправному сервису.

Состояния:

- Closed.
    
- Open.
    
- Half-open.
    

Цель — не допустить каскадного отказа системы.

---

## 15. Frameworks and PHP Ecosystem

### 15.1. Laravel

Laravel — PHP framework, предоставляющий готовые инструменты для разработки приложений.

Основные компоненты:

- Routing.
    
- Controllers.
    
- Middleware.
    
- Service Container.
    
- Eloquent ORM.
    
- Migrations.
    
- Queues.
    
- Events.
    
- Jobs.
    
- Validation.
    
- Authentication.
    
- Cache.
    
- Testing.
    

### 15.2. Symfony

Symfony — PHP framework и набор переиспользуемых компонентов.

Основные компоненты:

- HttpFoundation.
    
- HttpKernel.
    
- Routing.
    
- DependencyInjection.
    
- EventDispatcher.
    
- Console.
    
- Serializer.
    
- Validator.
    
- Messenger.
    
- Security.
    
- Doctrine integration.
    

### 15.3. Dependency Injection Container

В Symfony container создаёт и связывает сервисы.

Пример конфигурации:

```
services:
  App\Service\OrderService:
    arguments:
      $repository: '@App\Repository\OrderRepository'
```

Контейнер может использовать autowiring и autoconfiguration.

### 15.4. Middleware

Middleware обрабатывает HTTP request до или после основного handler.

Применение:

- Authentication.
    
- Authorization.
    
- Logging.
    
- Rate limiting.
    
- CORS.
    
- Request ID.
    
- Error handling.
    

### 15.5. ORM

ORM связывает объекты приложения с реляционной базой данных.

Примеры:

- Doctrine ORM.
    
- Eloquent.
    

Преимущества:

- Работа с объектами.
    
- Упрощение CRUD.
    
- Relationships.
    
- Unit of Work.
    
- Identity Map.
    

Недостатки:

- Сложные запросы могут быть неэффективными.
    
- Возможны N+1.
    
- Не всегда очевидно, какой SQL выполняется.
    
- Нужны знания самой СУБД.
    

Senior-разработчик должен понимать SQL независимо от ORM.

### 15.6. Migrations

Migrations позволяют версионировать изменения структуры базы данных.

Пример:

```
final class Version20260101000000
{
    public function up(): void
    {
        // Create table
    }

    public function down(): void
    {
        // Revert change
    }
}
```

В production migrations необходимо проектировать с учётом:

- Backward compatibility.
    
- Больших таблиц.
    
- Locking.
    
- Deployment order.
    
- Rollback strategy.
    

Для крупных изменений часто используется expand-and-contract:

1. Добавить новую структуру.
    
2. Поддерживать старый и новый формат.
    
3. Перенести данные.
    
4. Переключить приложение.
    
5. Удалить старую структуру в отдельном релизе.
    

### 15.7. Drupal Architecture

Drupal — CMS и framework, построенный на PHP и Symfony components.

Основные API:

- Entity API.
    
- Field API.
    
- Plugin API.
    
- Form API.
    
- Render API.
    
- Configuration API.
    
- Services and Dependency Injection.
    
- Events API.
    
- Hooks.
    
- Routing.
    
- Access API.
    
- Cache API.
    
- Queue API.
    
- Batch API.
    
- Migrate API.
    
- REST API.
    
- JSON.
    

### 15.8. Drupal Services

Сервисы определяются в `*.services.yml`.

```
services:
  app.order_manager:
    class: Drupal\app\Service\OrderManager
    arguments:
      - '@entity_type.manager'
```

Зависимости передаются через constructor injection.

Прямой вызов `\Drupal::service()` допустим в некоторых контекстах, но constructor injection предпочтительнее для тестируемости и явного управления зависимостями.

### 15.9. Drupal Entities

Entity — объект предметных данных Drupal.

Основные типы:

- Content entities.
    
- Configuration entities.
    

Content entities обычно представляют содержимое с полями и revision/translation support.

Configuration entities представляют конфигурационные объекты, например типы материалов или представления в соответствующих моделях Drupal.

### 15.10. Drupal Plugin API

Plugin API позволяет создавать расширяемые реализации определённых типов поведения.

Примеры:

- Blocks.
    
- Field widgets.
    
- Field formatters.
    
- Views plugins.
    
- Queue workers.
    

### 15.11. Drupal Cache API

Основные понятия:

- Cache tags.
    
- Cache contexts.
    
- Cache max-age.
    
- Cache metadata bubbling.
    

Cache tags отвечают за инвалидацию при изменении данных.

Cache contexts определяют, для какого контекста пользователя или запроса отличается результат.

Max-age определяет срок действия кэша.

### 15.12. Drupal Configuration Management

Конфигурация экспортируется и импортируется через YAML.

```
drush cex
drush cim
drush cr
```

В типичном workflow:

1. Изменить конфигурацию.
    
2. Экспортировать её.
    
3. Закоммитить YAML.
    
4. Развернуть код.
    
5. Импортировать конфигурацию.
    
6. Очистить кэш при необходимости.
    

---

## 16. Senior-Level Engineering Practices

### 16.1. Code Review

Senior-разработчик проверяет не только синтаксис.

Основные критерии:

- Correctness.
    
- Security.
    
- Performance.
    
- Maintainability.
    
- Testability.
    
- Backward compatibility.
    
- Error handling.
    
- Observability.
    
- Operational impact.
    

Вопросы при review:

- Что произойдёт при null?
    
- Что будет при повторном запросе?
    
- Возможна ли race condition?
    
- Не возникнет ли N+1?
    
- Что произойдёт при недоступности внешнего сервиса?
    
- Не раскрываются ли чувствительные данные?
    
- Есть ли обратная совместимость?
    
- Как откатить изменение?
    

### 16.2. Refactoring

Refactoring изменяет внутреннюю структуру кода без изменения внешнего поведения.

Примеры:

- Extract Method.
    
- Extract Class.
    
- Replace Conditional with Polymorphism.
    
- Introduce Parameter Object.
    
- Replace Magic Number with Constant.
    
- Dependency Injection.
    
- Reduce Coupling.
    

Перед серьёзным рефакторингом желательно иметь тесты, фиксирующие текущее поведение.

### 16.3. Technical Debt

Technical Debt — последствия решений, которые ускорили текущую разработку, но увеличили будущую стоимость изменений.

Примеры:

- Дублирование бизнес-логики.
    
- Отсутствие тестов.
    
- Непрозрачная архитектура.
    
- Устаревшие зависимости.
    
- Слишком большие классы.
    
- Неправильные границы модулей.
    

Senior должен уметь оценивать технический долг и приоритизировать его устранение.

### 16.4. Backward Compatibility

Изменение может быть технически правильным, но сломать существующих клиентов.

Нужно учитывать:

- Public API.
    
- Database schema.
    
- Event contracts.
    
- Message formats.
    
- Configuration.
    
- External integrations.
    

Для безопасных изменений применяются versioning, deprecation и staged migrations.

### 16.5. Performance Optimization

Правильный порядок:

1. Измерить проблему.
    
2. Найти bottleneck.
    
3. Сформулировать гипотезу.
    
4. Изменить код или инфраструктуру.
    
5. Повторить измерения.
    
6. Проверить отсутствие регрессий.
    

Инструменты:

- Xdebug profiler.
    
- Blackfire.
    
- Tideways.
    
- APM.
    
- Database EXPLAIN.
    
- Application metrics.
    

Оптимизировать без измерений — плохая инженерная практика.

### 16.6. Scalability

Vertical scaling — увеличение ресурсов одного сервера.

Horizontal scaling — увеличение количества экземпляров приложения.

Для горизонтального масштабирования обычно нужны:

- Stateless application instances.
    
- Shared session storage.
    
- Centralized cache.
    
- Shared file storage.
    
- Message broker.
    
- Load balancer.
    
- Database scaling strategy.
    

### 16.7. High Availability

Высокая доступность достигается через:

- Redundancy.
    
- Health checks.
    
- Failover.
    
- Replication.
    
- Monitoring.
    
- Backups.
    
- Disaster recovery.
    

Backup без проверенного восстановления не гарантирует возможность восстановить систему.

### 16.8. Deployment Strategy

Основные варианты:

- Rolling deployment.
    
- Blue-green deployment.
    
- Canary deployment.
    

Нужно учитывать совместимость старой и новой версии приложения с базой данных и сообщениями в очередях.

### 16.9. CI/CD

Типичный pipeline:

```
Commit
  |
  v
Composer install
  |
  v
Coding Standards
  |
  v
Static Analysis
  |
  v
Unit Tests
  |
  v
Integration Tests
  |
  v
Build Artifact
  |
  v
Deploy
  |
  v
Smoke Tests
```

CI — автоматическая проверка изменений.

CD — автоматизированная доставка и/или развёртывание изменений.

### 16.10. Architecture Decision Records

ADR фиксирует важные архитектурные решения.

Обычно включает:

- Context.
    
- Decision.
    
- Alternatives.
    
- Consequences.
    

Это помогает команде понимать, почему было принято определённое решение.

---

## 17. Senior PHP Interview Questions

### 17.1. PHP Language

**1. What is the difference between** `**==**` **and** `**===**`**?**

`==` сравнивает значения с возможным type juggling. `===` проверяет и значение, и тип без такого преобразования. В большинстве бизнес-сценариев предпочтительнее строгое сравнение.

**2. What is the difference between** `**isset()**` **and** `**array_key_exists()**`**?**

`isset()` возвращает false, если ключ отсутствует или его значение равно null. `array_key_exists()` проверяет именно наличие ключа, даже если значение null.

**3. What is the difference between** `**include**` **and** `**require**`**?**

Обе конструкции подключают файл. При отсутствии файла `include` генерирует warning, а `require` — ошибку, прекращающую нормальное выполнение. В современных проектах для классов используется Composer autoloading.

**4. What is the difference between a closure and an arrow function?**

Closure может содержать произвольный блок кода и захватывает внешние переменные через `use`. Arrow function имеет компактный синтаксис и автоматически захватывает внешние переменные по значению.

**5. What is a generator?**

Generator возвращает значения последовательно через `yield`, не создавая заранее весь массив. Это полезно для потоковой обработки больших наборов данных.

**6. What is late static binding?**

Late static binding позволяет использовать фактический класс вызова через `static`, в отличие от `self`, который связан с классом объявления метода.

### 17.2. OOP

**7. What is the difference between an abstract class and an interface?**

Abstract class может содержать общую реализацию и состояние. Interface определяет контракт, который могут реализовывать разные классы. Класс может реализовывать несколько интерфейсов, но наследоваться только от одного класса.

**8. What is composition over inheritance?**

Composition означает построение поведения через сотрудничество объектов. Она обычно уменьшает связанность и делает архитектуру более гибкой по сравнению с глубокой иерархией наследования.

**9. What is Dependency Injection?**

Dependency Injection — передача зависимостей объекту извне. Это снижает связанность, упрощает тестирование и позволяет заменять реализации.

**10. What is the difference between** `**self**` **and** `**static**`**?**

`self` относится к классу, в котором объявлен метод. `static` учитывает фактический класс вызова и поддерживает late static binding.

**11. What is the difference between an Entity and a Value Object?**

Entity имеет идентичность, обычно определяемую ID. Value Object определяется своими значениями и обычно проектируется как неизменяемый объект.

### 17.3. Runtime and Performance

**12. What is Opcache?**

Opcache хранит скомпилированные инструкции PHP в shared memory и позволяет повторно использовать их между запросами, уменьшая overhead компиляции.

**13. What is Copy-on-Write?**

PHP может разделять данные между копиями значений и создавать отдельную копию только при изменении. Это позволяет уменьшить ненужное потребление памяти.

**14. What is the difference between PHP-FPM and PHP CLI?**

CLI запускает PHP из командной строки. PHP-FPM управляет worker processes, которые обрабатывают веб-запросы через FastCGI.

**15. How would you optimize a slow PHP application?**

Сначала измерил бы производительность и определил bottleneck. Затем проверил SQL-запросы, N+1, внешние вызовы, алгоритмы, кэширование, сериализацию и использование памяти. После изменений повторил бы измерения.

### 17.4. Database

**16. What is a transaction?**

Транзакция объединяет несколько операций в одну логическую единицу, которая должна удовлетворять ACID.

**17. What is the N+1 problem?**

Это ситуация, когда один запрос получает список сущностей, а затем выполняется дополнительный запрос для каждого элемента. Решения — JOIN, eager loading и batch queries.

**18. What is the difference between optimistic and pessimistic locking?**

Optimistic locking обнаруживает конфликт при сохранении, обычно через version field. Pessimistic locking блокирует запись на время операции.

**19. Why are indexes important?**

Индексы ускоряют поиск и сортировку, но увеличивают место хранения и стоимость изменений данных. Их следует выбирать на основании реальных запросов и EXPLAIN.

### 17.5. Architecture

**20. What is the Dependency Inversion Principle?**

Высокоуровневые модули должны зависеть от абстракций, а не от конкретных низкоуровневых реализаций.

**21. What is the difference between an application service and a domain service?**

Application service координирует выполнение use case. Domain service содержит бизнес-операцию, которая не принадлежит естественным образом одной Entity или Value Object.

**22. What is the Repository Pattern?**

Repository предоставляет абстракцию доступа к данным, отделяя application/domain logic от конкретной persistence implementation.

**23. What is the Transactional Outbox Pattern?**

Он гарантирует, что изменение бизнес-данных и запись события в outbox выполняются в одной транзакции. Отдельный процесс публикует события в брокер.

### 17.6. Security

**24. How do you prevent SQL Injection?**

Использую prepared statements и parameter binding. Динамические имена таблиц и колонок выбираю из allowlist, поскольку SQL placeholders не предназначены для таких идентификаторов.

**25. What is the difference between authentication and authorization?**

Authentication устанавливает личность пользователя. Authorization определяет, какие действия ему разрешены.

**26. How do you store passwords securely?**

Использую `password_hash()` и `password_verify()`. Не храню пароли в plaintext и не применяю обычные быстрые хеш-функции вроде MD5 или SHA1.

**27. Is JWT encrypted?**

Обычный signed JWT не шифрует payload. Его содержимое можно прочитать. Подпись защищает целостность, но не конфиденциальность.

### 17.7. Distributed Systems

**28. What is idempotency?**

Повторное выполнение операции не должно создавать дополнительные эффекты после успешного первого выполнения. Для этого применяются idempotency keys и сохранение результатов операций.

**29. What is a Circuit Breaker?**

Это паттерн, который временно блокирует обращения к неисправному внешнему сервису, чтобы избежать каскадного отказа.

**30. How do you handle duplicate messages in a queue?**

Использую идемпотентных consumers, уникальные идентификаторы сообщений, ограничения БД и сохранение статуса обработки.

**31. What is eventual consistency?**

Данные в разных компонентах системы могут временно отличаться, но при успешной доставке событий и отсутствии новых изменений со временем приходят к согласованному состоянию.

---

## 18. Practical Senior PHP Tasks

Для Senior-собеседования важно уметь не только отвечать на вопросы, но и проектировать решения.

### Task 1. Design a Payment System

Нужно спроектировать систему оплаты с несколькими провайдерами.

Что следует предусмотреть:

- `PaymentGateway` interface.
    
- Отдельные реализации провайдеров.
    
- Dependency Injection.
    
- Payment entity.
    
- Transaction management.
    
- Idempotency key.
    
- Retry policy.
    
- Webhook verification.
    
- Logging.
    
- Reconciliation process.
    

### Task 2. Optimize a Slow API

API возвращает 1000 заказов и выполняет тысячи SQL-запросов.

План:

1. Измерить время и количество запросов.
    
2. Найти N+1.
    
3. Проверить индексы.
    
4. Использовать eager loading или JOIN.
    
5. Ввести pagination.
    
6. Добавить кэширование, если оно оправдано.
    
7. Проверить нагрузку повторно.
    

### Task 3. Design a Notification System

Нужно отправлять email, SMS и push notifications.

Возможная архитектура:

```
Application
    |
NotificationService
    |
NotificationInterface
    |
    +--> EmailNotification
    +--> SmsNotification
    +--> PushNotification
```

Дальнейшая обработка может выполняться через очередь.

### Task 4. Design an Order Processing System

Компоненты:

- Order.
    
- OrderRepository.
    
- PaymentService.
    
- InventoryService.
    
- OrderPlaced event.
    
- Message Broker.
    
- Notification Worker.
    
- Outbox Publisher.
    

Необходимо предусмотреть:

- Повторную доставку сообщений.
    
- Идемпотентность.
    
- Транзакции.
    
- Ошибки внешних сервисов.
    
- Мониторинг.
    
- Компенсирующие операции.
    

### Task 5. Refactor a Legacy PHP Application

План:

1. Зафиксировать текущее поведение тестами.
    
2. Найти критичные зависимости.
    
3. Выделить бизнес-логику.
    
4. Уменьшить связанность.
    
5. Внедрить interfaces и DI там, где они действительно нужны.
    
6. Добавить static analysis.
    
7. Постепенно улучшать архитектуру.
    
8. Сохранять backward compatibility.
    

---

## 19. What a Senior PHP Developer Should Know

Senior PHP Developer должен уверенно владеть следующими областями:

### PHP Language

- Type system.
    
- OOP.
    
- References.
    
- Closures.
    
- Generators.
    
- Iterators.
    
- Enums.
    
- Attributes.
    
- Reflection.
    
- Error handling.
    
- PHP internals at a practical level.
    

### Runtime

- Zend Engine.
    
- Opcache.
    
- PHP-FPM.
    
- Memory management.
    
- Garbage collection.
    
- Copy-on-Write.
    
- Long-running processes.
    

### Backend

- HTTP.
    
- REST API.
    
- Authentication.
    
- Authorization.
    
- Sessions.
    
- Caching.
    
- Queues.
    
- Database transactions.
    
- SQL optimization.
    
- Distributed systems.
    

### Architecture

- SOLID.
    
- Design patterns.
    
- Dependency Injection.
    
- Clean Architecture.
    
- DDD.
    
- Event-driven architecture.
    
- Modular monoliths.
    
- Microservices trade-offs.
    

### Engineering

- PHPUnit.
    
- Integration testing.
    
- Static analysis.
    
- Code review.
    
- Refactoring.
    
- CI/CD.
    
- Observability.
    
- Performance profiling.
    
- Security.
    

### Infrastructure

- Composer.
    
- Linux.
    
- Nginx.
    
- PHP-FPM.
    
- MySQL/PostgreSQL.
    
- Redis.
    
- RabbitMQ.
    
- Docker.
    
- Git.
    

---

## 20. Recommended Learning Order

Для системной подготовки рекомендуется проходить темы в следующем порядке:

1. PHP Fundamentals.
    
2. Arrays and Strings.
    
3. Advanced Language Features.
    
4. OOP.
    
5. Type System.
    
6. Error Handling.
    
7. PHP Runtime and Memory.
    
8. Composer and PSR.
    
9. SQL and PDO.
    
10. HTTP and REST.
    
11. Security.
    
12. Architecture and Design Patterns.
    
13. Testing and Static Analysis.
    
14. Caching and Queues.
    
15. Distributed Systems.
    
16. Framework Architecture.
    
17. Performance and Scalability.
    
18. Senior-level System Design.
    

Главный критерий Senior-уровня — не количество изученных функций PHP, а способность выбирать подходящее решение, аргументировать архитектурные решения, учитывать ограничения системы и отвечать за качество решения в production.# PHP — Senior Developer Interview Preparation

> Полный конспект PHP для подготовки к техническим собеседованиям уровня Senior. Основной фокус: язык, runtime, ООП, типизация, производительность, безопасность, архитектура и Backend-разработка.

## 1. PHP Fundamentals

### 1.1. Что такое PHP

PHP (PHP: Hypertext Preprocessor) — серверный интерпретируемый язык программирования общего назначения, преимущественно используемый для Backend-разработки и создания динамических веб-приложений.

Основные характеристики:

- Dynamically typed — тип переменной определяется во время выполнения.
    
- Weakly typed by default — в некоторых контекстах PHP выполняет автоматическое преобразование типов.
    
- Multi-paradigm — поддерживает процедурное, объектно-ориентированное и функциональное программирование.
    
- Garbage collected — управляет памятью автоматически.
    
- Request-oriented — традиционно веб-запрос обрабатывается отдельным жизненным циклом выполнения PHP.
    
- Extensible — функциональность расширяется через extensions.
    

PHP применяется для:

- REST API и GraphQL API.
    
- Backend-приложений.
    
- CMS: Drupal, WordPress.
    
- E-commerce.
    
- Микросервисов.
    
- CLI-инструментов и фоновых обработчиков.
    
- Интеграции с внешними сервисами.
    

### 1.2. Как работает PHP

В стандартной конфигурации PHP-FPM + Nginx запрос проходит следующие этапы:

1. Клиент отправляет HTTP request.
    
2. Nginx принимает запрос.
    
3. Nginx передаёт PHP-запрос в PHP-FPM через FastCGI.
    
4. PHP-FPM выбирает свободный worker.
    
5. PHP выполняет скрипт.
    
6. Zend Engine преобразует PHP-код в инструкции для выполнения.
    
7. Результат возвращается через PHP-FPM в Nginx.
    
8. Nginx отправляет HTTP response клиенту.
    

Основные компоненты:

|Компонент|Назначение|
|---|---|
|Zend Engine|Исполняет PHP-код|
|PHP Parser|Разбирает исходный код|
|Opcache|Кэширует скомпилированный bytecode|
|PHP-FPM|Управляет PHP-процессами|
|Extensions|Добавляют функциональность|
|SAPI|Интерфейс взаимодействия PHP с окружением|

PHP может работать через разные SAPI:

- CLI — командная строка.
    
- FPM/FastCGI — обработка веб-запросов.
    
- Apache module — интеграция с Apache.
    
- Другие специализированные интерфейсы.
    

### 1.3. Синтаксис PHP

```
<?php

declare(strict_types=1);

$name = 'John';
$age = 30;

echo "Name: {$name}";
echo PHP_EOL;

$result = $age + 10;
```

`<?php` начинает PHP-код.

`declare(strict_types=1)` включает строгую проверку скалярных типов для соответствующего файла в определённых контекстах вызова.

В современных PHP-приложениях рекомендуется использовать `strict_types=1` в исходных файлах, где это возможно.

### 1.4. Variables

Переменные в PHP начинаются с `$`.

```
$name = 'John';
$age = 30;
$price = 19.99;
$isActive = true;
$items = ['Apple', 'Orange'];
```

PHP не требует предварительного объявления переменной или указания её типа.

Переменные имеют динамический тип:

```
$value = 100;
$value = 'Hello';
$value = false;
```

Тип определяется текущим значением.

### 1.5. Constants

Константы — значения, которые нельзя переопределить после объявления.

```
define('APP_NAME', 'My Application');

const API_VERSION = 'v1';

echo APP_NAME;
echo API_VERSION;
```

Важные особенности:

- `define()` позволяет объявлять константы во время выполнения.
    
- `const` объявляется на этапе компиляции и имеет ограничения на контекст использования.
    
- Константы не используют `$`.
    
- Константы класса объявляются через `const`.
    

```
class User
{
    public const ROLE_ADMIN = 'admin';
}
```

### 1.6. Data Types

PHP поддерживает следующие основные типы:

|Категория|Типы|
|---|---|
|Scalar|int, float, string, bool|
|Compound|array, object, callable, iterable|
|Special|null, resource|
|Advanced type declarations|mixed, never, void, static, self, parent|

Пример:

```
$integer = 10;
$float = 10.5;
$string = 'PHP';
$boolean = true;
$array = [1, 2, 3];
$object = new stdClass();
$null = null;
```

#### Integer

Целые числа.

```
$number = 100;
$negative = -50;
$hex = 0xFF;
$binary = 0b1010;
```

Размер `int` зависит от платформы. На современных 64-битных системах обычно используется 64-битный integer.

#### Float

Числа с плавающей точкой.

```
$price = 19.99;
$rate = 0.15;
```

Float использует двоичное представление, поэтому некоторые десятичные значения не могут быть представлены точно.

```
var_dump(0.1 + 0.2);
```

Для финансовых вычислений не следует полагаться на обычный `float`.

Варианты:

- Integer в минимальных денежных единицах, например центах.
    
- Decimal в базе данных.
    
- Специализированные decimal-библиотеки.
    

#### String

```
$name = 'John';

$message = "Hello, {$name}";

$multiline = <<<TEXT
Hello
World
TEXT;
```

Основные функции:

- `strlen()`
    
- `mb_strlen()`
    
- `substr()`
    
- `mb_substr()`
    
- `strpos()`
    
- `str_contains()`
    
- `str_starts_with()`
    
- `str_ends_with()`
    
- `explode()`
    
- `implode()`
    
- `trim()`
    
- `str_replace()`
    
- `sprintf()`
    

Для Unicode необходимо учитывать разницу между байтовой длиной строки и количеством символов.

```
$text = 'Привет';

echo strlen($text);
echo mb_strlen($text);
```

`strlen()` возвращает количество байтов, `mb_strlen()` умеет работать с многобайтовыми кодировками.

#### Boolean

```
$isActive = true;

if ($isActive) {
    echo 'Active';
}
```

Falsy-значения в PHP:

- `false`
    
- `0`
    
- `0.0`
    
- `''`
    
- `'0'`
    
- `[]`
    
- `null`
    

Важно: строка `'0'` является falsy, несмотря на то что это непустая строка.

### 1.7. Type Juggling и Type Casting

Type Juggling — автоматическое преобразование типов.

```
$result = '10' + 5;

var_dump($result); // int(15)
```

Type Casting — явное преобразование.

```
$value = '100';

$integer = (int) $value;
$float = (float) $value;
$string = (string) $integer;
$boolean = (bool) $value;
```

В PHP существуют функции:

- `intval()`
    
- `floatval()`
    
- `strval()`
    
- `boolval()`
    
- `settype()`
    

Для проверки типа:

```
is_int($value);
is_string($value);
is_array($value);
is_object($value);
is_null($value);
```

Для проверки значения:

```
isset($value);
empty($value);
is_null($value);
```

Разница:

|Функция|Поведение|
|---|---|
|isset()|false для неопределённой переменной и null|
|empty()|true для falsy-значений|
|is_null()|Проверяет null|
|array_key_exists()|Проверяет существование ключа массива, даже если значение null|

```
$data = [
    'name' => null,
];

isset($data['name']);             // false
array_key_exists('name', $data); // true
```

### 1.8. Operators

#### Arithmetic

```
$a + $b;
$a - $b;
$a * $b;
$a / $b;
$a % $b;
$a ** $b;
```

#### Comparison

```
$a == $b;
$a === $b;
$a != $b;
$a !== $b;
$a > $b;
$a < $b;
$a >= $b;
$a <= $b;
```

Ключевое различие:

- `==` сравнивает значения с возможным преобразованием типов.
    
- `===` сравнивает значения и типы.
    

```
var_dump(10 == '10');  // true
var_dump(10 === '10'); // false
```

В бизнес-логике обычно предпочтительнее строгое сравнение.

#### Logical

```
&&
||
!
and
or
xor
```

`&&` и `and` имеют разный приоритет операторов.

#### Null coalescing

```
$name = $input['name'] ?? 'Guest';
```

Эквивалентно проверке существования значения на null, но не эквивалентно `isset()` в каждом контексте с массивами и выражениями.

#### Nullsafe operator

```
$city = $user?->getAddress()?->getCity();
```

Если один из объектов в цепочке равен `null`, результатом будет `null`, а дальнейшее обращение по цепочке не выполняется.

#### Spaceship operator

```
$result = $a <=> $b;
```

Возвращает:

- `-1`, если `$a < $b`.
    
- `0`, если значения равны.
    
- `1`, если `$a > $b`.
    

Используется, например, при сортировке.

### 1.9. Control Structures

#### if / elseif / else

```
if ($age >= 18) {
    echo 'Adult';
} elseif ($age >= 13) {
    echo 'Teenager';
} else {
    echo 'Child';
}
```

#### match

`match` — выражение, которое возвращает значение и использует строгое сравнение.

```
$status = match ($code) {
    200 => 'OK',
    404 => 'Not Found',
    500 => 'Server Error',
    default => 'Unknown',
};
```

Отличия от `switch`:

- Строгое сравнение.
    
- Возвращает значение.
    
- Не требует `break`.
    
- Не допускает fall-through.
    
- Необходимо обработать все варианты либо определить `default`.
    

#### Loops

```
for ($i = 0; $i < 5; $i++) {
    echo $i;
}

while ($condition) {
    // ...
}

foreach ($items as $item) {
    echo $item;
}

foreach ($items as $key => $value) {
    echo "{$key}: {$value}";
}
```

При изменении элементов массива по ссылке:

```
foreach ($items as &$item) {
    $item *= 2;
}

unset($item);
```

После `foreach` необходимо удалить ссылочную переменную через `unset()`, иначе она может продолжать ссылаться на последний элемент массива.

### 1.10. Functions

Функция — переиспользуемый блок кода, который может принимать аргументы и возвращать значение.

```
function calculateTotal(float $price, int $quantity): float
{
    return $price * $quantity;
}
```

Аргументы могут иметь:

- Тип.
    
- Значение по умолчанию.
    
- Named arguments.
    
- Variadic arguments.
    
- Ссылочную передачу.
    

```
function createUser(
    string $name,
    int $age = 18,
): array {
    return [
        'name' => $name,
        'age' => $age,
    ];
}
```

Named arguments:

```
createUser(
    age: 25,
    name: 'John',
);
```

Variadic arguments:

```
function sum(int ...$numbers): int
{
    return array_sum($numbers);
}

echo sum(1, 2, 3, 4);
```

Возвращаемый тип:

```
function getName(): string
{
    return 'John';
}
```

Функция может возвращать `void`, если не должна возвращать значение.

```
function logMessage(string $message): void
{
    echo $message;
}
```

### 1.11. Scope

Область видимости определяет, где доступна переменная.

Основные варианты:

- Local scope.
    
- Global scope.
    
- Static local variables.
    
- Function parameters.
    
- Object properties.
    

```
$name = 'Global';

function test(): void
{
    $name = 'Local';

    echo $name;
}
```

Для доступа к глобальной переменной:

```
$name = 'Global';

function test(): void
{
    global $name;

    echo $name;
}
```

Другой вариант:

```
$name = 'Global';

function test(): void
{
    echo $GLOBALS['name'];
}
```

`global` и `$GLOBALS` обычно не рекомендуются для бизнес-логики, поскольку создают скрытые зависимости.

Static local variable:

```
function counter(): int
{
    static $count = 0;

    return ++$count;
}
```

Значение сохраняется между вызовами функции в рамках текущего выполнения скрипта.

### 1.12. References

Reference позволяет нескольким переменным обращаться к одному значению через ссылочную семантику PHP.

```
$a = 10;
$b = &$a;

$b = 20;

echo $a; // 20
```

Передача аргумента по ссылке:

```
function increment(int &$number): void
{
    $number++;
}

$value = 10;

increment($value);

echo $value; // 11
```

References не следует путать с указателями в C/C++.

В PHP reference — механизм управления доступом к значению, а не обычный способ передачи объекта.

Для объектов отдельная модель: переменные объекта содержат идентификатор объекта, и копирование такой переменной обычно не клонирует сам объект.

---

## 2. PHP Data Structures

### 2.1. Arrays

В PHP массив — это ordered map, то есть упорядоченная структура ключ-значение.

Он может работать как:

- Indexed array.
    
- Associative array.
    
- Stack.
    
- Queue.
    
- Hash map.
    
- Multidimensional array.
    

```
$numbers = [10, 20, 30];

$user = [
    'name' => 'John',
    'age' => 30,
];
```

Массивы PHP не являются обычными непрерывными массивами, как в C. Они поддерживают ключи разных типов с определёнными правилами преобразования.

```
$data = [
    'name' => 'John',
    10 => 'value',
    true => 'boolean key',
];
```

Некоторые ключи автоматически преобразуются:

- Integer-like string может преобразоваться в integer при определённых условиях.
    
- Boolean преобразуется в integer.
    
- Float преобразуется в integer.
    
- Null преобразуется в пустую строку.
    

### 2.2. Array Operations

```
$items[] = 'New item';

array_push($items, 'A', 'B');

array_pop($items);

array_shift($items);

array_unshift($items, 'First');
```

Объединение массивов:

```
$a = ['name' => 'John', 'age' => 30];
$b = ['age' => 35, 'city' => 'Berlin'];

$result = $a + $b;
```

Оператор `+` сохраняет значения из левого массива при совпадении ключей.

Для полного объединения с перезаписью:

```
$result = array_merge($a, $b);
```

### 2.3. Array Functions

Часто используемые функции:

|Функция|Назначение|
|---|---|
|array_map()|Преобразование элементов|
|array_filter()|Фильтрация|
|array_reduce()|Агрегация|
|array_walk()|Обход с callback|
|array_column()|Получение колонки|
|array_keys()|Получение ключей|
|array_values()|Получение значений|
|array_key_exists()|Проверка ключа|
|in_array()|Поиск значения|
|array_search()|Поиск с возвратом ключа|
|array_unique()|Удаление дубликатов|
|array_slice()|Получение части массива|
|array_splice()|Изменение части массива|
|array_combine()|Создание массива из двух массивов|
|array_chunk()|Разбиение на части|

Пример:

```
$users = [
    ['name' => 'John', 'active' => true],
    ['name' => 'Anna', 'active' => false],
    ['name' => 'Mike', 'active' => true],
];

$activeUsers = array_filter(
    $users,
    fn(array $user): bool => $user['active'],
);

$names = array_map(
    fn(array $user): string => $user['name'],
    $activeUsers,
);
```

### 2.4. Array Reduce

`array_reduce()` позволяет свести массив к одному значению.

```
$prices = [100, 200, 300];

$total = array_reduce(
    $prices,
    fn(int $carry, int $price): int => $carry + $price,
    0,
);
```

Результат — `600`.

### 2.5. Destructuring

```
$user = ['John', 30];

[$name, $age] = $user;
```

Для ассоциативного массива:

```
$user = [
    'name' => 'John',
    'age' => 30,
];

[
    'name' => $name,
    'age' => $age,
] = $user;
```

### 2.6. Spread Operator

```
$first = [1, 2];
$second = [3, 4];

$result = [...$first, ...$second];
```

Для ассоциативных массивов важно учитывать правила объединения ключей и поведение одинаковых строковых ключей.

### 2.7. Sorting

```
sort($items);
rsort($items);

asort($items);
arsort($items);

ksort($items);
krsort($items);

usort($items, $callback);
uasort($items, $callback);
uksort($items, $callback);
```

Различия:

- `sort()` сортирует значения и переиндексирует массив.
    
- `asort()` сохраняет связь ключей со значениями.
    
- `ksort()` сортирует по ключам.
    
- `usort()` использует пользовательский callback.
    

```
usort(
    $users,
    fn(array $a, array $b): int => $a['age'] <=> $b['age'],
);
```

### 2.8. Array Performance

Для больших объёмов данных важно учитывать:

- Размер элементов.
    
- Количество копирований.
    
- Использование `array_map()` и `array_filter()`.
    
- Необходимость хранения всего набора данных в памяти.
    
- Использование генераторов для потоковой обработки.
    
- Выбор правильной структуры данных.
    

PHP arrays удобны, но могут потреблять значительно больше памяти, чем специализированные структуры.

---

## 3. Advanced PHP Language Features

### 3.1. include, require, include_once, require_once

Эти конструкции подключают PHP-файлы.

```
require_once __DIR__ . '/config.php';

include __DIR__ . '/helpers.php';
```

Разница:

|Конструкция|Повторное подключение|Ошибка при отсутствии|
|---|---|---|
|include|Допускается|Warning|
|require|Допускается|Error|
|include_once|Защищает от повторного подключения|Warning|
|require_once|Защищает от повторного подключения|Error|

Для современных приложений рекомендуется использовать Composer autoloading вместо ручного подключения большого количества классов.

### 3.2. Namespaces

Namespaces позволяют группировать классы, интерфейсы, функции и константы, избегая конфликтов имён.

```
<?php

namespace App\Service;

class UserService
{
}
```

Использование:

```
use App\Service\UserService;

$service = new UserService();
```

Alias:

```
use App\Repository\UserRepository as Repository;
```

Глобальное пространство имён обозначается через `\`.

```
$now = \time();
```

### 3.3. Anonymous Functions

Анонимная функция не имеет имени.

```
$greet = function (string $name): string {
    return "Hello, {$name}";
};

echo $greet('John');
```

### 3.4. Closures

Closure — анонимная функция, которая может захватывать переменные из внешнего контекста.

```
$message = 'Hello';

$greet = function (string $name) use ($message): string {
    return "{$message}, {$name}";
};
```

`use ($message)` захватывает значение переменной на момент создания closure.

`use (&$message)` захватывает переменную по ссылке.

### 3.5. Arrow Functions

Arrow functions — компактный синтаксис для коротких callback-функций.

```
$multiplier = 2;

$calculate = fn(int $value): int => $value * $multiplier;
```

Особенности:

- Автоматически захватывают внешние переменные по значению.
    
- Не используют `use`.
    
- Содержат одно выражение.
    
- Не подходят для сложной многооператорной логики.
    

### 3.6. Callable и Callbacks

Callable — значение, которое может быть вызвано как функция.

```
function process(callable $callback): void
{
    $callback();
}

process(function (): void {
    echo 'Executed';
});
```

Допустимые callable:

- Имя функции.
    
- Closure.
    
- Массив `[object, method]`.
    
- Массив `[ClassName::class, method]`.
    
- Invokable object.
    

Invokable object:

```
class Logger
{
    public function __invoke(string $message): void
    {
        echo $message;
    }
}

$logger = new Logger();

$logger('Hello');
```

### 3.7. Generators

Generator позволяет создавать значения последовательно, не загружая весь набор данных в память.

Используется `yield`.

```
function generateNumbers(int $max): Generator
{
    for ($i = 1; $i <= $max; $i++) {
        yield $i;
    }
}

foreach (generateNumbers(1000000) as $number) {
    // Process number
}
```

Обычный массив:

```
function getNumbers(int $max): array
{
    $result = [];

    for ($i = 1; $i <= $max; $i++) {
        $result[] = $i;
    }

    return $result;
}
```

Generator:

- Не хранит все результаты одновременно.
    
- Возвращает значения по мере обхода.
    
- Удобен для обработки больших файлов и потоков.
    
- Не делает сам по себе тяжёлые вычисления быстрее.
    
- Не заменяет полноценные очереди или асинхронную обработку.
    

### 3.8. Iterators

Iterator — интерфейс для объектов, которые можно обходить.

```
class NumberIterator implements Iterator
{
    private int $position = 0;

    private array $items = [10, 20, 30];

    public function current(): mixed
    {
        return $this->items[$this->position];
    }

    public function key(): int
    {
        return $this->position;
    }

    public function next(): void
    {
        $this->position++;
    }

    public function rewind(): void
    {
        $this->position = 0;
    }

    public function valid(): bool
    {
        return isset($this->items[$this->position]);
    }
}
```

`IteratorAggregate` позволяет вернуть другой iterator:

```
class UserCollection implements IteratorAggregate
{
    public function __construct(
        private array $users,
    ) {
    }

    public function getIterator(): Traversable
    {
        return new ArrayIterator($this->users);
    }
}
```

### 3.9. Fibers

Fibers появились в PHP 8.1.

Они позволяют приостанавливать и возобновлять выполнение кода.

```
$fiber = new Fiber(function (): void {
    echo "Start\n";

    Fiber::suspend('Paused');

    echo "Resumed\n";
});

$value = $fiber->start();

$fiber->resume();
```

Fiber — механизм cooperative multitasking, а не полноценная параллельная обработка.

Он не создаёт автоматически отдельный поток CPU.

Практическое применение зависит от event loop и библиотек, которые управляют переключением выполнения.

### 3.10. Enums

Enums появились в PHP 8.1.

Они позволяют определить ограниченный набор допустимых значений.

```
enum OrderStatus: string
{
    case Pending = 'pending';
    case Paid = 'paid';
    case Cancelled = 'cancelled';
}
```

Использование:

```
function processOrder(OrderStatus $status): void
{
    if ($status === OrderStatus::Paid) {
        // ...
    }
}
```

Backed enums:

- `int`
    
- `string`
    

Pure enums не имеют scalar value.

Enums могут реализовывать интерфейсы и содержать методы.

```
enum OrderStatus: string
{
    case Pending = 'pending';
    case Paid = 'paid';

    public function isFinal(): bool
    {
        return $this === self::Paid;
    }
}
```

### 3.11. Attributes

Attributes появились в PHP 8.0.

Они позволяют добавлять структурированные метаданные к классам, методам, свойствам, параметрам и другим элементам.

```
#[Attribute]
class Route
{
    public function __construct(
        public string $path,
    ) {
    }
}
```

Использование:

```
#[Route('/users')]
class UserController
{
}
```

Получение через Reflection:

```
$reflection = new ReflectionClass(UserController::class);

$attributes = $reflection->getAttributes(Route::class);

$route = $attributes[0]->newInstance();
```

Attributes сами по себе не выполняют бизнес-логику. Framework или application code должен прочитать их и определить поведение.

### 3.12. Reflection API

Reflection позволяет исследовать структуру PHP-классов во время выполнения.

```
$reflection = new ReflectionClass(UserService::class);

foreach ($reflection->getMethods() as $method) {
    echo $method->getName();
}
```

Применение:

- Dependency Injection containers.
    
- ORM.
    
- Serialization.
    
- Attribute processing.
    
- Testing frameworks.
    
- Framework metadata discovery.
    

Недостатки:

- Дополнительная сложность.
    
- Runtime overhead.
    
- Сложность статического анализа.
    
- Риск чрезмерной динамичности.
    

Для повторяющихся операций Reflection metadata обычно кэшируют.

### 3.13. Magic Methods

Основные magic methods:

|Метод|Назначение|
|---|---|
|`__construct()`|Создание объекта|
|`__destruct()`|Уничтожение объекта|
|`__get()`|Чтение недоступного свойства|
|`__set()`|Запись недоступного свойства|
|`__isset()`|Проверка через isset|
|`__unset()`|Удаление недоступного свойства|
|`__call()`|Вызов недоступного метода|
|`__callStatic()`|Статический вызов недоступного метода|
|`__toString()`|Преобразование в строку|
|`__invoke()`|Вызов объекта как функции|
|`__clone()`|Клонирование|
|`__serialize()`|Пользовательская сериализация|
|`__unserialize()`|Восстановление объекта|

Magic methods могут скрывать реальное поведение объекта. Их следует использовать осознанно.

### 3.14. Serialization

```
$data = serialize($object);

$object = unserialize($data);
```

JSON:

```
$json = json_encode($data, JSON_THROW_ON_ERROR);

$data = json_decode(
    $json,
    true,
    512,
    JSON_THROW_ON_ERROR,
);
```

Различия:

|PHP Serialization|JSON|
|---|---|
|Может сохранять структуру PHP-объектов|Стандарт обмена данными|
|PHP-специфичен|Языконезависим|
|Может вызывать magic methods|Не восстанавливает произвольные классы|
|Опасен при работе с недоверенными данными|Безопаснее для обычного обмена, но требует валидации|

Никогда не вызывайте `unserialize()` на недоверенных данных без строгого обоснования и безопасных ограничений.

---

## 4. Object-Oriented Programming

### 4.1. Основные принципы OOP

ООП строится вокруг объектов, которые объединяют состояние и поведение.

Основные принципы:

1. Encapsulation.
    
2. Abstraction.
    
3. Inheritance.
    
4. Polymorphism.
    

### 4.2. Classes and Objects

Класс описывает структуру и поведение объекта.

Объект — экземпляр класса.

```
class User
{
    public function __construct(
        private string $name,
        private string $email,
    ) {
    }

    public function getName(): string
    {
        return $this->name;
    }
}

$user = new User('John', 'john@example.com');
```

### 4.3. Visibility

|Modifier|Доступ|
|---|---|
|public|Из любого контекста|
|protected|Внутри класса и наследников|
|private|Только внутри объявившего класса|

По умолчанию свойства лучше делать `private` или `protected`, предоставляя контролируемый API.

### 4.4. Encapsulation

Инкапсуляция скрывает внутреннюю реализацию и предоставляет контролируемый интерфейс.

```
class BankAccount
{
    public function __construct(
        private int $balance,
    ) {
        if ($balance < 0) {
            throw new InvalidArgumentException();
        }
    }

    public function deposit(int $amount): void
    {
        if ($amount <= 0) {
            throw new InvalidArgumentException();
        }

        $this->balance += $amount;
    }

    public function getBalance(): int
    {
        return $this->balance;
    }
}
```

Внешний код не может напрямую присвоить произвольное значение приватному свойству.

### 4.5. Inheritance

Наследование позволяет создавать класс на основе другого класса.

```
class Animal
{
    public function eat(): void
    {
        echo 'Eating';
    }
}

class Dog extends Animal
{
    public function bark(): void
    {
        echo 'Woof';
    }
}
```

Наследование полезно при наличии реального отношения is-a.

Не следует использовать наследование только ради повторного использования кода.

### 4.6. Polymorphism

Полиморфизм позволяет работать с разными реализациями через общий контракт.

```
interface PaymentGateway
{
    public function pay(int $amount): void;
}

class StripeGateway implements PaymentGateway
{
    public function pay(int $amount): void
    {
        // Stripe integration
    }
}

class PaypalGateway implements PaymentGateway
{
    public function pay(int $amount): void
    {
        // PayPal integration
    }
}
```

Клиентский код зависит от интерфейса:

```
class PaymentService
{
    public function __construct(
        private PaymentGateway $gateway,
    ) {
    }

    public function process(int $amount): void
    {
        $this->gateway->pay($amount);
    }
}
```

### 4.7. Abstraction

Абстракция позволяет определить существенные характеристики объекта без раскрытия деталей реализации.

Используются:

- Interfaces.
    
- Abstract classes.
    
- Contracts.
    
- Domain abstractions.
    

### 4.8. Abstract Classes

Абстрактный класс нельзя создать напрямую.

```
abstract class Report
{
    abstract public function generate(): string;

    public function save(): void
    {
        // Common logic
    }
}
```

Класс-наследник обязан реализовать абстрактные методы.

### 4.9. Interfaces

Интерфейс определяет контракт поведения.

```
interface LoggerInterface
{
    public function log(string $message): void;
}
```

Класс может реализовывать несколько интерфейсов.

```
class FileLogger implements LoggerInterface
{
    public function log(string $message): void
    {
        // Write to file
    }
}
```

### 4.10. Abstract Class vs Interface

|Abstract Class|Interface|
|---|---|
|Может содержать состояние|Не предназначен для хранения состояния объекта|
|Может содержать реализацию методов|Определяет контракт методов и допускает некоторые константы|
|Только одно наследование класса|Можно реализовать несколько интерфейсов|
|Подходит для общей базовой реализации|Подходит для определения контракта|

Выбор зависит от архитектурной задачи.

### 4.11. Traits

Traits позволяют переиспользовать методы между классами без классического наследования.

```
trait Timestampable
{
    public function getTimestamp(): int
    {
        return time();
    }
}

class Order
{
    use Timestampable;
}
```

Traits не являются полноценной заменой композиции.

При конфликте методов можно использовать:

- `insteadof`
    
- `as`
    

```
class Example
{
    use FirstTrait, SecondTrait {
        FirstTrait::run insteadof SecondTrait;
        SecondTrait::run as runSecond;
    }
}
```

### 4.12. Composition vs Inheritance

Composition — объект использует другие объекты для выполнения задач.

```
class OrderService
{
    public function __construct(
        private PaymentGateway $gateway,
    ) {
    }
}
```

Преимущества композиции:

- Меньшая связанность.
    
- Простая замена реализаций.
    
- Удобное тестирование.
    
- Более гибкая архитектура.
    

В современных приложениях композиция обычно предпочтительнее глубоких иерархий наследования.

### 4.13. Static

Static properties и methods принадлежат классу, а не отдельному экземпляру.

```
class Counter
{
    private static int $count = 0;

    public static function increment(): void
    {
        self::$count++;
    }

    public static function getCount(): int
    {
        return self::$count;
    }
}
```

Чрезмерное использование static усложняет тестирование и управление зависимостями.

### 4.14. self, static, parent

- `self` — класс, в котором объявлен метод.
    
- `static` — late static binding, фактический класс вызова.
    
- `parent` — родительский класс.
    

```
class ParentClass
{
    public static function getName(): string
    {
        return self::class;
    }

    public static function getLateName(): string
    {
        return static::class;
    }
}

class ChildClass extends ParentClass
{
}

echo ChildClass::getName();
echo ChildClass::getLateName();
```

`self::class` возвращает имя класса, где метод был объявлен. `static::class` учитывает наследника.

### 4.15. Final

`final` запрещает переопределение метода или наследование класса.

```
final class SecurityService
{
}
```

```
class UserService
{
    final public function validate(): bool
    {
        return true;
    }
}
```

Используется для защиты инвариантов и ограничения расширения API.

### 4.16. Readonly

Readonly properties появились в PHP 8.1, readonly classes — в PHP 8.2.

```
class UserDto
{
    public function __construct(
        public readonly string $name,
        public readonly int $age,
    ) {
    }
}
```

Readonly property нельзя повторно присвоить после инициализации.

Readonly не означает глубокую неизменяемость объекта: если свойство содержит объект, внутреннее состояние этого объекта может изменяться.

### 4.17. Constructor Property Promotion

```
class User
{
    public function __construct(
        private string $name,
        private string $email,
    ) {
    }
}
```

Позволяет объявить и инициализировать свойства непосредственно в конструкторе.

### 4.18. Cloning

```
$copy = clone $original;
```

Обычное клонирование создаёт поверхностную копию.

Вложенные объекты остаются теми же объектами, если явно не клонировать их в `__clone()`.

### 4.19. Object Identity

Два объекта могут иметь одинаковые значения свойств, но быть разными экземплярами.

```
$a = new stdClass();
$b = new stdClass();

var_dump($a === $b); // false
```

`===` для объектов проверяет идентичность экземпляров.

`==` сравнивает объекты по классу и значениям свойств с учётом правил PHP.

### 4.20. Property Hooks — PHP 8.4

Property Hooks позволяют определить поведение при чтении и записи свойства.

```
class User
{
    public string $name {
        get => strtolower($this->name);
        set => trim($value);
    }
}
```

Property Hooks могут использоваться для:

- Валидации.
    
- Нормализации.
    
- Вычисляемых значений.
    
- Контролируемого доступа к свойствам.
    

Важно: это не то же самое, что обычные getter/setter-методы.

---

## 5. PHP Type System

### 5.1. Type Declarations

PHP поддерживает декларации типов для:

- Параметров функций.
    
- Возвращаемых значений.
    
- Свойств.
    
- Констант класса.
    
- Конструкторов и других членов класса.
    

```
function calculate(int $a, int $b): int
{
    return $a + $b;
}
```

### 5.2. Scalar Types

- `int`
    
- `float`
    
- `string`
    
- `bool`
    

### 5.3. Nullable Types

```
function findUser(int $id): ?User
{
    return null;
}
```

`?User` эквивалентен `User|null`.

### 5.4. Union Types

Union type допускает несколько типов.

```
function process(int|string $value): int|string
{
    return $value;
}
```

### 5.5. Intersection Types

Intersection type требует, чтобы значение соответствовало нескольким типам одновременно.

```
interface Logger
{
    public function log(string $message): void;
}

interface Formatter
{
    public function format(string $message): string;
}

function process(Logger&Formatter $service): void
{
}
```

### 5.6. DNF Types

Disjunctive Normal Form позволяет комбинировать union и intersection types.

```
function process((Logger&Formatter)|null $service): void
{
}
```

### 5.7. mixed

`mixed` означает, что значение может иметь любой тип.

```
function process(mixed $value): void
{
}
```

Не следует использовать `mixed` повсеместно. Это снижает возможности статического анализа.

### 5.8. never

`never` означает, что функция никогда не возвращает управление обычным образом.

```
function terminate(): never
{
    throw new RuntimeException();
}
```

Также `never` может использоваться для функций, которые завершают выполнение через `exit`.

### 5.9. void

Функция не возвращает полезного значения.

```
function logMessage(string $message): void
{
    echo $message;
}
```

### 5.10. static, self, parent

Эти типы полезны для деклараций, связанных с классами и наследованием.

```
class Factory
{
    public static function create(): static
    {
        return new static();
    }
}
```

### 5.11. strict_types

```
declare(strict_types=1);
```

Строгая типизация влияет на проверку скалярных типов при вызовах функций.

Важный нюанс: режим проверки аргументов определяется файлом вызывающего кода, а проверка возвращаемого значения — контекстом файла, в котором объявлена функция.

`strict_types` не заменяет полноценную статическую типизацию и не отключает все преобразования PHP.

### 5.12. Covariance and Contravariance

Covariance позволяет возвращать более конкретный тип, чем определён в родительском контракте.

Contravariance позволяет принимать более общий тип аргумента.

Принцип:

- Return type может сужаться.
    
- Parameter type может расширяться.
    

Это связано с принципом подстановки Лисков (LSP).

### 5.13. PHPDoc

PHPDoc используется для дополнительного описания типов, generic-подобных структур и контрактов.

```
/**
 * @param array<int, User> $users
 * @return array<string, User>
 */
function indexUsers(array $users): array
{
    $result = [];

    foreach ($users as $user) {
        $result[$user->getEmail()] = $user;
    }

    return $result;
}
```

PHPDoc особенно важен для PHPStan и Psalm.

### 5.14. Generics в PHP

Встроенной полноценной системы generics в PHP пока нет.

Но статические анализаторы поддерживают generics через PHPDoc.

```
/**
 * @template T
 */
class Collection
{
    /**
     * @param list<T> $items
     */
    public function __construct(
        private array $items,
    ) {
    }

    /**
     * @return list<T>
     */
    public function all(): array
    {
        return $this->items;
    }
}
```

### 5.15. Static Analysis

Инструменты:

- PHPStan.
    
- Psalm.
    

Они позволяют находить ошибки до запуска программы.

Проверяют:

- Типы аргументов.
    
- Возвращаемые типы.
    
- Возможные null.
    
- Недоступные методы.
    
- Неправильное использование массивов.
    
- Нарушения контрактов.
    
- Часть логических ошибок.
    

Для Senior-разработчика статический анализ — важная часть качества проекта.

---

## 6. Error Handling and Exceptions

### 6.1. Error vs Exception

В PHP есть общий интерфейс `Throwable`.

```
Throwable
├── Error
│   ├── TypeError
│   ├── ParseError
│   ├── ArgumentCountError
│   └── ...
└── Exception
    ├── RuntimeException
    ├── LogicException
    └── ...
```

`Exception` и `Error` — разные ветви.

Обе могут быть пойманы через `catch (Throwable $e)`.

### 6.2. Exceptions

```
try {
    $service->process();
} catch (RuntimeException $exception) {
    // Handle expected runtime failure
} finally {
    // Cleanup
}
```

`finally` обычно выполняется при выходе из `try` или `catch`, в том числе при исключении.

Однако принудительное завершение процесса и некоторые аварийные ситуации могут не позволить выполнить cleanup.

### 6.3. Custom Exceptions

```
class PaymentFailedException extends RuntimeException
{
}
```

Использование:

```
if (!$paymentSucceeded) {
    throw new PaymentFailedException(
        'Payment processing failed',
    );
}
```

Преимущества:

- Чёткие бизнес-сценарии ошибок.
    
- Упрощение обработки.
    
- Разделение технических и бизнес-ошибок.
    
- Более понятные API-контракты.
    

### 6.4. Exception Hierarchy

Рекомендуется проектировать иерархию исключений по смыслу.

Пример:

```
DomainException
├── OrderException
│   ├── InvalidOrderException
│   └── OrderAlreadyPaidException
└── PaymentException
    ├── PaymentDeclinedException
    └── PaymentProviderException
```

Не нужно создавать отдельный exception для каждого метода без необходимости.

### 6.5. Error Handling

PHP поддерживает:

- `error_reporting()`
    
- `set_error_handler()`
    
- `set_exception_handler()`
    
- `trigger_error()`
    
- `error_log()`
    

`set_error_handler()` может преобразовывать обрабатываемые PHP diagnostics в exceptions.

Не все ошибки могут быть перехвачены пользовательским error handler.

### 6.6. Production Error Handling

В production:

- Не показывать пользователю stack trace.
    
- Не выводить секреты и внутренние пути.
    
- Логировать ошибки.
    
- Использовать correlation/request ID.
    
- Возвращать корректный HTTP status code.
    
- Разделять публичное сообщение и внутренние детали.
    

Пример:

```
try {
    $orderService->create($data);
} catch (InvalidOrderException $exception) {
    $logger->warning(
        'Invalid order',
        ['exception' => $exception],
    );

    return new Response(
        'Invalid order',
        422,
    );
} catch (Throwable $exception) {
    $logger->error(
        'Unexpected error',
        ['exception' => $exception],
    );

    return new Response(
        'Internal Server Error',
        500,
    );
}
```

---

## 7. PHP Runtime and Memory Management

### 7.1. Zend Engine

Zend Engine — runtime engine PHP, отвечающий за выполнение PHP-кода.

Упрощённый процесс:

1. Lexing.
    
2. Parsing.
    
3. AST generation.
    
4. Compilation to opcodes.
    
5. Execution by Zend VM.
    

В реальности детали зависят от версии PHP и конфигурации.

### 7.2. Opcache

Opcache хранит скомпилированный bytecode в shared memory.

Без Opcache PHP обычно должен повторно разбирать и компилировать файлы при новых запросах.

С Opcache:

- Снижается повторная работа компилятора.
    
- Уменьшается CPU overhead.
    
- Улучшается производительность.
    
- Bytecode может использоваться несколькими PHP workers.
    

В production обычно рекомендуется включать Opcache.

### 7.3. JIT

JIT появился в PHP 8.0.

Он может компилировать некоторые участки bytecode в машинный код.

Однако JIT не гарантирует ускорение обычных веб-приложений.

Для типичных CRUD/API-систем часто важнее:

- SQL-запросы.
    
- Кэширование.
    
- Алгоритмы.
    
- Сетевые задержки.
    
- Сериализация.
    
- Профилирование.
    

### 7.4. Memory Management

PHP использует автоматическое управление памятью.

Основные механизмы:

- Reference counting.
    
- Copy-on-write.
    
- Garbage collection.
    

### 7.5. Reference Counting

Zend Engine отслеживает количество ссылок на значение.

Когда значение больше не используется, память может быть освобождена.

### 7.6. Copy-on-Write

При копировании массива PHP обычно не копирует все данные немедленно.

```
$a = [1, 2, 3];

$b = $a;
```

Пока массивы не изменяются, PHP может использовать общие данные.

При изменении одного из массивов выполняется копирование необходимых данных.

Это называется Copy-on-Write.

### 7.7. Garbage Collection

Reference counting не решает все случаи циклических ссылок.

```
$a = new stdClass();
$b = new stdClass();

$a->other = $b;
$b->other = $a;
```

Garbage Collector обнаруживает циклические структуры, которые больше не доступны из программы.

### 7.8. Memory Optimization

Для оптимизации:

- Не хранить большие результаты без необходимости.
    
- Использовать генераторы.
    
- Обрабатывать файлы потоково.
    
- Избегать ненужного копирования.
    
- Ограничивать размер batch processing.
    
- Не создавать лишние объекты.
    
- Использовать профилирование, а не предположения.
    

```
$handle = fopen('large.csv', 'rb');

while (($row = fgetcsv($handle)) !== false) {
    // Process one row
}

fclose($handle);
```

### 7.9. PHP-FPM

PHP-FPM управляет пулом PHP workers.

Основные параметры:

- `pm`
    
- `pm.max_children`
    
- `pm.start_servers`
    
- `pm.min_spare_servers`
    
- `pm.max_spare_servers`
    
- `pm.max_requests`
    

Режимы:

|Режим|Назначение|
|---|---|
|static|Фиксированное количество workers|
|dynamic|Количество workers изменяется в пределах настроек|
|ondemand|Workers запускаются по необходимости|

`pm.max_children` ограничивает максимальное количество одновременно работающих workers.

Слишком большое значение может привести к нехватке памяти.

Упрощённая оценка:

`max_children ≈ доступная память для PHP / среднее потребление памяти одним worker`

На практике нужно учитывать пики памяти, системные процессы и другие сервисы.

### 7.10. Request Lifecycle

В традиционной модели PHP-FPM:

- Worker принимает запрос.
    
- Выполняет PHP-код.
    
- Завершает request lifecycle.
    
- Освобождает request-scoped состояние.
    
- Worker может обработать следующий запрос.
    

Нельзя считать, что каждый запрос обязательно запускает новый системный процесс.

Долгоживущие процессы и приложения на event loop имеют другие требования к управлению состоянием.

### 7.11. Long-running PHP Processes

Примеры:

- Queue workers.
    
- RoadRunner.
    
- Swoole.
    
- FrankenPHP.
    
- CLI daemons.
    

В таких приложениях необходимо особенно внимательно относиться к:

- Утечкам памяти.
    
- Статическим свойствам.
    
- Глобальному состоянию.
    
- Повторному использованию объектов.
    
- Очистке request-scoped данных.
    
- Исключениям и восстановлению worker.
    

---

## 8. Composer and PHP Standards

### 8.1. Composer

Composer — стандартный dependency manager PHP.

Основные задачи:

- Установка библиотек.
    
- Управление версиями.
    
- Dependency resolution.
    
- Autoloading.
    
- Работа с пакетами.
    

### 8.2. composer.json

```
{
    "name": "app/project",
    "type": "project",
    "require": {
        "php": "^8.4",
        "guzzlehttp/guzzle": "^7.9"
    },
    "require-dev": {
        "phpunit/phpunit": "^11.0"
    },
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

### 8.3. composer.lock

`composer.lock` фиксирует конкретные версии установленных зависимостей.

Зачем он нужен:

- Повторяемые установки.
    
- Одинаковые версии в разных окружениях.
    
- Предсказуемые deployments.
    
- Контроль изменений зависимостей.
    

Для приложений lock-файл обычно коммитится в Git.

### 8.4. Composer Commands

```
composer install
composer update
composer require vendor/package
composer remove vendor/package
composer dump-autoload
composer validate
composer show
composer audit
```

Разница:

- `install` устанавливает версии из lock-файла.
    
- `update` разрешает зависимости заново и изменяет lock-файл.
    

В CI/CD обычно используется `composer install`, а не безусловный `composer update`.

### 8.5. Semantic Versioning

Формат:

`MAJOR.MINOR.PATCH`

- MAJOR — breaking changes.
    
- MINOR — новые возможности с обратной совместимостью.
    
- PATCH — исправления с обратной совместимостью.
    

Примеры Composer constraints:

- `^1.2` — совместимые обновления в пределах major 1.
    
- `~1.2.3` — обновления до следующего minor.
    
- `1.2.3` — точная версия.
    
- `*` — слишком широкое ограничение для многих production-сценариев.
    

### 8.6. Autoloading

Composer поддерживает:

- PSR-4.
    
- PSR-0.
    
- Classmap.
    
- Files autoload.
    

PSR-4 — основной современный стандарт autoloading.

```
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

Класс `App\Service\UserService` будет находиться в `src/Service/UserService.php` при соблюдении соглашения о регистре имён и каталогов.

### 8.7. PSR Standards

|Standard|Назначение|
|---|---|
|PSR-1|Basic Coding Standard|
|PSR-3|Logger Interface|
|PSR-4|Autoloading|
|PSR-7|HTTP Message Interfaces|
|PSR-11|Container Interface|
|PSR-12|Extended Coding Style|
|PSR-15|HTTP Server Request Handlers|
|PSR-17|HTTP Factories|
|PSR-18|HTTP Client|
|PSR-20|Clock|

PSR — PHP Standards Recommendations.

Они повышают совместимость библиотек и стандартизируют взаимодействие компонентов.

---

## 9. Database and PHP

### 9.1. Database Fundamentals

Backend-приложения обычно используют реляционные базы данных:

- MySQL.
    
- MariaDB.
    
- PostgreSQL.
    

Основные понятия:

- Tables.
    
- Rows.
    
- Columns.
    
- Primary keys.
    
- Foreign keys.
    
- Indexes.
    
- Constraints.
    
- Transactions.
    
- Views.
    
- Joins.
    

### 9.2. PDO

PDO — PHP Data Objects.

Предоставляет единый интерфейс работы с разными базами данных через соответствующие drivers.

```
$pdo = new PDO(
    'mysql:host=localhost;dbname=app;charset=utf8mb4',
    'user',
    'password',
    [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    ],
);
```

### 9.3. Prepared Statements

Prepared statements разделяют SQL-код и пользовательские значения.

```
$stmt = $pdo->prepare(
    'SELECT * FROM users WHERE email = :email',
);

$stmt->execute([
    'email' => $email,
]);

$user = $stmt->fetch();
```

Это основной способ защиты от SQL Injection при работе с динамическими значениями.

Важно: placeholders нельзя использовать для имён таблиц, колонок и SQL keywords. Такие динамические части нужно выбирать из заранее разрешённого списка.

### 9.4. SQL Injection

Опасный вариант:

```
$sql = "SELECT * FROM users WHERE email = '$email'";
```

Если `$email` содержит вредоносный SQL, запрос может изменить своё поведение.

Безопасный вариант — prepared statement.

### 9.5. Transactions

Транзакция объединяет несколько операций в единое логическое действие.

```
$pdo->beginTransaction();

try {
    $pdo->prepare(
        'UPDATE accounts SET balance = balance - ? WHERE id = ?',
    )->execute([100, 1]);

    $pdo->prepare(
        'UPDATE accounts SET balance = balance + ? WHERE id = ?',
    )->execute([100, 2]);

    $pdo->commit();
} catch (Throwable $exception) {
    if ($pdo->inTransaction()) {
        $pdo->rollBack();
    }

    throw $exception;
}
```

### 9.6. ACID

|Property|Значение|
|---|---|
|Atomicity|Все операции выполняются целиком или откатываются|
|Consistency|Данные сохраняют заданные ограничения|
|Isolation|Параллельные транзакции контролируют взаимное влияние|
|Durability|Подтверждённые изменения сохраняются|

### 9.7. Isolation Levels

Основные уровни:

- Read Uncommitted.
    
- Read Committed.
    
- Repeatable Read.
    
- Serializable.
    

Аномалии:

- Dirty Read.
    
- Non-repeatable Read.
    
- Phantom Read.
    
- Lost Update.
    

Конкретное поведение зависит от СУБД и её реализации MVCC/locking.

### 9.8. Indexes

Индексы ускоряют поиск и сортировку, но имеют стоимость:

- Дополнительная память и место на диске.
    
- Более дорогие INSERT/UPDATE/DELETE.
    
- Необходимость обслуживания.
    
- Возможность выбора неоптимального плана.
    

Основные типы:

- B-tree.
    
- Hash.
    
- Composite.
    
- Unique.
    
- Full-text.
    
- Partial indexes — поддерживаются не всеми СУБД одинаково.
    

### 9.9. N+1 Problem

N+1 возникает, когда приложение выполняет один запрос для получения списка, а затем по одному запросу на каждый элемент.

Например:

1. Получить 100 заказов.
    
2. Для каждого заказа получить пользователя.
    
3. Итого 101 запрос.
    

Решения:

- JOIN.
    
- Eager loading.
    
- Batch queries.
    
- DataLoader-подход.
    
- Корректное проектирование ORM-запросов.
    

### 9.10. Repository Pattern

Repository инкапсулирует получение и сохранение данных.

```
interface UserRepositoryInterface
{
    public function findById(int $id): ?User;

    public function save(User $user): void;
}
```

Реализация:

```
class PdoUserRepository implements UserRepositoryInterface
{
    public function __construct(
        private PDO $pdo,
    ) {
    }

    public function findById(int $id): ?User
    {
        $stmt = $this->pdo->prepare(
            'SELECT id, name FROM users WHERE id = ?',
        );

        $stmt->execute([$id]);

        $row = $stmt->fetch();

        if ($row === false) {
            return null;
        }

        return new User(
            id: (int) $row['id'],
            name: $row['name'],
        );
    }

    public function save(User $user): void
    {
        // Persistence implementation
    }
}
```

Repository полезен, когда он выражает понятную границу доступа к данным. Не нужно создавать бессмысленный repository для каждого SQL-запроса.

---

## 10. HTTP and Web Development

### 10.1. HTTP

HTTP — протокол взаимодействия клиента и сервера.

Основные части HTTP request:

- Method.
    
- URL.
    
- Headers.
    
- Body.
    

Основные части HTTP response:

- Status code.
    
- Headers.
    
- Body.
    

### 10.2. HTTP Methods

|Method|Назначение|
|---|---|
|GET|Получение ресурса|
|POST|Создание или выполнение операции|
|PUT|Полная замена ресурса|
|PATCH|Частичное обновление|
|DELETE|Удаление|
|HEAD|Получение заголовков|
|OPTIONS|Получение доступных возможностей|

Идемпотентность:

- GET — идемпотентен.
    
- PUT — идемпотентен по семантике.
    
- DELETE — идемпотентен по семантике.
    
- POST — обычно не идемпотентен.
    
- PATCH — зависит от операции.
    

### 10.3. HTTP Status Codes

|Code|Значение|
|---|---|
|200|OK|
|201|Created|
|202|Accepted|
|204|No Content|
|301|Moved Permanently|
|302|Found|
|304|Not Modified|
|400|Bad Request|
|401|Unauthorized|
|403|Forbidden|
|404|Not Found|
|409|Conflict|
|422|Unprocessable Content|
|429|Too Many Requests|
|500|Internal Server Error|
|502|Bad Gateway|
|503|Service Unavailable|
|504|Gateway Timeout|

401 означает отсутствие действительной аутентификации. 403 означает, что сервер отказывает в доступе, даже если запрос распознан.

### 10.4. REST API

REST — архитектурный стиль взаимодействия с ресурсами через стандартные HTTP-механизмы.

Пример:

```
GET    /api/users
GET    /api/users/10
POST   /api/users
PATCH  /api/users/10
DELETE /api/users/10
```

Основные принципы:

- Resource-oriented URLs.
    
- Stateless communication.
    
- Standard HTTP methods.
    
- Consistent representations.
    
- Cacheability.
    
- Layered architecture.
    

REST — не просто JSON поверх HTTP.

### 10.5. Request Data

PHP предоставляет:

```
$_GET;
$_POST;
$_COOKIE;
$_FILES;
$_SERVER;
$_SESSION;
```

Для JSON body:

```
$data = json_decode(
    file_get_contents('php://input'),
    true,
    512,
    JSON_THROW_ON_ERROR,
);
```

Полученные данные нужно валидировать до использования.

### 10.6. Cookies

Cookie хранится на стороне клиента и отправляется серверу в HTTP-запросах согласно правилам браузера.

Настройки:

- `Secure`
    
- `HttpOnly`
    
- `SameSite`
    
- `Path`
    
- `Domain`
    
- `Expires`
    
- `Max-Age`
    

### 10.7. Sessions

Session позволяет хранить серверное состояние, связанное с клиентом.

```
session_start();

$_SESSION['user_id'] = 10;
```

В production необходимо:

- Использовать HTTPS.
    
- Включать `HttpOnly`.
    
- Включать `Secure`.
    
- Настраивать `SameSite`.
    
- Регенерировать session ID после login.
    
- Настраивать expiration.
    
- Не хранить лишние чувствительные данные.
    

### 10.8. Authentication vs Authorization

Authentication — кто пользователь?

Authorization — что пользователю разрешено делать?

Пример:

- Authentication: пользователь вошёл в систему.
    
- Authorization: пользователь имеет право редактировать конкретный заказ.
    

Проверка доступа должна учитывать не только роль, но и объект, контекст и бизнес-правила.

### 10.9. JWT

JWT — JSON Web Token.

Структура:

```
Header.Payload.Signature
```

JWT обычно содержит claims.

Важно:

- Payload не является зашифрованным по умолчанию.
    
- Подпись подтверждает целостность.
    
- Необходимо проверять алгоритм.
    
- Проверять issuer, audience и expiration.
    
- Использовать безопасное управление ключами.
    
- Продумывать отзыв и ротацию токенов.
    

JWT не является автоматически более безопасным, чем server-side sessions.

### 10.10. CORS

CORS — механизм браузерной политики доступа к ресурсам между origins.

Основные заголовки:

- `Access-Control-Allow-Origin`
    
- `Access-Control-Allow-Methods`
    
- `Access-Control-Allow-Headers`
    
- `Access-Control-Allow-Credentials`
    

CORS не заменяет authentication и authorization.

### 10.11. CSRF

CSRF — атака, при которой браузер пользователя отправляет нежелательный запрос на сайт, где пользователь уже аутентифицирован.

Защита:

- CSRF tokens.
    
- SameSite cookies.
    
- Проверка Origin/Referer в подходящих сценариях.
    
- Не использовать GET для изменения состояния.
    
- Дополнительная проверка критичных операций.
    

### 10.12. cURL and HTTP Clients

PHP может выполнять HTTP-запросы через cURL.

```
$curl = curl_init('https://api.example.com/users');

curl_setopt_array($curl, [
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_TIMEOUT => 10,
    CURLOPT_CONNECTTIMEOUT => 3,
]);

$response = curl_exec($curl);

if ($response === false) {
    throw new RuntimeException(curl_error($curl));
}

$statusCode = curl_getinfo(
    $curl,
    CURLINFO_RESPONSE_CODE,
);

curl_close($curl);
```

Для приложений обычно удобнее использовать HTTP client library, например Guzzle.

Необходимо настраивать:

- Connection timeout.
    
- Request timeout.
    
- Retry policy.
    
- TLS verification.
    
- Logging.
    
- Circuit breaker для критичных интеграций.
    

---

## 11. PHP Security

### 11.1. Security Principles

Основные принципы:

- Least Privilege.
    
- Defense in Depth.
    
- Secure by Default.
    
- Input Validation.
    
- Output Encoding.
    
- Secrets Management.
    
- Fail Securely.
    

### 11.2. SQL Injection

Защита:

- Prepared statements.
    
- Parameter binding.
    
- Allowlist для динамических SQL identifiers.
    
- Минимальные права DB-пользователя.
    

### 11.3. XSS

XSS позволяет внедрять вредоносный JavaScript в страницы приложения.

Типы:

- Stored XSS.
    
- Reflected XSS.
    
- DOM-based XSS.
    

Защита зависит от контекста вывода:

```
echo htmlspecialchars(
    $userInput,
    ENT_QUOTES | ENT_SUBSTITUTE,
    'UTF-8',
);
```

Для HTML, JavaScript, CSS и URL используются разные правила encoding.

Нельзя считать одну функцию универсальной защитой для любого контекста.

### 11.4. CSRF

Защита через токены и настройки cookie.

Важно: CSRF актуален прежде всего там, где браузер автоматически прикрепляет credentials.

### 11.5. SSRF

SSRF позволяет злоумышленнику заставить сервер обращаться к нежелательным ресурсам.

Защита:

- Allowlist доменов.
    
- Проверка IP-адресов.
    
- Блокировка private/internal ranges.
    
- Контроль redirects.
    
- Ограничение протоколов.
    
- Egress network policies.
    

Особенно опасно разрешать произвольные URL в HTTP clients.

### 11.6. Path Traversal

Пример опасного сценария:

```
$file = $_GET['file'];

readfile('/var/app/files/' . $file);
```

Злоумышленник может попытаться обратиться к файлам за пределами разрешённого каталога.

Защита:

- Не принимать произвольные пути.
    
- Использовать идентификаторы файлов.
    
- Проверять разрешённую директорию.
    
- Использовать allowlist.
    
- Учитывать symlinks.
    

### 11.7. Command Injection

Опасный пример:

```
exec('convert ' . $filename);
```

Ввод пользователя может изменить команду.

Лучше избегать shell execution, если задача может быть решена PHP-библиотекой.

Если shell необходим:

- Использовать `escapeshellarg()` для отдельных аргументов.
    
- Не формировать произвольные команды из пользовательского ввода.
    
- Применять allowlist.
    
- Ограничивать права процесса.
    

### 11.8. Password Hashing

Нельзя хранить пароли в plaintext или использовать MD5/SHA1.

```
$hash = password_hash(
    $password,
    PASSWORD_DEFAULT,
);
```

Проверка:

```
if (password_verify($password, $hash)) {
    // Authenticated
}
```

Обновление хеша:

```
if (password_needs_rehash($hash, PASSWORD_DEFAULT)) {
    $hash = password_hash(
        $password,
        PASSWORD_DEFAULT,
    );
}
```

### 11.9. File Upload Security

Проверки:

- MIME type по содержимому.
    
- Размер файла.
    
- Разрешённые расширения.
    
- Безопасное имя.
    
- Права доступа.
    
- Запрет исполнения загруженных файлов.
    
- Хранение вне web root, если возможно.
    
- Проверка пользовательского доступа к скачиванию.
    

Нельзя доверять `$_FILES['type']` как единственному источнику MIME type.

### 11.10. Cryptography

Для случайных значений:

```
$token = bin2hex(random_bytes(32));
```

Для проверки подписи:

```
$signature = hash_hmac(
    'sha256',
    $payload,
    $secret,
);
```

Для сравнения секретных значений:

```
hash_equals($expected, $actual);
```

Не следует самостоятельно реализовывать криптографические алгоритмы. Используйте проверенные библиотеки и стандартные API.

---

## 12. Architecture and Design Patterns

### 12.1. SOLID

#### S — Single Responsibility Principle

Класс должен иметь одну основную ответственность и одну осмысленную причину для изменения.

Плохой пример: `UserService` одновременно выполняет SQL, отправляет email, формирует HTML и управляет HTTP response.

Хороший вариант — разделить обязанности между application service, repository, mailer и presentation layer.

#### O — Open/Closed Principle

Компоненты должны быть открыты для расширения и закрыты для модификации.

Например, новая платёжная система добавляется через новую реализацию `PaymentGateway`, а не через изменение большого `if/else`.

#### L — Liskov Substitution Principle

Наследник должен корректно заменять базовый тип, не нарушая его контракт.

Нельзя усиливать предусловия или ослаблять постусловия так, чтобы клиентский код ломался.

#### I — Interface Segregation Principle

Лучше несколько небольших интерфейсов, чем один большой интерфейс с методами, которые не нужны всем реализациям.

#### D — Dependency Inversion Principle

Высокоуровневые модули должны зависеть от абстракций, а не от конкретных реализаций.

### 12.2. DRY, KISS, YAGNI

- DRY — не дублировать знания и правила.
    
- KISS — выбирать простые решения.
    
- YAGNI — не реализовывать функциональность до появления реальной необходимости.
    

DRY не означает, что любой похожий код нужно немедленно объединять. Иногда похожие участки имеют разные причины для изменения.

### 12.3. Dependency Injection

Dependency Injection — передача зависимостей объекту извне.

```
class OrderService
{
    public function __construct(
        private OrderRepository $repository,
        private PaymentGateway $gateway,
    ) {
    }
}
```

Преимущества:

- Слабая связанность.
    
- Тестируемость.
    
- Явные зависимости.
    
- Возможность замены реализации.
    
- Упрощение управления жизненным циклом объектов.
    

### 12.4. Service Container

Service Container — объект, который управляет созданием и предоставлением сервисов.

Он может поддерживать:

- Autowiring.
    
- Constructor injection.
    
- Service definitions.
    
- Aliases.
    
- Factories.
    
- Scopes/lifetimes.
    

DI — принцип, Container — один из инструментов реализации.

### 12.5. Factory Pattern

Factory инкапсулирует создание объектов.

```
interface PaymentGateway
{
    public function pay(int $amount): void;
}

class PaymentGatewayFactory
{
    public function create(string $provider): PaymentGateway
    {
        return match ($provider) {
            'stripe' => new StripeGateway(),
            'paypal' => new PaypalGateway(),
            default => throw new InvalidArgumentException(),
        };
    }
}
```

В реальных проектах создание объектов с зависимостями часто делегируют DI container.

### 12.6. Strategy Pattern

Strategy инкапсулирует взаимозаменяемые алгоритмы.

```
interface DiscountStrategy
{
    public function calculate(int $price): int;
}

class RegularDiscount implements DiscountStrategy
{
    public function calculate(int $price): int
    {
        return $price;
    }
}

class PremiumDiscount implements DiscountStrategy
{
    public function calculate(int $price): int
    {
        return (int) ($price * 0.9);
    }
}
```

### 12.7. Observer Pattern

Observer позволяет подписчикам реагировать на событие.

Примеры:

- Отправка email после регистрации.
    
- Запись audit log после изменения заказа.
    
- Обновление статистики.
    
- Запуск фоновой задачи.
    

### 12.8. Adapter Pattern

Adapter позволяет использовать несовместимый внешний интерфейс через собственный контракт.

Например, интеграция нескольких payment providers через единый `PaymentGateway`.

### 12.9. Decorator Pattern

Decorator добавляет поведение объекту без изменения его исходного класса.

Пример: обернуть repository в caching decorator.

### 12.10. Repository Pattern

Инкапсулирует доступ к данным и отделяет application/domain logic от persistence implementation.

### 12.11. Unit of Work

Unit of Work отслеживает изменения объектов и объединяет их в одну операцию сохранения.

Часто используется ORM для управления транзакциями и изменёнными сущностями.

### 12.12. MVC

MVC разделяет приложение на:

- Model.
    
- View.
    
- Controller.
    

Controller принимает запрос и координирует выполнение операции.

Model представляет данные и бизнес-логику в зависимости от конкретной реализации архитектуры.

View отвечает за представление данных.

MVC — паттерн организации приложения, а не готовая архитектура всей системы.

### 12.13. Layered Architecture

Типичное разделение:

```
Presentation Layer
        |
Application Layer
        |
Domain Layer
        |
Infrastructure Layer
```

Presentation:

- Controllers.
    
- HTTP requests/responses.
    
- Serializers.
    

Application:

- Use cases.
    
- Application services.
    
- Command handlers.
    

Domain:

- Entities.
    
- Value Objects.
    
- Domain Services.
    
- Domain Events.
    

Infrastructure:

- Database.
    
- Redis.
    
- Message brokers.
    
- External APIs.
    
- File storage.
    

### 12.14. Clean Architecture

Основная идея — бизнес-правила не должны зависеть от инфраструктуры.

Зависимости направляются внутрь:

- Domain не зависит от Framework.
    
- Application зависит от Domain.
    
- Infrastructure реализует необходимые интерфейсы.
    
- Presentation вызывает application use cases.
    

Это помогает тестировать бизнес-логику отдельно от базы данных и HTTP.

### 12.15. Domain-Driven Design

DDD помогает моделировать сложную предметную область.

Основные понятия:

- Ubiquitous Language.
    
- Bounded Context.
    
- Entity.
    
- Value Object.
    
- Aggregate.
    
- Aggregate Root.
    
- Repository.
    
- Domain Service.
    
- Domain Event.
    

#### Entity

Объект с идентичностью.

Два объекта могут иметь одинаковые свойства, но оставаться разными сущностями из-за разных ID.

#### Value Object

Объект, определяемый значением, а не идентичностью.

Пример: Money, EmailAddress, DateRange.

#### Aggregate

Группа связанных объектов, изменения которой контролируются через Aggregate Root.

Aggregate Root обеспечивает соблюдение бизнес-инвариантов.

Не следует делать агрегат чрезмерно большим.

---

## 13. Testing and Code Quality

### 13.1. Types of Testing

|Type|Назначение|
|---|---|
|Unit|Проверка небольшого изолированного компонента|
|Integration|Проверка взаимодействия компонентов|
|Functional|Проверка пользовательского сценария|
|End-to-end|Проверка полного потока через систему|
|Performance|Проверка скорости и нагрузки|
|Security|Проверка уязвимостей|

### 13.2. PHPUnit

PHPUnit — основной инструмент тестирования PHP.

```
use PHPUnit\Framework\TestCase;

class CalculatorTest extends TestCase
{
    public function testAddition(): void
    {
        $calculator = new Calculator();

        self::assertSame(
            5,
            $calculator->add(2, 3),
        );
    }
}
```

### 13.3. Test Doubles

Основные виды:

- Dummy.
    
- Stub.
    
- Mock.
    
- Spy.
    
- Fake.
    

Stub предоставляет заранее заданные ответы.

Mock проверяет ожидаемые взаимодействия.

Fake — упрощённая рабочая реализация, например in-memory repository.

### 13.4. Unit Testing

Unit test должен проверять конкретное поведение изолированного компонента.

Хороший unit test:

- Независим.
    
- Детерминирован.
    
- Быстро выполняется.
    
- Не требует реальной внешней инфраструктуры.
    
- Проверяет поведение, а не внутреннюю реализацию без необходимости.
    

### 13.5. Integration Testing

Проверяет реальное взаимодействие компонентов.

Например:

- Repository + Database.
    
- HTTP client + test server.
    
- Queue producer + broker.
    
- Application service + persistence.
    

### 13.6. TDD

Test-Driven Development:

1. Red — написать падающий тест.
    
2. Green — реализовать минимальный код.
    
3. Refactor — улучшить реализацию без изменения поведения.
    

TDD не означает, что каждый технический шаг обязательно должен сопровождаться отдельным тестом. Важно, чтобы тесты направляли разработку поведения.

### 13.7. Code Coverage

Coverage показывает, какая часть кода выполнялась во время тестов.

Высокий coverage не гарантирует качество тестирования.

Важно проверять:

- Критичные бизнес-сценарии.
    
- Граничные значения.
    
- Ошибочные сценарии.
    
- Конкурентные операции.
    
- Права доступа.
    
- Интеграции.
    

### 13.8. Static Analysis

Инструменты:

- PHPStan.
    
- Psalm.
    

Они анализируют код без выполнения приложения.

### 13.9. Coding Standards

Инструменты:

- PHP_CodeSniffer.
    
- PHP-CS-Fixer.
    
- PHPStan.
    
- Psalm.
    
- Rector.
    

PHPCS проверяет соответствие стандартам.

PHP-CS-Fixer автоматически форматирует код.

Rector помогает выполнять автоматические преобразования и миграции.

### 13.10. Debugging

Xdebug поддерживает:

- Breakpoints.
    
- Step debugging.
    
- Stack traces.
    
- Variable inspection.
    
- Profiling.
    
- Code coverage.
    

Для production debugging лучше использовать структурированные логи, tracing и metrics, а не подключать интерактивный debugger к пользовательским запросам.

### 13.11. Observability

Три основных направления:

- Logs.
    
- Metrics.
    
- Traces.
    

Примеры:

- Logs: ошибка обработки заказа.
    
- Metrics: количество запросов в секунду.
    
- Traces: путь одного запроса через несколько сервисов.
    

Полезные метрики:

- Latency.
    
- Throughput.
    
- Error rate.
    
- CPU usage.
    
- Memory usage.
    
- Queue depth.
    
- Database query duration.
    

---

## 14. Advanced Backend Development

### 14.1. Caching

Кэширование сохраняет результаты вычислений или запросов, чтобы повторно не выполнять дорогие операции.

Варианты:

- Application cache.
    
- Database query cache.
    
- HTTP cache.
    
- Object cache.
    
- Distributed cache.
    
- CDN cache.
    

### 14.2. Cache Aside

Типичная схема:

1. Приложение проверяет кэш.
    
2. Если значение есть — возвращает его.
    
3. Если значения нет — обращается к БД.
    
4. Сохраняет результат в кэш.
    
5. Возвращает результат.
    

```
$value = $cache->get($key);

if ($value === null) {
    $value = $repository->find($id);

    $cache->set($key, $value, 300);
}
```

### 14.3. Cache Invalidation

Основная сложность кэширования — корректная инвалидация.

Стратегии:

- TTL.
    
- Explicit invalidation.
    
- Tag-based invalidation.
    
- Versioned keys.
    
- Event-driven invalidation.
    

Важно избегать ситуации, когда устаревшие данные продолжают возвращаться после изменения источника.

### 14.4. Redis

Redis — in-memory data store.

Применение:

- Cache.
    
- Sessions.
    
- Queues.
    
- Rate limiting.
    
- Distributed locks.
    
- Counters.
    
- Temporary data.
    

Структуры:

- Strings.
    
- Lists.
    
- Sets.
    
- Sorted Sets.
    
- Hashes.
    
- Streams.
    

Redis не должен автоматически становиться основной базой данных для всех сценариев.

### 14.5. Message Queues

Очереди позволяют отделить обработку задачи от основного HTTP request.

Пример:

1. Пользователь создаёт заказ.
    
2. Приложение сохраняет заказ.
    
3. Создаёт сообщение в очереди.
    
4. Worker получает сообщение.
    
5. Worker отправляет email или выполняет интеграцию.
    

Преимущества:

- Снижение времени HTTP response.
    
- Независимое масштабирование workers.
    
- Повторная обработка.
    
- Буферизация нагрузки.
    

### 14.6. RabbitMQ

RabbitMQ — message broker, поддерживающий AMQP.

Основные компоненты:

- Producer.
    
- Exchange.
    
- Queue.
    
- Binding.
    
- Consumer.
    
- Acknowledgement.
    

Типы exchanges:

- Direct.
    
- Fanout.
    
- Topic.
    
- Headers.
    

### 14.7. Delivery Semantics

Основные модели:

- At-most-once.
    
- At-least-once.
    
- Exactly-once.
    

Exactly-once processing трудно гарантировать в распределённых системах.

На практике часто используют at-least-once delivery и идемпотентную обработку.

### 14.8. Idempotency

Операция идемпотентна, если повторное выполнение не приводит к дополнительному изменению результата после первого успешного выполнения.

Пример:

```
POST /payments
Idempotency-Key: 7d9c...
```

Сервер хранит ключ и результат операции.

При повторном запросе возвращает ранее сохранённый результат, не создавая второй платёж.

### 14.9. Transactional Outbox

Проблема:

1. Записать заказ в БД.
    
2. Отправить событие в брокер.
    
3. Процесс падает между операциями.
    

Возможна ситуация, когда заказ сохранён, но событие не отправлено.

Transactional Outbox решает это:

- Заказ и outbox event записываются в одной DB transaction.
    
- Отдельный publisher читает outbox.
    
- Отправляет сообщения в брокер.
    
- Помечает события обработанными.
    

Это уменьшает риск потери событий между базой данных и брокером.

### 14.10. Retry and Dead Letter Queue

Retry используется для повторения временно неудачных операций.

Рекомендуется:

- Exponential backoff.
    
- Jitter.
    
- Ограниченное количество попыток.
    
- Разделение временных и постоянных ошибок.
    
- Dead Letter Queue для неисправимых сообщений.
    

Нельзя бесконечно повторять операцию без анализа причины ошибки.

### 14.11. Cron and Background Workers

Cron запускает периодические задачи.

Примеры:

- Очистка старых данных.
    
- Отправка отчётов.
    
- Проверка подписок.
    
- Обновление статистики.
    

Queue worker обрабатывает задачи асинхронно.

Cron подходит для расписания, queue worker — для обработки потока задач.

### 14.12. Event-Driven Architecture

Компоненты взаимодействуют через события.

Событие описывает факт, который уже произошёл.

Пример:

```
OrderPlaced
    |
    +--> SendConfirmationEmail
    |
    +--> UpdateAnalytics
    |
    +--> ReserveInventory
```

Важно разделять:

- Domain Events.
    
- Integration Events.
    
- Application Events.
    

События должны иметь стабильный контракт и продуманную схему версионирования.

### 14.13. WebSockets

WebSocket обеспечивает двустороннее соединение между клиентом и сервером.

Применение:

- Чаты.
    
- Live notifications.
    
- Real-time dashboards.
    
- Multiplayer applications.
    

В отличие от обычного HTTP request-response, соединение остаётся открытым.

### 14.14. Rate Limiting

Rate limiting ограничивает количество запросов.

Алгоритмы:

- Fixed Window.
    
- Sliding Window.
    
- Token Bucket.
    
- Leaky Bucket.
    

Используется для:

- Защиты API.
    
- Ограничения brute force.
    
- Защиты внешних интеграций.
    
- Контроля потребления ресурсов.
    

### 14.15. Pagination

Основные варианты:

- Offset pagination.
    
- Cursor pagination.
    

Offset:

```
SELECT *
FROM users
ORDER BY id
LIMIT 20 OFFSET 40;
```

Cursor:

```
SELECT *
FROM users
WHERE id > :last_id
ORDER BY id
LIMIT 20;
```

Cursor pagination обычно лучше масштабируется для больших и изменяющихся наборов данных, если используется подходящий стабильный индекс и порядок сортировки.

### 14.16. Database Locking

Варианты:

- Optimistic locking.
    
- Pessimistic locking.
    

Optimistic locking использует version или updated timestamp.

Pessimistic locking блокирует запись на время операции.

```
SELECT *
FROM accounts
WHERE id = 10
FOR UPDATE;
```

`FOR UPDATE` применяется внутри транзакции и зависит от возможностей СУБД.

### 14.17. Race Conditions

Race condition возникает, когда результат зависит от порядка выполнения параллельных операций.

Пример:

1. Два запроса одновременно читают баланс 100.
    
2. Оба пытаются списать 80.
    
3. Оба считают операцию допустимой.
    
4. Возникает некорректный результат.
    

Решения:

- Transactions.
    
- Row-level locking.
    
- Atomic SQL updates.
    
- Optimistic locking.
    
- Idempotency.
    
- Unique constraints.
    

### 14.18. Distributed Systems

Основные проблемы:

- Network failures.
    
- Partial failures.
    
- Latency.
    
- Retries.
    
- Duplicate messages.
    
- Data consistency.
    
- Clock differences.
    
- Service discovery.
    
- Cascading failures.
    

Важные паттерны:

- Circuit Breaker.
    
- Retry.
    
- Timeout.
    
- Bulkhead.
    
- Saga.
    
- Outbox.
    
- Idempotent Consumer.
    

### 14.19. Circuit Breaker

Circuit Breaker временно прекращает обращения к неисправному сервису.

Состояния:

- Closed.
    
- Open.
    
- Half-open.
    

Цель — не допустить каскадного отказа системы.

---

## 15. Frameworks and PHP Ecosystem

### 15.1. Laravel

Laravel — PHP framework, предоставляющий готовые инструменты для разработки приложений.

Основные компоненты:

- Routing.
    
- Controllers.
    
- Middleware.
    
- Service Container.
    
- Eloquent ORM.
    
- Migrations.
    
- Queues.
    
- Events.
    
- Jobs.
    
- Validation.
    
- Authentication.
    
- Cache.
    
- Testing.
    

### 15.2. Symfony

Symfony — PHP framework и набор переиспользуемых компонентов.

Основные компоненты:

- HttpFoundation.
    
- HttpKernel.
    
- Routing.
    
- DependencyInjection.
    
- EventDispatcher.
    
- Console.
    
- Serializer.
    
- Validator.
    
- Messenger.
    
- Security.
    
- Doctrine integration.
    

### 15.3. Dependency Injection Container

В Symfony container создаёт и связывает сервисы.

Пример конфигурации:

```
services:
  App\Service\OrderService:
    arguments:
      $repository: '@App\Repository\OrderRepository'
```

Контейнер может использовать autowiring и autoconfiguration.

### 15.4. Middleware

Middleware обрабатывает HTTP request до или после основного handler.

Применение:

- Authentication.
    
- Authorization.
    
- Logging.
    
- Rate limiting.
    
- CORS.
    
- Request ID.
    
- Error handling.
    

### 15.5. ORM

ORM связывает объекты приложения с реляционной базой данных.

Примеры:

- Doctrine ORM.
    
- Eloquent.
    

Преимущества:

- Работа с объектами.
    
- Упрощение CRUD.
    
- Relationships.
    
- Unit of Work.
    
- Identity Map.
    

Недостатки:

- Сложные запросы могут быть неэффективными.
    
- Возможны N+1.
    
- Не всегда очевидно, какой SQL выполняется.
    
- Нужны знания самой СУБД.
    

Senior-разработчик должен понимать SQL независимо от ORM.

### 15.6. Migrations

Migrations позволяют версионировать изменения структуры базы данных.

Пример:

```
final class Version20260101000000
{
    public function up(): void
    {
        // Create table
    }

    public function down(): void
    {
        // Revert change
    }
}
```

В production migrations необходимо проектировать с учётом:

- Backward compatibility.
    
- Больших таблиц.
    
- Locking.
    
- Deployment order.
    
- Rollback strategy.
    

Для крупных изменений часто используется expand-and-contract:

1. Добавить новую структуру.
    
2. Поддерживать старый и новый формат.
    
3. Перенести данные.
    
4. Переключить приложение.
    
5. Удалить старую структуру в отдельном релизе.
    

### 15.7. Drupal Architecture

Drupal — CMS и framework, построенный на PHP и Symfony components.

Основные API:

- Entity API.
    
- Field API.
    
- Plugin API.
    
- Form API.
    
- Render API.
    
- Configuration API.
    
- Services and Dependency Injection.
    
- Events API.
    
- Hooks.
    
- Routing.
    
- Access API.
    
- Cache API.
    
- Queue API.
    
- Batch API.
    
- Migrate API.
    
- REST API.
    
- JSON.
    

### 15.8. Drupal Services

Сервисы определяются в `*.services.yml`.

```
services:
  app.order_manager:
    class: Drupal\app\Service\OrderManager
    arguments:
      - '@entity_type.manager'
```

Зависимости передаются через constructor injection.

Прямой вызов `\Drupal::service()` допустим в некоторых контекстах, но constructor injection предпочтительнее для тестируемости и явного управления зависимостями.

### 15.9. Drupal Entities

Entity — объект предметных данных Drupal.

Основные типы:

- Content entities.
    
- Configuration entities.
    

Content entities обычно представляют содержимое с полями и revision/translation support.

Configuration entities представляют конфигурационные объекты, например типы материалов или представления в соответствующих моделях Drupal.

### 15.10. Drupal Plugin API

Plugin API позволяет создавать расширяемые реализации определённых типов поведения.

Примеры:

- Blocks.
    
- Field widgets.
    
- Field formatters.
    
- Views plugins.
    
- Queue workers.
    

### 15.11. Drupal Cache API

Основные понятия:

- Cache tags.
    
- Cache contexts.
    
- Cache max-age.
    
- Cache metadata bubbling.
    

Cache tags отвечают за инвалидацию при изменении данных.

Cache contexts определяют, для какого контекста пользователя или запроса отличается результат.

Max-age определяет срок действия кэша.

### 15.12. Drupal Configuration Management

Конфигурация экспортируется и импортируется через YAML.

```
drush cex
drush cim
drush cr
```

В типичном workflow:

1. Изменить конфигурацию.
    
2. Экспортировать её.
    
3. Закоммитить YAML.
    
4. Развернуть код.
    
5. Импортировать конфигурацию.
    
6. Очистить кэш при необходимости.
    

---

## 16. Senior-Level Engineering Practices

### 16.1. Code Review

Senior-разработчик проверяет не только синтаксис.

Основные критерии:

- Correctness.
    
- Security.
    
- Performance.
    
- Maintainability.
    
- Testability.
    
- Backward compatibility.
    
- Error handling.
    
- Observability.
    
- Operational impact.
    

Вопросы при review:

- Что произойдёт при null?
    
- Что будет при повторном запросе?
    
- Возможна ли race condition?
    
- Не возникнет ли N+1?
    
- Что произойдёт при недоступности внешнего сервиса?
    
- Не раскрываются ли чувствительные данные?
    
- Есть ли обратная совместимость?
    
- Как откатить изменение?
    

### 16.2. Refactoring

Refactoring изменяет внутреннюю структуру кода без изменения внешнего поведения.

Примеры:

- Extract Method.
    
- Extract Class.
    
- Replace Conditional with Polymorphism.
    
- Introduce Parameter Object.
    
- Replace Magic Number with Constant.
    
- Dependency Injection.
    
- Reduce Coupling.
    

Перед серьёзным рефакторингом желательно иметь тесты, фиксирующие текущее поведение.

### 16.3. Technical Debt

Technical Debt — последствия решений, которые ускорили текущую разработку, но увеличили будущую стоимость изменений.

Примеры:

- Дублирование бизнес-логики.
    
- Отсутствие тестов.
    
- Непрозрачная архитектура.
    
- Устаревшие зависимости.
    
- Слишком большие классы.
    
- Неправильные границы модулей.
    

Senior должен уметь оценивать технический долг и приоритизировать его устранение.

### 16.4. Backward Compatibility

Изменение может быть технически правильным, но сломать существующих клиентов.

Нужно учитывать:

- Public API.
    
- Database schema.
    
- Event contracts.
    
- Message formats.
    
- Configuration.
    
- External integrations.
    

Для безопасных изменений применяются versioning, deprecation и staged migrations.

### 16.5. Performance Optimization

Правильный порядок:

1. Измерить проблему.
    
2. Найти bottleneck.
    
3. Сформулировать гипотезу.
    
4. Изменить код или инфраструктуру.
    
5. Повторить измерения.
    
6. Проверить отсутствие регрессий.
    

Инструменты:

- Xdebug profiler.
    
- Blackfire.
    
- Tideways.
    
- APM.
    
- Database EXPLAIN.
    
- Application metrics.
    

Оптимизировать без измерений — плохая инженерная практика.

### 16.6. Scalability

Vertical scaling — увеличение ресурсов одного сервера.

Horizontal scaling — увеличение количества экземпляров приложения.

Для горизонтального масштабирования обычно нужны:

- Stateless application instances.
    
- Shared session storage.
    
- Centralized cache.
    
- Shared file storage.
    
- Message broker.
    
- Load balancer.
    
- Database scaling strategy.
    

### 16.7. High Availability

Высокая доступность достигается через:

- Redundancy.
    
- Health checks.
    
- Failover.
    
- Replication.
    
- Monitoring.
    
- Backups.
    
- Disaster recovery.
    

Backup без проверенного восстановления не гарантирует возможность восстановить систему.

### 16.8. Deployment Strategy

Основные варианты:

- Rolling deployment.
    
- Blue-green deployment.
    
- Canary deployment.
    

Нужно учитывать совместимость старой и новой версии приложения с базой данных и сообщениями в очередях.

### 16.9. CI/CD

Типичный pipeline:

```
Commit
  |
  v
Composer install
  |
  v
Coding Standards
  |
  v
Static Analysis
  |
  v
Unit Tests
  |
  v
Integration Tests
  |
  v
Build Artifact
  |
  v
Deploy
  |
  v
Smoke Tests
```

CI — автоматическая проверка изменений.

CD — автоматизированная доставка и/или развёртывание изменений.

### 16.10. Architecture Decision Records

ADR фиксирует важные архитектурные решения.

Обычно включает:

- Context.
    
- Decision.
    
- Alternatives.
    
- Consequences.
    

Это помогает команде понимать, почему было принято определённое решение.

---

## 17. Senior PHP Interview Questions

### 17.1. PHP Language

**1. What is the difference between** `**==**` **and** `**===**`**?**

`==` сравнивает значения с возможным type juggling. `===` проверяет и значение, и тип без такого преобразования. В большинстве бизнес-сценариев предпочтительнее строгое сравнение.

**2. What is the difference between** `**isset()**` **and** `**array_key_exists()**`**?**

`isset()` возвращает false, если ключ отсутствует или его значение равно null. `array_key_exists()` проверяет именно наличие ключа, даже если значение null.

**3. What is the difference between** `**include**` **and** `**require**`**?**

Обе конструкции подключают файл. При отсутствии файла `include` генерирует warning, а `require` — ошибку, прекращающую нормальное выполнение. В современных проектах для классов используется Composer autoloading.

**4. What is the difference between a closure and an arrow function?**

Closure может содержать произвольный блок кода и захватывает внешние переменные через `use`. Arrow function имеет компактный синтаксис и автоматически захватывает внешние переменные по значению.

**5. What is a generator?**

Generator возвращает значения последовательно через `yield`, не создавая заранее весь массив. Это полезно для потоковой обработки больших наборов данных.

**6. What is late static binding?**

Late static binding позволяет использовать фактический класс вызова через `static`, в отличие от `self`, который связан с классом объявления метода.

### 17.2. OOP

**7. What is the difference between an abstract class and an interface?**

Abstract class может содержать общую реализацию и состояние. Interface определяет контракт, который могут реализовывать разные классы. Класс может реализовывать несколько интерфейсов, но наследоваться только от одного класса.

**8. What is composition over inheritance?**

Composition означает построение поведения через сотрудничество объектов. Она обычно уменьшает связанность и делает архитектуру более гибкой по сравнению с глубокой иерархией наследования.

**9. What is Dependency Injection?**

Dependency Injection — передача зависимостей объекту извне. Это снижает связанность, упрощает тестирование и позволяет заменять реализации.

**10. What is the difference between** `**self**` **and** `**static**`**?**

`self` относится к классу, в котором объявлен метод. `static` учитывает фактический класс вызова и поддерживает late static binding.

**11. What is the difference between an Entity and a Value Object?**

Entity имеет идентичность, обычно определяемую ID. Value Object определяется своими значениями и обычно проектируется как неизменяемый объект.

### 17.3. Runtime and Performance

**12. What is Opcache?**

Opcache хранит скомпилированные инструкции PHP в shared memory и позволяет повторно использовать их между запросами, уменьшая overhead компиляции.

**13. What is Copy-on-Write?**

PHP может разделять данные между копиями значений и создавать отдельную копию только при изменении. Это позволяет уменьшить ненужное потребление памяти.

**14. What is the difference between PHP-FPM and PHP CLI?**

CLI запускает PHP из командной строки. PHP-FPM управляет worker processes, которые обрабатывают веб-запросы через FastCGI.

**15. How would you optimize a slow PHP application?**

Сначала измерил бы производительность и определил bottleneck. Затем проверил SQL-запросы, N+1, внешние вызовы, алгоритмы, кэширование, сериализацию и использование памяти. После изменений повторил бы измерения.

### 17.4. Database

**16. What is a transaction?**

Транзакция объединяет несколько операций в одну логическую единицу, которая должна удовлетворять ACID.

**17. What is the N+1 problem?**

Это ситуация, когда один запрос получает список сущностей, а затем выполняется дополнительный запрос для каждого элемента. Решения — JOIN, eager loading и batch queries.

**18. What is the difference between optimistic and pessimistic locking?**

Optimistic locking обнаруживает конфликт при сохранении, обычно через version field. Pessimistic locking блокирует запись на время операции.

**19. Why are indexes important?**

Индексы ускоряют поиск и сортировку, но увеличивают место хранения и стоимость изменений данных. Их следует выбирать на основании реальных запросов и EXPLAIN.

### 17.5. Architecture

**20. What is the Dependency Inversion Principle?**

Высокоуровневые модули должны зависеть от абстракций, а не от конкретных низкоуровневых реализаций.

**21. What is the difference between an application service and a domain service?**

Application service координирует выполнение use case. Domain service содержит бизнес-операцию, которая не принадлежит естественным образом одной Entity или Value Object.

**22. What is the Repository Pattern?**

Repository предоставляет абстракцию доступа к данным, отделяя application/domain logic от конкретной persistence implementation.

**23. What is the Transactional Outbox Pattern?**

Он гарантирует, что изменение бизнес-данных и запись события в outbox выполняются в одной транзакции. Отдельный процесс публикует события в брокер.

### 17.6. Security

**24. How do you prevent SQL Injection?**

Использую prepared statements и parameter binding. Динамические имена таблиц и колонок выбираю из allowlist, поскольку SQL placeholders не предназначены для таких идентификаторов.

**25. What is the difference between authentication and authorization?**

Authentication устанавливает личность пользователя. Authorization определяет, какие действия ему разрешены.

**26. How do you store passwords securely?**

Использую `password_hash()` и `password_verify()`. Не храню пароли в plaintext и не применяю обычные быстрые хеш-функции вроде MD5 или SHA1.

**27. Is JWT encrypted?**

Обычный signed JWT не шифрует payload. Его содержимое можно прочитать. Подпись защищает целостность, но не конфиденциальность.

### 17.7. Distributed Systems

**28. What is idempotency?**

Повторное выполнение операции не должно создавать дополнительные эффекты после успешного первого выполнения. Для этого применяются idempotency keys и сохранение результатов операций.

**29. What is a Circuit Breaker?**

Это паттерн, который временно блокирует обращения к неисправному внешнему сервису, чтобы избежать каскадного отказа.

**30. How do you handle duplicate messages in a queue?**

Использую идемпотентных consumers, уникальные идентификаторы сообщений, ограничения БД и сохранение статуса обработки.

**31. What is eventual consistency?**

Данные в разных компонентах системы могут временно отличаться, но при успешной доставке событий и отсутствии новых изменений со временем приходят к согласованному состоянию.

---

## 18. Practical Senior PHP Tasks

Для Senior-собеседования важно уметь не только отвечать на вопросы, но и проектировать решения.

### Task 1. Design a Payment System

Нужно спроектировать систему оплаты с несколькими провайдерами.

Что следует предусмотреть:

- `PaymentGateway` interface.
    
- Отдельные реализации провайдеров.
    
- Dependency Injection.
    
- Payment entity.
    
- Transaction management.
    
- Idempotency key.
    
- Retry policy.
    
- Webhook verification.
    
- Logging.
    
- Reconciliation process.
    

### Task 2. Optimize a Slow API

API возвращает 1000 заказов и выполняет тысячи SQL-запросов.

План:

1. Измерить время и количество запросов.
    
2. Найти N+1.
    
3. Проверить индексы.
    
4. Использовать eager loading или JOIN.
    
5. Ввести pagination.
    
6. Добавить кэширование, если оно оправдано.
    
7. Проверить нагрузку повторно.
    

### Task 3. Design a Notification System

Нужно отправлять email, SMS и push notifications.

Возможная архитектура:

```
Application
    |
NotificationService
    |
NotificationInterface
    |
    +--> EmailNotification
    +--> SmsNotification
    +--> PushNotification
```

Дальнейшая обработка может выполняться через очередь.

### Task 4. Design an Order Processing System

Компоненты:

- Order.
    
- OrderRepository.
    
- PaymentService.
    
- InventoryService.
    
- OrderPlaced event.
    
- Message Broker.
    
- Notification Worker.
    
- Outbox Publisher.
    

Необходимо предусмотреть:

- Повторную доставку сообщений.
    
- Идемпотентность.
    
- Транзакции.
    
- Ошибки внешних сервисов.
    
- Мониторинг.
    
- Компенсирующие операции.
    

### Task 5. Refactor a Legacy PHP Application

План:

1. Зафиксировать текущее поведение тестами.
    
2. Найти критичные зависимости.
    
3. Выделить бизнес-логику.
    
4. Уменьшить связанность.
    
5. Внедрить interfaces и DI там, где они действительно нужны.
    
6. Добавить static analysis.
    
7. Постепенно улучшать архитектуру.
    
8. Сохранять backward compatibility.
    

---

## 19. What a Senior PHP Developer Should Know

Senior PHP Developer должен уверенно владеть следующими областями:

### PHP Language

- Type system.
    
- OOP.
    
- References.
    
- Closures.
    
- Generators.
    
- Iterators.
    
- Enums.
    
- Attributes.
    
- Reflection.
    
- Error handling.
    
- PHP internals at a practical level.
    

### Runtime

- Zend Engine.
    
- Opcache.
    
- PHP-FPM.
    
- Memory management.
    
- Garbage collection.
    
- Copy-on-Write.
    
- Long-running processes.
    

### Backend

- HTTP.
    
- REST API.
    
- Authentication.
    
- Authorization.
    
- Sessions.
    
- Caching.
    
- Queues.
    
- Database transactions.
    
- SQL optimization.
    
- Distributed systems.
    

### Architecture

- SOLID.
    
- Design patterns.
    
- Dependency Injection.
    
- Clean Architecture.
    
- DDD.
    
- Event-driven architecture.
    
- Modular monoliths.
    
- Microservices trade-offs.
    

### Engineering

- PHPUnit.
    
- Integration testing.
    
- Static analysis.
    
- Code review.
    
- Refactoring.
    
- CI/CD.
    
- Observability.
    
- Performance profiling.
    
- Security.
    

### Infrastructure

- Composer.
    
- Linux.
    
- Nginx.
    
- PHP-FPM.
    
- MySQL/PostgreSQL.
    
- Redis.
    
- RabbitMQ.
    
- Docker.
    
- Git.
    

---

## 20. Recommended Learning Order

Для системной подготовки рекомендуется проходить темы в следующем порядке:

1. PHP Fundamentals.
    
2. Arrays and Strings.
    
3. Advanced Language Features.
    
4. OOP.
    
5. Type System.
    
6. Error Handling.
    
7. PHP Runtime and Memory.
    
8. Composer and PSR.
    
9. SQL and PDO.
    
10. HTTP and REST.
    
11. Security.
    
12. Architecture and Design Patterns.
    
13. Testing and Static Analysis.
    
14. Caching and Queues.
    
15. Distributed Systems.
    
16. Framework Architecture.
    
17. Performance and Scalability.
    
18. Senior-level System Design.
    

Главный критерий Senior-уровня — не количество изученных функций PHP, а способность выбирать подходящее решение, аргументировать архитектурные решения, учитывать ограничения системы и отвечать за качество решения в production.