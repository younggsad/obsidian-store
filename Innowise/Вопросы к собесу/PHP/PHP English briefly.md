---
custom-width: 80
---
## 1. PHP Fundamentals

### What is PHP?

PHP is a server-side scripting language mainly used for web development. It can generate dynamic pages, communicate with databases, process requests, and build APIs.

### Is PHP interpreted or compiled?

PHP is compiled into bytecode (opcodes) and executed by the Zend Engine. OPcache can store compiled bytecode to avoid recompilation on every request.

### What are the main PHP data types?

- **Scalar:** int, float, string, bool.
    
- **Compound:** array, object, callable, iterable.
    
- **Special:** null, resource.
    

### What is the difference between `==` and `===`?

- `==` compares values after type conversion.
    
- `===` compares both value and type.
    

```
5 == "5";   // true
5 === "5";  // false
```

I generally prefer strict comparison because it makes type conversion explicit.

### What is the difference between `isset()` and `empty()`?

- `isset()` checks whether a variable exists and is not null.
    
- `empty()` checks whether a variable is empty or evaluates to false.
    

```
$value = 0;

isset($value); // true
empty($value); // true
```

### What is the difference between `null`, `false`, and `0`?

- `null` means no value.
    
- `false` is a boolean false value.
    
- `0` is an integer.
    

They are different values, even though all of them can evaluate to false in a boolean context.

### What is type juggling?

Type juggling is automatic type conversion performed by PHP.

```
$result = "10" + 5; // 15
```

### What is strict typing?

Strict typing enables stricter scalar type checking for function calls from the declaring file.

```
<?php

declare(strict_types=1);

function calculate(int $number): int
{
    return $number * 2;
}
```

It helps prevent unexpected type conversions.

### What is the difference between `include` and `require`?

- `include` produces a warning if the file cannot be loaded.
    
- `require` produces an error and stops execution.
    

`include_once` and `require_once` prevent the same file from being included multiple times.

### What is the difference between `echo` and `print`?

- `echo` can output multiple arguments and has no return value.
    
- `print` outputs one argument and returns `1`.
    

In practice, `echo` is more commonly used.

---

## 2. Variables, Constants and Operators

### What is the difference between `const` and `define()`?

Both create constants.

- `const` is language-level syntax and can be used for class constants.
    
- `define()` creates constants at runtime.
    

### What is the difference between `??` and `?:`?

- `??` checks whether a variable exists and is not null.
    
- `?:` checks whether a value evaluates to true.
    

```
$name = $input ?? 'Guest';
$name = $input ?: 'Guest';
```

### What is the nullsafe operator?

The nullsafe operator `?->` allows method or property access without checking for null manually.

```
$name = $user?->getProfile()?->getName();
```

If any step returns null, the whole expression returns null.

### What is the difference between `match` and `switch`?

- `switch` uses loose comparison.
    
- `match` uses strict comparison.
    
- `match` returns a value.
    
- `match` does not fall through between branches.
    
- `match` must be exhaustive or have a `default` branch.
    

```
$result = match ($status) {
    'active' => 'Enabled',
    'inactive' => 'Disabled',
    default => 'Unknown',
};
```

---

## 3. Arrays and Iteration

### What types of arrays exist in PHP?

PHP has one flexible array type that can work as:

- Indexed array.
    
- Associative array.
    
- Multidimensional array.
    

### What is the difference between `array_map()`, `array_filter()`, and `array_reduce()`?

- `array_map()` transforms every element.
    
- `array_filter()` selects elements based on a condition.
    
- `array_reduce()` combines elements into one result.
    

```
$numbers = [1, 2, 3, 4];

$mapped = array_map(
    fn($number) => $number * 2,
    $numbers
);

$filtered = array_filter(
    $numbers,
    fn($number) => $number > 2
);

$total = array_reduce(
    $numbers,
    fn($carry, $number) => $carry + $number,
    0
);
```

