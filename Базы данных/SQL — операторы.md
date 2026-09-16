---
custom-width: 80
---
### SELECT

> Выбирает данные (какие столбцы получить) из таблицы.

```sql
SELECT name, email FROM users WHERE age > 18;

SELECT * FROM users;
-- плохая практика в проде: ломается при добавлении новых столбцов,
-- передаёт лишние данные по сети, мешает индексу "покрыть" запрос
```

`DISTINCT` — убирает дубликаты строк из результата:

```sql
SELECT DISTINCT country FROM users; -- список уникальных стран
```

`CASE WHEN` — условная логика прямо в запросе:

```sql
SELECT
  name,
  CASE
    WHEN total > 1000 THEN 'VIP'
    WHEN total > 100  THEN 'regular'
    ELSE 'new'
  END AS customer_tier
FROM orders;
```

### JOIN

> Объединяет строки из двух (или более) таблиц по указанному условию совпадения (например, по внешнему ключу).

```sql
SELECT u.name, o.total
FROM users u
JOIN orders o ON u.id = o.user_id; -- INNER JOIN — только совпадения в обеих таблицах
```

- **`INNER JOIN`** — объединяет только строки с совпадением в обеих таблицах.
- **`LEFT JOIN` (LEFT OUTER JOIN)** — все строки из левой таблицы, даже без совпадений (поля правой — `NULL`).
- **`RIGHT JOIN`** — зеркало LEFT JOIN, все строки из правой таблицы. На практике реже — проще переписать как LEFT JOIN, поменяв таблицы местами.
- **`FULL JOIN` (FULL OUTER JOIN)** — все строки из обеих таблиц, независимо от совпадений; не совпавшие поля заполняются `NULL` с обеих сторон.
- **`CROSS JOIN`** — декартово произведение, каждая строка первой таблицы с каждой строкой второй, без условия соединения. Результат — `N × M` строк.
- **`SELF JOIN`** — таблица соединяется сама с собой (обычно для иерархических данных, например "сотрудник — его руководитель"):

```sql
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

### GROUP BY

> Группирует строки с одинаковым значением столбца в одну "сводную" строку — используется вместе с агрегатными функциями.

```sql
SELECT user_id, COUNT(*) AS order_count, SUM(total) AS total_spent
FROM orders
GROUP BY user_id;
```

**Правило:** в `SELECT` вместе с `GROUP BY` можно указывать только столбцы группировки или агрегатные функции — нельзя произвольный столбец вне группировки (неясно, из какой строки группы его брать).

### HAVING

> Фильтрует **уже сгруппированные** результаты — в отличие от `WHERE`, который фильтрует строки **до** группировки.

```sql
SELECT user_id, COUNT(*) AS order_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 5;
```

**WHERE vs HAVING:**

> `WHERE` — до группировки, на отдельных строках, без агрегатных функций. `HAVING` — после группировки, может использовать агрегатные функции.

**Логический порядок выполнения запроса** (важно запомнить для собеса):

```
FROM → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT/OFFSET
```

(Это порядок логической **обработки**, а не порядок написания в запросе — писать всё равно нужно `SELECT ... FROM ... WHERE ... GROUP BY ... HAVING ... ORDER BY ... LIMIT ...`.)

Именно из-за этого порядка алиас, заданный в `SELECT`, формально ещё "не существует" на этапе `WHERE`/`GROUP BY`/`HAVING` в строгом SQL (хотя PostgreSQL и MySQL как расширение разрешают использовать алиасы в `GROUP BY`/`HAVING`/`ORDER BY`).

### ORDER BY

> Сортирует итоговый результат по одному или нескольким столбцам (по возрастанию **ASC**, по умолчанию, или по убыванию **DESC**).

```sql
SELECT * FROM users ORDER BY created_at DESC;
SELECT * FROM users ORDER BY country ASC, name ASC; -- сначала по стране, потом по имени
```

Работа с `NULL` при сортировке — `NULL` можно явно разместить в начале или конце:

```sql
SELECT * FROM users ORDER BY last_login DESC NULLS LAST; -- PostgreSQL
```

### LIMIT / OFFSET

> **LIMIT** — ограничивает количество возвращаемых строк. **OFFSET** — сколько строк пропустить перед началом выборки (используется вместе с **LIMIT** для пагинации — "страница 2, 3..." и т.д.).

```sql
SELECT * FROM users ORDER BY id LIMIT 10;            -- первые 10
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 20;   -- страница 3 (по 10 на страницу)
```

**Проблема большого OFFSET:** база сначала "проходит" через все пропускаемые строки, потом отдаёт нужные — медленно на больших таблицах. Решение — **keyset pagination (seek pagination)**:

```sql
SELECT * FROM users WHERE id > 12345 ORDER BY id LIMIT 10; -- эффективнее большого OFFSET,
-- т.к. использует индекс на id для прямого перехода, а не сканирование с начала
```

### UNION

> Объединяет результаты двух (или более) запросов "по вертикали" (складывает строки).

```sql
SELECT name FROM customers
UNION
SELECT name FROM suppliers;
```

> `UNION` — удаляет дубликаты (требует внутренней сортировки/дедупликации — дороже). `UNION ALL` — не удаляет, быстрее, используется по умолчанию, если известно, что дублей не будет или они не важны. Оба запроса должны возвращать одинаковое количество столбцов совместимых типов, в одинаковом порядке.

### EXISTS

> Проверяет, существует ли хотя бы одна строка по условию подзапроса — возвращает `true`/`false`.

```sql
SELECT name FROM users u
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);
```

**Почему EXISTS часто эффективнее IN:** может остановиться на первом совпадении (semi-join), в то время как `IN` иногда собирает весь список результатов подзапроса перед сравнением. Также `EXISTS` безопаснее при `NULL` в списке подзапроса — `NOT IN` с `NULL` в результатах подзапроса может неожиданно вернуть пустой результат целиком, а `NOT EXISTS` этой ловушки не имеет.

```sql
-- Ловушка NOT IN:
SELECT * FROM users WHERE id NOT IN (SELECT user_id FROM orders); 
-- если хотя бы один user_id в orders — NULL, запрос вернёт 0 строк!

