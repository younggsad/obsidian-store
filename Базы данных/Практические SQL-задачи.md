Дано:

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name TEXT,
  country TEXT
);

CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id),
  total NUMERIC,
  created_at DATE
);
```

### Задача 1. Пользователи без заказов

```sql
SELECT u.name
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL;
```

Идея: `LEFT JOIN` сохраняет всех пользователей, даже без заказов (поля `orders` будут `NULL`) — фильтруем именно тех, у кого `o.id IS NULL`, то есть совпадений не нашлось.

Альтернатива через `NOT EXISTS` (часто эффективнее на больших таблицах, т.к. может останавливаться раньше и не требует полного join):

```sql
SELECT name FROM users u
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);
```

### Задача 2. Топ-3 пользователей по общей сумме заказов

```sql
SELECT u.name, SUM(o.total) AS total_spent
FROM users u
JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name
ORDER BY total_spent DESC
LIMIT 3;
```

Обрати внимание: в `GROUP BY` указаны `u.id, u.name` — хотя выводим только `name`, группировать нужно по всем не-агрегированным столбцам из `SELECT` (в PostgreSQL технически достаточно группировать по PK, если остальные столбцы функционально от него зависят, но `GROUP BY u.id, u.name` — более переносимый и явный вариант).

### Задача 3. Пользователи с >5 заказов и средним чеком >100

```sql
SELECT u.name, COUNT(o.id) AS order_count, AVG(o.total) AS avg_order
FROM users u
JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name
HAVING COUNT(o.id) > 5 AND AVG(o.total) > 100;
```

Ключевой момент: оба условия зависят от агрегатных функций — значит, это `HAVING`, а не `WHERE` (`WHERE` не может использовать `COUNT`/`AVG`, так как выполняется до группировки).

### Задача 4. Все пользователи и их заказы за последний месяц (даже без заказов)

```sql
SELECT u.name, o.total, o.created_at
FROM users u
LEFT JOIN orders o
  ON u.id = o.user_id
  AND o.created_at >= CURRENT_DATE - INTERVAL '1 month';
```

Важный нюанс: условие по дате стоит **в `ON`**, а не в `WHERE`. Если бы оно было в `WHERE`, `LEFT JOIN` фактически превратился бы в `INNER JOIN` — `WHERE o.created_at >= ...` отфильтровал бы (убрал) все строки, где `o.created_at` равен `NULL` (то есть как раз пользователей без заказов), что противоречит задаче "показать всех, даже без заказов".

### Задача 5. Второй по величине заказ (без LIMIT/OFFSET)

```sql
SELECT MAX(total) AS second_highest
FROM orders
WHERE total < (SELECT MAX(total) FROM orders);
```

Идея: находим максимум среди всех значений, которые **меньше** глобального максимума — это и есть второе по величине значение. Работает корректно даже при дубликатах максимума (если два заказа имеют одинаковый максимальный total, оба считаются "первым местом", и второй результат — уже следующий по величине).

Альтернатива через `LIMIT`/`OFFSET` (проще, но не различает дубликаты максимума так же строго):

```sql
SELECT total FROM orders ORDER BY total DESC LIMIT 1 OFFSET 1;
```

Более гибкая альтернатива через `DENSE_RANK` (учитывает "N-е по величине уникальное значение"):

```sql
SELECT total FROM (
  SELECT total, DENSE_RANK() OVER (ORDER BY total DESC) AS rnk
  FROM orders
) t WHERE rnk = 2 LIMIT 1;
```

### Задача 6. Заказы по странам, где заказы делали больше одного пользователя

```sql
SELECT u.country, COUNT(DISTINCT u.id) AS unique_customers, COUNT(o.id) AS order_count
FROM users u
JOIN orders o ON u.id = o.user_id
GROUP BY u.country
HAVING COUNT(DISTINCT u.id) > 1;
```

`COUNT(DISTINCT u.id)` — считает уникальных пользователей (не заказы), важно не перепутать с обычным `COUNT(u.id)`, который считал бы количество **строк** (с повторами пользователя, если у него несколько заказов).

### Задача 7. Объединить email из двух таблиц без дублей

```sql
SELECT email FROM users
UNION
SELECT email FROM newsletter_subscribers;
```

`UNION` (не `UNION ALL`) автоматически убирает дубли — если один и тот же email есть в обеих таблицах, в результате он появится один раз.

### Задача 8. Спровоцировать full table scan несмотря на индекс

```sql
CREATE INDEX idx_email ON users(email);

-- Индекс НЕ используется:
SELECT * FROM users WHERE UPPER(email) = 'ALICE@MAIL.COM';
```

Причина: применена функция `UPPER()` к индексированному столбцу — обычный B-tree индекс построен на **исходных** значениях `email`, а не на результате применения функции к ним, поэтому база не может напрямую воспользоваться индексом для этого сравнения (нужен отдельный функциональный индекс `CREATE INDEX ON users(UPPER(email))`, либо переписать запрос, храня email в нормализованном виде и не применяя функцию в `WHERE`).

### Задача 9. Найти пользователей с наибольшим "разрывом" между заказами (оконные функции)

```sql
WITH ordered_orders AS (
  SELECT
    user_id,
    created_at,
    LAG(created_at) OVER (PARTITION BY user_id ORDER BY created_at) AS prev_order
  FROM orders
)
SELECT user_id, MAX(created_at - prev_order) AS max_gap
FROM ordered_orders
WHERE prev_order IS NOT NULL
GROUP BY user_id
ORDER BY max_gap DESC;
```

Идея: `LAG` даёт дату предыдущего заказа того же пользователя в отсортированном по дате окне; разница между текущей и предыдущей датой — это "разрыв" в днях; дальше стандартная агрегация `MAX` по пользователю.

### Задача 10. Пагинация большого списка без деградации на больших OFFSET

```sql
-- Плохо на больших страницах:
SELECT * FROM orders ORDER BY id LIMIT 20 OFFSET 100000;

-- Хорошо (keyset pagination), при условии что клиент передаёт id последней увиденной строки:
SELECT * FROM orders WHERE id > 458210 ORDER BY id LIMIT 20;
```

Идея: вместо "пропусти N строк" используем "продолжи после конкретного значения", что напрямую использует B-tree индекс по `id` для перехода к нужной точке, а не последовательный подсчёт пропускаемых строк.