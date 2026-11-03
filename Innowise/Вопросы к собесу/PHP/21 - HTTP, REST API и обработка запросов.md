---
custom-width: 80
---
# 21 — HTTP, REST API и обработка запросов

## 1. Теория

### Суперглобальные массивы (Superglobals)
Доступные отовсюду встроенные массивы для работы с HTTP-запросом:
- `$_GET` — параметры строки запроса (Query parameters).
- `$_POST` — данные формы (POST).
- `$_REQUEST` — комбинация `$_GET`, `$_POST` и `$_COOKIE` (не рекомендуется из-за уязвимости к загрязнению параметров).
- `$_SERVER` — серверные и окружные переменные (заголовки, IP, метод запроса).
- `$_FILES` — загруженные файлы.
- `$_COOKIE` — куки.
- `$_SESSION` — данные сессии.

### REST API в PHP
- Использование правильных HTTP-методов (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`).
- Возврат корректных HTTP статус-кодов (200, 201, 400, 401, 403, 404, 422, 500).
- Установка заголовков ответа (`header('Content-Type: application/json')`).

---

## 2. Примеры кода

```php
<?php
declare(strict_types=1);

// Обработка REST API ответа
header('Content-Type: application/json; charset=utf-8');

$method = $_SERVER['REQUEST_METHOD'];

if ($method === 'GET') {
    http_response_code(200);
    echo json_encode(['status' => 'success', 'data' => []]);
} else {
    http_response_code(405); // Method Not Allowed
    echo json_encode(['error' => 'Method not allowed']);
}
```

---

## 3. Вопросы на собеседовании

1. **Почему не рекомендуется использовать суперглобальный массив `$_REQUEST`?**
   *Ответ*: `$_REQUEST` содержит смесь данных из `$_GET`, `$_POST` и `$_COOKIE`. Это создает потенциальные уязвимости безопасности (Parameter Pollution), когда злоумышленник может перезаписать GET-параметр с помощью POST или Cookie с тем же именем.
2. **Как получить заголовки входящего HTTP-запроса в PHP?**
   *Ответ*: С помощью функции `getallheaders()` (в среде Apache/FPM) или путем чтения массива `$_SERVER`, где заголовки представлены в верхнем регистре с префиксом `HTTP_` (например, `$_SERVER['HTTP_AUTHORIZATION']`).
3. **Какие HTTP статус-коды используются для успешного создания ресурса и для ошибок валидации?**
   *Ответ*: Для успешного создания — `201 Created`. Для ошибок валидации данных — `422 Unprocessable Entity` (или `400 Bad Request`).