### What is the difference between `foreach` and `for`?

- `for` is useful when working with numeric indexes or a known number of iterations.
    
- `foreach` is designed for iterating over arrays and iterables.
    

### What happens when you use `&` in foreach?

It creates a reference to the original array element. Changes affect the original array.

```
foreach ($numbers as &$number) {
    $number *= 2;
}

unset($number);
```

`unset()` is important because the reference remains attached to the last element after the loop.

### What is destructuring?

Destructuring allows you to extract array values into separate variables.

```
[$first, $second] = [10, 20];

['name' => $name, 'age' => $age] = $user;
```

### What is the spread operator?

The spread operator `...` expands an array into individual values.

```
$first = [1, 2];
$second = [3, 4];

$result = [...$first, ...$second];
```

### What is the difference between `array_merge()` and the `+` operator?

- `array_merge()` combines arrays. Numeric keys are reindexed, and later string keys overwrite earlier ones.
    
- `+` is the array union operator. Existing keys from the left array take priority.
    

### What is a Generator?

A Generator allows you to iterate over values without storing the entire result in memory.

```
function getNumbers(): Generator
{
    for ($i = 1; $i <= 1000; $i++) {
        yield $i;
    }
}
```

Generators are useful when processing large datasets.

---

## 4. Functions and Closures

### What is a function?

A function is a reusable block of code that performs a specific operation.

### What is the difference between passing by value and passing by reference?

- By value: the function receives a value.
    
- By reference: the function can modify the original variable.
    

```
function update(int &$number): void
{
    $number = 100;
}
```

### What is a Closure?

A Closure is an anonymous function that can be assigned to a variable, passed as an argument, or returned from another function.

```
$multiply = function (int $number): int {
    return $number * 2;
};
```

### What is the difference between Closure and arrow function?

- A Closure can capture variables using `use`.
    
- An arrow function automatically captures variables from the parent scope by value.
    
- Arrow functions contain one expression.
    

```
$factor = 2;

$multiply = fn(int $number): int => $number * $factor;
```

### What is a callback?

A callback is a function passed to another function as an argument.

```
function process(callable $callback): void
{
    $callback();
}

process(fn() => print('Hello'));
```

### What is recursion?

Recursion is when a function calls itself.

It must have a termination condition to prevent infinite recursion.

### What is a variadic function?

A variadic function accepts an unlimited number of arguments using `...`.

```
function sum(int ...$numbers): int
{
    return array_sum($numbers);
}
```

### What is a higher-order function?

A higher-order function accepts another function as an argument or returns a function.

Examples include `array_map()`, `array_filter()`, and `array_reduce()`.

---

## 5. Object-Oriented Programming (OOP)

### What is OOP?

Object-Oriented Programming is a programming paradigm based on objects that combine data and behavior.

The four main principles are:

1. Encapsulation.
    
2. Inheritance.
    
3. Polymorphism.
    
4. Abstraction.
    

### What is encapsulation?

Encapsulation means hiding internal implementation details and controlling access to an object's data.

PHP provides:

- `public`
    
- `protected`
    
- `private`
    

### What is inheritance?

Inheritance allows a class to reuse and extend functionality from another class using `extends`.

```
class Animal
{
    public function eat(): void {}
}

class Dog extends Animal
{
    public function bark(): void {}
}
```

### What is polymorphism?

Polymorphism allows different classes to implement the same interface or inherit the same contract while providing different behavior.

```
interface Payment
{
    public function pay(float $amount): void;
}

class CardPayment implements Payment
{
    public function pay(float $amount): void {}
}

class PayPalPayment implements Payment
{
    public function pay(float $amount): void {}
}
```

### What is abstraction?

Abstraction hides implementation details and exposes only essential functionality.

It can be implemented using interfaces and abstract classes.

### What is the difference between an interface and an abstract class?

**Interface:**

- Defines a contract.
    
- A class can implement multiple interfaces.
    
