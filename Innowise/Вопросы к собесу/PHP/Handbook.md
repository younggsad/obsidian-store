---
title: "PHP 8.4+ Senior Technical Interview: Comprehensive Bilingual Q&A Handbook"
tags:
  - php
  - interview-prep
  - architecture
  - internals
  - security
  - database
cssclasses:
  - wide-page
custom-width: 80
---

# PHP 8.4+ Senior Technical Interview: Comprehensive Bilingual Q&A Handbook

> [!abstract] Format Guide
> Each entry follows the strict four-step pattern:
> 1. **RU Question:** Вопрос на русском языке
> 2. **EN Question:** Question in English
> 3. **RU Answer:** Детальный ответ и технические нюансы
> 4. **EN Answer:** Verbatim professional interview response ready to speak aloud

---

## 1. Engine Internals, OPcache & Execution Lifecycle

### Q1: Пайплайн выполнения кода в Zend Engine / Code Execution Pipeline

> [!question] RU Question
> Как именно Zend Engine выполняет PHP-код от исходного файла до выполнения процессором?

> [!question] EN Question
> How does the Zend Engine execute PHP code from source file to CPU execution?

> [!check] RU Answer
> PHP не интерпретирует код построчно. Движок проходит 4 фазы:
> 1. **Лексический анализ (Lexing / Scanning):** Исходный текст на C-уровне разбивается генератором сканеров `re2c` на поток лексем/токенов (`T_VARIABLE`, `T_STRING`).
> 2. **Синтаксический анализ (Parsing):** Парсер LALR(1) на базе Bison преобразует токены в Абстрактное синтаксическое дерево (AST). На этой стадии происходит constant folding (сворачивание констант, например вычисление `60 * 60 * 24` прямо при сборке дерева).
> 3. **Компиляция (Compilation):** AST обходится рекурсивно и преобразуется в массив опкодов (`zend_op_array`). Опкоды оперируют типами операндов: `CV` (скомпилированные переменные), `TMPVAR`, `VAR`, `CONST`.
> 4. **Исполнение (Virtual Machine Execution):** Виртуальная машина Zend VM запускает цикл выполнения опкодов с прямым переходом (`goto *handler`), вызывая обработчики на языке C.

> [!quote] EN Answer
> *"PHP does not interpret source code line by line. The Zend Engine processes scripts in four phases: first, lexical analysis converts raw code into tokens via `re2c`. Second, the Bison parser builds an Abstract Syntax Tree (AST), performing optimizations like constant folding. Third, the AST is compiled into a `zend_op_array` containing low-level opcodes. Finally, the Zend Virtual Machine dispatches these opcodes via direct-threaded C handlers to execute the program."*

---

### Q2: Работа OPcache и Preloading / OPcache & Preloading Internals

> [!question] RU Question
> Что именно делает расширение OPcache и как работает механизм Preloading в PHP 7.4+?

> [!question] EN Question
> What exactly does OPcache do under the hood, and how does Preloading work in PHP 7.4+?

> [!check] RU Answer
> * **OPcache:** Хранит уже скомпилированные массивы опкодов (`zend_op_array`) в разделяемой оперативной памяти (Shared Memory, SHM). При повторном запросе движок полностью пропускает шаги Tokenizing, Parsing и Compilation, читая готовые опкоды из RAM. Также OPcache дедуплицирует строковые литералы в пул интернированных строк (`IS_STR_INTERNED`).
> * **Preloading (`opcache.preload`):** Запускает специальный PHP-скрипт один раз при инициализации процесса веб-сервера (до того, как PHP-FPM открывает сокет и начинает принимать запросы). Выбранные классы, трейты и интерфейсы компилируются и навечно вшиваются в глобальную таблицу классов ядра (`CG(class_table)`). Они становятся доступны всем запросам без autoloading, экономя время на `include`/`require`. Минус: обновление кода требует перезагрузки мастер-процесса FPM (`SIGUSR2` или рестарт контейнера).

> [!quote] EN Answer
> *"OPcache stores compiled opcode arrays (`zend_op_array`) directly in shared memory (`shm`), completely bypassing lexical scanning, AST generation, and compilation for subsequent requests. Preloading compiles framework and application classes into the persistent global class table (`CG(class_table)`) during FPM master process startup, before binding to the socket. This eliminates class-loading overhead per request, but deploying code changes requires restarting the PHP-FPM master process."*

---

### Q3: Shared-Nothing (FPM) против Worker Mode / Shared-Nothing vs Worker Mode

> [!question] RU Question
> В чём архитектурная разница между PHP-FPM (Shared-Nothing) и воркер-серверами вроде RoadRunner/FrankenPHP/Swoole?

> [!question] EN Question
> What is the architectural difference between PHP-FPM (Shared-Nothing) and resident worker runtimes like RoadRunner, FrankenPHP, or Swoole?

> [!check] RU Answer
> * **PHP-FPM (Shared-Nothing):** Каждый запрос попадает в изолированный процесс. По завершении ответа ядро вызывает `sapi_deactivate`, полностью сбрасывая всю аллоцированную память, глобальные массивы и статические переменные. Это гарантирует 100% изоляцию пользователей и устойчивость к утечкам памяти, но заставляет инициализировать фреймворк (bootstrap) на каждый запрос.
> * **Worker Mode (RoadRunner / FrankenPHP / Swoole):** Приложение инициализируется один раз и крутится в памяти постоянно в бесконечном цикле обработки событий (Event Loop). Bootstrap выполняется единожды, что снижает задержку (latency) до единиц миллисекунд.
> * **Опасность Worker Mode:** Статические свойства, сервисы-синглтоны и кэши в памяти не сбрасываются. Обязателен сброс состояния: `EntityManager::clear()` в Doctrine, очистка очередей событий и ручной контроль утечек памяти.

