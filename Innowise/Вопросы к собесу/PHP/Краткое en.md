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
## 1. Engine Internals & Runtimes

> [!theory] Compilation & Execution Pipeline
> 
> PHP does not interpret code line by line. The **Zend Engine** tokenizes source code, builds an **AST**, compiles it into intermediate **Opcodes**, and executes them on the **Zend Virtual Machine**.

- **OPcache:** Caches compiled opcodes in shared memory (`shm`). It completely bypasses lexical analysis, AST parsing, and compilation on subsequent hits.
    
- **Preloading (PHP 7.4+):** Compiles framework and vendor classes directly into shared memory on server boot. Classes stay permanently accessible to all workers without per-request loading.
    
- **PHP-FPM (Shared-Nothing):** Each request runs in an isolated worker. On exit, `sapi_deactivate` destroys all user memory, globals, and instances. Leaks between requests are prevented, but framework bootstrap overhead occurs on every call.
    
- **Worker Runtimes (RoadRunner, FrankenPHP, Swoole):** The application boots once and stays resident in memory. Request latency drops significantly.
    
- **State Management in Workers:** Singletons persist across calls. You must clear ORM identity maps (`$em->clear()`), reset static caches, and avoid retaining request-scoped references.
    

## 2. Memory: zval, Copy-on-Write & Garbage Collection

> [!tip] Memory Allocation
> 
> The engine stores variables in a 16-byte `zval` container. Scalars (`int`, `float`, `bool`, `null`) live directly inside the `zval` union without heap allocations.

- **Copy-on-Write (CoW):** Complex structures (arrays, strings) share memory buffers. Assigning or passing an array increments `gc.refcount`. Physical duplication occurs only when one variable is modified.
    
- **Zend References (`&`):** Wraps data in a `zend_reference` structure and bypasses standard CoW optimizations.
    
- **The Foreach Reference Trap:** `foreach ($arr as &$v)` leaves `$v` bound to the last element after loop completion. A subsequent `foreach ($arr as $v)` will overwrite that final element on each pass. Always call `unset($v);` immediately after a reference loop.
    
- **Cycle Collector:** Reference counting immediately frees memory when `refcount === 0`. It fails on circular object references ($A \to B \to A$).
    
- **Root Buffer Algorithm:** The engine places suspected circular roots into a buffer (capacity: 10,000). When full, it runs trial-deletion coloring (Black, Purple, Grey, White) to detect and deallocate unreferenced cyclic islands.
    

## 3. Modern PHP 8.0 – 8.4 Language Mechanics

> [!note] Type System Evolution
> 
> Modern PHP provides a mathematically sound, statically verifiable type system with full runtime enforcement.

- **First-Class Callables (8.1+):** `$this->method(...)` compiles to `ZEND_CALLABLE_CONVERT`. Scope visibility and method presence are validated at creation, returning a typed `\Closure`.
    
- **Composite Types:**
    
    - **Union (`A|B`):** Matches either type.
        
    - **Intersection (`A&B`):** Matches both types simultaneously. Classes and interfaces only.
        
    - **DNF (`(A&B)|C`):** Disjunctive Normal Form. Combines unions and intersections using explicit parentheses.
        
- **Type Literals:** `true`, `false`, and `null` function as standalone types.
    
- **`void` vs `never`:** `void` returns control without a value. `never` (8.1+) guarantees execution halts via `exit()` or an uncaught exception.
    
- **Enums (8.1+):** Immutable singleton objects. Cannot be instantiated with `new`, cloned, or inherited. Pure Enums hold no state; Backed Enums map to `string` or `int` with built-in `from()` and `tryFrom()`. Implement `JsonSerializable` for API payloads.
    
- **Property Hooks (8.4):** Inline property interceptors (`get => ...;`, `set => ...;`). Eliminates getter/setter boilerplate for virtual and backed fields.
    
- **Asymmetric Visibility (8.4):** Declares distinct access levels for read and write operations (e.g., `public private(set) string $id`).
    

## 4. Architecture & SOLID Principles

> [!important] Architectural Rule
> 
> SOLID principles are practical tools to eliminate tight coupling, make code testable, and support safe horizontal expansion.

- **Single Responsibility (SRP):** One class, one business actor. Keep persistence, validation, HTTP handling, and notifications in dedicated services.
    
- **Open/Closed (OCP):** Open for extension, closed for modification. Replace expanding `switch`/`match` blocks with polymorphism and the **Strategy Pattern**.
    
- **Liskov Substitution (LSP):** Subtypes must preserve parent behavior. The engine enforces **contravariant arguments** (can widen) and **covariant returns** (can narrow). Child classes must never throw unexpected exceptions like `NotImplementedException`.
    
- **Interface Segregation (ISP):** Prefer multiple small, client-specific interfaces over large, monolithic ones (e.g., separate `FileReaderInterface` from `FileWriterInterface`).
    
