---
title: PHP 8.4+ Senior Engineering & System Architecture Revision Sheet
tags:
  - php
  - architecture
  - backend
  - performance
  - internals
  - security
  - interview-prep
cssclasses:
  - wide-page
custom-width: 80
---

# PHP 8.4+ Senior Technical & System Architecture Revision Sheet

## Quick Navigation
- [[#1. PHP Runtime, Zend Engine & Lifecycle]]
- [[#2. Memory Model: zval, Copy-on-Write & Cycle Collector]]
- [[#3. Modern PHP (8.0 – 8.4): Strict Typing, Enums & Engine Features]]
- [[#4. Functional Programming & Closures Internals]]
- [[#5. Streaming & Concurrency: Generators & Fibers]]
- [[#6. OOP Core, Property Hooks & Metaprogramming]]
- [[#7. SOLID Principles & Enterprise Design Patterns]]
- [[#8. Error Architecture & Exceptions]]
- [[#9. Relational Databases: PDO, Transactions, Locks & B-Tree Indexes]]
- [[#10. Enterprise Security & OWASP Top 10]]
- [[#11. Advanced HTTP, REST Standards & PSR Ecosystem]]
- [[#12. Caching & Message Queues Architecture]]
- [[#13. Verbatim English Interview Formulations ("What to Say")]]

---

## 1. PHP Runtime, Zend Engine & Lifecycle

### Execution Pipeline
1. **Scanning/Lexing (Re2c):** Stream of source characters $\to$ token stream (`T_VARIABLE`, `T_STRING`).
2. **Parsing (Bison LALR):** Token stream $\to$ Abstract Syntax Tree (AST). Constant folding occurs here (e.g., `86400` evaluated at compile time).
3. **Compilation:** AST $\to$ `zend_op_array` (opcodes with operands `CV`, `TMPVAR`, `VAR`, `CONST`).
4. **Execution (Zend VM):** Direct-threaded code dispatch loop executing opcode C handlers via `goto *handler`.

### OPcache & Shared Memory (SHM)
- **Opcode Caching:** Eliminates lexing, parsing, and compilation for subsequent requests by storing `zend_op_array` in SHM.
- **Interned Strings:** Deduplicates identical string literals (class names, keys, constants) into an immutable SHM pool (`IS_STR_INTERNED`), bypassing refcounting.
- **Preloading (`opcache.preload`):** Compiles code into the Zend core symbol table (`CG(class_table)`) before PHP-FPM binds its socket. Stored code is globally accessible without autoloader overhead, but requires a master process restart (`SIGUSR2`) to update.
- **JIT (Just-In-Time Compiler):** Translates opcodes into native x86/ARM CPU machine instructions using DynASM. Effective for CPU-bound computations; negligible for I/O-bound web requests.

### Execution Models & State Lifecycles
- **Shared-Nothing (PHP-FPM):** Process per request. Everything allocated during the request is purged at `sapi_deactivate`. High fault isolation, zero cross-tenant memory contamination, but has framework bootstrap overhead per request.
- **Worker Mode (RoadRunner / FrankenPHP / Swoole):** Single persistent process loop across thousands of requests. Bypasses bootstrap costs, but introduces high risk of **memory leaks and state leaks** across requests.
  - *Hard requirement:* Reset Singletons, static variables, and run `EntityManager::clear()` between requests.

---

## 2. Memory Model: zval, Copy-on-Write & Cycle Collector

### The `zval` Structure (16 Bytes)
```text
┌────────────────────────┬───────────┬───────────┬───────────────────────┐
│ value (8 bytes)        │ type (1B) │ flags (1B)| u2 (4B - hash/GC info)│
└────────────────────────┴───────────┴───────────┴───────────────────────┘
```
- **Direct Stack Storage (Zero Heap Allocation):** `int`, `float`, `bool`, `null`. Stored directly in the `value` union; no `refcount`.
- **Heap Pointer Types:** `string`, `array`, `object`, `resource`, `reference`. Points to a heap structure with a `zend_refcounted_h` header.

### Copy-on-Write (CoW)
- Passing/assigning complex types (`$b = $a`) copies only the 16-byte `zval` and increments `gc.refcount`. No memory is duplicated.
- Physical memory duplication occurs **only upon mutation** if `refcount > 1`.
- If `refcount === 1`, mutation happens **in-place** (zero allocation overhead).

### Reference Traps (`&`)
- Converts `zval` into `IS_REFERENCE`, wrapping the value inside a `zend_reference` container and bypassing CoW optimizations.
- **Classic Loop Bug:** In `foreach ($arr as &$v)`, `$v` remains bound to the last element after the loop. A subsequent `foreach ($arr as $v)` will progressively mutate that last element.
- *Fix:* Always call `unset($v);` immediately after a reference loop.

### Cycle Collector (Bacon-Rajan Algorithm)
- Standard refcounting frees memory immediately when `refcount === 0`.
- **Cyclic References:** If Object A references Object B and B references A, their refcounts never reach zero after `unset($a, $b)`.
- **Detection Algorithm:**
  1. Composite zvals whose refcount decreases but remains $> 0$ are marked **Purple** and added to the Root Buffer (default capacity: 10,000).
  2. When the buffer is full, GC runs **Trial Deletion**: it traverses the subgraph, decrements refcounts for internal links, and marks nodes **Grey**.
  3. Nodes whose refcount drops to 0 have no external references; they are marked **White** (garbage).
  4. Nodes with remaining references are restored and marked **Black** (alive).
  5. All **White** nodes are swept and freed.

---

## 3. Modern PHP (8.0 – 8.4): Strict Typing, Enums & Engine Features

### Strict Types (`declare(strict_types=1);`)
- Scope is strictly **file-local** and applies to the **caller**, not the callee declaration.
- Allows only one implicit scalar conversion: widening `int` to `float`. Any other mismatch throws a `TypeError`.

### Type System Matrix
- **Union Types (`A|B`):** Must satisfy at least one type. Redundancy (e.g., `bool|false`) causes compile-time errors.
- **Intersection Types (`A&B`):** Must satisfy all types simultaneously. Supports **only class/interface names**; scalars and `null` are forbidden.
- **DNF Types (`(A&B)|null`):** Disjunctive Normal Form. Combines unions and intersections; intersections must be wrapped in parentheses.
- **Return Types:**
  - `void`: Method completes without a return value. Writing `return null;` causes a fatal error.
  - `never`: Method never returns control to the caller (it always throws an exception or calls `exit()`). Enables unreachable-code analysis in static tools.

### Enums (PHP 8.1+)
- Built as final, immutable singleton objects.
- **Pure Enum:** `enum Status { case Draft; case Active; }` (no backing values).
- **Backed Enum:** `enum Status: string { case Draft = 'draft'; }` (typed strictly as `string` or `int`).
  - Methods: `Status::from($val)` (throws `ValueError` on failure), `Status::tryFrom($val)` (returns `null`), `Status::cases()`.
- *Engine Guarantees:* Cannot be instantiated via `new`, cannot be cloned, cannot inherit (`extends`), and have no dynamic properties. Compare via strict identity (`===`).

### PHP 8.4 Specifics
- **Property Hooks:** Eliminates boilerplate getter/setter methods:
  ```php
  public string $fullName {
      get => "$this->first $this->last";
      set (string $value) => [$this->first, $this->last] = explode(' ', $value, 2);
  }
  ```
- **Asymmetric Visibility:** Separates read and write access levels:
  ```php
  public private(set) string $status = 'pending';
  ```
- **New without parentheses chaining:** `new Logger()->log('msg');`.

---

## 4. Functional Programming & Closures Internals

### `callable` vs `\Closure`
- `callable`: Typehint accepting strings (`'trim'`), arrays (`[$obj, 'method']`), invocable objects, and Closures. **Cannot type class properties**.
- `\Closure`: Concrete final class representing anonymous functions and first-class callables. Valid for properties, arguments, and return types.

### First-Class Callables (`...`)
- Compiles via `ZEND_CALLABLE_CONVERT`.
- Evaluates method existence and visibility (`private`/`protected`) **at reference creation time**, not during invocation.
- Fully traceable by static analyzers: `$pipeline = array_map($this->sanitize(...), $data);`.

### Scope Manipulation (`bindTo` & `call`)
- `Closure::bindTo($obj, ScopeClass::class)`: Rebinds `$this` context and lexical scope.
- `$closure->call($targetObj)`: Instantly executes a closure within the target object's scope, allowing hydration of private properties in ORMs without running constructors.

---

## 5. Streaming & Concurrency: Generators & Fibers

### Generators (`yield`)
- Function returns a `\Generator` object implementing `\Iterator`.
- Implements a **stackless coroutine**: upon `yield`, local execution variables are saved in a heap context (`zend_generator_context`), yielding control to the caller.
- Guarantees **$\mathcal{O}(1)$ memory consumption** regardless of stream size. Forward-only: cannot be rewound.
- Bidirectional: `$gen->send($val)` injects values into the generator; `$gen->throw($e)` raises exceptions inside the generator frame.

### Fibers (PHP 8.1+)
- **Stackful cooperative coroutines:** Each Fiber has its own isolated C/Zend execution stack allocated on the heap.
- Unlike Generators, `Fiber::suspend()` can pause execution **deep within nested function calls** without altering parent method signatures.
- Forms the foundation for non-blocking asynchronous event loops (e.g., Revolt PHP, Amp v3).

---

## 6. OOP Core, Property Hooks & Metaprogramming

### Object Model Internals
- Variables hold an **Object Handle** (integer index in the executor's object store), not the physical object structure itself.
- Object assignment (`$b = $a`) copies only the handle; both variables reference the same `zend_object`.
- **Cloning:** `clone $a` makes a **shallow copy**. Scalar fields are copied; nested object handles are shared. Deep copies require an explicit `__clone()` implementation.

### Late Static Binding (LSB)
- `self::`: Resolved at compile time to the declaring class.
- `static::`: Resolved at runtime to the class handling the active call (enables polymorphic static factories: `return new static();`).

### Readonly Semantics
- `readonly` properties must be typed and can only be initialized once within the declaring class scope.
- *Shallow Immutability Warning:* If a readonly property holds an object, its reference cannot be reassigned, but the inner object's internal state remains mutable.

---

## 7. SOLID Principles & Enterprise Design Patterns

### SOLID Implementation Guide
- **S (Single Responsibility):** A class must have only one reason to change, serving a single actor. Decouple domain logic from persistence and transport formatting.
- **O (Open/Closed):** Open for extension, closed for modification. Replace brittle `switch/match` blocks with the **Strategy Pattern** and polymorphic interfaces.
- **L (Liskov Substitution):** Subtypes must fulfill base contracts without breaking expectations:
  - *Covariant returns:* Subclasses may narrow return types.
  - *Contravariant parameters:* Subclasses may widen argument types.
  - Never throw unexpected exceptions (e.g., `NotImplementedException`) in child implementations.
- **I (Interface Segregation):** Favor narrow, role-specific interfaces (`Stringable`, `Countable`) over monolithic ones.
- **D (Dependency Inversion):** High-level domain logic must depend on abstractions (interfaces), never on low-level concrete implementations. Inject interfaces via Constructor Injection.

---

## 8. Error Architecture & Exceptions

### `Throwable` Hierarchy
```text
                  Throwable (Internal Interface)
                  /                            \
        Exception (Domain/App)             Error (Engine Faults)
        ├── LogicException                 ├── TypeError
        │   └── InvalidArgumentException   ├── ValueError
        ├── RuntimeException               ├── ArithmeticError / DivisionByZeroError
        └── JsonException                  └── UnhandledMatchError
```

### Critical Rules
- User classes **cannot** implement `Throwable` directly; they must extend `Exception` or its subclasses.
- `catch (Exception $e)` **will not catch** engine-level `Error`s (e.g., `TypeError`, `DivisionByZeroError`).
- Use `catch (Throwable $e)` **only at top-level boundary layers** (global exception handlers, root HTTP middleware) for logging and returning clean 500 responses.
- **Finally Trap:** A `return` inside a `finally` block **silently suppresses any active exceptions** thrown in the `try` block. Never return from `finally`.

---

## 9. Relational Databases: PDO, Transactions, Locks & B-Tree Indexes

### PDO Hardening
```php
$pdo = new PDO($dsn, $user, $pass, [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    PDO::ATTR_EMULATE_PREPARES => false, // Uses genuine engine-level prepared statements
]);
```
- Placeholders (`?`, `:name`) can only parameterize **scalar values**. Identifiers (tables, columns, `ORDER BY` directions) must be validated against a strict in-code **whitelist**.

### Isolation Levels & Concurrency Anomalies
- **READ UNCOMMITTED:** Allows Dirty Reads (reading uncommitted changes from another transaction).
- **READ COMMITTED (PostgreSQL default):** Prevents Dirty Reads. Susceptible to Non-Repeatable Reads.
- **REPEATABLE READ (MySQL InnoDB default):** Snapshot isolation via MVCC. Prevents Dirty and Non-Repeatable Reads. InnoDB also prevents Phantom Reads during non-locking reads via Undo Logs.
- **SERIALIZABLE:** Strict sequential execution. Highest safety, lowest throughput.

### Concurrency Controls
- **Pessimistic Locking (`SELECT ... FOR UPDATE`):** Acquires an exclusive row-level lock (X-Lock). Blocks competing transactions until `COMMIT` or `ROLLBACK`. Best for high-contention writes (financial transfers, inventory checks).
- **Optimistic Locking:** Checks an incrementing version column (`WHERE id = :id AND version = :v`). If `rowCount() === 0`, a concurrent modification occurred; throw a concurrency exception and retry.
- **Deadlock Mitigation:**
  1. Enforce a **deterministic lock order** across transactions (e.g., sort record IDs numerically before acquiring row locks).
  2. Keep transactions short: never perform external HTTP/network calls inside an active database transaction.
  3. Catch SQLSTATE `40001` (deadlock/serialization failure) and implement an exponential backoff retry.

### B-Tree Indexes & `EXPLAIN`
- **Leftmost Prefix Rule:** A composite index `(A, B, C)` is usable only if querying left-to-right (`A`, `A, B`, `A, B, C`). Querying `(B, C)` bypasses the index entirely.
- **Range Breaks:** Range conditions (`>`, `<`, `BETWEEN`) break index usage for any subsequent columns in a composite index.
- **Covering Index (`Using index`):** All columns requested by `SELECT`, `WHERE`, and `ORDER BY` exist within the secondary index itself. Bypasses the table lookup entirely (avoids Double Lookup).
- **Keyset Pagination:** Replaces `OFFSET` ($\mathcal{O}(N)$ scan) with cursor checks: `WHERE id > :last_id ORDER BY id ASC LIMIT 50` ($\mathcal{O}(\log N)$ index jump).

---

## 10. Enterprise Security & OWASP Top 10

### Vulnerabilities & Countermeasures
- **SQLi:** Real prepared statements (`ATTR_EMULATE_PREPARES => false`) + whitelists for identifier columns.
- **XSS:** Context-aware escaping on output via `htmlspecialchars($input, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8')`. Set `HttpOnly` on session cookies; enforce a Content Security Policy (CSP). Never escape data prior to database persistence.
- **CSRF:** Cryptographically secure tokens (`random_bytes(32)`) validated via `hash_equals()`. Set session cookies to `SameSite=Lax` or `Strict`.
- **IDOR:** Enforce object-level permissions (Policies/Voters) verifying tenancy (`WHERE id = :id AND account_id = :current_user_account_id`).
- **Timing Attacks:** Regular string comparisons (`===`) exit on the first mismatched byte, leaking secrets via execution time. Always use `hash_equals()` for comparing hashes, tokens, and HMAC signatures.
- **Password Storage:** Use slow Key Derivation Functions: `password_hash($pass, PASSWORD_DEFAULT)` (Bcrypt or Argon2id) and verify via `password_verify()`.

---

## 11. Advanced HTTP, REST Standards & PSR Ecosystem

### Input Processing (`php://input`)
- PHP populates `$_POST` only for `application/x-www-form-urlencoded` and `multipart/form-data`.
- JSON payloads (`Content-Type: application/json`) must be read from the read-only stream:
  ```php
  $payload = json_decode(file_get_contents('php://input'), true, 512, JSON_THROW_ON_ERROR);
  ```

### Output Buffering & Header Rules
- HTTP protocol mandates that all response headers precede response body bytes.
- Sending content (including BOM characters or leading whitespace before `<?php`) triggers the `"Headers already sent"` warning.
- `ob_start()` intercepts script output into an in-memory buffer, deferring transmission to the SAPI layer and allowing header modifications downstream.

### REST Semantics
- **Safe Methods (Read-only):** `GET`, `HEAD`, `OPTIONS`.
- **Idempotent Methods (Same end-state on repeat):** `GET`, `PUT` (complete overwrite), `DELETE`, `HEAD`.
- **Non-Idempotent:** `POST` (resource creation), `PATCH` (partial delta updates).
  - *Production Safeguard:* Protect financial `POST` requests via an `Idempotency-Key` header cached in Redis.

### PSR Standards
- **PSR-7:** Immutable HTTP message interfaces (`withHeader()` returns a clone).
- **PSR-15:** Request Handlers & Middleware using the **Onion Model** (pre-processing $\to$ inner layer handler $\to$ post-processing).

---

## 12. Caching & Message Queues Architecture

### Cache Strategies & Hazards
- **Cache-Aside Pattern:** Read from cache $\to$ on cache miss, query database $\to$ write to cache with TTL.
- **Cache Stampede (Thundering Herd):** High-frequency cache key expires; hundreds of concurrent requests hit the database simultaneously.
  - *Fix:* Distributed mutual exclusion lock (Redis `SET resource_key my_random_token NX PX 5000`) or probabilistic early expiration (XFetch algorithm).

### Queue Resiliency
- Consuming workers must be **idempotent**: processing the same queue event twice should not cause duplicate side effects (e.g., track processed message IDs in Redis or use database unique constraints).
- Supervisor should manage worker lifecycles; configure workers to restart after handling a fixed number of jobs (e.g., `--max-jobs=1000`) to clear low-level memory leaks.

---

## 13. Verbatim English Interview Formulations ("What to Say")

Use these concise, professional phrasings during your interview:

### Explaining Copy-on-Write (CoW)
> *"In PHP, variables and arrays are passed by value, but values are not physically duplicated in memory right away. Under the hood, the zval shares the same memory buffer and increments the refcount. Physical memory duplication occurs only when one of the variables is mutated, which keeps read-heavy operations fast and memory-efficient."*

### Explaining Race Conditions & Locking
> *"To prevent race conditions during concurrent financial balance updates, we wrap the operation in a transaction and apply a pessimistic lock using `SELECT ... FOR UPDATE`. To prevent deadlocks when multiple accounts are involved, we sort the account IDs deterministically before acquiring row locks, ensuring all processes request resources in the same sequence."*

### Explaining Dependency Inversion (DIP)
> *"Dependency Inversion states that high-level domain logic should not depend on low-level infrastructure details; both must depend on abstractions. In practice, I inject interfaces into services via Constructor Injection. This decouples our business rules from specific database or API drivers and allows straightforward unit testing with test doubles."*

### Explaining LSB (`self::` vs `static::`)
> *"The difference comes down to binding resolution time. `self::` is resolved at compile time to the specific class where the code is written, whereas `static::` uses Late Static Binding to resolve at runtime to the class that actually initiated the call. This is why `static::` is required when writing polymorphic factory methods in abstract base classes."*

### Explaining Generators & Streams
> *"Generators act as stackless coroutines via the `yield` keyword. Instead of buffering a massive dataset in memory, the engine preserves the function's execution frame and yields control back to the caller item by item, maintaining $\mathcal{O}(1)$ constant memory usage regardless of data volume."*