> [!quote] EN Answer
> *"PHP-FPM implements a Shared-Nothing architecture where `sapi_deactivate` cleans up all memory, objects, and static state at the end of every request. This prevents state leakage across tenants, but introduces framework bootstrap overhead per request. In contrast, worker runtimes like RoadRunner or FrankenPHP bootstrap the framework once and keep the process resident in memory, handling requests sequentially or via event loops. This reduces latency, but requires strict state management: singletons persist, meaning ORM identity maps and static caches must be explicitly flushed between requests to prevent memory and tenant leaks."*

---

## 2. Memory Internals, zval & Garbage Collection

### Q4: Внутреннее устройство структуры `zval` / Internal Structure of `zval`

> [!question] RU Question
> Как значение переменной хранится внутри ядра PHP (структура `zval`) и почему примитивы не аллоцируют память в куче?

> [!question] EN Question
> How is a variable stored inside the Zend Engine (`zval` structure), and why do primitives not allocate heap memory?

> [!check] RU Answer
> Начиная с PHP 7, структура `zval` оптимизирована до **16 байт**:
> * `value` (8 байт): C-union. Хранит либо непосредственно скалярное значение, либо указатель на сложную структуру в куче (`zend_string*`, `zend_array*`, `zend_object*`).
> * `u1` (4 байта): метаданные типа (`type`, `type_flags`).
> * `u2` (4 байта): служебные флаги для оптимизатора, кэш хэш-таблицы.
> 
> Скаляры (`int`, `float`, `bool`, `null`) хранятся прямо внутри union `value`. Они лежат в стеке вызовов (call stack), не вызывают системных вызовов аллокации памяти `malloc`/`emalloc` и не имеют счетчика ссылок (`refcount`).

> [!quote] EN Answer
> *"In modern PHP, a `zval` is a 16-byte C structure consisting of an 8-byte value union, 4 bytes for type tags and flags (`u1`), and 4 bytes for internal engine metadata (`u2`). Primitives like integers, floats, booleans, and null are stored directly inside the value union on the execution stack. They never trigger heap allocations via `emalloc` and do not utilize reference counting."*

---

### Q5: Механизм Copy-on-Write (CoW) / Copy-on-Write (CoW) Mechanism

> [!question] RU Question
> Как работает механизм Copy-on-Write (CoW) и в какой момент происходит физическое дублирование памяти?

> [!question] EN Question
> How does the Copy-on-Write (CoW) mechanism work, and at what exact moment does physical memory duplication occur?

> [!check] RU Answer
> При присваивании `$b = $a` или передаче массива в функцию по значению физического копирования данных в памяти **не происходит**.
> 1. Второй `zval` начинает указывать на ту же область памяти (`zend_array` или `zend_string`).
> 2. Поле `gc.refcount` составной структуры увеличивается на 1.
> 3. Физическое дублирование памяти (аллокация нового буфера и копирование элементов) происходит **только при попытке записи (модификации)** в одну из переменных, если её `refcount > 1`.
> 4. Если `refcount === 1`, движок выполняет модификацию прямо на месте (in-place mutation) без выделения новой памяти.

> [!quote] EN Answer
> *"PHP passes arrays and strings by value using Copy-on-Write (CoW). Assigning an array `$b = $a` does not clone memory; both `zval` structs point to the same heap buffer and increment `gc.refcount`. Physical allocation and duplication take place only when one variable undergoes mutation while its `refcount` is greater than 1. If `refcount` equals 1, the mutation is performed in-place without memory allocation."*

---

### Q6: Циклические ссылки и сборщик мусора / Circular References & Cycle Collector

> [!question] RU Question
> Почему счетчик ссылок не может освободить циклические графы объектов и как работает Cycle Collector в PHP?

> [!question] EN Question
> Why can standard reference counting not collect cyclic object graphs, and how does the Cycle Collector work in PHP?

> [!check] RU Answer
> Обычный refcounting немедленно освобождает память, как только `refcount` становится равен 0. Однако если Объект A ссылается на Объект B, а B ссылается на A, то после вызова `unset($a, $b)` обе внешние переменные уничтожаются, но взаимный `refcount` каждого объекта остаётся равен 1. Возникает утечка памяти.
> 
> **Алгоритм Бэкона-Раджана (Cycle Collector):**
> 1. Составные типы, у которых `refcount` уменьшился, но не стал равен 0, помечаются фиолетовым цветом (Purple) и помещаются в Root Buffer (размер: 10 000 ячеек).
> 2. Когда буфер заполняется, запускается сборка: алгоритм виртуально обходит ссылки внутри графа и уменьшает счетчики на 1 (раскрашивая узлы в Grey).
> 3. Если после этого у узла `refcount` стал равен 0 — значит, он поддерживался только ссылками из своего замкнутого кольца. Он помечается белым (White) — это мусор.
> 4. Если `refcount > 0`, узел достижим снаружи, счетчик восстанавливается (Black).
> 5. Все белые (White) узлы физически удаляются из памяти.

> [!quote] EN Answer
> *"Standard reference counting deallocates memory immediately when `refcount` drops to zero, but fails on circular references because mutually referring objects maintain a `refcount` of at least 1 even after `unset()`. To resolve this, the Bacon-Rajan Cycle Collector buffers suspicious nodes whose refcount decrements without reaching zero into a 10,000-entry root buffer. When full, it performs trial deletion by temporarily decrementing internal reference counts: nodes reaching zero are confirmed garbage (marked White) and swept from memory, while surviving nodes are restored and marked Black."*

---

### Q7: Опасность ссылок `&` в циклах foreach / The Foreach Reference Trap

> [!question] RU Question
> В чём заключается баг использования ссылок в цикле `foreach ($arr as &$item)` и как его предотвратить?

> [!question] EN Question
> What is the bug associated with references in `foreach ($arr as &$item)`, and how do you prevent it?

> [!check] RU Answer
> Конструкция `&$item` оборачивает значение в `zend_reference`. После завершения цикла переменная `$item` **не уничтожается** — она остаётся висеть в текущей области видимости как активная ссылка на последний элемент массива (`$arr[last]`).
> Если дальше по коду запустить обычный цикл `foreach ($arr as $item)`, то на каждой итерации значение `$item` будет перезаписываться текущим элементом, и поскольку это ссылка, оно будет физически перезаписывать **последний элемент оригинального массива**.
> **Решение:** Всегда явно вызывать `unset($item);` сразу после завершения цикла по ссылке.

