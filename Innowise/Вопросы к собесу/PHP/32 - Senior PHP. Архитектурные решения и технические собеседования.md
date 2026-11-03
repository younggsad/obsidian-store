---
custom-width: 80
---
# 32 — Senior PHP. Архитектурные решения и технические собеседования

## 1. Теория

### Уровень Senior PHP-разработчика
На позиции Senior от специалиста ждут не просто знания синтаксиса языка, а глубокого понимания:
- **Архитектурных паттернов**: DDD (Domain-Driven Design), CQRS, Event Sourcing, Clean Architecture.
- **Масштабирования**: Шардирование, репликация баз данных, балансировка нагрузки, кэширование (Redis Cluster).
- **Отказоустойчивости**: Graceful degradation, Circuit Breaker, Rate Limiting, очереди сообщений.
- **Code Review и менторства**: Контроль качества кода, архитектурный надзор, статический анализ (PHPStan, Psalm).

---

## 2. Примеры кода

```php
<?php
declare(strict_types=1);

// Пример паттерна Circuit Breaker (концептуальный скелет)
class CircuitBreaker {
    private int $failures = 0;
    private const THRESHOLD = 5;

    public function execute(callable $serviceCall): mixed {
        if ($this->failures >= self::THRESHOLD) {
            throw new \RuntimeException("Circuit breaker is OPEN. Service unavailable.");
        }

        try {
            $result = $serviceCall();
            $this->failures = 0; // Сброс при успехе
            return $result;
        } catch (\Throwable $e) {
            $this->failures++;
            throw $e;
        }
    }
}
```

---

## 3. Вопросы на собеседовании

1. **Что такое CQRS (Command Query Responsibility Segregation)?**
   *Ответ*: Архитектурный паттерн, разделяющий операции записи (Commands — изменяющие состояние) и операции чтения (Queries — возвращающие данные). Позволяет оптимизировать хранилища данных и масштабировать чтение и запись независимо друг от друга.
2. **Как реализовать паттерн Circuit Breaker и зачем он нужен?**
   *Ответ*: Circuit Breaker защищает систему от каскадных сбоев при падении внешнего сервиса или БД. Если количество ошибок превышает порог, он «размыкает цепь» и сразу возвращает ошибку или кэш, не нагружая падающий сервис повторными запросами.
3. **Какие инструменты статического анализа кода вы используете и в чем их польза?**
   *Ответ*: PHPStan (на уровне максимальной строжности level max) и Psalm. Они позволяют находить баги, несоответствие типов, потенциальные ошибки нулевых указателей и проблемы логики еще до запуска тестов и деплоя на production.
