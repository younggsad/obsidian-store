---
custom-width: 80
---
## Оглавление

1. [[#1. Типы баз данных: Реляционные vs NoSQL]]
2. [[#2. Классификация команд SQL: DDL, DML, DQL, DCL, TCL]]
3. [[#3. Базовые команды и операции: JOIN, UNION, DELETE vs TRUNCATE vs DROP]]
4. [[#4. ACID и жизненный цикл транзакции (WAL)]]
5. [[#5. Параллельный доступ: MVCC, аномалии и уровни изоляции]]
6. [[#6. Индексы: B-Tree, Hash, GIN, составные и покрывающие]]
7. [[#7. Оптимизация запросов и чтение EXPLAIN]]
8. [[#8. Проектирование схемы: Нормализация, Денормализация, Ключи]]
9. [[#9. Блокировки (Pessimistic vs Optimistic) и Deadlocks]]
10. [[#10. Продвинутый SQL: GROUP BY/HAVING, Оконные функции, CTE]]
11. [[#11. Масштабирование БД: Репликация, Партиционирование, Шардирование]]
12. [[#12. Шпаргалка перед собеседованием]]

---

## 1. Типы баз данных: Реляционные vs NoSQL

> [!abstract] Коротко
> Реляционные БД (PostgreSQL, MySQL) хранят структурированные данные в таблицах со связями и строгими гарантиями ACID. NoSQL решает задачи высокой скорости записи, неструктурированных документов или кэширования.

### Теория (🇷🇺 Русский)

- **Реляционные СУБД (RDBMS — PostgreSQL, MySQL/InnoDB)**:
  - Данные организованы в таблицы со строгой схемой (Schema-on-Write), внешними ключами и связями.
  - Поддерживают декларативный язык SQL, сложные `JOIN`-ы, агрегации и полные **ACID-транзакции**.
  - **Когда использовать**: 90% бизнес-задач (финансы, заказы, пользователи, каталоги, CRM), где важна целостность и непротиворечивость данных.
- **NoSQL базы данных**:
  - **Key-Value (Redis, Memcached)**: хранение в RAM по ключу за $O(1)$. Идеально для кэша, сессий, счётчиков, rate limiting, очередей.
  - **Document-oriented (MongoDB)**: хранение JSON/BSON документов с гибкой схемой. Удобно для логов, профилей с разным набором атрибутов.
  - **Column-family (ClickHouse, Cassandra)**: колоночное хранение на диске. ClickHouse идеален для аналитики (OLAP) и агрегации миллиардов строк за миллисекунды.
  - **Search Engines (Elasticsearch, OpenSearch)**: полнотекстовый поиск на базе инвертированных индексов.

### Theory (🇬🇧 English)

- **Relational DBMS (PostgreSQL, MySQL/InnoDB)**:
  - Stores structured data in tables with strict schemas (Schema-on-Write), foreign keys, and full **ACID** compliance.
  - **Best for**: Core business domains (payments, orders, users) requiring strong consistency and complex relational queries.
- **NoSQL Databases**:
  - **Key-Value (Redis)**: In-memory $O(1)$ lookups; used for caching, sessions, rate limiting, and locks.
  - **Document (MongoDB)**: Schema-less BSON documents; good for variable attributes and unstructured payloads.
  - **Columnar (ClickHouse)**: Column-oriented storage optimized for analytical (OLAP) queries over massive datasets.
  - **Search Engines (Elasticsearch)**: Inverted indexes for full-text search and log aggregation.

---

## 2. Классификация команд SQL: DDL, DML, DQL, DCL, TCL

> [!abstract] Коротко
> Весь язык SQL делится на 5 функциональных групп в зависимости от того, с чем работает команда: со структурой таблиц, с самими данными, с правами доступа или с транзакциями.

### Теория (🇷🇺 Русский & 🇬🇧 English)

| Группа | Расшифровка | Назначение (🇷🇺 / 🇬🇧) | Основные команды |
| :--- | :--- | :--- | :--- |
| **DDL** | *Data Definition Language* | Определение и изменение **структуры** объектов БД (таблиц, индексов, схем) / *Defines and alters database schema structure* | `CREATE`, `ALTER`, `DROP`, `TRUNCATE`, `RENAME` |
| **DML** | *Data Manipulation Language* | Изменение **данных** внутри таблиц / *Modifies data inside tables* | `INSERT`, `UPDATE`, `DELETE`, `MERGE` / `UPSERT` |
| **DQL** | *Data Query Language* | Выборка и чтение данных / *Queries and retrieves data* | `SELECT` |
| **DCL** | *Data Control Language* | Управление правами доступа пользователей / *Manages user permissions and access control* | `GRANT`, `REVOKE` |
| **TCL** | *Transaction Control Language* | Управление транзакциями / *Controls transaction boundaries* | `BEGIN` / `START TRANSACTION`, `COMMIT`, `ROLLBACK`, `SAVEPOINT` |

> [!warning] Важный нюанс про DDL и транзакции
> В **MySQL** любая DDL-команда (`ALTER TABLE`, `DROP`, `TRUNCATE`) вызывает **неявный (автоматический) `COMMIT`** текущей транзакции и сама не может быть отменена через `ROLLBACK`.
> В **PostgreSQL** DDL-команды являются **транзакционными** — вы можете сделать `BEGIN; ALTER TABLE ...; DROP TABLE ...; ROLLBACK;`, и структура базы вернётся в исходное состояние!

---

## 3. Базовые команды и операции: JOIN, UNION, DELETE vs TRUNCATE vs DROP

> [!abstract] Коротко
> Чётко различай удаление данных с логированием (`DELETE`), мгновенную очистку таблицы (`TRUNCATE`) и полное уничтожение таблицы (`DROP`), а также логику объединения таблиц по горизонтали (`JOIN`) и вертикали (`UNION`).

### Теория (🇷🇺 Русский)

#### 1. Сравнение `DELETE` vs `TRUNCATE` vs `DROP`

| Критерий | `DELETE` | `TRUNCATE` | `DROP` |
| :--- | :--- | :--- | :--- |
| **Категория SQL** | DML (работа с данными) | DDL (работа со структурой) | DDL (работа со структурой) |
| **Что удаляет** | Конкретные строки (по условию `WHERE`) или все строки | **Все** строки в таблице (условие `WHERE` запрещено) | Саму таблицу целиком: данные, структуру, индексы, триггеры, права |
| **Скорость работы** | Медленно (удаляет построчно, обновляет индексы на каждой строке) | Очень быстро (освобождает страницы данных / пересоздаёт таблицу) | Мгновенно (удаляет метаданные и файлы таблицы) |
| **Журнал (WAL / Undo Log)** | Записывает каждую удалённую строку в лог | Минимальное логирование (только факт освобождения страниц) | Минимальное логирование |
| **Откат (`ROLLBACK`)** | Да (внутри транзакции) | В PostgreSQL — да; в MySQL — **нет** (неявный `COMMIT`) | В PostgreSQL — да; в MySQL — **нет** |
| **Сброс `AUTO_INCREMENT` / `SERIAL`** | **Нет** (счётчик продолжает расти с последнего ID) | **Да** (сбрасывает счётчик в 1; в Postgres с опцией `RESTART IDENTITY`) | Таблица перестаёт существовать |
| **Триггеры `ON DELETE`** | Срабатывают для каждой строки | **Не срабатывают** (только `ON TRUNCATE` в Postgres) | Не срабатывают |

#### 2. Объединение таблиц: Все виды `JOIN`

`JOIN` соединяет таблицы **по горизонтали** (добавляя столбцы из второй таблицы) на основе условия совпадения ключей (`ON`).

- **`INNER JOIN` (или просто `JOIN`)**: возвращает только те строки, для которых нашлось совпадение **в обеих** таблицах.
- **`LEFT JOIN` (`LEFT OUTER JOIN`)**: возвращает **все** строки из левой таблицы и совпавшие из правой. Если пары справа нет — подставляет `NULL`.
  - *Классический приём (`LEFT JOIN ... WHERE right.id IS NULL`)*: найти записи, у которых **нет** связей (например, пользователей без единого заказа).
- **`RIGHT JOIN` (`RIGHT OUTER JOIN`)**: зеркально `LEFT JOIN` — возвращает все строки из правой таблицы и совпавшие из левой. На практике стараются не использовать, заменяя на `LEFT JOIN` для читаемости.
- **`FULL OUTER JOIN`**: возвращает все строки из обеих таблиц; там, где пары нет, подставляет `NULL`. (Встроен в PostgreSQL; в MySQL эмулируется через `LEFT JOIN ... UNION ALL RIGHT JOIN ...`).
- **`CROSS JOIN`**: декартово произведение (каждая строка первой таблицы соединяется с каждой строкой второй: $N \times M$ строк). Условие `ON` не используется.
- **`SELF JOIN`**: соединение таблицы самой с собой (например, таблица `employees` с колонкой `manager_id`, ссылающейся на `employees.id`).

#### 3. Вертикальное объединение: `UNION` vs `UNION ALL`

`UNION` склеивает результаты двух и более `SELECT`-запросов **по вертикали** (количество колонок и их типы должны совпадать).
- **`UNION`**: объединяет результаты и **удаляет дубликаты** (выполняет сортировку/хэширование под капотом, поэтому медленнее).
- **`UNION ALL`**: просто склеивает результаты как есть **без проверки на дубликаты** (работает намного быстрее — всегда предпочитай `UNION ALL`, если дубликаты невозможны или не мешают).

#### 4. Основные DML-команды и `UPSERT`

- **`INSERT INTO ... VALUES (...)`**: вставка одной или нескольких строк (batch insert).
- **`UPDATE ... SET ... WHERE ...`**: обновление строк. **Никогда не запускай `UPDATE` или `DELETE` без `WHERE`**, если не хочешь перезаписать всю таблицу!
- **`UPSERT` (Insert or Update при конфликте уникального ключа)**:
  - *PostgreSQL*: `INSERT INTO users (email, name) VALUES ('a@b.c', 'Ivan') ON CONFLICT (email) DO UPDATE SET name = EXCLUDED.name;`
  - *MySQL*: `INSERT INTO users (email, name) VALUES ('a@b.c', 'Ivan') ON DUPLICATE KEY UPDATE name = VALUES(name);`

### Theory (🇬🇧 English)

- **SQL Command Categories**:
  - **DDL** (*Data Definition Language*): `CREATE`, `ALTER`, `DROP`, `TRUNCATE` — modifies schema structure. Implicitly commits transactions in MySQL, whereas PostgreSQL supports transactional DDL.
  - **DML** (*Data Manipulation Language*): `INSERT`, `UPDATE`, `DELETE`, `MERGE` — modifies row data.
  - **DQL** (*Data Query Language*): `SELECT` — retrieves data.
  - **DCL** (*Data Control Language*): `GRANT`, `REVOKE` — manages permissions.
  - **TCL** (*Transaction Control Language*): `BEGIN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT`.
- **`DELETE` vs `TRUNCATE` vs `DROP`**:
  - `DELETE` (DML): Row-by-row removal supporting `WHERE`, fires triggers, logs every row, preserves `AUTO_INCREMENT`, can be rolled back.
  - `TRUNCATE` (DDL): Fast deallocation of all data pages (`WHERE` not allowed), resets `AUTO_INCREMENT`, does not fire row-level `ON DELETE` triggers.
  - `DROP` (DDL): Completely removes the table schema, data, indexes, and constraints from the database.
- **`JOIN` & `UNION`**:
  - `INNER JOIN` returns matching rows from both tables; `LEFT JOIN` preserves all left-table rows (filling unmatched right columns with `NULL`).
  - `UNION` deduplicates combined result sets (slower); `UNION ALL` concatenates without deduplication (faster).

### Схема и Код

```mermaid
flowchart TD
    subgraph SQL_Commands [Категории SQL]
        DDL[DDL: CREATE, ALTER, DROP, TRUNCATE]
        DML[DML: INSERT, UPDATE, DELETE]
        DQL[DQL: SELECT]
        TCL[TCL: BEGIN, COMMIT, ROLLBACK]
        DCL[DCL: GRANT, REVOKE]
    end
````

```SQL
-- DDL: Создание таблицы с ограничениями и внешним ключом
CREATE TABLE orders (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    total NUMERIC(10, 2) NOT NULL CHECK (total >= 0),
    status VARCHAR(20) NOT NULL DEFAULT 'new',
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- DDL: Изменение структуры
ALTER TABLE orders ADD COLUMN comment TEXT;

-- DML: Upsert в PostgreSQL (вставка или обновление при конфликте уникального ключа)
INSERT INTO user_stats (user_id, logins_count)
VALUES (42, 1)
ON CONFLICT (user_id) 
DO UPDATE SET logins_count = user_stats.logins_count + 1;

-- LEFT JOIN для поиска пользователей БЕЗ заказов
SELECT u.id, u.email
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE o.id IS NULL;

-- SELF JOIN: вывод сотрудника и имени его руководителя из одной таблицы
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

### Вопросы и ответы

> [!question] В чём ловушка фильтрации правой таблицы при `LEFT JOIN`?
> 
> **Ловушка**: если написать условие по правой таблице в блоке `WHERE`, `LEFT JOIN` неявно превратится в `INNER JOIN`:
> 
> ```SQL
> -- ПЛОХО: отфильтрует всех пользователей без заказов, так как o.status для них равен NULL!
> SELECT u.id, o.total FROM users u
> LEFT JOIN orders o ON o.user_id = u.id
> WHERE o.status = 'paid';
> 
> -- ХОРОШО: условие должно быть внутри ON, чтобы сохранить всех пользователей
> SELECT u.id, o.total FROM users u
> LEFT JOIN orders o ON o.user_id = u.id AND o.status = 'paid';
> ```

## 4. ACID и жизненный цикл транзакции (WAL)

> [!abstract] Коротко
> 
> Транзакция — это неделимая единица работы с БД. Надёжность (Durability) и атомарность обеспечиваются через **WAL (Write-Ahead Log)** — журнал упреждающей записи.

### Теория (🇷🇺 Русский)

- **Atomicity (Атомарность)**: принцип «всё или ничего». Если в транзакции из 5 запросов упал 4-й, первые 3 полностью откатываются (`ROLLBACK`).
    
- **Consistency (Согласованность)**: транзакция переводит БД из одного корректного состояния в другое, не нарушая инварианты (`NOT NULL`, `UNIQUE`, `FOREIGN KEY`, `CHECK`).
    
- **Isolation (Изолированность)**: параллельно выполняющиеся транзакции не видят промежуточных (незафиксированных) изменений друг друга в соответствии с выбранным уровнем изоляции.
    
- **Durability (Долговечность)**: если СУБД вернула `COMMIT OK`, данные гарантированно не пропадут даже при внезапном отключении электричества.
    
- **Как работает WAL (Write-Ahead Log / Redo Log)**:
    
    - Писать изменения сразу в случайные места файлов таблиц на диске (Random I/O) очень медленно.
        
    - При `COMMIT` СУБД сначала последовательно (Sequential I/O — максимально быстро) дописывает изменение в журнал **WAL** на диске, а страницы данных в оперативной памяти (Buffer Pool) помечает как «грязные» (dirty pages) и сбрасывает на диск в фоне. При сбое питания БД при старте просто «проигрывает» WAL и восстанавливает все зафиксированные транзакции.
        

### Theory (🇬🇧 English)

- **Atomicity**: "All or nothing" execution; partial failures trigger a full `ROLLBACK`.
    
- **Consistency**: Preserves all database invariants and constraints (`UNIQUE`, `FOREIGN KEY`, `CHECK`).
    
- **Isolation**: Concurrent transactions execute without interfering with each other based on the configured isolation level.
    
- **Durability**: Committed transactions survive crashes or power loss.
    
- **Write-Ahead Logging (WAL)**: Instead of performing slow random disk writes to table files on every commit, the DBMS appends changes sequentially to the WAL on disk first, flushing modified data pages from RAM (Buffer Pool) asynchronously. On crash recovery, the WAL is replayed.
    

## 5. Параллельный доступ: MVCC, аномалии и уровни изоляции

> [!abstract] Коротко
> 
> Современные СУБД (PostgreSQL, MySQL InnoDB) используют **MVCC**, чтобы читатели не блокировали писателей, а писатели не блокировали читателей. Уровни изоляции определяют баланс между консистентностью и производительностью.

### Теория (🇷🇺 Русский)

- **MVCC (Multi-Version Concurrency Control)**: механизм многоверсионности. При обновлении строки БД не перезаписывает её на месте, а создаёт **новую версию строки** с номером текущей транзакции (в PostgreSQL — скрытые поля `xmin`/`xmax`; в MySQL InnoDB — цепочка версий в Undo Log). Каждая транзакция видит «снимок» (snapshot) данных на момент своего старта.
    
- **Аномалии параллельных транзаций**:
    
    1. **Dirty Read (Грязное чтение)**: транзакция А читает данные, которые транзакция Б изменила, но ещё **не закоммитила** (а потом Б может сделать `ROLLBACK`).
        
    2. **Non-Repeatable Read (Неповторяющееся чтение)**: транзакция А дважды читает одну и ту же строку и получает разные значения, потому что между чтениями транзакция Б сделала `UPDATE` и `COMMIT`.
        
    3. **Phantom Read (Фантомное чтение)**: транзакция А дважды выполняет запрос с условием (`WHERE amount > 100`) и во второй раз видит новые строки, которые в это время вставила (`INSERT`) и закоммитила транзакция Б.
        
    4. **Lost Update (Потерянное обновление)**: две транзакции одновременно прочитали `balance = 100`, обе прибавили `50` и записали `150` вместо `200`.
        

|**Уровень изоляции**|**Dirty Read**|**Non-Repeatable Read**|**Phantom Read**|**Примечание по СУБД**|
|---|---|---|---|---|
|**Read Uncommitted**|Возможен|Возможен|Возможен|В Postgres работает как Read Committed|
|**Read Committed**|Нет|Возможен|Возможен|**По умолчанию в PostgreSQL** (новый снимок на каждый SQL-запрос)|
|**Repeatable Read**|Нет|Нет|Возможен*|**По умолчанию в MySQL InnoDB** (снимок на всю транзакцию)|
|**Serializable**|Нет|Нет|Нет|Полная изоляция; при конфликтах выбрасывает ошибку сериализации (нужен retry)|

> [!tip] Важный нюанс на собеседованиях
> 
> По стандарту SQL на уровне `Repeatable Read` возможен Phantom Read. Но на практике:
> 
> - в **PostgreSQL** `Repeatable Read` использует Snapshot Isolation и **полностью предотвращает** фантомное чтение (но возможна аномалия _Write Skew_);
>     
> - в **MySQL (InnoDB)** обычные `SELECT` читают из MVCC-снимка без фантомов, а блокирующие `SELECT ... FOR UPDATE` используют **Next-Key Locks** (блокировка записи + интервала), также защищая от фантомов.
>     

### Theory (🇬🇧 English)

- **MVCC (Multi-Version Concurrency Control)**: Allows readers and writers to operate without blocking each other by maintaining multiple versions of rows (`xmin`/`xmax` in PostgreSQL; Undo Logs in MySQL InnoDB). Each transaction reads from a consistent point-in-time snapshot.
    
- **Read Phenomena**:
    
    1. **Dirty Read**: Reading uncommitted changes from another transaction.
        
    2. **Non-Repeatable Read**: Re-reading a row yields different values due to a concurrent committed `UPDATE`/`DELETE`.
        
    3. **Phantom Read**: Re-running a range query returns newly committed `INSERT` rows.
        
    4. **Lost Update**: Two concurrent read-modify-write cycles overwrite each other's changes.
        

## 6. Индексы: B-Tree, Hash, GIN, составные и покрывающие

> [!abstract] Коротко
> 
> Индекс — это отдельная отсортированная структура данных, ускоряющая поиск с $O(N)$ до $O(\log N)$, но замедляющая операции записи (`INSERT/UPDATE/DELETE`) и занимающая место.

### Теория (🇷🇺 Русский)

- **B-Tree (Сбалансированное дерево)**: стандартный индекс по умолчанию. Ключи хранятся в отсортированном виде в листовых узлах (связанных двусвязным списком).
    
    - _Поддерживает_: точное совпадение (`=`), диапазоны (`>`, `<`, `>=`, `<=`, `BETWEEN`), сортировку (`ORDER BY`) и префиксный поиск (`LIKE 'ivan%'`).
        
    - _Не работает_: при ведущем wildcard (`LIKE '%ivan%'`) или применении функции к колонке (`WHERE LOWER(email) = ...`).
        
- **Hash-индекс**: хэш-таблица. Поиск за $O(1)$, но работает **только** на точное равенство (`=`). Не поддерживает диапазоны и сортировку.
    
- **GIN (Generalized Inverted Index — в PostgreSQL)**: инвертированный индекс. Незаменим для поиска внутри массивов, `JSONB` и полнотекстового поиска.
    
- **Clustered vs Non-Clustered (Кластерный и некластерный)**:
    
    - В **MySQL InnoDB** первичный ключ (`PRIMARY KEY`) является **кластерным** — сама таблица физически отсортирована и хранится в листьях B-Tree первичного ключа. Вторичные индексы хранят значение `PRIMARY KEY`, поэтому поиск по вторичному индексу делает два прохода по дереву (double lookup).
        
    - В **PostgreSQL** все индексы **некластерные** — таблица хранится в неупорядоченной куче (Heap), а индексы хранят физический адрес строки (`ctid`).
        
- **Составной индекс (Composite Index) `(a, b, c)`**: работает строго по **правилу левого префикса (Leftmost Prefix Rule)**. Индекс будет задействован для условий: `WHERE a = 1`, `WHERE a = 1 AND b = 2`, `WHERE a = 1 AND b = 2 AND c = 3`, `WHERE a = 1 AND b > 5`. Но он **бесполезен** для `WHERE b = 2` или `WHERE c = 3`.
    
- **Покрывающий индекс (Covering Index)**: индекс включает в себя все колонки, указанные в `SELECT` и `WHERE` (в Postgres можно добавить колонки в лист через `INCLUDE (col)`). СУБД отдаёт ответ прямо из индекса без чтения самой таблицы (`Index Only Scan` / `Using index`).
    

### Theory (🇬🇧 English)

- **B-Tree Index**: Default balanced tree structure storing sorted keys in leaf nodes. Supports equality (`=`), range queries (`>`, `<`, `BETWEEN`), sorting (`ORDER BY`), and prefix matching (`LIKE 'abc%'`) in $O(\log N)$ time.
    
- **Hash Index**: $O(1)$ lookup for exact equality (`=`) only; cannot sort or handle ranges.
    
- **GIN (PostgreSQL)**: Inverted index designed for composite values like `JSONB`, arrays, and full-text search.
    
- **Composite Index `(a, b, c)`**: Obeys the **Leftmost Prefix Rule**. Usable for `(a)`, `(a, b)`, and `(a, b, c)`, but cannot optimize queries filtering solely on `(b)` or `(c)`.
    
- **Covering Index**: Contains all columns needed by the query (`SELECT` + `WHERE`), allowing an `Index Only Scan` without touching table heap pages.
    

## 7. Оптимизация запросов и чтение EXPLAIN

> [!abstract] Коротко
> 
> Для диагностики медленных запросов используем `EXPLAIN ANALYZE`, устраняем Full Table Scan, избегаем больших `OFFSET` и не делаем запросов внутри циклов (N+1).

### Теория (🇷🇺 Русский & 🇬🇧 English)

- **`EXPLAIN` vs `EXPLAIN ANALYZE`**:
    
    - `EXPLAIN` строит только теоретический план выполнения на основе статистики планировщика (без реального запуска запроса).
        
    - `EXPLAIN ANALYZE` реально выполняет запрос (осторожно с `UPDATE/DELETE` — оборачивайте в транзакцию с `ROLLBACK`!) и показывает фактическое время и количество прочитанных строк.
        
- **На что смотреть в плане выполнения**:
    
    - **Тип сканирования**: `Seq Scan` / `ALL` (полное чтение таблицы — плохо для больших таблиц) → `Index Scan` / `ref` → `Index Only Scan` / `const`.
        
    - **Расхождение `estimated rows` и `actual rows`**: если планировщик думал, что вернётся 10 строк, а вернулось 500 000, значит устарела статистика — нужно запустить `ANALYZE table_name`.
        
    - **Сортировка и временные таблицы**: `Sort Method: external merge Disk` (PostgreSQL) или `Using filesort; Using temporary` (MySQL) говорят о том, что сортировка не поместилась в индекс или в оперативную память (`work_mem` / `sort_buffer_size`).
        
- **Проблема пагинации с большим `OFFSET`**:
    
    - Запрос `SELECT * FROM orders ORDER BY id LIMIT 20 OFFSET 1000000` заставляет БД прочитать и отсортировать 1 000 020 строк, чтобы выбросить миллион и отдать 20.
        
    - **Решение (Keyset / Cursor Pagination)**: передавать последний увиденный ID: `SELECT * FROM orders WHERE id < :last_id ORDER BY id DESC LIMIT 20`. Выполняется мгновенно по индексу первичного ключа за $O(\log N)$.
        

### Код

```SQL
-- Плохо: индекс по created_at не сработает из-за функции DATE()
SELECT * FROM orders WHERE DATE(created_at) = '2026-11-13';

-- Хорошо: диапазонный поиск использует B-Tree индекс
SELECT id, total FROM orders 
WHERE created_at >= '2026-11-13 00:00:00' 
  AND created_at < '2026-11-14 00:00:00';

-- Покрывающий индекс в PostgreSQL (ускоряет выборку без чтения таблицы)
CREATE INDEX idx_orders_user_status_include 
ON orders (user_id, status) INCLUDE (total, created_at);
```

## 8. Проектирование схемы: Нормализация, Денормализация, Ключи

> [!abstract] Коротко
> 
> Нормализация устраняет избыточность и аномалии при записи; денормализация осознанно возвращает дублирование ради скорости чтения на высоких нагрузках.

### Теория (🇷🇺 Русский)

- **Нормальные формы**:
    
    - **1НФ (1NF)**: в каждой ячейке атомарное (неделимое) значение, нет повторяющихся групп столбцов (`phone1, phone2`), есть первичный ключ.
        
    - **2НФ (2NF)**: таблица в 1НФ + каждый неключевой столбец зависит от **всего** составного первичного ключа целиком, а не от его части.
        
    - **3НФ (3NF)**: таблица в 2НФ + отсутствуют транзитивные зависимости (неключевые столбцы зависят только от первичного ключа, но не друг от друга; например, `city_name` не должен храниться в таблице `orders` рядом с `city_id`).
        
- **Денормализация**: осознанное нарушение 3НФ (например, хранение счётчика `comments_count` прямо в таблице `posts` или копирование цены товара в таблицу `order_items` на момент покупки). Применяется, когда `JOIN`-ы или `COUNT(*)` становятся узким местом.
    
- **Выбор первичного ключа: Auto-Increment (`BIGINT`) vs UUID**:
    
    - **`BIGINT` (Sequential ID)**: занимает 8 байт, новые записи вставляются строго в конец B-Tree дерева (нет фрагментации страниц, максимальная скорость вставки). Минус: предсказуемость (риск IDOR, раскрытие объёма базы) и сложности при слиянии данных с нескольких шардов.
        
    - **`UUID v4` (случайный)**: 16 байт, можно генерировать на клиенте/бэкенде до вставки в БД. **Главный минус**: случайные значения вставляются в случайные места B-Tree, вызывая постоянное расщепление страниц (page splits) и фрагментацию индекса.
        
    - **Компромисс — `UUID v7` / `ULID`**: содержат временную метку (timestamp) в начале, поэтому являются монотонно возрастающими и не фрагментируют B-Tree!
        

### Theory (🇬🇧 English)

- **Normal Forms**:
    
    - **1NF**: Atomic column values, no repeating groups, defined primary key.
        
    - **2NF**: 1NF + no partial dependencies on a composite primary key.
        
    - **3NF**: 2NF + no transitive dependencies (non-key attributes depend strictly on the primary key).
        
- **Denormalization**: Intentional redundancy (e.g., storing `comments_count` on `posts`) to optimize read-heavy workloads at the cost of write complexity.
    
- **Primary Keys (`BIGINT` vs `UUID`)**:
    
    - `BIGINT`: Compact (8 bytes), sequential appends to B-Tree (zero page fragmentation), but leaks record counts and complicates multi-master sharding.
        
    - `UUID v4`: Globally unique, safe from enumeration, but random inserts cause severe B-Tree page splits and cache misses.
        
    - `UUID v7` / `ULID`: Time-ordered UUIDs combining global uniqueness with sequential B-Tree insert performance.
        

## 9. Блокировки (Pessimistic vs Optimistic) и Deadlocks

> [!abstract] Коротко
> 
> Для предотвращения гонок (Race Conditions) при изменении общих данных используют пессимистическую блокировку на уровне БД (`FOR UPDATE`) или оптимистическую через версионирование строк.

### Теория (🇷🇺 Русский & 🇬🇧 English)

- **Пессимистическая блокировка (Pessimistic Locking)**:
    
    - Предполагаем, что конфликты будут происходить часто, поэтому блокируем строку заранее при чтении: `SELECT balance FROM wallets WHERE id = 1 FOR UPDATE;`.
        
    - Другие транзакции, пытающиеся изменить или заблокировать эту строку, встают в очередь до нашего `COMMIT`/`ROLLBACK`.
        
    - _Плюс_: гарантирует отсутствие конфликтов. _Минус_: снижает параллелизм и может привести к Deadlock.
        
- **Оптимистическая блокировка (Optimistic Locking)**:
    
    - Предполагаем, что конфликты редки. Не держим блокировок в БД, а добавляем колонку `version`.
        
    - При обновлении проверяем версию: `UPDATE products SET stock = stock - 1, version = version + 1 WHERE id = 10 AND version = 5;`.
        
    - Если `affected_rows == 0`, значит строку успел изменить другой поток — приложение перечитывает данные и повторяет операцию.
        
- **Deadlock (Взаимная блокировка)**:
    
    - Транзакция 1 заблокировала строку А и ждёт строку Б. Транзакция 2 заблокировала строку Б и ждёт строку А. Обе ждут вечно.
        
    - СУБД отслеживает граф ожиданий и принудительно откатывает одну из транзакций (выбрасывая ошибку `Deadlock found`).
        
    - **Как избежать**: 1) всегда блокировать сущности в одном строгом порядке (например, по возрастанию `ORDER BY id ASC`); 2) держать транзакции максимально короткими (не делать внутри них HTTP-запросы к внешним API!).
        

## 10. Продвинутый SQL: GROUP BY/HAVING, Оконные функции, CTE

> [!abstract] Коротко
> 
> Обязательный минимум для собеседований: понимать порядок выполнения SQL-запроса, разницу между `WHERE` и `HAVING`, и уметь решать задачи аналитики через оконные функции.

### Теория (🇷🇺 Русский)

- **Логический порядок выполнения SQL-запроса** (почему в `WHERE` нельзя использовать алиас из `SELECT`):
    
    1. `FROM` и `JOIN` (сборка исходного набора данных)
        
    2. `WHERE` (фильтрация отдельных строк **до** группировки)
        
    3. `GROUP BY` (сворачивание строк в группы)
        
    4. `HAVING` (фильтрация **групп** по агрегатным функциям `COUNT`, `SUM`, `AVG`)
        
    5. `SELECT` (вычисление выражений и оконных функций, присвоение алиасов)
        
    6. `DISTINCT` (удаление дубликатов)
        
    7. `ORDER BY` (сортировка результата)
        
    8. `LIMIT / OFFSET` (обрезка выборки)
        
- **Оконные функции (Window Functions)**: позволяют выполнять вычисления по набору строк (окну), связанному с текущей строкой, **не схлопывая** строки в одну, как это делает `GROUP BY`.
    
    - Синтаксис: `ФУНКЦИЯ() OVER (PARTITION BY отдел ORDER BY зарплата DESC)`.
        
    - Популярные функции: `ROW_NUMBER()` (сквозная нумерация 1, 2, 3), `RANK()` (с пропусками при равенстве 1, 1, 3), `DENSE_RANK()` (без пропусков 1, 1, 2), `LAG()` / `LEAD()` (доступ к предыдущей / следующей строке).
        
- **CTE (Common Table Expressions — `WITH`)**: именованные временные результирующие наборы, повышающие читаемость сложных запросов и позволяющие писать рекурсивные запросы (например, обход дерева категорий).
    

### Примеры сложных SQL-запросов

```SQL
-- 1. Найти топ-2 самых высокооплачиваемых сотрудников в каждом отделе (Оконная функция)
WITH RankedEmployees AS (
    SELECT 
        name,
        department_id,
        salary,
        DENSE_RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) as rnk
    FROM employees
)
SELECT name, department_id, salary
FROM RankedEmployees
WHERE rnk <= 2;

-- 2. Найти пользователей, у которых больше 3 оплаченных заказов на общую сумму > 10 000
SELECT 
    user_id,
    COUNT(*) AS orders_cnt,
    SUM(total) AS total_spent
FROM orders
WHERE status = 'paid'             -- фильтруем строки ДО группировки
GROUP BY user_id
HAVING COUNT(*) > 3               -- фильтруем группы ПОСЛЕ группировки
   AND SUM(total) > 10000;

-- 3. Удалить дубликаты email, оставив запись с наименьшим id (PostgreSQL)
DELETE FROM users a
USING users b
WHERE a.id > b.id 
  AND a.email = b.email;
```

## 11. Масштабирование БД: Репликация, Партиционирование, Шардирование

> [!abstract] Коротко
> 
> Когда один сервер БД перестаёт справляться, сначала оптимизируют запросы и добавляют кэш, затем выносят чтение на реплики, и только в крайнем случае делят данные между серверами (шардирование).

### Теория (🇷🇺 Русский & 🇬🇧 English)

- **Вертикальное масштабирование (Scale Up)**: увеличение CPU, RAM и скорости дисков (NVMe) на одном сервере. Самый простой и дешёвый способ до определённого предела.
    
- **Репликация (Primary-Replica / Master-Slave)**:
    
    - Все запросы на запись (`INSERT/UPDATE/DELETE`) идут на **Primary**, а чтение (`SELECT`) распределяется между несколькими **Replica**-серверами.
        
    - _Синхронная репликация_: Primary ждёт подтверждения от реплики перед ответом клиенту (нет потери данных, но выше задержка записи).
        
    - _Асинхронная репликация_: Primary отвечает клиенту сразу, изменения доезжают на реплики с задержкой (**Replication Lag**). _Проблема_: пользователь обновил профиль, страница перезагрузилась, `SELECT` попал на отставшую реплику и показал старые данные (лечится чтением критичных данных с Primary или sticky-сессиями на пару секунд после записи).
        
- **Партиционирование (Partitioning)**: разделение одной гигантской таблицы на множество физических подтаблиц (например, `logs_2026_01`, `logs_2026_02`) **внутри одного сервера БД**. Ускоряет обслуживание (можно мгновенно удалить старый месяц через `DROP TABLE`, не нагружая `VACUUM`/индексы) и поиск по ключу партиционирования (Partition Pruning).
    
- **Шардирование (Sharding)**: горизонтальное разделение данных **по разным независимым серверам (шардам)** на основе ключа шардирования (например, по `tenant_id` или хэшу `user_id % N`).
    
    - _Плюсы_: линейное масштабирование записи и объёма данных.
        
    - _Минусы_: невозможность делать `JOIN` между разными шардами, сложность распределённых транзакций (2PC), болезненная процедура ребалансировки при добавлении новых серверов (решается через **Consistent Hashing**).
        

## 12. Шпаргалка перед собеседованием

> [!important] Ключевые тезисы по Базам данных:
> 
> 1. **Команды SQL**: `DELETE` (DML, построчно, с `WHERE`, не сбрасывает счётчик ID), `TRUNCATE` (DDL, очищает всю таблицу мгновенно, сбрасывает счётчик ID), `DROP` (DDL, удаляет саму таблицу).
>     
> 2. **JOIN и UNION**: `WHERE` по правой таблице превращает `LEFT JOIN` в `INNER JOIN`; `UNION ALL` быстрее `UNION`, так как не тратит ресурсы на удаление дубликатов.
>     
> 3. **ACID и WAL**: Durability обеспечивается последовательной записью в Write-Ahead Log перед обновлением страниц данных.
>     
> 4. **MVCC**: читатели не блокируют писателей; PostgreSQL по умолчанию работает на `Read Committed`, MySQL InnoDB — на `Repeatable Read`.
>     
> 5. **Индексы**: B-Tree работает на `=`, диапазоны и `LIKE 'abc%'`; составной индекс `(a, b)` бесполезен без `a` (правило левого префикса).
>     
> 6. **Конкурентность**: `SELECT ... FOR UPDATE` (пессимистическая блокировка) для высокой конкуренции; `version` (оптимистическая) для редких конфликтов; единый порядок блокировки строк защищает от Deadlock.
>