> [!quote] EN Answer
> *"Using `foreach ($arr as &$item)` leaves `$item` as an active reference pointing to the array's last element even after the loop exits. If a subsequent loop reuses `$item` by value (`foreach ($arr as $item)`), every iteration writes the current value directly into that referenced final element, corrupting the data. To prevent this, always explicitly destroy the reference variable by calling `unset($item);` immediately after the loop."*

---

## 3. Modern PHP (8.0 – 8.4): Types, OOP & Engine Features

### Q8: Callable против Closure / Callable vs Closure

> [!question] RU Question
> В чём принципиальная разница между псевдотипом `callable` и объектом `\Closure`?

> [!question] EN Question
> What is the fundamental difference between the `callable` typehint and the `\Closure` class?

> [!check] RU Answer
> * `callable`: Псевдотип. Принимает строки с именами функций (`'trim'`), массивы методов (`[$this, 'action']`), инвокабельные объекты с `__invoke` и замыкания. Валидируется только в рантайме при попытке вызова. **Нельзя использовать для типизации свойств класса** (`public callable $fn;` — Fatal Error).
> * `\Closure`: Конкретный финальный встроенный класс в PHP. Представляет анонимные функции и результаты First-Class Callable синтаксиса (`...`). Его **можно использовать для типизации свойств класса** (`public Closure $fn;`).

> [!quote] EN Answer
> *"`callable` is a loose pseudotype accepting strings, array method pointers, invokable objects, and closures, evaluated only at runtime when invoked. Crucially, `callable` cannot be used as a class property type. In contrast, `\Closure` is a concrete, final internal class representing an anonymous function or a first-class callable. It is fully valid as a typed class property, return type, and argument."*

---

### Q9: First-Class Callable Syntax (`...`) / First-Class Callable Syntax

> [!question] RU Question
> Как устроен First-Class Callable синтаксис (PHP 8.1+) под капотом и какие преимущества он даёт?

> [!question] EN Question
> How does the First-Class Callable syntax (PHP 8.1+) work under the hood, and what advantages does it offer?

> [!check] RU Answer
> Синтаксис вида `strlen(...)` или `$this->format(...)` компилируется в специальный опкод `ZEND_CALLABLE_CONVERT`.
> **Преимущества перед старыми массивами `[$this, 'format']`:**
> 1. Существование метода и права видимости (`public`, `protected`, `private`) проверяются **в момент создания ссылки**, а не через 10 минут во время вызова.
> 2. Возвращает строгий объект `\Closure`, привязанный к текущему `$this`.
> 3. Доступен автокомплит в IDE, переименование через рефакторинг и статический анализ типов в PHPStan/Psalm.

> [!quote] EN Answer
> *"The First-Class Callable syntax (`$this->method(...)`) compiles to the dedicated opcode `ZEND_CALLABLE_CONVERT`. Unlike fragile string or array callbacks, it evaluates method existence and visibility (`private`/`protected`) immediately at reference creation time rather than at execution time. It returns an instance of `\Closure` with `$this` bound, enabling full static analysis and IDE refactoring support."*

---

### Q10: DNF Types (Disjunctive Normal Form) / DNF Types

> [!question] RU Question
> Что такое DNF-типы (PHP 8.2+) и какие правила их составления?

> [!question] EN Question
> What are DNF types (PHP 8.2+), and what are the rules for writing them?

> [!check] RU Answer
> DNF (Дизъюнктивная нормальная форма) — это способ объединения пересечений: **ИЛИ из групп И**.
> Позволяет комбинировать Union (`|`) и Intersection (`&`) типы.
> **Правила:**
> 1. Группы пересечений (`&`) **обязаны** быть обёрнуты в круглые скобки: `(A&B)|C`.
> 2. Пересечения (`&`) разрешены только для имён классов и интерфейсов (скаляры в `&` запрещены).
> 3. Нельзя делать двойную вложенность скобок: `((A&B)|C)&D` запрещено.
> 
> *Пример:* `(HasId&HasSlug)|string|null` — принимает либо объект с ID и Slug одновременно, либо строку, либо null.

> [!quote] EN Answer
> *"DNF (Disjunctive Normal Form) types introduced in PHP 8.2 standardize combining Union (`|`) and Intersection (`&`) types in the format: OR of ANDs. The syntax strictly mandates wrapping intersection groups in parentheses, such as `(A&B)|(C&D)|null`. Intersection segments apply exclusively to classes and interfaces, while scalars can only be added via top-level unions."*

---

### Q11: Возвращаемые типы: `void` против `never` / Return Types: `void` vs `never`

> [!question] RU Question
> В чём разница между типами возврата `void` и `never`?

> [!question] EN Question
> What is the difference between `void` and `never` return types?

> [!check] RU Answer
> * `void`: Функция штатно завершает работу и **возвращает управление** вызывающему коду, но не возвращает значения. Инструкция `return;` без аргументов разрешена, но `return null;` приведёт к фатальной ошибке.
> * `never`: Функция **вообще никогда не возвращает управление** вызывающему коду. Она обязана либо завершить процесс (`exit()`, `die()`), либо выбросить исключение (`throw`). Это тип «дна» (Bottom Type). Помогает статическим анализаторам понимать, что весь код после вызова такой функции является недостижимым.

> [!quote] EN Answer
> *"`void` indicates that a function completes execution normally and returns control to the caller without producing a value; writing `return;` is allowed, but `return null;` fails. In contrast, `never` is a bottom type indicating that the function never returns control to the caller at all: it must terminate the process via `exit()` or throw an exception. It guarantees to static analyzers that subsequent code is unreachable."*

---

### Q12: Enums под капотом Zend Engine / Enums Under the Hood

> [!question] RU Question
> Как Enums устроены внутри ядра PHP и чем они отличаются от обычных классов?