-- Безопасная альтернатива:
SELECT * FROM users u WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);
```

### Подзапросы (subqueries)

> Запрос внутри другого запроса. Бывают:
> 
> - **скалярные** — возвращают одно значение, используются в `SELECT`/`WHERE` как выражение;
> - **строчные/табличные** — возвращают набор строк, используются с `IN`, `EXISTS`, `FROM`;
> - **коррелированные (correlated)** — ссылаются на столбцы внешнего запроса, выполняются заново для каждой строки внешнего запроса (обычно медленнее);
> - **некоррелированные** — независимы от внешнего запроса, выполняются один раз.

```sql
-- скалярный подзапрос
SELECT name, (SELECT COUNT(*) FROM orders o WHERE o.user_id = u.id) AS order_count
FROM users u;

-- подзапрос в FROM (производная таблица)
SELECT country, AVG(total_spent) FROM (
  SELECT u.country, u.id, SUM(o.total) AS total_spent
  FROM users u JOIN orders o ON u.id = o.user_id
  GROUP BY u.country, u.id
) AS per_user
GROUP BY country;
```

### CTE (Common Table Expression, `WITH`)

> Именованный временный "подзапрос", объявленный через `WITH`, который можно использовать в основном запросе как обычную таблицу. Делает сложные запросы читаемыми, позволяет переиспользовать один и тот же промежуточный результат несколько раз в запросе.

```sql
WITH user_totals AS (
  SELECT user_id, SUM(total) AS total_spent
  FROM orders
  GROUP BY user_id
)
SELECT u.name, ut.total_spent
FROM users u
JOIN user_totals ut ON ut.user_id = u.id
WHERE ut.total_spent > 1000;
```

**Рекурсивный CTE** — для иерархических/древовидных данных (например, дерево категорий, оргструктура):

```sql
WITH RECURSIVE subordinates AS (
  SELECT id, name, manager_id FROM employees WHERE id = 1        -- базовый случай (корень)
  UNION ALL
  SELECT e.id, e.name, e.manager_id
  FROM employees e
  JOIN subordinates s ON e.manager_id = s.id                     -- рекурсивный шаг
)
SELECT * FROM subordinates;
```

Важный нюанс по производительности: в PostgreSQL до 12-й версии обычный `CTE` был **оптимизационным барьером** (materialized) — планировщик не мог "протолкнуть" внутрь него условия из внешнего запроса. Начиная с PostgreSQL 12 планировщик может встраивать (inline) CTE автоматически, если это выгодно; принудительно можно управлять через `MATERIALIZED`/`NOT MATERIALIZED`.