- **Dependency Inversion (DIP):** Depend on abstractions, not concrete implementations. High-level use cases accept interfaces through **Constructor Injection**; concrete adapters live in the infrastructure layer.
    

## 5. Database Concurrency, Transactions & Indexing

> [!warning] Concurrency Rule
> 
> Prevent race conditions in high-contention operations (e.g., balances, inventory) at the database layer, not within PHP memory.

- **ACID Transactions:** Enforce atomicity with explicit transaction boundaries:
    
    PHP
    
    ```
    $pdo->beginTransaction();
    try {
        // business mutations
        $pdo->commit();
    } catch (Throwable $e) {
        if ($pdo->inTransaction()) { $pdo->rollBack(); }
        throw $e;
    }
    ```
    
- **Isolation Levels:** PostgreSQL defaults to `READ COMMITTED` (prevents dirty reads). MySQL InnoDB defaults to `REPEATABLE READ`. InnoDB avoids phantom reads during locking operations using **Next-Key Locks** (record + gap locks) and during reads using **MVCC**.
    
- **Pessimistic Locking:** `SELECT ... FOR UPDATE` applies an exclusive lock (X-lock). Concurrent transactions queue until the lock holder commits.
    
- **Optimistic Locking:** Adds a `version` column. Updates verify state: `WHERE id = :id AND version = :v`. If `rowCount() === 0`, handle the collision in userland.
    
- **Deadlock Prevention:** Always acquire row locks in a deterministic order (e.g., sort IDs in ascending order). Implement automated retry logic for SQL error code `40001`.
    
- **B-Tree Indexes:**
    
    - **Leftmost Prefix Rule:** A composite index `(A, B, C)` serves queries on `(A)`, `(A, B)`, and `(A, B, C)`. It will not index `(B)` or `(C)` alone. Range operators (`>`, `<`, `BETWEEN`) on a column prevent subsequent columns from using the index.
        
    - **Covering Index:** Leaf nodes contain all queried fields (`SELECT`, `WHERE`, `ORDER BY`). The database skips the table lookup step (`Using index` in `EXPLAIN`).
        
    - **Keyset Pagination:** Replace slow `OFFSET` scans with cursor queries: `WHERE id > :last_id ORDER BY id ASC LIMIT 50`.
        

## 6. OWASP Web Security

> [!danger] Defense in Depth
> 
> Never trust client input. Apply context-aware validation on entry and escaping on exit.

- **SQL Injection (SQLi):** Enforce real prepared statements (`PDO::ATTR_EMULATE_PREPARES => false`). Parameters cannot bind identifiers; validate column and table names against explicit whitelists.
    
- **Cross-Site Scripting (XSS):** Store raw data in the database. Escape on output using `htmlspecialchars($str, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8')`. Mark session cookies as `HttpOnly` and set a restrictive Content Security Policy (CSP).
    
- **Cross-Site Request Forgery (CSRF):** Protect mutable requests (`POST`, `PUT`, `DELETE`) with session-backed tokens checked via `hash_equals()`. Set `SameSite=Lax` or `Strict` on cookies.
    
- **Insecure Direct Object References (IDOR):** Never query by client-supplied IDs alone. Always scope records to the authenticated tenant (e.g., `WHERE id = :id AND user_id = :auth_id`).
    
- **Password Hashing:** Use adaptive slow hashes: `password_hash($pass, PASSWORD_DEFAULT)` (Bcrypt or Argon2id) with `password_verify()`. Never use SHA/MD5. Use `random_bytes()` for crypto-secure tokens.
    

## 7. HTTP Protocols, REST & PSR Standards

> [!quote] Professional Communication
> 
> Model HTTP communication around standard status codes, explicit streaming, and immutable message representations.

- **Raw JSON Payloads:** The `$_POST` array ignores JSON requests. Read raw payload bytes directly from `php://input` and parse with `json_decode(..., flags: JSON_THROW_ON_ERROR)`.
    
- **Output Buffering:** Sending content before `header()` triggers _Headers already sent_. Use `ob_start()` and `ob_get_clean()` to manage output delivery and templating safely.
    
- **REST Idempotency:**
    
    - `GET`: Safe and idempotent.
        
    - `PUT`: Idempotent (full replacement).
        
    - `DELETE`: Idempotent (repeated calls leave state unchanged).
        
    - `POST`: Non-idempotent. Use an `Idempotency-Key` header cached in Redis to safely retry sensitive actions (e.g., payments).
        
- **PSR-7 & PSR-15:** PSR-7 treats HTTP messages as immutable value objects; modifications return clones via `with*()` methods. PSR-15 defines the middleware pipeline (the Onion Model), where requests pass down the chain and responses bubble back up through `$handler->handle($request)`.