> [!question] EN Question
> How are Enums implemented inside the Zend Engine, and how do they differ from standard classes?

> [!check] RU Answer
> Enums в PHP 8.1+ — это **неизменяемые объекты-синглтоны (Value Objects)**.
> * **Pure Enum:** Не имеют скалярных значений (`case Draft;`).
> * **Backed Enum:** Привязаны к скаляру `string` или `int` (`case Draft = 'draft';`), имеют свойства `$case->value` и `$case->name`, а также статические методы `::from()` и `::tryFrom()`.
> * **Ограничения ядра:**
>   1. Нельзя создать через `new` (`new Status()` — Fatal Error).
>   2. Нельзя клонировать (`clone $status` — Error).
>   3. Нельзя наследовать (`extends` запрещён).
>   4. Запрещены динамические и нестатические свойства.
>   5. Сравнение `$a === Status::Draft` проверяет identity по handle объекта за $O(1)$.

> [!quote] EN Answer
> *"Enums in PHP 8.1+ are immutable singleton objects. Pure Enums represent distinct state cases, while Backed Enums map each case to a strict `string` or `int` scalar with `from()` and `tryFrom()` parsing methods. Under the hood, Enums cannot be instantiated via `new`, cloned, inherited, or augmented with dynamic properties. Comparisons using strict equality (`===`) are instantaneous because cases are singleton object handles."*

---

### Q13: Property Hooks и Asymmetric Visibility (PHP 8.4) / Property Hooks & Asymmetric Visibility

> [!question] RU Question
> Как работают Property Hooks и асимметричная видимость свойств в PHP 8.4?

> [!question] EN Question
> How do Property Hooks and Asymmetric Visibility work in PHP 8.4?

> [!check] RU Answer
> * **Property Hooks:** Позволяют объявлять логику чтения (`get`) и записи (`set`) прямо в свойстве класса, избавляясь от boilerplate геттеров и сеттеров:
>   * *Виртуальные свойства:* Не выделяют память в объекте (`public string $fullName { get => "$this->first $this->last"; }`).
>   * *Backed свойства:* Используют внутреннее хранилище через `$value` (`set => strtolower($value);`).
>   * Хуки можно декларировать в интерфейсах!
> * **Асимметричная видимость:** Разделяет область видимости на чтение и запись.
>   `public private(set) string $status = 'pending';` — читать свойство `$order->status` может любой внешний код, но изменить `$this->status = 'paid'` разрешено только методам самого класса.

> [!quote] EN Answer
> *"PHP 8.4 introduces Property Hooks, which embed read (`get`) and write (`set`) logic directly into property definitions, eliminating boilerplate getter and setter methods. Properties can be virtual (computed on the fly without memory allocation) or backed by underlying storage. Combined with Asymmetric Visibility, such as `public private(set) string $status`, properties can be publicly readable across the application while restricting modifications exclusively to internal class methods."*

---

### Q14: Late Static Binding: `self::` против `static::` / Late Static Binding: `self::` vs `static::`

> [!question] RU Question
> В чём разница между `self::`, `parent::` и `static::` (Late Static Binding)?

> [!question] EN Question
> What is the difference between `self::`, `parent::`, and `static::` (Late Static Binding)?

> [!check] RU Answer
> * `self::`: Разрешается на этапе **компиляции** и указывает строго на тот класс, в котором физически написана строчка кода.
> * `parent::`: Разрешается на этапе компиляции и указывает на родительский класс текущего файла.
> * `static::` (LSB): Разрешается на этапе **выполнения (рантайм)** и ссылается на класс, через который метод был фактически вызван.
> * *Применение:* Фабричные методы в базовых абстрактных классах (`return new static();`) для создания экземпляров правильных дочерних классов.

> [!quote] EN Answer
> *"`self::` is resolved at compile time to the exact class where the code is written. In contrast, `static::` invokes Late Static Binding, resolving at runtime to the class that initiated the actual call. This makes `static::` essential when writing factory methods in abstract base classes, such as `return new static();`, ensuring the correct child instance is instantiated."*

---

## 4. Architecture, SOLID & Design Patterns

### Q15: Принцип единственной ответственности (SRP) / Single Responsibility Principle

> [!question] RU Question
> Как правильно понимать Single Responsibility Principle (SRP) и в чём главная ошибка его трактовки?

> [!question] EN Question
> What is the correct definition of the Single Responsibility Principle (SRP), and what is the most common misconception about it?

> [!check] RU Answer
> * **Заблуждение:** «Класс должен делать ровно одно действие или содержать один метод».
> * **Правильное определение (Роберт Мартин):** «У класса должна быть только одна причина для изменения, то есть он должен обслуживать ровно одного актора (бизнес-роль)».
> * **Пример:** Класс `OrderProcessor`, который валидирует данные, проводит cURL в банк, делает SQL-запрос `INSERT INTO orders` и шлёт email клиенту, нарушает SRP. Изменение в вёрстке письма или переход на другой платежный шлюз заставляют менять один и тот же монолитный класс.
> * **Решение:** Разделение на `OrderValidator`, `PaymentGatewayInterface`, `OrderRepositoryInterface`, `NotifierInterface`.

> [!quote] EN Answer
> *"The common misconception is that a class should only perform one method or action. The true definition of SRP is that a class should have only one reason to change, meaning it must serve a single business actor. An `OrderProcessor` that validates payloads, communicates via cURL with a bank, writes SQL to the database, and sends confirmation emails violates SRP. It should be refactored into dedicated services for validation, payment processing, repository persistence, and notification delivery."*

---

### Q16: Open/Closed Principle и устранение switch/match / Open/Closed Principle & Refactoring Switches

> [!question] RU Question
> Как принцип открытости/закрытости (OCP) применяется на практике для устранения ветвлений `switch`/`match`?

> [!question] EN Question
> How is the Open/Closed Principle (OCP) applied in practice to eliminate expanding `switch` or `match` statements?