- Does not provide instance properties or constructor implementations as an abstract class does.
    
- Can contain constants and method declarations; modern PHP also supports interface method defaults through traits, not interfaces.
    

**Abstract class:**

- Can contain implemented and abstract methods.
    
- Can have properties and a constructor.
    
- A class can extend only one class.
    

### What is the difference between `self`, `static`, and `parent`?

- `self` refers to the class where the method is defined.
    
- `static` uses late static binding and refers to the called class.
    
- `parent` refers to the parent class.
    

### What is late static binding?

Late static binding allows PHP to resolve `static` based on the class that was called at runtime rather than the class where the method was originally defined.

### What is method overriding?

Method overriding happens when a child class provides its own implementation of a method inherited from a parent class.

### What is method overloading in PHP?

PHP does not support traditional method overloading by different method signatures like Java or C++.

It provides magic methods such as `__get()`, `__set()`, and `__call()` for dynamic behavior.

### What is a constructor?

A constructor is a special method `__construct()` called when an object is created. It initializes the object.

### What is a destructor?

`__destruct()` is called when an object is destroyed or during shutdown. It can be used for cleanup, but should not be relied on for critical business operations.

### What are static methods and properties?

Static members belong to the class rather than a particular object.

They can be accessed without creating an instance.

### What is a Trait?

A Trait allows code reuse across multiple classes without inheritance.

```
trait LoggerTrait
{
    public function log(string $message): void {}
}

class UserService
{
    use LoggerTrait;
}
```

### What is an Enum?

An Enum defines a fixed set of possible values.

```
enum Status: string
{
    case ACTIVE = 'active';
    case INACTIVE = 'inactive';
}
```

Enums improve type safety and make domain values explicit.

### What is a readonly property?

A readonly property can be initialized once and cannot be reassigned afterward.

```
class User
{
    public function __construct(
        public readonly int $id,
    ) {}
}
```

PHP 8.2 introduced readonly classes.

### What is a final class or method?

- `final class` cannot be extended.
    
- `final method` cannot be overridden by child classes.
    

### What is dependency injection?

Dependency Injection means providing an object with its dependencies from outside instead of creating them internally.

```
class UserService
{
    public function __construct(
        private UserRepository $repository,
    ) {}
}
```

It improves testability, flexibility, and separation of concerns.

---

## 6. Advanced OOP and Type System

### What is an anonymous class?

An anonymous class is a class without a declared name. It is useful for small, one-time implementations.

### What are magic methods?

Magic methods are special methods called automatically by PHP in specific situations.

Examples:

- `__construct()` — object creation.
    
- `__destruct()` — object destruction.
    
- `__get()` — inaccessible property read.
    
- `__set()` — inaccessible property write.
    
- `__call()` — inaccessible method call.
    
- `__toString()` — object converted to string.
    
- `__invoke()` — object called as a function.
    
- `__clone()` — object cloning.
    

### What is object cloning?

Cloning creates a shallow copy of an object.

`__clone()` can customize the cloning process.

### What is the difference between shallow copy and deep copy?

- Shallow copy copies the object, but nested object references may still point to the same objects.
    
- Deep copy duplicates the nested object structure as well.
    

### What is covariance and contravariance?

- Covariance allows a child implementation to return a more specific type.
    
- Contravariance allows a child implementation to accept a less specific parameter type.
    

These concepts make method overriding more flexible while preserving type compatibility.

### What are union and intersection types?

- Union types allow one of several types: `int|string`.
    
- Intersection types require a value to satisfy multiple types: `Countable&Iterator`.
    

PHP also supports DNF types, combining unions and intersections.

### What is a nullable type?

A nullable type allows a value or null.

```
function findUser(): ?User
{
    return null;
}
```

### What is a backed Enum?

A backed Enum has scalar values such as strings or integers.

```
enum Role: string
{
    case ADMIN = 'admin';
    case USER = 'user';
}
```

### What is an Iterator?

