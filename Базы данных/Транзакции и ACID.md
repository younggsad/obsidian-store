> **Транзакция** — группа операций, которые должны выполниться **всё или ничего**.

```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;   -- фиксируем окончательно

-- ROLLBACK; -- откатываем ВСЁ, если что-то пошло не так
```

> **SAVEPOINT** — точка частичного отката внутри транзакции:

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
SAVEPOINT after_withdrawal;

UPDATE accounts SET balance = balance + 100 WHERE id = 999; -- ошибка, id нет
ROLLBACK TO after_withdrawal; -- откат только к точке, первое обновление остаётся

UPDATE accounts SET balance = balance + 100 WHERE id = 2; -- повторная попытка
COMMIT;
```

### ACID

> **ACID** — четыре свойства, которые гарантирует надёжная транзакционная база данных, чтобы данные оставались целостными и корректными:

- **Atomicity (атомарность)** — всё или ничего, частичного выполнения не бывает. Достигается через журнал транзакций (WAL — Write-Ahead Log) и механизм отката.
- **Consistency (согласованность)** — транзакция переводит БД из одного валидного состояния в другое, не нарушая ограничения (constraints, foreign keys, unique, check).
- **Isolation (изолированность)** — параллельные транзакции не влияют друг на друга непредсказуемо (см. раздел 8).
- **Durability (устойчивость)** — после `COMMIT` изменения сохраняются навсегда, даже при сбое сразу после (данные физически записаны на диск, обычно через WAL, который проигрывается при восстановлении после сбоя).

**BASE — альтернатива ACID в NoSQL-мире (кратко, для сравнения):**

> **Basically Available, Soft state, Eventually consistent** — вместо строгих гарантий ACID многие NoSQL-системы (Cassandra, DynamoDB) выбирают доступность и производительность ценой временной несогласованности, которая "выравнивается" со временем.