> [!check] RU Answer
> * **Суть:** Программные сущности открыты для расширения (добавление нового функционала), но закрыты для модификации (не нужно переписывать протестированный рабочий код).
> * **Code Smell:** Растущие конструкции `match ($provider)` или цепочки `if-elseif-else`. При появлении нового провайдера (например, службы доставки СДЭК вместо DHL/UPS) разработчик вынужден править исходный класс, рискуя внести регрессионный баг.
> * **Решение:** Паттерн «Стратегия» (Strategy). Создаётся общий интерфейс `DeliveryStrategyInterface` с методом `calculate(Order $order): int`. Новый провайдер — это просто новый отдельный класс. Сервис-контекст принимает коллекцию стратегий через DI и не меняется ни на строчку.

> [!quote] EN Answer
> *"The Open/Closed Principle mandates that software entities should be open for extension but closed for modification. Growing `match` or `switch` blocks that inspect business types are an architectural code smell. We resolve this using polymorphism and the Strategy Pattern: define a shared interface (like `DeliveryStrategyInterface`), implement concrete strategy classes per provider, and inject them into the consumer. Adding a new provider requires adding a new class without modifying existing, tested code."*

---

### Q17: Liskov Substitution Principle (LSP) и вариантность типов / Liskov Substitution Principle & Type Variance

> [!question] RU Question
> В чём суть принципа подстановки Лисков (LSP) и как PHP проверяет вариантность типов при наследовании?

> [!question] EN Question
> What is the core idea of the Liskov Substitution Principle (LSP), and how does PHP enforce type variance during inheritance?

> [!check] RU Answer
> Дочерний класс должен безболезненно подменяться вместо базового класса, не ломая поведение вызывающего кода.
> **Правила проверки вариантности типов в PHP (на уровне компилятора):**
> 1. **Ковариантность возвращаемого типа (Narrowing):** Метод наследника имеет право сузить тип возврата (родитель возвращает `Animal`, наследник возвращает `Dog`).
> 2. **Контравариантность типов аргументов (Widening):** Метод наследника имеет право расширить принимаемый тип (родитель принимает `Dog`, наследник принимает `Animal`).
> 
> **Семантические нарушения LSP (которые компилятор не поймает):**
> * Наследник бросает `BadMethodCallException` или `NotImplementedException` на методы родителя.
> * Усиление предусловий (родитель принимает любое число, наследник — только положительные).

> [!quote] EN Answer
> *"Liskov Substitution Principle requires that subclasses must be usable in place of their parent types without altering program correctness. The PHP engine enforces this via type variance rules: return types are covariant (a child method can narrow the return type), while parameter types are contravariant (a child method can widen the accepted parameter type). Semantically, LSP also forbids child classes from throwing unexpected exceptions like `NotImplementedException` or strengthening preconditions."*

---

### Q18: Dependency Inversion (DIP) против Dependency Injection (DI) / DIP vs DI

> [!question] RU Question
> В чём разница между Dependency Injection (DI) и Dependency Inversion Principle (DIP)?

> [!question] EN Question
> What is the difference between Dependency Injection (DI) and the Dependency Inversion Principle (DIP)?

> [!check] RU Answer
> * **Dependency Injection (DI):** Это просто прикладной шаблон проектирования (технический паттерн). Зависимости не создаются внутри класса через `new MyDependency()`, а передаются снаружи (чаще всего через аргументы конструктора).
> * **Dependency Inversion Principle (DIP):** Это глобальный архитектурный принцип. Модули верхнего уровня (Use Cases / Бизнес-логика) не должны зависеть от модулей нижнего уровня (БД, cURL, файловая система). Оба уровня обязаны зависеть от абстракций (интерфейсов).
> * **Подвох:** Можно использовать DI (передавать зависимость в конструктор), но грубо нарушать DIP, если передаваемый класс — это конкретный `MySQLDatabase` вместо интерфейса `DatabaseInterface`.

> [!quote] EN Answer
> *"Dependency Injection is a mechanical design pattern where dependencies are passed from the outside—typically via constructor injection—rather than instantiated internally using `new`. Dependency Inversion Principle is an architectural rule stating that high-level business policies must not depend on low-level infrastructure details; both must depend on abstractions. You can use DI while still violating DIP if you inject concrete classes instead of interfaces."*

---

## 5. Databases, Transactions, Locks & Indexes

### Q19: Уровни изоляции транзакций и аномалии данных / Transaction Isolation Levels & Anomalies

> [!question] RU Question
> Назови 4 уровня изоляции транзакций и какие аномалии чтения они предотвращают?

> [!question] EN Question
> Name the 4 standard transaction isolation levels and explain the concurrency anomalies they prevent.

> [!check] RU Answer
> 1. **READ UNCOMMITTED:** Допускает **грязное чтение (Dirty Read)** — чтение незафиксированных данных другой транзакции, которая может затем сделать rollback.
> 2. **READ COMMITTED (дефолт в PostgreSQL):** Устраняет Dirty Read. Допускает **неповторяемое чтение (Non-Repeatable Read)** — повторный SELECT строки возвращает другие данные, так как параллельная транзакция успела закоммитить UPDATE.
> 3. **REPEATABLE READ (дефолт в MySQL InnoDB):** Устраняет Dirty Read и Non-Repeatable Read. В стандарте SQL допускает **фантомное чтение (Phantom Read)** — появление новых строк в выборке при параллельном INSERT. *(В MySQL InnoDB фантомы также устранены благодаря MVCC и Next-Key Locks).*
> 4. **SERIALIZABLE:** Полная сериализация транзакций, аномалии исключены, наименьшая скорость.

> [!quote] EN Answer
> *"The four standard isolation levels are: READ UNCOMMITTED, which permits Dirty Reads; READ COMMITTED (PostgreSQL default), which prevents Dirty Reads but allows Non-Repeatable Reads; REPEATABLE READ (MySQL InnoDB default), which uses MVCC snapshots to prevent Dirty and Non-Repeatable Reads, while InnoDB also eliminates Phantom Reads using Next-Key Locks; and SERIALIZABLE, which enforces strict sequential execution at the cost of high lock contention."*

---