An Iterator allows an object to be traversed using `foreach`. It implements methods such as `current()`, `key()`, `next()`, `rewind()`, and `valid()`.

### What is the difference between Iterator and Generator?

An Iterator is an object implementing an iteration interface. A Generator is a simpler way to create an iterator using `yield`.

---

## 7. Error Handling and Exceptions

### What is the difference between an Exception and an Error?

Both implement `Throwable`.

- `Exception` represents conditions that application code may handle.
    
- `Error` represents engine-level or programming errors, such as type errors.
    

### What is Throwable?

`Throwable` is the base interface for both `Exception` and `Error`.

### What is the difference between `try`, `catch`, and `finally`?

- `try` contains code that may throw an exception.
    
- `catch` handles an exception.
    
- `finally` executes after the try/catch process, including when an exception is thrown.
    

```
try {
    processPayment();
} catch (PaymentException $exception) {
    logError($exception);
} finally {
    cleanup();
}
```

### What is a custom exception?

A custom exception is an application-specific exception class that extends `Exception` or another suitable exception class.

```
class UserNotFoundException extends RuntimeException {}
```

### When should you throw an exception?

When an operation cannot continue normally and the caller needs to handle the failure.

### Should exceptions be used for normal application flow?

No. Exceptions should represent exceptional situations, not ordinary business decisions.

### What is exception propagation?

Exception propagation means an exception moves up the call stack until a matching `catch` block handles it.

---

## 8. Memory Management and Performance

### How does PHP manage memory?

PHP automatically manages memory using reference counting and garbage collection.

### What is reference counting?

PHP tracks how many references point to a value. When the reference count reaches zero, the value can be released.

### What is garbage collection?

Garbage collection identifies and removes circular references that cannot be released by normal reference counting.

### What is OPcache?

OPcache stores compiled PHP bytecode in shared memory. It reduces repeated compilation and improves performance.

### What is JIT?

JIT (Just-In-Time compilation) is a feature that compiles certain bytecode paths into machine code at runtime.

It can help some CPU-intensive workloads, but it does not automatically make typical web applications faster.

### How can you optimize PHP performance?

- Use OPcache.
    
- Avoid unnecessary database queries.
    
- Add database indexes.
    
- Cache frequently used data.
    
- Avoid loading large datasets into memory.
    
- Use generators for streaming large data.
    
- Profile before optimizing.
    
- Reduce unnecessary object creation.
    

### What is the difference between time complexity and space complexity?

- Time complexity describes how execution time grows with input size.
    
- Space complexity describes how memory usage grows with input size.
    

### What is Big O notation?

Big O describes the growth rate of an algorithm.

Examples:

- O(1) — constant.
    
- O(log n) — logarithmic.
    
- O(n) — linear.
    
- O(n log n) — linearithmic.
    
- O(n²) — quadratic.
    

---

## 9. Composer and Autoloading

### What is Composer?

Composer is PHP's dependency manager. It installs packages, manages versions, and provides autoloading.

### What is `composer.json`?

It defines project metadata, dependencies, autoloading rules, and scripts.

### What is `composer.lock`?

It stores exact installed dependency versions to ensure consistent installations across environments.

### What is the difference between `composer install` and `composer update`?

- `composer install` installs versions from `composer.lock`.
    
- `composer update` resolves newer compatible versions and updates the lock file.
    

### What is PSR-4?

PSR-4 is an autoloading standard that maps namespaces to directory structures.

```
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

### What is autoloading?

Autoloading automatically loads class files when classes are needed, without manually including each file.

### What is semantic versioning?

Semantic Versioning uses three numbers:

`MAJOR.MINOR.PATCH`

- MAJOR — breaking changes.
    
- MINOR — backward-compatible features.
    
- PATCH — backward-compatible bug fixes.
    

### What is the difference between `require` and `require-dev` in Composer?

- `require` contains dependencies needed in production.
    
- `require-dev` contains development dependencies such as testing and code-quality tools.
    

---

## 10. Database and SQL

### What is PDO?

PDO (PHP Data Objects) provides a consistent interface for accessing different database systems.

### What are prepared statements?

Prepared statements separate SQL code from user input. They help prevent SQL injection.

```
$stmt = $pdo->prepare(
    'SELECT * FROM users WHERE id = :id'
);

