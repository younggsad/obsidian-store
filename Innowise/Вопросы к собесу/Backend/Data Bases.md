---
custom-width: 80
tags: [databases, sql, interview]
---
---
# Contents

## Part 1. Foundations

- [[#1. What is a database and a DBMS]]
    
- [[#2. Relational databases vs NoSQL]]
    
- [[#3. SQL command groups]]
    
- [[#4. Basic SQL queries, filtering, grouping and NULL]]
    
- [[#5. JOIN, UNION and subqueries]]
    
- [[#6. DELETE vs TRUNCATE vs DROP]]
    
- [[#7. Keys, constraints, relations and views]]
    
- [[#8. Normalization and denormalization]]
    

## Part 2. Core interview topics

- [[#9. Transactions and ACID]]
    
- [[#10. Indexes, B-Tree and index optimization]]
    
- [[#11. Concurrent access, isolation levels and MVCC]]
    
- [[#12. Query optimization and EXPLAIN]]
    
- [[#13. SQL injection and prepared statements]]
    
- [[#14. Window functions and CTE]]
    

## Part 3. Advanced topics

- [[#15. Locks, optimistic vs pessimistic locking and deadlocks]]
    
- [[#16. WAL and crash recovery]]
    
- [[#17. Scaling, replication, partitioning, sharding and CAP]]
    
- [[#18. Final interview cheat sheet]]
    

---

# Part 1. Foundations

---

# 1. What is a database and a DBMS

## Basic concepts

**RU:**

- **Database** — организованное хранилище данных.
    
- **DBMS (Database Management System)** — программное обеспечение для управления базами данных.
    
- **Table** — таблица, содержащая данные об определённом типе объектов.
    
- **Column** — столбец, описывающий определённое свойство объекта.
    
- **Row** — строка, представляющая одну запись.
    
- **Primary Key** — уникальный идентификатор строки.
    
- **Foreign Key** — ссылка на запись в другой или той же таблице.
    
- **Schema** — структура базы данных: таблицы, столбцы, типы и ограничения.
    
- **Query** — запрос к базе данных.
    

**EN:**

A database is an organized collection of data. A DBMS (Database Management System) is software that allows applications to store, retrieve, update, and manage this data.

A table stores information about a particular type of entity. A column represents an attribute, while a row represents one record.

A primary key uniquely identifies each row. A foreign key establishes a relationship between tables by referencing a candidate key, usually a primary key, in another table.

A schema defines the structure of the database, including tables, columns, data types, and constraints.

## Database vs DBMS

**RU:**

База данных — это сами данные, а DBMS — программное обеспечение, которое ими управляет. Например, PostgreSQL является DBMS, а конкретная база данных внутри PostgreSQL содержит таблицы и записи приложения.

**EN:**

A database contains the actual data, while a DBMS is the software responsible for managing it. PostgreSQL is a DBMS, and a single PostgreSQL server can manage multiple databases.

## How an application communicates with a database

**RU:**

Приложение устанавливает соединение с сервером базы данных, отправляет SQL-запрос и получает результат. Соединения имеют определённую стоимость, поэтому приложения обычно используют connection pooling для повторного использования соединений.

Стандартные порты: PostgreSQL — 5432, MySQL — 3306.

**EN:**

An application connects to a database server, sends SQL queries, and receives results. Establishing database connections has overhead, so production applications commonly use connection pooling to reuse existing connections.

The default ports are 5432 for PostgreSQL and 3306 for MySQL.

## Example

```sql
CREATE TABLE users (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE
);

INSERT INTO users (name, email)
VALUES ('Anna', 'anna@example.com');

SELECT name, email
FROM users
WHERE id = 1;
```

## Interview questions

**Q: What is the difference between a database and a DBMS?**

**RU:** База данных содержит информацию, а DBMS предоставляет инструменты для её хранения, изменения, поиска и защиты.

**EN:** A database stores information, while a DBMS provides the functionality to store, retrieve, modify, and protect that information.

**Q: What is a primary key?**

**RU:** Primary Key — это столбец или комбинация столбцов, которые уникально идентифицируют каждую запись. Значения не могут повторяться или быть NULL.

**EN:** A primary key is a column or a combination of columns that uniquely identifies every row in a table. It must contain unique, non-NULL values. A table can have only one primary key constraint, although that key may consist of multiple columns.

---

# 2. Relational databases vs NoSQL

## Main idea

**RU:**

Реляционные базы данных хранят информацию в таблицах и используют связи между ними. NoSQL — это группа нереляционных моделей хранения, которые подходят для определённых сценариев, например кэширования, поиска, распределённого хранения и аналитики.

**EN:**

Relational databases store information in tables and use relationships between them. NoSQL is a broad category of non-relational database systems designed for different use cases, such as caching, document storage, distributed data processing, and analytics.

## Relational databases

**RU:**

PostgreSQL и MySQL используют реляционную модель данных. Они поддерживают SQL, JOIN, ограничения целостности, внешние ключи и транзакции.

Их обычно выбирают для бизнес-приложений, где важны согласованность данных и сложные связи: платежи, заказы, пользователи, CRM и интернет-магазины.

**EN:**

Relational databases such as PostgreSQL and MySQL organize data into tables with a predefined schema. They support SQL, JOIN operations, constraints, foreign keys, and transactions.

They are commonly used in business applications where data integrity and relationships are important, such as payment systems, e-commerce platforms, and CRM applications.

## NoSQL database types

**RU:**

Основные категории NoSQL:

- **Key-Value:** Redis. Хранение данных по ключу. Используется для кэширования, сессий, счётчиков и временных данных.
    
- **Document:** MongoDB. Хранение JSON-подобных документов с гибкой структурой.
    
- **Wide-column:** Cassandra. Распределённое хранение больших объёмов данных с доступом по ключам и заранее спроектированным запросам.
    
- **Search engines:** Elasticsearch и OpenSearch. Полнотекстовый поиск, логирование и аналитика.
    
- **Graph databases:** Neo4j. Хранение сложных связей между сущностями.
    

**EN:**

The main NoSQL database categories are:

- **Key-Value stores:** Redis stores values using unique keys. It is commonly used for caching, sessions, and counters.
    
- **Document databases:** MongoDB stores JSON-like documents with flexible structures.
    
- **Wide-column databases:** Cassandra is designed for distributed storage and high write throughput.
    
- **Search engines:** Elasticsearch and OpenSearch provide full-text search and log analytics.
    
- **Graph databases:** Neo4j stores entities and their relationships, making it useful for recommendation systems and social networks.
    

## OLTP vs OLAP

**RU:**

- **OLTP (Online Transaction Processing)** — большое количество небольших операций: создание заказов, платежи, обновление пользователей.
    
- **OLAP (Online Analytical Processing)** — аналитические запросы, обработка больших объёмов данных и построение отчётов.
    

Реляционные row-oriented базы часто используются для OLTP, а columnar-системы, например ClickHouse, хорошо подходят для OLAP.

**EN:**

OLTP systems handle many short transactions, such as creating orders, processing payments, and updating user information.

OLAP systems are designed for analytical queries over large datasets, such as generating reports and calculating business metrics.

Row-oriented databases are commonly used for OLTP, while columnar databases such as ClickHouse are often preferred for analytical workloads.

## Polyglot persistence

**RU:**

Polyglot persistence означает использование нескольких типов хранилищ в одном приложении. Например, PostgreSQL хранит основные данные, Redis используется для кэширования, а Elasticsearch обеспечивает полнотекстовый поиск.

**EN:**

Polyglot persistence means using different database technologies within the same application. For example, PostgreSQL can store the primary business data, Redis can handle caching, and Elasticsearch can provide full-text search.

## Interview questions

**Q: Why not store everything in MongoDB?**

**RU:** Гибкая структура не всегда является преимуществом. Если приложению нужны сложные JOIN, транзакции, ограничения целостности и строгие связи между сущностями, реляционная база часто будет более подходящим решением.

**EN:** A flexible schema is not always an advantage. Applications that require complex joins, relational constraints, and strong transactional guarantees often benefit more from relational databases.

**Q: When would you use Redis instead of PostgreSQL?**

**RU:** Redis подходит для временных данных, кэширования, сессий и счётчиков. PostgreSQL лучше использовать как основное хранилище бизнес-данных.

**EN:** I would use Redis for caching, sessions, counters, and temporary data. PostgreSQL would be my primary database for persistent business data that requires relational queries and strong integrity guarantees.

---

# 3. SQL command groups

## Main idea

**RU:**

SQL-команды традиционно разделяют на пять групп в зависимости от их назначения: управление структурой, изменение данных, чтение данных, управление разрешениями и управление транзакциями.

**EN:**

SQL commands are traditionally divided into five groups based on their purpose: defining database structures, manipulating data, querying data, controlling permissions, and managing transactions.

## DDL — Data Definition Language

**RU:**

DDL отвечает за создание и изменение структуры объектов базы данных.

Основные команды:

- CREATE
    
- ALTER
    
- DROP
    
- TRUNCATE
    

**EN:**

DDL is used to define and modify database structures, such as tables, indexes, and schemas.

Common commands include CREATE, ALTER, DROP, and TRUNCATE.

## DML — Data Manipulation Language

**RU:**

DML отвечает за изменение данных внутри таблиц.

Команды: INSERT, UPDATE, DELETE, MERGE.

**EN:**

DML is used to insert, update, and delete records in database tables.

Common commands include INSERT, UPDATE, DELETE, and MERGE.

## DQL — Data Query Language

**RU:**

DQL отвечает за получение информации из базы данных. Основная команда — SELECT.

**EN:**

DQL is used to retrieve information from database tables. The main command is SELECT.

## DCL — Data Control Language

**RU:**

DCL управляет разрешениями пользователей и ролей.

Команды: GRANT и REVOKE.

**EN:**

DCL manages database access permissions through commands such as GRANT and REVOKE.

## TCL — Transaction Control Language

**RU:**

TCL управляет транзакциями.

Команды: BEGIN, COMMIT, ROLLBACK, SAVEPOINT.

**EN:**

TCL controls database transactions using commands such as BEGIN, COMMIT, ROLLBACK, and SAVEPOINT.

## Important difference: PostgreSQL vs MySQL

**RU:**

В PostgreSQL большинство DDL-операций можно выполнять внутри транзакций и откатывать через ROLLBACK.

В MySQL многие DDL-команды вызывают implicit commit, поэтому их нельзя откатить обычным ROLLBACK.

**EN:**

PostgreSQL supports transactional DDL for most common schema operations, which means that many DDL changes can be rolled back.

MySQL performs implicit commits for many DDL statements, so those operations generally cannot be rolled back using ROLLBACK.

## Interview questions

**Q: Which SQL group does TRUNCATE belong to?**

**RU:** Традиционно TRUNCATE относится к DDL, потому что удаляет данные на уровне структуры таблицы, а не отдельными строками.

**EN:** TRUNCATE is traditionally classified as DDL because it removes all rows by operating on the table as a whole rather than deleting individual rows.

**Q: What is the difference between GRANT and REVOKE?**

**RU:** GRANT предоставляет права доступа, а REVOKE отзывает ранее предоставленные права.

**EN:** GRANT assigns database privileges, while REVOKE removes previously granted privileges.

---

# 4. Basic SQL queries, filtering, grouping and NULL

## Main idea

**RU:**

Базовые SQL-запросы строятся вокруг SELECT, FROM, WHERE, ORDER BY и LIMIT. Важно понимать разницу между фильтрацией строк, группировкой данных и фильтрацией агрегированных результатов.

**EN:**

Basic SQL queries are built around SELECT, FROM, WHERE, ORDER BY, and LIMIT. It is important to understand the difference between filtering individual rows, grouping records, and filtering aggregated results.

## SELECT and WHERE

**RU:**

SELECT определяет, какие столбцы необходимо вернуть. WHERE фильтрует строки по заданным условиям.

**EN:**

SELECT specifies which columns should be returned, while WHERE filters individual rows based on specified conditions.

```sql
SELECT name, salary
FROM employees
WHERE salary > 3000;
```

## Common operators

|Operator|Meaning|
|---|---|
|=|Equal|
|<>|Not equal|
|>, <|Greater / less|
|>=, <=|Greater or equal / less or equal|
|AND|Both conditions|
|OR|At least one condition|
|NOT|Negation|
|IN|Match a list|
|BETWEEN|Match a range|
|LIKE|Pattern matching|
|IS NULL|Check for NULL|

**RU:**

LIKE использует `%` для произвольного количества символов и `_` для одного символа.

**EN:**

LIKE uses `%` to match any number of characters and `_` to match exactly one character.

```sql
SELECT *
FROM employees
WHERE name LIKE 'A%';
```

## ORDER BY, LIMIT and DISTINCT

**RU:**

ORDER BY сортирует результат. LIMIT ограничивает количество возвращаемых строк. DISTINCT удаляет повторяющиеся комбинации выбранных столбцов.

**EN:**

ORDER BY sorts the result set. LIMIT restricts the number of returned rows, while DISTINCT removes duplicate combinations of selected columns.

```sql
SELECT DISTINCT department
FROM employees
ORDER BY department ASC
LIMIT 10;
```

## NULL

**RU:**

NULL означает неизвестное или отсутствующее значение. NULL не равен нулю, пустой строке или FALSE.

SQL использует трёхзначную логику: TRUE, FALSE и UNKNOWN.

Поэтому NULL = NULL возвращает UNKNOWN, а не TRUE. Для проверки NULL нужно использовать IS NULL или IS NOT NULL.

**EN:**

NULL represents a missing or unknown value. It is not the same as zero, an empty string, or FALSE.

SQL uses three-valued logic: TRUE, FALSE, and UNKNOWN.

Therefore, NULL = NULL evaluates to UNKNOWN rather than TRUE. We should use IS NULL or IS NOT NULL to check for NULL values.

```sql
SELECT *
FROM employees
WHERE department_id IS NULL;
```

## COALESCE

**RU:**

COALESCE возвращает первое значение, которое не является NULL. Часто используется для установки значения по умолчанию.

**EN:**

COALESCE returns the first non-NULL expression from its arguments. It is commonly used to provide default values.

```sql
SELECT
    name,
    COALESCE(department, 'Not assigned')
FROM employees;
```

## Aggregate functions

**RU:**

Агрегатные функции вычисляют одно значение на основе набора строк.

- COUNT — количество.
    
- SUM — сумма.
    
- AVG — среднее значение.
    
- MIN — минимальное значение.
    
- MAX — максимальное значение.
    

COUNT(*) считает все строки, а COUNT(column) игнорирует NULL в указанном столбце.

**EN:**

Aggregate functions calculate a single result from multiple rows.

COUNT counts rows or non-NULL values, SUM calculates a total, AVG calculates an average, and MIN and MAX return the smallest and largest values.

COUNT(*) counts all rows, while COUNT(column) ignores NULL values in that column.

## GROUP BY

**RU:**

GROUP BY объединяет строки с одинаковыми значениями указанных столбцов в группы. После этого агрегатные функции вычисляют результат для каждой группы.

Каждый столбец в SELECT должен быть сгруппирован или использоваться внутри агрегатной функции, за исключением случаев, когда СУБД может доказать функциональную зависимость.

**EN:**

GROUP BY groups rows that have the same values in specified columns. Aggregate functions then calculate results for each group.

Every selected column must either appear in GROUP BY or be used inside an aggregate function, unless the database can establish a functional dependency.

```sql
SELECT department, AVG(salary)
FROM employees
GROUP BY department;
```

## WHERE vs HAVING

**RU:**

WHERE фильтрует отдельные строки до группировки. HAVING фильтрует уже сформированные группы после GROUP BY.

WHERE не может непосредственно использовать агрегатные функции для фильтрации строк, а HAVING может использовать COUNT, SUM и AVG.

**EN:**

WHERE filters individual rows before grouping and aggregation. HAVING filters groups after GROUP BY has been applied.

WHERE cannot directly filter using aggregate functions, while HAVING can use functions such as COUNT, SUM, and AVG.

```sql
SELECT department, AVG(salary)
FROM employees
WHERE salary > 3000
GROUP BY department
HAVING AVG(salary) > 5000;
```

## Logical execution order

**RU:**

Логический порядок обработки SQL-запроса отличается от порядка написания команд.

1. FROM
    
2. JOIN / ON
    
3. WHERE
    
4. GROUP BY
    
5. HAVING
    
6. SELECT
    
7. DISTINCT
    
8. ORDER BY
    
9. LIMIT / OFFSET
    

Поэтому алиас из SELECT обычно нельзя использовать в WHERE: на логическом этапе WHERE он ещё не сформирован.

**EN:**

The logical execution order of a SQL query differs from the order in which its clauses are written.

The typical logical order is FROM, JOIN, WHERE, GROUP BY, HAVING, SELECT, DISTINCT, ORDER BY, and finally LIMIT/OFFSET.

This explains why a SELECT alias generally cannot be referenced in WHERE: the SELECT expressions have not been evaluated at that logical stage.

## INSERT, UPDATE and DELETE

**RU:**

INSERT добавляет новые строки, UPDATE изменяет существующие, а DELETE удаляет записи.

Перед массовым UPDATE или DELETE полезно выполнить SELECT с тем же условием, чтобы проверить затрагиваемые строки. Для важных операций желательно использовать транзакцию.

**EN:**

INSERT adds new rows, UPDATE modifies existing records, and DELETE removes rows.

Before performing a bulk UPDATE or DELETE, it is good practice to run a SELECT query with the same WHERE condition to verify which records will be affected. Important operations should be protected by transactions.

## UPSERT

**RU:**

UPSERT — это операция, которая вставляет запись, если её ещё нет, либо обновляет существующую запись при конфликте уникальности.

В PostgreSQL используется INSERT ... ON CONFLICT.

**EN:**

UPSERT is an operation that inserts a new row or updates an existing row when a uniqueness conflict occurs.

PostgreSQL provides this functionality through INSERT ... ON CONFLICT.

```sql
INSERT INTO user_stats (user_id, logins_count)
VALUES (42, 1)
ON CONFLICT (user_id)
DO UPDATE
SET logins_count = user_stats.logins_count + 1;
```

## Interview questions

**Q: What is the difference between WHERE and HAVING?**

**EN:** WHERE filters individual rows before grouping, while HAVING filters groups after aggregation.

**Q: What is the difference between COUNT(*) and COUNT(column)?**

**EN:** COUNT(*) counts all rows, while COUNT(column) counts only rows where the specified column is not NULL.

**Q: Why does NULL = NULL not return TRUE?**

**EN:** Because NULL represents an unknown value. Comparisons involving NULL generally return UNKNOWN, so we must use IS NULL or IS NOT NULL.

**Q: What does COALESCE do?**

**EN:** COALESCE returns the first non-NULL value from a list of expressions.

**Q: Why can't we use a SELECT alias in WHERE?**

**EN:** Because WHERE is logically evaluated before SELECT, so the alias is not available at that stage.

---

# 5. JOIN, UNION and subqueries

## Main idea

**RU:**

JOIN объединяет таблицы горизонтально, добавляя столбцы из связанных таблиц. UNION объединяет результаты запросов вертикально, добавляя строки.

**EN:**

JOIN combines tables horizontally by adding columns from related tables. UNION combines query results vertically by adding rows from multiple SELECT statements.

## INNER JOIN

**RU:**

INNER JOIN возвращает только строки, для которых найдено совпадение в обеих таблицах. Если совпадения нет, строка не попадает в результат.

**EN:**

INNER JOIN returns only rows that have matching records in both tables. Rows without a matching record are excluded from the result.

```sql
SELECT u.name, o.total
FROM users u
INNER JOIN orders o
    ON o.user_id = u.id;
```

## LEFT JOIN

**RU:**

LEFT JOIN возвращает все строки левой таблицы и совпадающие строки правой таблицы. Если совпадения нет, столбцы правой таблицы содержат NULL.

**EN:**

LEFT JOIN returns every row from the left table and matching rows from the right table. If there is no match, the columns from the right table contain NULL.

## RIGHT JOIN

**RU:**

RIGHT JOIN работает аналогично LEFT JOIN, но сохраняет все строки правой таблицы. На практике часто переписывается через LEFT JOIN для читаемости.

**EN:**

RIGHT JOIN returns all rows from the right table and matching rows from the left table. In practice, it can often be rewritten using LEFT JOIN for readability.

## FULL OUTER JOIN

**RU:**

FULL OUTER JOIN возвращает все строки обеих таблиц. Если совпадения нет, недостающие столбцы заполняются NULL.

В PostgreSQL поддерживается напрямую. В MySQL нет встроенного FULL OUTER JOIN.

**EN:**

FULL OUTER JOIN returns all rows from both tables, including unmatched records. Missing values from the other table are represented by NULL.

PostgreSQL supports FULL OUTER JOIN directly, while MySQL requires an alternative implementation.

## CROSS JOIN

**RU:**

CROSS JOIN возвращает декартово произведение двух таблиц. Если первая таблица содержит N строк, а вторая M строк, результат содержит N × M строк.

**EN:**

CROSS JOIN produces the Cartesian product of two tables. If the first table contains N rows and the second contains M rows, the result contains N × M rows.

## SELF JOIN

**RU:**

SELF JOIN — соединение таблицы с самой собой. Используется, например, для получения сотрудника и его руководителя из одной таблицы.

**EN:**

A SELF JOIN joins a table with itself. It is commonly used to retrieve hierarchical relationships, such as employees and their managers.

```sql
SELECT
    e.name AS employee,
    m.name AS manager
FROM employees e
LEFT JOIN employees m
    ON e.manager_id = m.id;
```

## Anti-join

**RU:**

Anti-join — техника поиска записей, у которых отсутствует связанная запись. Обычно реализуется через LEFT JOIN с проверкой IS NULL или через NOT EXISTS.

**EN:**

An anti-join is a technique for finding rows that do not have matching records in another table. It is commonly implemented using LEFT JOIN with IS NULL or NOT EXISTS.

```sql
SELECT u.id, u.email
FROM users u
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

## LEFT JOIN trap

**RU:**

Если поместить условие на правую таблицу в WHERE, можно случайно превратить LEFT JOIN в INNER JOIN. Строки без совпадений будут исключены, потому что условие на NULL не даст TRUE.

Если нужно сохранить все строки левой таблицы, условие на правую таблицу часто следует помещать в ON.

**EN:**

Placing a condition on the right-hand table in the WHERE clause can unintentionally turn a LEFT JOIN into an INNER JOIN. Unmatched rows are removed because comparisons with NULL do not evaluate to TRUE.

If we want to preserve all rows from the left table, the condition should often be placed in the ON clause.

```sql
-- Incorrect if we want to keep users without paid orders
SELECT u.id, o.total
FROM users u
LEFT JOIN orders o
    ON o.user_id = u.id
WHERE o.status = 'paid';

-- Keeps all users
SELECT u.id, o.total
FROM users u
LEFT JOIN orders o
    ON o.user_id = u.id
   AND o.status = 'paid';
```

## UNION vs UNION ALL

**RU:**

Обе команды объединяют результаты нескольких SELECT-запросов. Количество столбцов должно совпадать, а соответствующие типы данных должны быть совместимы.

UNION удаляет дубликаты. Для этого СУБД выполняет дополнительную обработку, например сортировку или hashing.

UNION ALL сохраняет все строки, включая дубликаты, поэтому обычно работает быстрее.

**EN:**

Both UNION and UNION ALL combine the results of multiple SELECT statements. The queries must return the same number of columns with compatible data types.

UNION removes duplicate rows, which requires additional processing such as sorting or hashing.

UNION ALL preserves all rows, including duplicates, and is generally faster because it does not perform duplicate elimination.

## Subqueries

**RU:**

Subquery — это запрос, вложенный внутрь другого SQL-запроса.

Основные виды:

- Scalar subquery — возвращает одно значение.
    
- Row subquery — возвращает одну строку.
    
- Table subquery — возвращает набор строк.
    
- Correlated subquery — использует значения из внешнего запроса.
    
- Non-correlated subquery — не зависит от внешнего запроса.
    

**EN:**

A subquery is a query nested inside another SQL query.

A scalar subquery returns a single value. A correlated subquery references columns from the outer query, while a non-correlated subquery can be evaluated independently.

A correlated subquery logically depends on the current outer row. The optimizer may transform it into a more efficient execution plan, so it does not necessarily mean that the inner query is physically executed once for every row.

## Example: non-correlated subquery

```sql
SELECT name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

## Example: correlated subquery

```sql
SELECT e.name, e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
);
```

## EXISTS and IN

**RU:**

EXISTS проверяет, возвращает ли подзапрос хотя бы одну строку. Часто применяется для проверки наличия связанных записей.

IN проверяет, входит ли значение в набор результатов.

NOT IN может дать неожиданный результат, если подзапрос возвращает NULL. Для проверки отсутствия связанных записей часто безопаснее использовать NOT EXISTS.

**EN:**

EXISTS checks whether a subquery returns at least one row. It is commonly used to check whether related records exist.

IN checks whether a value belongs to a set of returned values.

NOT IN can produce unexpected results when the subquery contains NULL values. NOT EXISTS is often safer for checking the absence of matching records.

## Interview questions

**Q: What is the difference between INNER JOIN and LEFT JOIN?**

**EN:** INNER JOIN returns only matching rows from both tables, while LEFT JOIN preserves every row from the left table and returns NULL for unmatched right-hand records.

**Q: What is the difference between UNION and UNION ALL?**

**EN:** UNION removes duplicates, while UNION ALL preserves them. UNION ALL is generally faster because it does not need duplicate elimination.

**Q: What is a correlated subquery?**

**EN:** A correlated subquery references columns from the outer query and logically depends on the current outer row.

**Q: What is the difference between EXISTS and IN?**

**EN:** EXISTS checks whether a subquery returns at least one row, while IN checks whether a value matches one of the values returned by a subquery.

---

# 6. DELETE vs TRUNCATE vs DROP

## Main idea

**RU:**

DELETE удаляет выбранные строки, TRUNCATE очищает всю таблицу, сохраняя её структуру, а DROP удаляет саму таблицу вместе с её структурой.

**EN:**

DELETE removes selected rows, TRUNCATE removes all rows while preserving the table structure, and DROP removes the entire table, including its structure.

## DELETE

**RU:**

DELETE удаляет строки с использованием WHERE или без него. PostgreSQL обрабатывает удаление через MVCC: строки становятся невидимыми для новых снимков данных, а физическая очистка происходит позднее через VACUUM.

DELETE записывает изменения в WAL и поддерживает транзакционный откат.

**EN:**

DELETE removes rows using an optional WHERE condition. In PostgreSQL, deleted rows become invisible to new transaction snapshots because of MVCC. Their physical space is reclaimed later by VACUUM.

DELETE generates WAL records and can be rolled back before the transaction is committed.

## TRUNCATE

**RU:**

TRUNCATE быстро удаляет все строки таблицы без индивидуальной обработки каждой записи. Структура таблицы остаётся.

В PostgreSQL TRUNCATE транзакционный и может быть отменён через ROLLBACK. Он не удаляет каждую строку по отдельности и не вызывает обычные row-level DELETE triggers. PostgreSQL поддерживает отдельные ON TRUNCATE triggers.

В PostgreSQL для сброса identity используется RESTART IDENTITY.

**EN:**

TRUNCATE removes all rows from a table without processing every row individually. The table structure remains unchanged.

In PostgreSQL, TRUNCATE is transactional and can be rolled back. It does not fire regular row-level DELETE triggers, but PostgreSQL supports separate ON TRUNCATE triggers.

The RESTART IDENTITY option resets associated identity sequences.

## DROP

**RU:**

DROP TABLE удаляет таблицу вместе с данными, индексами, ограничениями и другими объектами, принадлежащими таблице.

В PostgreSQL операцию можно откатить внутри транзакции. В MySQL DROP TABLE обычно вызывает implicit commit.

**EN:**

DROP TABLE removes the table itself, including its data, indexes, constraints, and associated table objects.

In PostgreSQL, it can be rolled back inside a transaction. In MySQL, DROP TABLE normally causes an implicit commit.

## Comparison

|Feature|DELETE|TRUNCATE|DROP|
|---|---|---|---|
|Removes rows|Yes|All rows|Yes|
|Removes structure|No|No|Yes|
|WHERE|Yes|No|No|
|Row-level DELETE triggers|Yes|No|No|
|PostgreSQL rollback|Yes|Yes|Yes|
|MySQL rollback|Usually yes|No|No|
|Typical performance|Slower for full-table deletion|Usually faster|Removes object|

**RU:**

TRUNCATE обычно быстрее DELETE без WHERE, потому что не выполняет индивидуальное удаление каждой строки. Однако точная стоимость зависит от СУБД, блокировок, внешних ключей и размера таблицы.

**EN:**

TRUNCATE is generally faster than DELETE without WHERE because it avoids processing every row individually. However, the exact performance depends on the database engine, locks, foreign-key dependencies, and table size.

## Interview questions

**Q: Why is TRUNCATE faster than DELETE?**

**EN:** TRUNCATE operates on the table as a whole instead of deleting rows individually. This reduces the amount of row-level processing and logging.

**Q: Can TRUNCATE be rolled back?**

**EN:** In PostgreSQL, TRUNCATE is transactional and can be rolled back. In MySQL, TRUNCATE causes an implicit commit and cannot normally be rolled back.

**Q: Which command would you use to clear a table?**

**EN:** If I need to remove all rows but keep the table structure, I would use TRUNCATE. If I need to remove only specific rows, I would use DELETE.

---

# 7. Keys, constraints, relations and views

## Constraints

**RU:**

Constraints — это правила целостности данных, которые проверяются самой базой данных. Они позволяют защитить данные независимо от того, какое приложение выполняет запрос.

**EN:**

Constraints are rules enforced by the database to maintain data integrity. They protect the data regardless of which application or service performs the operation.

## PRIMARY KEY

**RU:**

PRIMARY KEY уникально идентифицирует каждую запись. Он объединяет свойства UNIQUE и NOT NULL.

На таблицу можно определить только один PRIMARY KEY constraint, но он может включать несколько столбцов.

**EN:**

A PRIMARY KEY uniquely identifies every row in a table. It combines uniqueness and non-NULL requirements.

A table can have only one primary key constraint, but that key can consist of multiple columns.

## FOREIGN KEY

**RU:**

FOREIGN KEY обеспечивает ссылочную целостность между таблицами. Он гарантирует, что значение внешнего ключа соответствует существующей записи в связанной таблице, если значение не NULL и ограничение не отключено.

**EN:**

A FOREIGN KEY enforces referential integrity between tables. It ensures that a non-NULL foreign key value references an existing record in the related table.

## UNIQUE

**RU:**

UNIQUE запрещает повторение значений. В PostgreSQL обычный UNIQUE допускает несколько NULL, потому что NULL по умолчанию считается отличным от другого NULL при проверке уникальности.

Для альтернативного поведения PostgreSQL поддерживает UNIQUE NULLS NOT DISTINCT.

**EN:**

UNIQUE prevents duplicate values in a column or combination of columns. In PostgreSQL, a standard UNIQUE constraint allows multiple NULL values because NULL values are considered distinct by default for uniqueness checks.

PostgreSQL also supports UNIQUE NULLS NOT DISTINCT when NULL values should be treated as equal for uniqueness.

## NOT NULL

**RU:**

NOT NULL запрещает хранение NULL в столбце.

**EN:**

NOT NULL prevents a column from containing NULL values.

## CHECK

**RU:**

CHECK проверяет заданное условие для каждой записи.

**EN:**

CHECK enforces a condition that must be satisfied by every inserted or updated row.

## DEFAULT

**RU:**

DEFAULT устанавливает значение по умолчанию, если при вставке значение столбца не указано.

**EN:**

DEFAULT provides a value when an INSERT statement does not explicitly specify one for the column.

```sql
CREATE TABLE orders (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    user_id BIGINT NOT NULL REFERENCES users(id),
    total NUMERIC(10,2) NOT NULL CHECK (total >= 0),
    status VARCHAR(20) NOT NULL DEFAULT 'new',
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

## Relationships

**RU:**

Существует три основных типа отношений между сущностями.

- One-to-one: одна запись связана максимум с одной записью другой таблицы.
    
- One-to-many: одна запись может иметь несколько связанных записей.
    
- Many-to-many: несколько записей первой таблицы связаны с несколькими записями второй.
    

Для many-to-many используется промежуточная таблица с двумя внешними ключами.

**EN:**

There are three common relationship types:

- One-to-one: one record is associated with at most one record in another table.
    
- One-to-many: one record can be associated with multiple records.
    
- Many-to-many: multiple records from one table can be associated with multiple records from another table.
    

Many-to-many relationships are implemented using a junction table containing foreign keys to both related tables.

```sql
CREATE TABLE user_roles (
    user_id BIGINT REFERENCES users(id),
    role_id BIGINT REFERENCES roles(id),
    PRIMARY KEY (user_id, role_id)
);
```

## Views

**RU:**

View — сохранённый SQL-запрос, который можно использовать как виртуальную таблицу. Обычный view не хранит результат запроса, а вычисляет его при обращении.

Materialized View хранит результат запроса физически. Это ускоряет чтение, но данные необходимо периодически обновлять.

**EN:**

A view is a saved SQL query that can be used like a virtual table. A regular view does not store the query result; it evaluates the underlying query when accessed.

A materialized view physically stores the query result. This can improve read performance, but the data must be refreshed to reflect changes in the underlying tables.

```sql
CREATE VIEW active_users AS
SELECT id, email
FROM users
WHERE is_active = TRUE;
```

PostgreSQL:

```sql
REFRESH MATERIALIZED VIEW active_users_summary;
```

## Interview questions

**Q: What is the difference between PRIMARY KEY and UNIQUE?**

**EN:** A primary key uniquely identifies each row and cannot contain NULL values. A table can have only one primary key constraint but multiple UNIQUE constraints.

**Q: How do you implement a many-to-many relationship?**

**EN:** I create a junction table containing foreign keys to both tables. I usually add a composite primary key or unique constraint to prevent duplicate relationships.

**Q: What is the difference between a view and a materialized view?**

**EN:** A regular view stores a query definition, while a materialized view stores the result of that query. Materialized views can improve read performance but require refreshing.

---

# 8. Normalization and denormalization

## Main idea

**RU:**

Нормализация уменьшает дублирование данных и предотвращает аномалии вставки, обновления и удаления. Денормализация намеренно добавляет избыточность, когда это необходимо для улучшения производительности чтения.

**EN:**

Normalization reduces data redundancy and prevents insertion, update, and deletion anomalies. Denormalization intentionally introduces redundancy when it is necessary to improve read performance.

## Problems normalization solves

**RU:**

Представим таблицу заказов, в которой данные клиента повторяются в каждой записи.

|order_id|customer_name|customer_city|
|---|---|---|
|1|Anna|Warsaw|
|2|Anna|Warsaw|
|3|Piotr|Krakow|

Это создаёт три проблемы:

- Update anomaly: изменение города клиента требует обновить несколько записей.
    
- Insert anomaly: невозможно сохранить клиента без заказа, если данные клиента существуют только в таблице заказов.
    
- Delete anomaly: удаление последнего заказа может привести к потере информации о клиенте.
    

Решение — разделить клиентов и заказы на отдельные таблицы.

**EN:**

Consider an orders table where customer information is repeated in every order.

This design creates three types of anomalies:

- Update anomaly: changing a customer's city requires updating multiple records.
    
- Insert anomaly: we cannot store a customer independently if customer information exists only in the orders table.
    
- Delete anomaly: deleting the customer's last order may also remove the only copy of their information.
    

The solution is to separate customers and orders into different tables.

## First Normal Form (1NF)

**RU:**

1NF требует, чтобы значения атрибутов соответствовали атомарной структуре выбранной реляционной модели. Также не должно быть повторяющихся групп столбцов вроде phone1, phone2, phone3.

**EN:**

First Normal Form requires attributes to contain atomic values according to the chosen relational model. Tables should not contain repeating groups of columns such as phone1, phone2, and phone3.

## Second Normal Form (2NF)

**RU:**

2NF требует соблюдения 1NF и отсутствия частичных зависимостей неключевых атрибутов от составного кандидатного ключа.

Если первичный ключ состоит из student_id и course_id, student_name не должен зависеть только от student_id внутри этой таблицы.

**EN:**

Second Normal Form requires a table to satisfy 1NF and ensures that every non-key attribute depends on the entire candidate key rather than only part of a composite key.

For example, if the primary key consists of student_id and course_id, student_name should not depend only on student_id in that table.

## Third Normal Form (3NF)

**RU:**

3NF требует соблюдения 2NF и отсутствия недопустимых транзитивных зависимостей неключевых атрибутов от кандидатных ключей.

Например, если city_id определяет city_name, то city_name лучше хранить в отдельной таблице cities, а не дублировать в orders.

**EN:**

Third Normal Form requires a table to satisfy 2NF and eliminates problematic transitive dependencies of non-key attributes on candidate keys.

For example, if city_id determines city_name, city_name should generally be stored in a separate cities table instead of being repeated in orders.

## BCNF

**RU:**

BCNF — более строгая форма нормализации, чем 3NF. Для каждой нетривиальной функциональной зависимости детерминант должен быть суперключом.

**EN:**

Boyce-Codd Normal Form is stricter than 3NF. For every non-trivial functional dependency, the determinant must be a superkey.

## Denormalization

**RU:**

Денормализация намеренно дублирует данные для ускорения чтения. Например, можно хранить comments_count непосредственно в posts вместо постоянного COUNT по таблице comments.

Но при изменении комментариев необходимо корректно обновлять счётчик. Это усложняет запись и создаёт риск рассинхронизации.

**EN:**

Denormalization intentionally duplicates data to improve read performance. For example, we may store comments_count directly in the posts table instead of calculating COUNT(*) from the comments table every time.

However, the counter must be updated correctly whenever comments change. This makes writes more complex and introduces the risk of inconsistent data.

## Primary key choices

**RU:**

- BIGINT IDENTITY — компактный, простой и удобный для обычных приложений.
    
- UUID v4 — случайный идентификатор, который можно генерировать вне базы, но случайное расположение ключей может увеличить количество page splits.
    
- UUID v7 — временно упорядоченный UUID, который лучше подходит для локальности вставок в B-Tree.
    

UUID не гарантирует безопасность API сам по себе. Защита ресурсов всё равно должна строиться на проверке авторизации.

**EN:**

BIGINT IDENTITY is compact and efficient for traditional applications.

UUID v4 provides globally unique random identifiers that can be generated outside the database, but random insertion patterns may cause additional B-Tree page splits.

UUID v7 contains time-ordered information, which generally provides better insertion locality than UUID v4.

UUIDs should not be treated as an authorization mechanism. Access control must still be enforced by the application.

## Money and numeric types

**RU:**

Для денежных значений лучше использовать NUMERIC/DECIMAL или BIGINT в минимальных денежных единицах, например в центах.

FLOAT и DOUBLE PRECISION используют двоичное представление с плавающей точкой и могут давать ошибки округления.

**EN:**

For monetary values, NUMERIC/DECIMAL or BIGINT storing the smallest currency unit is generally preferable.

FLOAT and DOUBLE PRECISION use binary floating-point representation and may introduce rounding errors.

## Soft delete

**RU:**

Soft delete — это логическое удаление, обычно реализованное через deleted_at. Запись физически остаётся в базе, но приложение считает её удалённой.

В PostgreSQL можно использовать partial unique indexes для ограничения уникальности только среди активных записей.

**EN:**

Soft deletion means marking a record as deleted instead of physically removing it, usually through a deleted_at column.

In PostgreSQL, partial unique indexes can enforce uniqueness only among active records.

```sql
CREATE UNIQUE INDEX unique_active_email
ON users(email)
WHERE deleted_at IS NULL;
```

## Interview questions

**Q: What is normalization?**

**EN:** Normalization is the process of organizing relational data to reduce redundancy and prevent insertion, update, and deletion anomalies.

**Q: What is the difference between 2NF and 3NF?**

**EN:** 2NF eliminates partial dependencies on composite keys, while 3NF eliminates problematic transitive dependencies between non-key attributes.

**Q: When would you denormalize a database?**

**EN:** I would denormalize only after identifying a real performance bottleneck. For example, I might store a precomputed counter to avoid an expensive aggregation on a frequently accessed page.

**Q: Why store the product price in order_items?**

**EN:** Because product prices can change over time, but an order must preserve the price that was valid at the moment of purchase. This is intentional duplication to preserve historical accuracy.

---

# Part 2. Core interview topics

---

# 9. Transactions and ACID

## What is a transaction?

**RU:**

Транзакция — это последовательность операций, которые рассматриваются базой данных как одна логическая единица работы.

Либо все операции успешно завершаются, либо изменения отменяются. Классический пример — перевод денег между счетами.

**EN:**

A transaction is a sequence of database operations treated as one logical unit of work.

Either all operations succeed, or the changes are rolled back. A common example is transferring money between two bank accounts.

## Transaction commands

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

UPDATE accounts
SET balance = balance + 100
WHERE id = 2;

COMMIT;
```

If something goes wrong:

```sql
ROLLBACK;
```

## ACID properties

### Atomicity

**RU:**

Atomicity означает «всё или ничего». Если одна из операций транзакции завершается ошибкой, транзакция не должна частично сохранять свои изменения.

**EN:**

Atomicity means all or nothing. If a transaction fails, its changes are rolled back instead of being partially committed.

### Consistency

**RU:**

Consistency означает, что транзакция переводит базу данных из одного корректного состояния в другое, сохраняя ограничения целостности и бизнес-правила.

**EN:**

Consistency means that a transaction moves the database from one valid state to another while preserving integrity constraints and business rules.

### Isolation

**RU:**

Isolation контролирует взаимодействие параллельных транзакций. Уровень изоляции определяет, какие изменения других транзакций могут быть видны текущей транзакции.

**EN:**

Isolation controls how concurrent transactions interact with each other. The isolation level determines which changes made by other transactions are visible to the current transaction.

### Durability

**RU:**

Durability означает, что после успешного COMMIT изменения должны сохраняться даже после сбоя сервера. В PostgreSQL это обеспечивается в том числе механизмом WAL и настройками надёжной записи на диск.

**EN:**

Durability means that committed changes must survive database crashes. PostgreSQL provides this guarantee through mechanisms including WAL and durable disk synchronization, depending on the configuration.

## Autocommit

**RU:**

Autocommit означает, что каждый отдельный SQL-запрос автоматически выполняется как отдельная транзакция, если явно не открыта другая транзакция.

**EN:**

Autocommit means that each individual SQL statement is automatically committed as its own transaction unless an explicit transaction is opened.

## SAVEPOINT

**RU:**

SAVEPOINT создаёт точку внутри транзакции, к которой можно откатиться, не отменяя всю транзакцию.

**EN:**

SAVEPOINT creates a checkpoint inside a transaction, allowing us to roll back part of the transaction without cancelling everything.

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

SAVEPOINT after_withdrawal;

UPDATE accounts
SET balance = balance + 100
WHERE id = 2;

ROLLBACK TO SAVEPOINT after_withdrawal;

COMMIT;
```

## Interview questions

**Q: What are ACID properties?**

**EN:** ACID describes four transaction guarantees: Atomicity, Consistency, Isolation, and Durability. They help ensure that database operations remain reliable even when failures or concurrent transactions occur.

**Q: Why should transactions be short?**

**EN:** Long-running transactions can hold locks, retain old row versions, increase contention, and contribute to database bloat. They can also increase the risk of deadlocks.

**Q: Is the C in ACID the same as the C in CAP?**

**EN:** No. ACID consistency means preserving database integrity constraints. CAP consistency refers to consistency of replicated data in a distributed system.

---

# 10. Indexes, B-Tree and index optimization

## What is an index?

**RU:**

Индекс — дополнительная структура данных, которая позволяет находить записи без полного сканирования таблицы.

В PostgreSQL по умолчанию используется B-Tree. Индексы могут ускорять поиск, фильтрацию, сортировку и JOIN, но занимают место и увеличивают стоимость операций записи.

**EN:**

An index is an additional data structure that allows the database to locate records without scanning the entire table.

PostgreSQL uses B-Tree indexes by default. Indexes can improve searching, filtering, sorting, and JOIN operations, but they require additional storage and introduce overhead for write operations.

## B-Tree

**RU:**

B-Tree — сбалансированная древовидная структура, которая хранит ключи в упорядоченном виде.

Она подходит для:

- Equality: `WHERE id = 10`
    
- Range queries: `WHERE salary > 5000`
    
- BETWEEN
    
- ORDER BY
    
- Prefix matching: `LIKE 'abc%'`, при подходящей collation и operator class.
    

B-Tree не является универсальным решением для поиска по произвольной подстроке, например `LIKE '%abc%'`.

**EN:**

B-Tree is a balanced tree structure that stores indexed keys in sorted order.

It supports equality comparisons, range queries, BETWEEN, and many ORDER BY operations. It can also support prefix matching such as LIKE 'abc%' when the collation and operator class allow it.

However, a standard B-Tree is not generally suitable for arbitrary substring searches such as LIKE '%abc%'.

## Hash index

**RU:**

Hash index предназначен для поиска по точному равенству. Он не поддерживает диапазоны и сортировку по индексированному значению.

В PostgreSQL Hash indexes являются WAL-logged и могут использоваться для equality comparisons.

**EN:**

A Hash index is designed for equality comparisons. It does not support range scans or ordering by indexed values.

PostgreSQL Hash indexes are WAL-logged and can be used for equality lookups.

## GIN

**RU:**

GIN (Generalized Inverted Index) хранит обратные соответствия между элементами и строками. Подходит для массивов, JSONB и полнотекстового поиска.

**EN:**

GIN (Generalized Inverted Index) stores inverted mappings between indexed elements and rows. It is commonly used for arrays, JSONB, and full-text search.

## GiST

**RU:**

GiST — обобщённая индексная структура, используемая для специализированных типов данных, геопространственного поиска и поиска по диапазонам.

**EN:**

GiST is a generalized index framework used for specialized data types, geometric searches, spatial queries, and range operations.

## BRIN

**RU:**

BRIN хранит сводную информацию о диапазонах физических блоков таблицы. Особенно полезен для очень больших таблиц, где значения физически коррелируют с порядком хранения, например для временных данных.

**EN:**

BRIN stores summarized information about ranges of physical table blocks. It is useful for very large tables where indexed values correlate with physical row order, such as time-series data.

## Composite indexes

**RU:**

Composite index — индекс по нескольким столбцам.

Например, индекс `(user_id, status)` может эффективно использоваться для условий по `user_id` и по комбинации `user_id` + `status`.

Порядок столбцов важен. PostgreSQL использует ведущие столбцы индекса наиболее эффективно. В современных версиях PostgreSQL возможен skip scan при определённых условиях, поэтому нельзя утверждать, что поиск только по второму столбцу всегда абсолютно невозможен.

**EN:**

A composite index contains multiple columns.

For example, an index on `(user_id, status)` is particularly useful for queries filtering by `user_id` or by both `user_id` and `status`.

Column order matters because B-Tree indexes are primarily organized by their leading columns. Modern PostgreSQL versions can also use skip scans in some situations, so an index on the second column is not necessarily completely unusable.

```sql
CREATE INDEX idx_orders_user_status
ON orders (user_id, status);
```

## Covering indexes

**RU:**

Covering index содержит все столбцы, необходимые конкретному запросу. Это позволяет базе данных получить результат непосредственно из индекса, если условия видимости MVCC позволяют выполнить Index Only Scan.

В PostgreSQL дополнительные столбцы можно добавить через INCLUDE.

**EN:**

A covering index contains all columns required by a particular query. This can allow the database to retrieve the result directly from the index using an Index Only Scan when MVCC visibility conditions permit it.

PostgreSQL supports additional non-key columns through the INCLUDE clause.

```sql
CREATE INDEX idx_orders_user
ON orders (user_id)
INCLUDE (total, created_at);
```

## Partial indexes

**RU:**

Partial index содержит только строки, которые удовлетворяют заданному условию. Это позволяет уменьшить размер индекса, если часто выполняются запросы по определённому подмножеству данных.

**EN:**

A partial index contains only rows that satisfy a specified condition. It can reduce index size when queries frequently access a particular subset of records.

```sql
CREATE INDEX idx_orders_new
ON orders (created_at)
WHERE status = 'new';
```

## Functional indexes

**RU:**

Functional index создаётся по выражению или функции. Он полезен, когда запросы регулярно используют функцию над столбцом.

**EN:**

A functional index is created on an expression or function. It is useful when queries frequently apply the same expression to an indexed column.

```sql
CREATE INDEX idx_users_lower_email
ON users (LOWER(email));
```

```sql
SELECT *
FROM users
WHERE LOWER(email) = 'john@example.com';
```

## Selectivity

**RU:**

Selectivity показывает, насколько сильно условие уменьшает количество подходящих строк. Индекс по столбцу с малым количеством уникальных значений иногда оказывается неэффективным, потому что PostgreSQL может предпочесть Sequential Scan.

Но низкая кардинальность сама по себе не означает, что индекс бесполезен: важны распределение данных и конкретный запрос.

**EN:**

Selectivity describes how effectively a condition reduces the number of matching rows.

An index on a low-cardinality column may be inefficient if a query returns a large percentage of the table. However, low cardinality does not automatically make an index useless; data distribution and query patterns also matter.

## Clustered vs non-clustered indexes

**RU:**

В MySQL InnoDB таблица физически организована вокруг clustered primary key index. Вторичные индексы содержат значение первичного ключа для поиска записи.

В PostgreSQL таблица хранится как heap, а индексы отдельно содержат ссылки на физические версии строк. Команда CLUSTER может физически переупорядочить таблицу по индексу, но это не поддерживает порядок автоматически после последующих изменений.

**EN:**

In MySQL InnoDB, the table is organized around the clustered primary key index. Secondary indexes store primary key values to locate the corresponding rows.

In PostgreSQL, table data is stored in a heap, and indexes are separate structures containing references to physical row versions. The CLUSTER command can physically reorder a table according to an index, but this ordering is not automatically maintained after subsequent modifications.

## Why can an index not be used?

**RU:**

Возможные причины:

- Запрос возвращает большую часть таблицы.
    
- Индекс не соответствует условиям фильтрации.
    
- Используется ведущий wildcard в LIKE.
    
- Используется функция над столбцом без подходящего expression index.
    
- Статистика устарела.
    
- Типы данных или преобразования мешают использованию индекса.
    
- Планировщик считает Sequential Scan дешевле.
    

**EN:**

An index may not be used for several reasons:

- The query returns a large percentage of the table.
    
- The index does not match the query conditions.
    
- LIKE uses a leading wildcard.
    
- A function is applied to a column without a matching expression index.
    
- Database statistics are outdated.
    
- Type conversions interfere with index usage.
    
- The query planner estimates that a sequential scan is cheaper.
    

## Interview questions

**Q: What is an index?**

**EN:** An index is an additional data structure that improves data retrieval performance. It can reduce the amount of data scanned, but it requires storage and increases the cost of INSERT, UPDATE, and DELETE operations.

**Q: What is the difference between B-Tree and Hash indexes?**

**EN:** B-Tree supports equality, range queries, and ordering. Hash indexes are designed primarily for equality comparisons.

**Q: What is a composite index?**

**EN:** A composite index contains multiple columns. Its column order matters because queries filtering by the leading columns can generally use it most effectively.

**Q: Why are too many indexes bad?**

**EN:** Every additional index requires storage and must be maintained during data modifications. Too many indexes increase write overhead and maintenance costs.

**Q: Can an index make a query slower?**

**EN:** An index can introduce additional maintenance overhead and may lead to a less efficient execution plan if the planner chooses an unsuitable access path. For queries returning many rows, a sequential scan may be faster.

---

# 11. Concurrent access, isolation levels and MVCC

## Main idea

**RU:**

Параллельный доступ возникает, когда несколько транзакций одновременно работают с одними данными. Уровни изоляции определяют, какие аномалии могут возникнуть и насколько транзакции изолированы друг от друга.

**EN:**

Concurrent access occurs when multiple transactions operate on the same data at the same time. Isolation levels determine which concurrency anomalies are possible and how strongly transactions are isolated from one another.

## Concurrency anomalies

### Dirty Read

**RU:**

Dirty Read возникает, когда транзакция читает изменения другой транзакции, которые ещё не были подтверждены COMMIT и могут быть отменены.

**EN:**

A dirty read occurs when a transaction reads changes made by another transaction before those changes have been committed. The other transaction may still roll back.

### Non-Repeatable Read

**RU:**

Non-Repeatable Read возникает, когда транзакция повторно читает одну строку и получает другое значение из-за изменения и COMMIT другой транзакции.

**EN:**

A non-repeatable read occurs when a transaction reads the same row twice and gets different values because another transaction updated and committed the row between the two reads.

### Phantom Read

**RU:**

Phantom Read возникает, когда повторный запрос с тем же условием возвращает другое количество строк из-за конкурентных INSERT или DELETE.

**EN:**

A phantom read occurs when repeating a query with the same condition returns a different set of rows because another transaction inserted or deleted matching records.

### Lost Update

**RU:**

Lost Update возникает, когда две транзакции читают одно значение, независимо изменяют его и одна запись перезаписывает результат другой.

**EN:**

A lost update occurs when two transactions read the same value, independently modify it, and one transaction overwrites the changes made by the other.

### Write Skew

**RU:**

Write Skew возникает, когда транзакции читают пересекающиеся данные, но изменяют разные строки. В результате нарушается общее бизнес-правило.

Например, две транзакции одновременно отключают разных дежурных врачей, не проверяя, что хотя бы один врач должен остаться на смене.

**EN:**

Write skew occurs when concurrent transactions read overlapping data but modify different rows, resulting in a violation of a shared business invariant.

For example, two transactions may independently deactivate different doctors even though at least one doctor must remain on duty.

## Isolation levels

|Level|Dirty Read|Non-repeatable Read|Phantom Read|
|---|---|---|---|
|Read Uncommitted|Possible|Possible|Possible|
|Read Committed|No|Possible|Possible|
|Repeatable Read|No|No|Possible by standard|
|Serializable|No|No|No|

**RU:**

Это таблица стандартной модели SQL. Реальное поведение зависит от конкретной СУБД и реализации MVCC.

PostgreSQL использует Read Committed по умолчанию. При Repeatable Read PostgreSQL обеспечивает snapshot isolation и предотвращает фантомные чтения, но допускает некоторые формы write skew.

MySQL InnoDB использует Repeatable Read по умолчанию.

**EN:**

This table represents the standard SQL isolation model. Actual behavior depends on the database engine and its concurrency implementation.

PostgreSQL uses Read Committed by default. Its Repeatable Read implementation provides snapshot isolation and prevents phantom reads, but some forms of write skew remain possible.

MySQL InnoDB uses Repeatable Read by default.

## PostgreSQL isolation behavior

**RU:**

- Read Committed: каждый SQL statement получает собственный snapshot.
    
- Repeatable Read: транзакция использует согласованный snapshot, созданный в начале её первого запроса.
    
- Serializable: PostgreSQL отслеживает опасные зависимости и может отменить транзакцию с serialization failure. Приложение должно повторить всю транзакцию.
    

**EN:**

- Read Committed: each SQL statement receives its own snapshot.
    
- Repeatable Read: the transaction uses a consistent snapshot established by its first relevant statement.
    
- Serializable: PostgreSQL detects potentially unsafe dependency patterns and may abort a transaction with a serialization failure. The application must retry the entire transaction.
    

## MVCC

**RU:**

MVCC (Multi-Version Concurrency Control) — механизм управления конкурентным доступом через несколько версий строк.

В PostgreSQL при UPDATE создаётся новая версия строки, а старая остаётся доступной для транзакций, которым она ещё необходима согласно snapshot.

Это позволяет читателям видеть согласованные данные, не блокируя обычные операции записи, и наоборот.

**EN:**

MVCC (Multi-Version Concurrency Control) is a concurrency mechanism that maintains multiple versions of database rows.

In PostgreSQL, an UPDATE creates a new row version while the old version remains available to transactions whose snapshots still require it.

This allows ordinary readers and writers to work concurrently without blocking each other in many situations.

## xmin and xmax

**RU:**

PostgreSQL хранит служебную информацию о версиях строк. В частности, xmin связан с транзакцией, создавшей версию строки, а xmax — с транзакцией, удалившей или изменившей её в соответствующем контексте.

**EN:**

PostgreSQL stores system metadata for row versions. In particular, xmin identifies the transaction that created a row version, while xmax is associated with a transaction that deleted or otherwise invalidated that version.

## VACUUM and dead tuples

**RU:**

После UPDATE и DELETE старые версии строк не всегда могут быть удалены немедленно. Они становятся dead tuples после того, как больше не нужны активным snapshots.

VACUUM освобождает пространство для повторного использования, а autovacuum выполняет обслуживание автоматически.

**EN:**

After UPDATE and DELETE operations, old row versions cannot always be removed immediately. They become dead tuples once they are no longer needed by active snapshots.

VACUUM makes their space available for reuse, while autovacuum performs this maintenance automatically.

## Interview questions

**Q: What is MVCC?**

**EN:** MVCC is a concurrency control mechanism that maintains multiple versions of rows. It allows transactions to read consistent snapshots while other transactions modify data.

**Q: What is the default isolation level in PostgreSQL?**

**EN:** PostgreSQL uses Read Committed by default. Each statement receives a new snapshot of committed data.

**Q: What is the difference between Repeatable Read and Serializable?**

**EN:** Repeatable Read provides a stable snapshot for a transaction, while Serializable provides stronger guarantees by ensuring that committed concurrent transactions behave as if they were executed sequentially. Serializable transactions may need to be retried.

**Q: What is VACUUM?**

**EN:** VACUUM reclaims space occupied by obsolete row versions and helps PostgreSQL maintain database health. Autovacuum performs this maintenance automatically.

---

# 12. Query optimization and EXPLAIN

## Main idea

**RU:**

Оптимизация SQL-запросов направлена на уменьшение времени выполнения, количества прочитанных данных, операций ввода-вывода и использования ресурсов.

Первым шагом при расследовании медленного запроса обычно является анализ execution plan.

**EN:**

SQL query optimization aims to reduce execution time, the amount of data scanned, I/O operations, and resource consumption.

When investigating a slow query, the first step is usually to examine its execution plan.

## EXPLAIN

**RU:**

EXPLAIN показывает предполагаемый план выполнения запроса, построенный планировщиком на основе статистики. Сам запрос при этом не выполняется.

**EN:**

EXPLAIN displays the estimated execution plan generated by the query planner using database statistics. The query itself is not executed.

## EXPLAIN ANALYZE

**RU:**

EXPLAIN ANALYZE реально выполняет запрос и показывает фактические показатели: время, количество строк и другие параметры выполнения.

Важно: EXPLAIN ANALYZE для UPDATE или DELETE действительно изменяет данные, если не использовать транзакцию с последующим ROLLBACK.

**EN:**

EXPLAIN ANALYZE actually executes the query and reports real execution metrics, including execution time and actual row counts.

For UPDATE and DELETE statements, it performs the modification. When testing such queries, we can wrap them in a transaction and use ROLLBACK afterward.

```sql
BEGIN;

EXPLAIN ANALYZE
UPDATE orders
SET status = 'old'
WHERE created_at < '2025-01-01';

ROLLBACK;
```

## Important scan types

**RU:**

- Sequential Scan — чтение таблицы последовательно.
    
- Index Scan — поиск через индекс с последующим чтением строк.
    
- Index Only Scan — получение данных непосредственно из индекса, если это возможно с учётом visibility map.
    
- Bitmap Index Scan — создание bitmap подходящих строк, часто с последующим Bitmap Heap Scan.
    

Sequential Scan не всегда является проблемой. Для маленьких таблиц или запросов, возвращающих большую часть данных, он может быть эффективнее индекса.

**EN:**

- Sequential Scan reads the table sequentially.
    
- Index Scan uses an index to locate matching rows and then retrieves the corresponding table rows.
    
- Index Only Scan retrieves data directly from an index when visibility information allows it.
    
- Bitmap Index Scan builds a bitmap of matching row locations, often followed by a Bitmap Heap Scan.
    

A Sequential Scan is not necessarily bad. It may be more efficient for small tables or queries that return a large percentage of the data.

## Estimated rows vs actual rows

**RU:**

Если estimated rows сильно отличаются от actual rows, планировщик может выбрать неоптимальный план.

Нужно проверить статистику и при необходимости выполнить ANALYZE.

**EN:**

If estimated row counts differ significantly from actual row counts, the query planner may choose a suboptimal execution plan.

We should check database statistics and run ANALYZE when necessary.

## JOIN algorithms

**RU:**

PostgreSQL использует несколько алгоритмов соединения:

- Nested Loop Join — для небольших наборов данных или эффективного индексного поиска.
    
- Hash Join — для соединения больших наборов данных по равенству.
    
- Merge Join — для уже отсортированных наборов данных или ситуаций, когда сортировка оправдана.
    

**EN:**

PostgreSQL supports several join algorithms:

- Nested Loop Join is useful for small datasets or when indexed lookups are efficient.
    
- Hash Join is commonly used for joining larger datasets using equality conditions.
    
- Merge Join is effective when both inputs are sorted or can be sorted efficiently.
    

## OFFSET problem

**RU:**

Большой OFFSET может быть дорогим, потому что база данных должна обработать пропускаемые строки перед возвратом результата.

Для больших наборов данных часто лучше использовать keyset pagination.

**EN:**

A large OFFSET can be expensive because the database must process the skipped rows before returning the requested result.

For large datasets, keyset pagination is often a more efficient alternative.

```sql
SELECT id, total
FROM orders
WHERE id < :last_id
ORDER BY id DESC
LIMIT 20;
```

## N+1 problem

**RU:**

N+1 возникает, когда приложение сначала получает список записей одним запросом, а затем выполняет отдельный запрос для каждой записи, чтобы получить связанные данные.

Решения: JOIN, eager loading или получение связанных данных через IN.

**EN:**

The N+1 problem occurs when an application executes one query to retrieve a list of records and then performs an additional query for each record to load related data.

Common solutions include JOIN operations, eager loading, and fetching related records using IN.

## Other optimization techniques

**RU:**

- Не использовать SELECT *, если нужны только несколько столбцов.
    
- Создавать индексы на основании реальных запросов.
    
- Использовать batch INSERT.
    
- Избегать функций над индексированными столбцами без expression index.
    
- Использовать EXISTS для проверки наличия записей.
    
- Использовать кэширование для дорогих повторяющихся запросов.
    
- Анализировать pg_stat_statements и медленные запросы.
    

**EN:**

Other optimization techniques include:

- Selecting only the required columns instead of using SELECT *.
    
- Creating indexes based on actual query patterns.
    
- Using batch INSERT operations.
    
- Avoiding functions on indexed columns unless an appropriate expression index exists.
    
- Using EXISTS for existence checks.
    
- Caching expensive and frequently repeated results.
    
- Monitoring slow queries with pg_stat_statements.
    

## Interview questions

**Q: A query is slow. What would you do first?**

**EN:** First, I would inspect the execution plan using EXPLAIN ANALYZE. I would check scan types, estimated versus actual row counts, JOIN algorithms, sorting operations, and index usage. Then I would identify the bottleneck and optimize the query or database structure.

**Q: Why is a large OFFSET slow?**

**EN:** Because the database still needs to process the skipped rows. Keyset pagination can be more efficient because it uses the last retrieved indexed value as the starting point.

**Q: What is the N+1 problem?**

**EN:** It occurs when an application performs one query to retrieve a collection and then executes additional queries for each item. It can be solved using eager loading, JOINs, or batch queries.

---

# 13. SQL injection and prepared statements

## Main idea

**RU:**

SQL Injection — уязвимость, возникающая, когда пользовательский ввод напрямую включается в SQL-код. Злоумышленник может изменить логику запроса и получить несанкционированный доступ к данным.

Основная защита — parameterized queries и prepared statements.

**EN:**

SQL injection is a security vulnerability that occurs when user input is directly included in SQL statements. An attacker may manipulate the query and gain unauthorized access to data.

The primary defense is to use parameterized queries and prepared statements.

## Dangerous example

**RU:**

Нельзя формировать SQL-запрос простым объединением строк с пользовательским вводом.

**EN:**

We should never build SQL queries by directly concatenating user input into SQL strings.

```php
// DANGEROUS
$sql = "SELECT * FROM users WHERE email = '$email'";
```

## Prepared statements

**RU:**

Prepared statements отделяют SQL-код от передаваемых значений. Параметры передаются отдельно и интерпретируются как данные, а не как SQL-команды.

**EN:**

Prepared statements separate SQL code from user-provided values. Parameters are supplied separately and treated as data rather than executable SQL code.

```php
$stmt = $pdo->prepare(
    'SELECT * FROM users WHERE email = :email'
);

$stmt->execute([
    'email' => $email,
]);

$user = $stmt->fetch();
```

## Important limitation

**RU:**

Параметры нельзя использовать вместо названий таблиц или столбцов. Если имя таблицы или столбца динамическое, необходимо использовать строгий whitelist допустимых значений.

Также prepared statements не заменяют авторизацию, проверку прав и бизнес-валидацию.

**EN:**

Parameters cannot be used for table names or column names. If identifiers must be dynamic, we should use a strict whitelist of allowed values.

Prepared statements also do not replace authorization, access control, or business validation.

## Interview questions

**Q: How do you prevent SQL injection?**

**EN:** I use prepared statements with bound parameters to separate SQL code from user input. I also apply least-privilege database permissions and validate dynamic identifiers using an allowlist.

**Q: Are prepared statements enough for application security?**

**EN:** They protect against SQL injection in parameterized values, but they do not replace authorization, access control, input validation, or secure application design.

---

# 14. Window functions and CTE

## Window functions

**RU:**

Window functions вычисляют значения на основе набора связанных строк, но не объединяют строки в одну, как GROUP BY.

Они особенно полезны для ранжирования, аналитики, расчёта накопительных сумм и сравнения текущих значений с предыдущими.

**EN:**

Window functions calculate values based on a set of related rows without collapsing them into a single row, unlike GROUP BY.

They are especially useful for ranking, analytics, running totals, and comparing current values with previous records.

## OVER and PARTITION BY

**RU:**

OVER определяет окно строк, для которых будет выполняться вычисление. PARTITION BY разделяет данные на независимые группы, а ORDER BY задаёт порядок обработки внутри каждой группы.

**EN:**

OVER defines the window of rows used by a window function. PARTITION BY divides the data into independent groups, while ORDER BY defines the processing order within each group.

## ROW_NUMBER

**RU:**

ROW_NUMBER присваивает каждой строке уникальный порядковый номер внутри окна.

**EN:**

ROW_NUMBER assigns a unique sequential number to each row within a window.

## RANK

**RU:**

RANK присваивает одинаковый ранг строкам с одинаковыми значениями, но оставляет пропуски после совпадений.

Например: 1, 1, 3.

**EN:**

RANK assigns the same rank to rows with equal values but leaves gaps after ties.

For example: 1, 1, 3.

## DENSE_RANK

**RU:**

DENSE_RANK также присваивает одинаковые ранги совпадающим значениям, но не оставляет пропусков.

Например: 1, 1, 2.

**EN:**

DENSE_RANK assigns the same rank to tied values but does not leave gaps in the ranking.

For example: 1, 1, 2.

## LAG and LEAD

**RU:**

LAG позволяет получить значение предыдущей строки, а LEAD — следующей строки в рамках окна.

**EN:**

LAG retrieves a value from a previous row, while LEAD retrieves a value from a following row within the window.

## Running total

**RU:**

Running total — накопительная сумма. Она рассчитывается с использованием агрегатной функции как оконной.

**EN:**

A running total is a cumulative sum calculated by using an aggregate function as a window function.

```sql
SELECT
    day,
    revenue,
    SUM(revenue) OVER (
        ORDER BY day
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM daily_revenue;
```

## CTE

**RU:**

CTE (Common Table Expression) — именованный временный результат запроса, определяемый через WITH.

Он улучшает читаемость сложных запросов и позволяет разделить вычисления на логические этапы.

**EN:**

A CTE (Common Table Expression) is a named temporary result set defined using the WITH clause.

It improves query readability and allows complex queries to be divided into logical steps.

```sql
WITH high_salary AS (
    SELECT *
    FROM employees
    WHERE salary > 5000
)
SELECT *
FROM high_salary;
```

## Recursive CTE

**RU:**

Recursive CTE позволяет выполнять рекурсивную обработку иерархических данных, например дерева категорий или организационной структуры.

**EN:**

A recursive CTE allows us to process hierarchical data, such as category trees or organizational structures.

```sql
WITH RECURSIVE tree AS (
    SELECT id, parent_id, name, 1 AS depth
    FROM categories
    WHERE parent_id IS NULL

    UNION ALL

    SELECT c.id, c.parent_id, c.name, t.depth + 1
    FROM categories c
    JOIN tree t ON c.parent_id = t.id
)
SELECT *
FROM tree;
```

## Top N per group

**RU:**

Одна из распространённых задач SQL-собеседования — получить несколько самых высокооплачиваемых сотрудников в каждом отделе.

Для этого удобно использовать DENSE_RANK или ROW_NUMBER в CTE.

**EN:**

A common SQL interview problem is retrieving the highest-paid employees in each department.

This can be solved using DENSE_RANK or ROW_NUMBER inside a CTE.

```sql
WITH ranked AS (
    SELECT
        name,
        department_id,
        salary,
        DENSE_RANK() OVER (
            PARTITION BY department_id
            ORDER BY salary DESC
        ) AS position
    FROM employees
)
SELECT *
FROM ranked
WHERE position <= 2;
```

## Interview questions

**Q: What is the difference between GROUP BY and window functions?**

**EN:** GROUP BY combines rows into groups and returns one result per group. Window functions calculate values across related rows while preserving the individual rows.

**Q: What is the difference between RANK and DENSE_RANK?**

**EN:** RANK leaves gaps after tied values, while DENSE_RANK does not.

**Q: How would you find the second-highest salary?**

**EN:** I would use DENSE_RANK with ORDER BY salary DESC and filter for rank 2. This also handles duplicate salary values correctly.

---

# Part 3. Advanced topics

---

# 15. Locks, optimistic vs pessimistic locking and deadlocks

## Main idea

**RU:**

Блокировки управляют конкурентным доступом к данным. Pessimistic locking блокирует запись заранее, а optimistic locking обнаруживает конфликт в момент обновления.

**EN:**

Locks control concurrent access to database records. Pessimistic locking prevents conflicts by locking data in advance, while optimistic locking detects conflicts when an update is attempted.

## Pessimistic locking

**RU:**

Pessimistic locking предполагает, что конфликты вероятны. Поэтому приложение блокирует запись ещё на этапе чтения.

В PostgreSQL для этого используется SELECT FOR UPDATE.

**EN:**

Pessimistic locking assumes that conflicts are likely to occur. Therefore, the application locks the record when reading it.

In PostgreSQL, this can be implemented using SELECT FOR UPDATE.

```sql
BEGIN;

SELECT id, balance
FROM wallets
WHERE id = 1
FOR UPDATE;

UPDATE wallets
SET balance = balance - 100
WHERE id = 1;

COMMIT;
```

## NOWAIT and SKIP LOCKED

**RU:**

NOWAIT немедленно возвращает ошибку, если запись уже заблокирована несовместимой блокировкой.

SKIP LOCKED пропускает заблокированные записи. Это удобно для распределённых worker-ов, которые обрабатывают общую очередь задач.

**EN:**

NOWAIT returns an error immediately if a row is already locked by an incompatible lock.

SKIP LOCKED skips locked rows. It is useful when multiple workers process the same job queue concurrently.

## Optimistic locking

**RU:**

Optimistic locking предполагает, что конфликты редки. Обычно в таблице есть version column.

При обновлении приложение передаёт ожидаемую версию. Если обновлено ноль строк, значит запись уже изменилась и операцию необходимо повторить или обработать конфликт.

**EN:**

Optimistic locking assumes that conflicts are relatively rare. A common implementation uses a version column.

When updating a record, the application checks the expected version. If zero rows are updated, the record has changed and the application must handle the conflict.

```sql
UPDATE products
SET stock = stock - 1,
    version = version + 1
WHERE id = 10
  AND version = 5;
```

## Atomic updates

**RU:**

Иногда отдельная блокировка не нужна. Можно использовать атомарный UPDATE с условием, чтобы изменение выполнялось только при соблюдении бизнес-правила.

**EN:**

Sometimes an explicit lock is unnecessary. We can use an atomic UPDATE with a condition to ensure that the modification happens only when the business rule is satisfied.

```sql
UPDATE wallets
SET balance = balance - 100
WHERE id = 1
  AND balance >= 100;
```

## Deadlocks

**RU:**

Deadlock возникает, когда две или более транзакции ожидают блокировки, удерживаемые друг другом.

Например, транзакция A заблокировала строку 1 и ждёт строку 2, а транзакция B заблокировала строку 2 и ждёт строку 1.

СУБД обнаруживает deadlock и отменяет одну из транзакций.

**EN:**

A deadlock occurs when two or more transactions wait for locks held by each other.

For example, transaction A locks row 1 and waits for row 2, while transaction B locks row 2 and waits for row 1.

The database detects the deadlock and aborts one of the transactions.

## How to prevent deadlocks

**RU:**

- Блокировать ресурсы в одинаковом порядке.
    
- Не держать транзакции открытыми слишком долго.
    
- Не выполнять внешние HTTP-запросы внутри транзакций.
    
- Обрабатывать ошибки deadlock и повторять транзакцию.
    
- Использовать минимально необходимый уровень блокировки.
    

**EN:**

To reduce deadlocks, we should:

- Acquire locks in a consistent order.
    
- Keep transactions short.
    
- Avoid external HTTP requests inside transactions.
    
- Handle deadlock errors and retry the entire transaction when appropriate.
    
- Use the least restrictive locking strategy that satisfies the business requirements.
    

## Idempotency

**RU:**

Для финансовых операций важно не только блокировать конкурентные запросы, но и обеспечивать идемпотентность.

Idempotency Key позволяет определить повторную отправку одного и того же запроса и не выполнить списание дважды.

**EN:**

For financial operations, preventing concurrent conflicts is not enough. We also need idempotency.

An Idempotency Key allows the application to recognize repeated requests and prevent the same payment from being processed more than once.

## Interview questions

**Q: What is the difference between optimistic and pessimistic locking?**

**EN:** Pessimistic locking prevents conflicts by locking records before modification. Optimistic locking allows concurrent operations and detects conflicts using a version column or another concurrency check.

**Q: What is a deadlock?**

**EN:** A deadlock occurs when transactions wait for resources locked by each other, creating a circular dependency. The database detects the cycle and aborts one transaction.

**Q: How would you prevent double charging?**

**EN:** I would use database transactions, atomic updates or row-level locks, and an idempotency key. This protects against both concurrent requests and repeated client submissions.

---

# 16. WAL and crash recovery

## Main idea

**RU:**

WAL (Write-Ahead Logging) — механизм журналирования изменений, который позволяет PostgreSQL восстанавливать базу после сбоя и обеспечивать надёжность подтверждённых транзакций.

Главный принцип: соответствующая запись WAL должна быть сохранена до записи изменённой страницы данных на диск.

**EN:**

WAL (Write-Ahead Logging) is a logging mechanism that allows PostgreSQL to recover the database after a crash and provide durability for committed transactions.

The main principle is that the corresponding WAL record must be safely written before a modified data page is written to disk.

## How WAL works

**RU:**

При изменении данных PostgreSQL сначала изменяет страницы в shared buffers в памяти и формирует соответствующие записи WAL.

При COMMIT необходимые WAL-записи сбрасываются на надёжное хранилище в соответствии с настройками durability.

Сами изменённые страницы таблиц могут записываться позже, например во время checkpoint.

**EN:**

When data changes, PostgreSQL modifies pages in shared buffers and generates corresponding WAL records.

At COMMIT, the required WAL records are flushed to durable storage according to the durability configuration.

The modified table pages can be written later, for example during a checkpoint.

## Dirty pages

**RU:**

Dirty page — страница в памяти, которая была изменена, но ещё не записана в основной файл таблицы на диске.

**EN:**

A dirty page is a database page that has been modified in memory but has not yet been written to its corresponding data file on disk.

## Checkpoints

**RU:**

Checkpoint — процесс, во время которого PostgreSQL записывает изменённые страницы данных на диск и создаёт точку, от которой восстановление после сбоя становится проще.

**EN:**

A checkpoint is a process during which PostgreSQL writes modified data pages to disk and establishes a recovery point that reduces the amount of WAL that must be replayed after a crash.

## Crash recovery

**RU:**

После аварийного завершения PostgreSQL использует WAL для восстановления согласованного состояния базы. Необходимые изменения повторно применяются при восстановлении.

**EN:**

After an unexpected shutdown, PostgreSQL uses WAL during crash recovery to restore a consistent database state by replaying the necessary logged changes.

## MySQL comparison

**RU:**

В MySQL InnoDB похожую роль выполняет Redo Log. Undo Log используется для отката изменений и поддержки MVCC.

**EN:**

In MySQL InnoDB, the Redo Log provides functionality similar to PostgreSQL WAL. The Undo Log is used for rolling back changes and supporting MVCC.

## Interview questions

**Q: What is WAL?**

**EN:** WAL is a write-ahead logging mechanism that records database changes before modified data pages are written to disk. It allows the database to recover committed changes after a crash.

**Q: Why do we need WAL?**

**EN:** WAL provides durability and crash recovery. It also supports replication and point-in-time recovery when combined with appropriate backup and archiving configuration.

**Q: What is a checkpoint?**

**EN:** A checkpoint is a process that flushes modified data pages to disk and reduces the amount of WAL that must be replayed during recovery.

---

# 17. Scaling, replication, partitioning, sharding and CAP

## Main idea

**RU:**

Масштабирование базы данных необходимо, когда текущая инфраструктура не справляется с нагрузкой или объёмом данных.

Обычно сначала оптимизируют запросы и индексы, затем рассматривают кэширование, репликацию, партиционирование и только после этого сложное горизонтальное распределение данных.

**EN:**

Database scaling becomes necessary when the current infrastructure cannot efficiently handle the workload or data volume.

A common approach is to optimize queries and indexes first, then consider caching, replication, partitioning, and finally more complex horizontal data distribution.

## Vertical scaling

**RU:**

Vertical scaling — увеличение ресурсов одного сервера: CPU, RAM, дискового пространства и скорости дисков.

Это самый простой способ масштабирования, но он ограничен возможностями оборудования.

**EN:**

Vertical scaling means increasing the resources of a single database server, such as CPU, RAM, storage capacity, and disk performance.

It is relatively simple to implement but is limited by the hardware capabilities of one machine.

## Replication

**RU:**

Replication — копирование данных с основного сервера на один или несколько реплик-серверов.

Primary обычно принимает записи, а replicas могут использоваться для чтения и повышения доступности.

**EN:**

Replication is the process of copying data from a primary database server to one or more replica servers.

The primary usually handles writes, while replicas can serve read requests and improve availability.

## Synchronous replication

**RU:**

При синхронной репликации primary ожидает подтверждения от синхронной реплики согласно выбранной политике подтверждения. Это повышает надёжность, но может увеличить задержку записи.

**EN:**

With synchronous replication, the primary waits for acknowledgment from the required synchronous replica or replicas according to the configured acknowledgment policy.

This improves durability guarantees but can increase write latency.

## Asynchronous replication

**RU:**

При асинхронной репликации primary может подтвердить транзакцию до получения подтверждения от реплики.

Это снижает задержку, но создаёт replication lag и риск потери последних подтверждённых на primary изменений при определённых сценариях отказа.

**EN:**

With asynchronous replication, the primary can acknowledge a transaction before receiving confirmation from the replica.

This reduces write latency but introduces replication lag and may result in the loss of recent primary commits under certain failure scenarios.

## Replication lag

**RU:**

Replication lag — задержка между применением изменений на primary и их доступностью на replica.

Из-за этого пользователь может записать новые данные, а затем прочитать устаревшую версию с реплики.

Для критичных операций можно читать с primary или использовать стратегию read-your-writes.

**EN:**

Replication lag is the delay between changes being committed on the primary and becoming available on a replica.

As a result, a user may write new data and then read an outdated version from a replica.

For critical operations, we can read from the primary or implement a read-your-writes strategy.

## Failover

**RU:**

Failover — переключение роли primary на другую реплику при отказе основного сервера.

Для автоматизации используются специальные инструменты или возможности managed database services.

**EN:**

Failover is the process of promoting a replica to become the new primary when the original primary fails.

It can be automated using dedicated tools or managed database services.

## Partitioning

**RU:**

Partitioning разделяет одну логическую таблицу на физические части внутри одной базы данных.

Основные стратегии:

- Range partitioning.
    
- List partitioning.
    
- Hash partitioning.
    

Это помогает обслуживать большие таблицы, удалять старые данные и сокращать объём сканирования через partition pruning.

**EN:**

Partitioning divides one logical table into multiple physical partitions within the same database system.

Common strategies include range, list, and hash partitioning.

It can improve maintenance, simplify data retention, and reduce the amount of data scanned through partition pruning.

## PostgreSQL partitioning example

```sql
CREATE TABLE logs (
    id BIGINT,
    created_at DATE NOT NULL,
    message TEXT
)
PARTITION BY RANGE (created_at);

CREATE TABLE logs_2026_01
PARTITION OF logs
FOR VALUES FROM ('2026-01-01')
TO ('2026-02-01');
```

## Sharding

**RU:**

Sharding — распределение частей данных между несколькими независимыми серверами.

Например, пользователи распределяются по shard на основании user_id.

Sharding помогает масштабировать объём данных и операции записи, но усложняет JOIN, транзакции, балансировку и обслуживание.

**EN:**

Sharding distributes subsets of data across multiple independent database servers.

For example, users can be distributed between shards based on user_id.

Sharding can improve write scalability and storage capacity, but it makes joins, transactions, rebalancing, and maintenance more complex.

## Partitioning vs Sharding

|Feature|Partitioning|Sharding|
|---|---|---|
|Distribution|One database system|Multiple independent servers|
|Main goal|Manage large tables|Scale storage and writes|
|Cross-partition queries|Usually supported|May require distributed coordination|
|Complexity|Lower|Higher|

## Connection pooling

**RU:**

Connection pooling позволяет переиспользовать соединения с базой данных вместо создания нового соединения для каждого запроса.

Для PostgreSQL широко используется PgBouncer. Это особенно важно для приложений с большим количеством PHP-FPM workers.

**EN:**

Connection pooling allows applications to reuse database connections instead of creating a new connection for every request.

PgBouncer is commonly used with PostgreSQL. Connection pooling is particularly important for applications running many PHP-FPM workers.

## Backups and PITR

**RU:**

Основные варианты резервного копирования:

- Logical backup — например pg_dump.
    
- Physical backup — копирование физических файлов базы данных.
    
- PITR (Point-in-Time Recovery) — восстановление базы на определённый момент времени с использованием base backup и архивных WAL.
    

Резервная копия должна регулярно проверяться через тестовое восстановление.

**EN:**

Common backup strategies include:

- Logical backups, such as pg_dump.
    
- Physical backups of database files.
    
- Point-in-Time Recovery (PITR), which restores the database to a specific moment using a base backup and archived WAL files.
    

Backups should be regularly tested by performing actual restoration procedures.

## CAP theorem

**RU:**

CAP theorem описывает компромисс в распределённых системах во время сетевого разделения (Partition).

При возникновении сетевого разделения система должна выбирать между:

- Consistency — согласованностью данных между узлами.
    
- Availability — доступностью ответов на запросы.
    

Важно: CAP не означает, что любая база данных всегда выбирает только два свойства. Ограничение относится к поведению системы во время сетевого разделения.

**EN:**

The CAP theorem describes a trade-off in distributed systems during a network partition.

When a partition occurs, a distributed system must choose between:

- Consistency — ensuring that nodes return consistent data.
    
- Availability — ensuring that every request receives a response.
    

CAP does not mean that a database can permanently provide only two of these three properties. The trade-off specifically applies when a network partition occurs.

## Interview questions

**Q: What is the difference between replication and sharding?**

**EN:** Replication copies the same data to multiple servers, mainly improving availability and read scalability. Sharding distributes different parts of the data across servers to scale storage and write workloads.

**Q: What is the difference between partitioning and sharding?**

**EN:** Partitioning divides a logical table into smaller physical parts within a database system. Sharding distributes data across multiple independent database servers.

**Q: What is replication lag?**

**EN:** Replication lag is the delay between a transaction being committed on the primary and becoming visible on a replica.

**Q: What is the CAP theorem?**

**EN:** CAP states that during a network partition, a distributed system must choose between consistency and availability.

**Q: When would you introduce sharding?**

**EN:** I would consider sharding only when query optimization, indexing, caching, vertical scaling, and replication are no longer sufficient. Sharding introduces significant architectural complexity.

---

# 18. Final interview cheat sheet

## 18.1. Key concepts

|Topic|Main point|
|---|---|
|Database|Organized collection of data|
|DBMS|Software managing databases|
|Primary Key|Unique and non-NULL identifier|
|Foreign Key|Enforces referential integrity|
|WHERE|Filters rows|
|HAVING|Filters groups|
|GROUP BY|Groups rows for aggregation|
|NULL|Unknown or missing value|
|INNER JOIN|Only matching rows|
|LEFT JOIN|All left rows and matching right rows|
|UNION|Combines and removes duplicates|
|UNION ALL|Combines and keeps duplicates|
|Subquery|Query inside another query|
|DELETE|Removes rows|
|TRUNCATE|Clears all rows|
|DROP|Removes the table|
|Normalization|Reduces redundancy|
|Denormalization|Deliberate duplication|
|ACID|Transaction guarantees|
|B-Tree|Default PostgreSQL index|
|MVCC|Multiple row versions|
|VACUUM|Reclaims reusable space|
|EXPLAIN ANALYZE|Executes and analyzes a query|
|CTE|Named temporary query result|
|Window function|Calculates values without collapsing rows|
|WAL|Write-ahead logging|
|Replication|Copies data between servers|
|Partitioning|Splits one logical table|
|Sharding|Distributes data across servers|

## 18.2. Important SQL distinctions

|Comparison|Difference|
|---|---|
|WHERE vs HAVING|Rows vs groups|
|INNER vs LEFT JOIN|Matching rows vs all left rows|
|UNION vs UNION ALL|Removes duplicates vs preserves duplicates|
|DELETE vs TRUNCATE|Row deletion vs clearing the table|
|PRIMARY KEY vs UNIQUE|One primary key constraint vs multiple unique constraints|
|View vs Materialized View|Query definition vs stored result|
|B-Tree vs Hash|Equality and ranges vs equality|
|GROUP BY vs Window|Collapses rows vs preserves rows|
|Optimistic vs Pessimistic|Detect conflicts vs lock in advance|
|Partitioning vs Sharding|One database system vs multiple servers|
|Replication vs Sharding|Copies data vs distributes data|

## 18.3. SQL patterns to remember

**Find duplicate emails:**

```sql
SELECT email, COUNT(*)
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

**Find users without orders:**

```sql
SELECT u.*
FROM users u
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

**Find the second-highest salary:**

```sql
SELECT salary
FROM (
    SELECT DISTINCT salary
    FROM employees
    ORDER BY salary DESC
    OFFSET 1 LIMIT 1
) AS ranked;
```

**Find the top 3 employees in each department:**

```sql
WITH ranked AS (
    SELECT *,
        DENSE_RANK() OVER (
            PARTITION BY department_id
            ORDER BY salary DESC
        ) AS rnk
    FROM employees
)
SELECT *
FROM ranked
WHERE rnk <= 3;
```

**Find records with NULL:**

```sql
SELECT *
FROM users
WHERE email IS NULL;
```

**Calculate average salary by department:**

```sql
SELECT department_id, AVG(salary)
FROM employees
GROUP BY department_id;
```

**Update only if enough balance exists:**

```sql
UPDATE accounts
SET balance = balance - 100
WHERE id = 1
  AND balance >= 100;
```

## 18.4. Useful interview phrases

- "The main difference between these two concepts is..."
    
- "The main advantage of this approach is..."
    
- "The trade-off is that we gain performance at the cost of additional storage."
    
- "It depends on the specific use case and workload."
    
- "I would first inspect the execution plan using EXPLAIN ANALYZE."
    
- "The database optimizer chooses the execution plan based on statistics and estimated costs."
    
- "This operation is atomic because all changes are committed or rolled back together."
    
- "We can prevent this race condition using row-level locking."
    
- "I would use a transaction to ensure data consistency."
    
- "I would avoid premature optimization and measure the actual bottleneck first."
    
- "I'm not completely sure about the implementation details, but my understanding is that..."