### Q20: Пессимистичные и оптимистичные блокировки / Pessimistic vs Optimistic Locking

> [!question] RU Question
> Как защититься от состояния гонки (Race Condition) при списании баланса: пессимистичная (`FOR UPDATE`) против оптимистичной блокировки?

> [!question] EN Question
> How do you prevent Race Conditions during balance deductions: Pessimistic (`FOR UPDATE`) vs Optimistic Locking?

> [!check] RU Answer
> * **Пессимистичная блокировка (`SELECT ... FOR UPDATE`):**
>   Выполняется внутри транзакции. Накладывает эксклюзивную блокировку (X-Lock) на строки в базе данных. Любые параллельные транзакции встают в очередь и ждут commit или rollback первой. Применяется при высокой вероятности одновременной записи (биллинг, бронирование билетов).
> * **Оптимистичная блокировка (Optimistic Locking):**
>   Строки не блокируются в базе. Таблица содержит колонку `version` (int). Обновление происходит с проверкой версии: `UPDATE accounts SET balance = balance - 100, version = version + 1 WHERE id = :id AND version = :current_version`. Если `rowCount() === 0`, значит, запись успели изменить параллельно — приложение ловит коллизию и повторяет попытку. Применяется при редких конфликтах.

> [!quote] EN Answer
> *"To prevent race conditions during balance deductions under high contention, we use Pessimistic Locking: within an active transaction, we execute `SELECT ... FOR UPDATE`. This acquires an exclusive row lock, making concurrent transactions wait until the lock holder commits. Under low-contention scenarios, we use Optimistic Locking by introducing a `version` column and updating with `WHERE id = :id AND version = :expected_version`. If `rowCount()` returns 0, a conflict occurred and the application can safely retry."*

---

### Q21: Предотвращение дедлоков (Deadlock Prevention) / Deadlock Prevention

> [!question] RU Question
> Почему возникают взаимные блокировки (Deadlocks) и как архитектор обязан защитить код от дедлоков?

> [!question] EN Question
> Why do database deadlocks occur, and what architectural strategies must be used to prevent them?

> [!check] RU Answer
> Дедлок возникает, когда Транзакция 1 заблокировала Ресурс A и ждёт Ресурс B, а Транзакция 2 заблокировала Ресурс B и ждёт Ресурс A. СУБД убивает одну из транзакций с ошибкой SQLSTATE `40001` (Deadlock found).
> 
> **Стратегии защиты:**
> 1. **Детерминированный порядок блокировок:** Если операция затрагивает несколько строк (например, перевод со счета A на счет B), массив ID счетов **всегда сортируется по возрастанию (`sort($ids)`) перед запросом с `FOR UPDATE`**. Все транзакции будут захватывать строки в одинаковом порядке.
> 2. **Короткие транзакции:** Не делать внутри открытой транзакции внешних HTTP/cURL вызовов.
> 3. **Retry-механизм:** Ловить исключение дедлока в PHP и повторять транзакцию заново (с экспоненциальной задержкой).

> [!quote] EN Answer
> *"Deadlocks occur when two concurrent transactions hold locks on separate records and cyclically wait to acquire locks held by each other. We mitigate this using three rules: first, enforce a deterministic lock acquisition order by always sorting entity IDs (e.g., numerically ascending) before acquiring `FOR UPDATE` locks. Second, keep transactions as short as possible, strictly avoiding external HTTP calls while holding a connection. Third, implement an exponential backoff retry loop targeting SQLSTATE `40001`."*

---

### Q22: B-Tree Индексы и правило левого префикса / B-Tree Indexes & Leftmost Prefix Rule

> [!question] RU Question
> Как устроен B-Tree индекс, что такое правило левого префикса для составного индекса `(A, B, C)` и как диапазонные условия влияют на поиск?

> [!question] EN Question
> How does a B-Tree index work, what is the Leftmost Prefix Rule for composite index `(A, B, C)`, and how do range conditions affect searching?

> [!check] RU Answer
> * **B-Tree:** Сбалансированное дерево на диске со сложностью поиска $O(\log N)$. Листья соединены двусвязным списком для эффективных диапазонов.
> * **Правило левого префикса:** Составной индекс `(A, B, C)` работает только слева направо без пропусков:
>   * `WHERE A = 1 AND B = 2` — использует индекс.
>   * `WHERE B = 2 AND C = 3` — **не использует** индекс (нет колонки A).
>   * `WHERE A = 1 AND C = 3` — использует индекс только по части `A`.
> * **Диапазонные операторы (`>`, `<`, `BETWEEN`, `LIKE 'abc%'`):**
>   Если на колонке `B` стоит диапазон (`WHERE A = 1 AND B > 10 AND C = 5`), то для колонки `C` индекс **больше не работает**! В составном индексе диапазонная колонка обязана стоять на последнем месте.

> [!quote] EN Answer
> *"A B-Tree is a balanced search tree offering $\mathcal{O}(\log N)$ lookups with leaf nodes linked for rapid range scans. Under the Leftmost Prefix Rule, a composite index `(A, B, C)` can only satisfy queries filtering from left to right without gaps: queries filtering on `(B, C)` cannot use the index because prefix `A` is absent. Additionally, applying a range operator (`>`, `<`, `BETWEEN`) on a column breaks index usage for all subsequent columns; therefore, equality columns must always precede range columns in composite indexes."*

---

### Q23: Покрывающий индекс (Covering Index) и Keyset-пагинация / Covering Index & Keyset Pagination

> [!question] RU Question
> Что такое покрывающий индекс (Covering Index) и почему Keyset-пагинация в разы быстрее `LIMIT ... OFFSET`?

> [!question] EN Question
> What is a Covering Index, and why is Keyset pagination significantly faster than `LIMIT ... OFFSET`?