$stmt->execute(['id' => $id]);

$user = $stmt->fetch();
```

### What is SQL injection?

SQL injection is an attack where malicious SQL is inserted into a query through untrusted input.

Prevention:

- Prepared statements.
    
- Parameter binding.
    
- Input validation.
    
- Least-privilege database accounts.
    

### What is a database transaction?

A transaction groups multiple database operations into one logical unit.

### What are ACID properties?

- Atomicity — all operations succeed or none do.
    
- Consistency — data remains valid.
    
- Isolation — concurrent transactions do not interfere incorrectly.
    
- Durability — committed changes persist.
    

### What is an index?

An index improves data retrieval speed but requires additional storage and can slow down writes.

### What is the N+1 query problem?

It occurs when one query retrieves a collection and then an additional query is executed for each item.

Solution: use JOINs, eager loading, or batch queries.

### What is normalization?

Normalization organizes database tables to reduce data duplication and improve consistency.

### What is the difference between INNER JOIN and LEFT JOIN?

- INNER JOIN returns matching records from both tables.
    
- LEFT JOIN returns all records from the left table and matching records from the right table.
    

### What is optimistic locking?

Optimistic locking assumes conflicts are uncommon and checks whether a record changed before updating it, often using a version column.

### What is a database deadlock?

A deadlock happens when transactions wait for resources locked by each other.

Solutions include consistent lock ordering, short transactions, proper indexes, and retry logic.

---

## 11. HTTP, REST and APIs

### What is HTTP?

HTTP is a protocol used for communication between clients and servers.

### What are common HTTP methods?

- GET — retrieve data.
    
- POST — create or process data.
    
- PUT — replace a resource.
    
- PATCH — partially update a resource.
    
- DELETE — delete a resource.
    

### What is the difference between PUT and PATCH?

- PUT replaces the resource representation.
    
- PATCH applies partial modifications.
    

### What are common HTTP status codes?

- 200 — OK.
    
- 201 — Created.
    
- 204 — No Content.
    
- 301 — Moved Permanently.
    
- 400 — Bad Request.
    
- 401 — Unauthorized.
    
- 403 — Forbidden.
    
- 404 — Not Found.
    
- 409 — Conflict.
    
- 422 — Unprocessable Content.
    
- 500 — Internal Server Error.
    
- 503 — Service Unavailable.
    

### What is REST?

REST is an architectural style for designing APIs around resources and standard HTTP operations.

### What is JSON?

JSON is a lightweight data-interchange format commonly used in APIs.

PHP provides `json_encode()` and `json_decode()`.

### What is the difference between authentication and authorization?

- Authentication verifies who the user is.
    
- Authorization determines what the user is allowed to do.
    

### What is JWT?

JWT (JSON Web Token) is a token format used to transmit claims between parties.

A JWT contains a header, payload, and signature.

Important: JWT payloads are normally encoded, not encrypted.

### What is idempotency?

An operation is idempotent if repeating it produces the same intended result as executing it once.

GET, PUT, and DELETE are defined as idempotent HTTP methods. POST is not idempotent by default.

### What is pagination?

Pagination splits large result sets into smaller responses.

Common approaches:

- Offset pagination.
    
- Cursor pagination.
    

Cursor pagination is generally more stable for large or frequently changing datasets.

---

## 12. Security

### How do you prevent SQL injection?

Use prepared statements and parameterized queries.

### How do you prevent XSS?

Escape output according to its context and avoid inserting untrusted content into executable HTML or JavaScript.

### What is CSRF?

Cross-Site Request Forgery tricks an authenticated user into performing an unwanted action.

Prevention:

- CSRF tokens.
    
- SameSite cookies.
    
- Appropriate origin checks.
    

### How should passwords be stored?

Use `password_hash()` and verify with `password_verify()`.

Never store plain-text passwords or use MD5/SHA1 for password storage.

### What is the difference between hashing and encryption?

- Hashing is one-way.
    
- Encryption is reversible using a key.
    

### What is session fixation?

Session fixation is an attack where an attacker makes a victim use a known session ID.

Prevention: regenerate the session ID after authentication.

### What is the principle of least privilege?

Users and services should receive only the permissions necessary to perform their tasks.

### How should secrets be managed?

Use environment variables or a dedicated secrets manager. Never commit passwords, API keys, or private credentials to Git.

---

## 13. SOLID Principles

### What is SOLID?

SOLID is a set of five principles for writing maintainable and flexible object-oriented code.

### S — Single Responsibility Principle

A class should have one responsibility and one main reason to change.

### O — Open/Closed Principle

Software entities should be open for extension but closed for modification.

Example: introduce a new payment implementation instead of modifying an existing payment service.

### L — Liskov Substitution Principle

A child class should be usable wherever its parent type is expected without breaking behavior.

### I — Interface Segregation Principle

Prefer small, specific interfaces instead of large interfaces that force classes to implement unnecessary methods.

### D — Dependency Inversion Principle

High-level modules should depend on abstractions rather than concrete implementations.

### What is the difference between Dependency Injection and Dependency Inversion?

- Dependency Injection is a technique for providing dependencies.
    
- Dependency Inversion is a design principle about depending on abstractions.
    

---

## 14. Design Patterns

### What is a design pattern?

A design pattern is a reusable solution to a common software design problem.

### What is Singleton?

Ensures that a class has only one instance and provides a global access point.

Use carefully because it introduces global state and makes testing harder.

### What is Factory?

Creates objects without exposing the exact creation logic to the caller.

### What is Strategy?

Defines interchangeable algorithms behind a common interface.

Example: different payment methods.

### What is Repository?

Encapsulates data access logic and separates persistence from business logic.

### What is Adapter?

Allows incompatible interfaces to work together.

### What is Observer?

Allows multiple objects to react when an event occurs.

### What is Decorator?

Adds behavior to an object without modifying its original class.

### What is Dependency Injection Container?

A container manages object creation and resolves dependencies automatically.

### What is the difference between Factory and Repository?

- Factory creates objects.
    
- Repository retrieves and persists data.
    

---

## 15. Testing

### What is unit testing?

Unit testing checks a small, isolated piece of application logic.

### What is integration testing?

Integration testing checks whether multiple components work correctly together.

### What is functional testing?

Functional testing verifies application behavior from the perspective of a feature or user workflow.

### What is PHPUnit?

PHPUnit is a testing framework for PHP applications.

### What is a mock?

A mock object simulates a dependency and allows you to control its behavior and verify interactions.

### What is a stub?

A stub provides predefined responses to method calls.

### What is the difference between a mock and a stub?

A stub provides data. A mock is also used to verify expected interactions.

### What is test coverage?

Test coverage measures which parts of the code are executed during tests. High coverage does not automatically mean high test quality.

### What is TDD?

Test-Driven Development follows:

1. Write a failing test.
    
2. Implement the minimum code to pass.
    
3. Refactor the implementation.
    

### What is static analysis?

Static analysis examines code without executing it.

Examples: PHPStan, Psalm.

### What is code style checking?

It checks whether code follows agreed coding standards.

Example: PHP_CodeSniffer (PHPCS).

---

## 16. Architecture and Senior-Level Concepts

### What is separation of concerns?

It means keeping different responsibilities in separate components.

For example, controllers handle HTTP requests, services contain business logic, and repositories handle persistence.

### What is a Service Layer?

A service layer contains application or business operations and coordinates different components.

### What is a DTO?

A Data Transfer Object is an object used to transfer structured data between application layers without exposing internal domain entities.

### What is an Entity?

An Entity is an object defined by its identity, which remains important even if its attributes change.

### What is a Value Object?

A Value Object is defined by its values rather than identity. It is usually immutable.

Example: Money, EmailAddress, DateRange.

### What is Domain-Driven Design?

DDD is an approach to designing software around business domains and their concepts.

Important concepts:

- Entity.
    
- Value Object.
    
- Aggregate.
    
- Repository.
    
- Domain Service.
    
- Bounded Context.
    
- Ubiquitous Language.
    

### What is an Aggregate?

An Aggregate is a group of domain objects treated as one consistency boundary.

The Aggregate Root controls access to the objects inside the aggregate.

### What is CQRS?

Command Query Responsibility Segregation separates operations that change state (commands) from operations that read data (queries).

### What is Event-Driven Architecture?

Components communicate through events instead of directly depending on each other.

Example: after an order is created, separate listeners can send an email, update statistics, and publish a notification.

### What is a Message Queue?

A message queue allows tasks to be processed asynchronously.

Examples: RabbitMQ, Redis-based queues.

Benefits:

- Reduces request processing time.
    
- Supports background jobs.
    
- Improves scalability.
    
- Helps handle temporary failures through retries.
    

### What is eventual consistency?

Eventual consistency means that distributed systems may temporarily contain different data, but they are expected to become consistent over time.

### What is caching?

Caching stores frequently accessed data to reduce computation and database load.

Examples: Redis, Memcached, OPcache.

### What is horizontal scaling?

Horizontal scaling means adding more application servers.

### What is vertical scaling?

Vertical scaling means increasing the resources of an existing server, such as CPU and RAM.

### What is a race condition?

A race condition occurs when the result depends on the timing or order of concurrent operations.

Prevention includes transactions, locking, atomic operations, and proper concurrency control.

### What is technical debt?

Technical debt is the future cost caused by choosing a simpler or faster implementation instead of a more maintainable solution.

### How do you approach an unfamiliar codebase?

1. Understand the project structure.
    
2. Read documentation and configuration.
    
3. Identify the main entry points.
    
4. Trace the request lifecycle.
    
5. Understand dependencies and data flow.
    
6. Run existing tests.
    
7. Make a small change and verify its impact.
    

### How do you approach a production bug?

1. Reproduce the issue.
    
2. Check logs and monitoring.
    
3. Identify the root cause.
    
4. Assess the impact.
    
5. Implement the smallest safe fix.
    
6. Add regression tests.
    
7. Deploy and monitor the result.
    

### How do you perform a code review?

I check correctness, readability, security, performance, test coverage, maintainability, and consistency with project architecture.

---

## 17. PHP 8.x Features

### PHP 8.0

- Named arguments.
    
- Attributes.
    
- Constructor property promotion.
    
- Union types.
    
- Match expressions.
    
- Nullsafe operator.
    
- JIT compilation.
    

### PHP 8.1

- Enums.
    
- Fibers.
    
- Readonly properties.
    
- Intersection types.
    
- First-class callable syntax.
    
- `never` return type.
    

### PHP 8.2

- Readonly classes.
    
- Disjunctive Normal Form (DNF) types.
    
- Standalone `null`, `false`, and `true` types.
    
- Deprecation of dynamic properties.
    

### PHP 8.3

- Typed class constants.
    
- Dynamic class constant fetch.
    
- `#[\Override]` attribute.
    