> [!check] RU Answer
> * **Покрывающий индекс:** Запрос запрашивает в `SELECT`, фильтрует в `WHERE` и сортирует в `ORDER BY` только те колонки, которые уже содержатся в самом вторичном B-Tree индексе. СУБД забирает данные напрямую из дерева индекса, не открывая таблицу на диске (в `EXPLAIN` появляется отметка `Using index`).
> * **Keyset (курсорная) пагинация:**
>   Запрос `OFFSET 1000000 LIMIT 50` заставляет СУБД физически считать и отбросить миллион строк. Keyset-пагинация запоминает последний ID: `WHERE id > :last_id ORDER BY id ASC LIMIT 50`. СУБД по индексу переходит к узлу за $O(\log N)$ и считывает следующие 50 строк моментально.

> [!quote] EN Answer
> *"A Covering Index contains all columns requested by `SELECT`, `WHERE`, and `ORDER BY` within the index structure itself. The database reads data entirely from the index leaf nodes without performing a secondary table lookup, indicated by `Using index` in `EXPLAIN`. Keyset (cursor) pagination replaces `OFFSET 1000000`—which forces the engine to scan and discard one million rows—with `WHERE id > :last_id ORDER BY id ASC LIMIT 50`, jumping directly to the cursor position via B-Tree in $\mathcal{O}(\log N)$ time."*

---

## 6. Enterprise Security & OWASP Top 10

### Q24: Настоящие Prepared Statements против эмуляции в PDO / Real vs Emulated Prepared Statements

> [!question] RU Question
> В чём разница между реальными Prepared Statements и `ATTR_EMULATE_PREPARES => true` в PDO?

> [!question] EN Question
> What is the difference between genuine Prepared Statements and `ATTR_EMULATE_PREPARES => true` in PDO?

> [!check] RU Answer
> * **Честные Prepared Statements (`false`):** Запрос отправляется в СУБД в два сетевых раунда: сначала шаблон запроса без данных компилируется на сервере БД в дерево разбора, а затем вторым пакетом передаются бинарные параметры. Данные физически не могут изменить синтаксис команды.
> * **Эмуляция (`true`):** PDO в PHP сам подставляет параметры в строку запроса через строковое экранирование перед отправкой в MySQL.
> * **Опасность:** При редких многобайтовых кодировках соединения (GBK/Big5) эмуляция подвержена обходу экранирования.
> * *Senior-правило:* Всегда отключать эмуляцию (`PDO::ATTR_EMULATE_PREPARES => false`) и передавать `charset=utf8mb4` прямо в строке DSN.

> [!quote] EN Answer
> *"With genuine prepared statements (`ATTR_EMULATE_PREPARES => false`), the query template is sent to the database server and compiled first, after which parameter values are sent separately as binary packets. This makes SQL injection physically impossible. With emulation enabled (`true`), PHP simply quotes and concatenates parameters into the SQL string locally before sending it, which can be vulnerable under non-standard multibyte character sets. In production, always set emulation to false and define `charset=utf8mb4` in the DSN."*

---

### Q25: Контекстное экранирование от XSS / Context-Aware Escaping against XSS

> [!question] RU Question
> Нужно ли экранировать данные перед сохранением в базу данных для защиты от XSS?

> [!question] EN Question
> Should user data be escaped before saving to the database to prevent XSS?

> [!check] RU Answer
> **КАТЕГОРИЧЕСКИ НЕТ.**
> Данные обязаны храниться в базе данных в чистом, сыром виде.
> **Почему нельзя экранировать перед сохранением:**
> 1. Это ломает поиск по базе данных, расчёт длины строк и сортировку.
> 2. При выводе в мобильное приложение, генерации PDF или отправке в REST API клиенты получат испорченные HTML-сущности (`&quot;`, `&amp;`).
> 3. При повторном редактировании сущности в админке возникнет двойное экранирование.
> 
> **Правило:** Экранирование выполняется **строго на этапе вывода (Output Encoding)** в зависимости от контекста: для HTML-контекста используется `htmlspecialchars($data, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8')`.

> [!quote] EN Answer
> *"Never escape user data before persisting it to the database. Data must be stored in its raw, authentic format; otherwise, string lengths, searches, and non-HTML consumers like REST APIs or mobile apps become corrupted. Escaping is strictly a presentation-layer responsibility performed at the moment of output (Output Encoding), using context-appropriate functions like `htmlspecialchars($data, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8')` for HTML body contexts."*

---

### Q26: Защита от CSRF и cookie SameSite / CSRF Prevention & SameSite Cookies

> [!question] RU Question
> Как работает атака CSRF и как атрибут SameSite защищает сессии?

> [!question] EN Question
> How does a CSRF attack work, and how does the `SameSite` cookie attribute mitigate it?

> [!check] RU Answer
> * **Механизм CSRF:** Жертва заходит на вредоносный сайт `evil.com`, где размещена скрытая HTML-форма, отправляющая POST-запрос на `bank.com/transfer`. Браузер автоматически прикрепляет куки сессии `bank.com`, и сервер банка выполняет перевод от имени жертвы.
> * **Защита токенами (Synchronizer Token Pattern):** Сервер генерирует случайный токен в сессии и требует его наличия в скрытом поле формы или HTTP-заголовке. Сравнение выполняется строго через `hash_equals()`.
> * **Cookie-атрибут `SameSite`:**
>   * `SameSite=Strict`: Cookie никогда не отправляется в кросс-доменных запросах (даже при клике по ссылке из Google).
>   * `SameSite=Lax` (современный дефолт): Cookie отправляется только при безопасных GET-переходах верхнего уровня, но **блокируется при любых межсайтовых POST/PUT/DELETE запросах**, полностью нейтрализуя классический CSRF.

> [!quote] EN Answer
> *"CSRF occurs when a malicious site exploits a user's active session by tricking their browser into dispatching unauthorized requests to the target domain, which automatically includes stored session cookies. We mitigate this using CSRF tokens validated via constant-time comparison (`hash_equals()`). Additionally, setting the cookie attribute `SameSite=Lax` or `Strict` instructs the browser to omit session cookies on cross-site state-changing POST requests, stopping CSRF at the protocol level."*

---

### Q27: Тайминг-атаки и функция `hash_equals()` / Timing Attacks & `hash_equals()`