- `json_validate()`.
    

### PHP 8.4

- Property hooks.
    
- Asymmetric visibility.
    
- Property `new` initializers.
    
- Lazy objects.
    
- `#[\Deprecated]` attribute.
    
- New array functions such as `array_find()` and `array_any()`.
    

---

## 18. Common Practical Interview Questions

### How do you handle large files?

I process files in chunks or use streams and generators instead of loading everything into memory.

### How do you handle large database results?

I use pagination, cursor-based iteration, or database-level chunking.

### How do you avoid duplicate code?

I extract reusable logic into services, functions, traits, or shared abstractions where appropriate.

### How do you make code testable?

I use dependency injection, small responsibilities, interfaces where useful, and avoid global state.

### How do you improve a slow application?

I profile the application, identify bottlenecks, optimize database queries, introduce caching, and measure the results.

### How do you handle external API failures?

I use timeouts, retries with backoff, error handling, logging, and circuit breakers when appropriate.

### How do you prevent duplicate payment processing?

I use idempotency keys, database constraints, transactions, and proper state transitions.

### How do you manage database schema changes?

I use version-controlled migrations, review their impact, and plan backward-compatible deployments.

### How do you ensure backward compatibility?

I avoid breaking public interfaces, introduce changes gradually, use deprecation periods, and maintain tests for existing behavior.