> [!question] RU Question
> Что такое атака по времени (Timing Attack) и почему секреты нельзя сравнивать через `$a === $b`?

> [!question] EN Question
> What is a Timing Attack, and why must cryptographic secrets never be compared using `$a === $b`?

> [!check] RU Answer
> Стандартный оператор сравнения `$a === $b` сравнивает строки побайтово слева направо и **немедленно прерывает выполнение на первом несовпавшем байте**.
> Если первый байт неверен, проверка занимает, например, 2 микросекунды. Если первый байт угадан, а второй неверен — 4 микросекунды. Замеряя миллионы сетевых запросов статистически, атакующий может посимвольно восстановить секретный токен, подпись HMAC или API-ключ.
> **Решение:** Функция `hash_equals($known, $user)` работает за **константное время**: она всегда проверяет строку до конца независимо от позиции несовпадения.

> [!quote] EN Answer
> *"Standard string comparison (`===`) terminates on the first mismatched byte. An attacker measuring millisecond-level network latency differences across thousands of requests can guess secret tokens character-by-character based on execution duration. `hash_equals()` mitigates timing attacks by executing in constant time, always scanning the entire string length regardless of where differences occur."*

---

## 7. HTTP Protocols, Streaming & PSR Standards

### Q28: Чтение тела JSON-запроса через `php://input` / Reading JSON Payloads via `php://input`

> [!question] RU Question
> Почему при отправке JSON в теле POST-запроса массив `$_POST` пуст и как правильно вычитывать тело?

> [!question] EN Question
> Why is the `$_POST` superglobal empty when receiving a JSON POST request, and how do you properly extract the body?

> [!check] RU Answer
> Ядро PHP автоматически парсит тело входящего запроса в суперглобальный массив `$_POST` **только для двух MIME-типов**:
> 1. `application/x-www-form-urlencoded`
> 2. `multipart/form-data`
> 
> При отправке современных REST-запросов с заголовком `Content-Type: application/json` PHP не производит автоматический парсинг, и `$_POST` остаётся пустым.
> **Как читать:** Вычитывать сырой байтовый поток через специальную обёртку:
> `$raw = file_get_contents('php://input');`
> `$data = json_decode($raw, true, 512, JSON_THROW_ON_ERROR);`

> [!quote] EN Answer
> *"PHP populates `$_POST` exclusively for `application/x-www-form-urlencoded` and `multipart/form-data` content types. When clients submit modern REST payloads using `application/json`, the engine leaves `$_POST` empty. The payload must be read directly as raw bytes from the read-only stream wrapper `php://input` and parsed via `json_decode($raw, true, 512, JSON_THROW_ON_ERROR)`."*

---

### Q29: Природа ошибки «Headers already sent» / The "Headers already sent" Error

> [!question] RU Question
> Из-за чего возникает ошибка `Headers already sent` и как буферизация вывода (`ob_*`) решает эту проблему?

> [!question] EN Question
> What triggers the `Headers already sent` error, and how does Output Buffering (`ob_*`) resolve it?

> [!check] RU Answer
> По протоколу HTTP все заголовки ответа (headers, status codes, cookies) обязаны быть переданы веб-серверу строго **до первого байта тела ответа (body)**.
> Если скрипт вывел хотя бы один символ (случайный пробел до тега `<?php`, невидимый символ BOM в начале файла, или результат `echo`), PHP сбрасывает заголовки в сокет. Любой последующий вызов `header()` или `setcookie()` выбросит предупреждение `Cannot modify header information - headers already sent`.
> **Решение:** 
> 1. Не закрывать чисто PHP-файлы тегом `?>` (требование PSR-12).
> 2. Использовать Output Buffering (`ob_start()`), который перехватывает весь вывод в оперативную память, позволяя безопасно модифицировать заголовки до сброса буфера через `ob_end_flush()`.

> [!quote] EN Answer
> *"The HTTP specification requires response headers to precede body content. If a script emits even a single byte—such as whitespace before `<?php` or an invisible UTF-8 BOM—PHP flushes headers to the web server socket, causing subsequent `header()` calls to fail with `Headers already sent`. Output Buffering (`ob_start()`) resolves this by capturing all output in an in-memory buffer, allowing developers to set response headers freely until the buffer is flushed."*

---

### Q30: Архитектура PSR-7 и PSR-15 Middleware / PSR-7 & PSR-15 Middleware Architecture

> [!question] RU Question
> В чём суть стандарта PSR-7 (HTTP-сообщения) и как работает конвейер PSR-15 Middleware («модель луковицы»)?

> [!question] EN Question
> What is the architectural philosophy of PSR-7 HTTP messages, and how does the PSR-15 Middleware pipeline ("Onion Model") operate?

> [!check] RU Answer
> * **PSR-7:** Представляет HTTP-запросы и ответы как **иммутабельные объекты-значения (Value Objects)**. Методы изменения (`withHeader()`, `withStatus()`) не модифицируют текущий объект, а возвращают новый клон. Это гарантирует отсутствие скрытых сайд-эффектов при передаче запроса через десятки несвязанных сервисов.
> * **PSR-15 Middleware:** Реализует паттерн «Луковица» (Onion Model). Каждый слой Middleware реализует интерфейс с методом `process(ServerRequestInterface $request, RequestHandlerInterface $handler): ResponseInterface`.
>   1. Middleware перехватывает запрос на входе (валидация, аутентификация).
>   2. Передаёт управление вглубь цепочки вызовом `$response = $handler->handle($request);`.
>   3. На обратном пути обогащает полученный ответ (добавляет заголовки кэширования, CORS, логирует тайминги).

> [!quote] EN Answer
> *"PSR-7 models HTTP requests and responses as immutable Value Objects where mutating methods (`withHeader()`) return clones, preventing unexpected side effects across services. PSR-15 formalizes HTTP middleware around the Onion Model: each middleware implements `process()`, performing upstream pre-processing, invoking `$handler->handle($request)` to delegate deeper into the pipeline, and applying downstream post-processing (such as adding CORS headers or logging execution times) on the returning response."*