### How do you choose between inheritance and composition?

I generally prefer composition because it creates more flexible and loosely coupled code. I use inheritance when there is a clear and stable IS-A relationship.

### When would you use an interface?

When different implementations should follow the same contract.

### When would you use an abstract class?

When related classes share common state or implementation.

### When would you use a Generator?

When I need to process a large sequence of values without keeping the entire collection in memory.

---

## 19. Essential PHP Functions to Remember

|Function|Purpose|
|---|---|
|`isset()`|Check if variable exists and is not null|
|`empty()`|Check if value is empty|
|`unset()`|Remove variable or array element|
|`count()`|Count elements|
|`array_map()`|Transform elements|
|`array_filter()`|Filter elements|
|`array_reduce()`|Reduce array to one value|
|`array_merge()`|Merge arrays|
|`array_keys()`|Get array keys|
|`array_values()`|Get array values|
|`in_array()`|Search for a value|
|`array_key_exists()`|Check if a key exists|
|`array_find()`|Find first matching element (PHP 8.4)|
|`usort()`|Sort using a custom comparison|
|`explode()`|Convert string to array|
|`implode()`|Convert array to string|
|`str_contains()`|Check substring|
|`str_starts_with()`|Check string prefix|
|`str_ends_with()`|Check string suffix|
|`trim()`|Remove whitespace|
|`substr()`|Extract part of string|
|`password_hash()`|Hash password|
|`password_verify()`|Verify password|
|`json_encode()`|Convert PHP value to JSON|
|`json_decode()`|Convert JSON to PHP value|
|`serialize()`|Serialize PHP value|
|`unserialize()`|Restore serialized value|
|`file_get_contents()`|Read file|
|`file_put_contents()`|Write file|
|`filter_var()`|Validate or sanitize a value|
|`class_exists()`|Check whether class exists|
|`method_exists()`|Check whether method exists|
|`property_exists()`|Check whether property exists|

---

## 20. Key Differences — Quick Revision

|Concept A|Concept B|
|---|---|
|`==` — loose comparison|`===` — strict comparison|
|`include` — warning|`require` — fatal error|
|Interface — contract|Abstract class — shared behavior|
|Trait — code reuse|Inheritance — parent-child relationship|
|`self` — defining class|`static` — called class|
|`array_map()` — transform|`array_filter()` — select|
|`array_reduce()` — aggregate|`foreach` — iterate|
|Exception — application failure|Error — engine/programming failure|
|Hashing — one-way|Encryption — reversible|
|Authentication — identity|Authorization — permissions|
|PUT — replace|PATCH — partial update|
|Unit test — isolated unit|Integration test — component interaction|
|Stub — predefined response|Mock — verify interactions|
|Factory — create objects|Repository — manage data access|
|DI — provide dependencies|DIP — depend on abstractions|
|Vertical scaling — stronger server|Horizontal scaling — more servers|
|Synchronous — wait for result|Asynchronous — process independently|

---
