# 20 — Работа с JSON, XML и HTTP

## 1. Теория

### Работа с JSON
- `json_encode(mixed $value, int $flags = 0): string|false` — сериализация в JSON.
- `json_decode(string $json, ?bool $assoc = null, int $depth = 512, int $flags = 0): mixed` — десериализация.
  - Если `$assoc` равен `true`, возвращает ассоциативный массив, если `false` — объект.
- Ошибки кодирования проверяются через `json_last_error()` или с помощью флага `JSON_THROW_ON_ERROR` (выбрасывает `JsonException`).

### Работа с XML
- Для базового парсинга: `SimpleXMLElement`.
- Для сложных задач: `DOMDocument`, `XMLReader` (потоковый парсер для больших файлов) и `XMLWriter`.

---

## 2. Примеры кода

```php
<?php
declare(strict_types=1);

// JSON с выбросом исключений при ошибках
try {
    $jsonString = '{"name": "Alice", "age": 30}';
    $data = json_decode($jsonString, associative: true, flags: JSON_THROW_ON_ERROR);
    
    $encoded = json_encode($data, JSON_THROW_ON_ERROR | JSON_UNESCAPED_UNICODE);
    echo $encoded;
} catch (\JsonException $e) {
    echo "JSON error: " . $e->getMessage();
}
```

---

## 3. Вопросы на собеседовании

1. **Почему рекомендуется использовать флаг `JSON_THROW_ON_ERROR` при работе с JSON в PHP?**
   *Ответ*: По умолчанию при ошибке кодирования/декодирования `json_encode` и `json_decode` возвращают `false` или `null`, а ошибку нужно было проверять через `json_last_error()`. Флаг `JSON_THROW_ON_ERROR` заставляет функции выбрасывать `JsonException`, что упрощает обработку ошибок в блоках `try-catch`.
2. **Какой инструмент лучше использовать для парсинга огромных XML файлов и почему?**
   *Ответ*: Лучше использовать `XMLReader`, так как он работает как потоковый итератор (pull-parser), обрабатывая XML по частям и не загружая весь документ в память. `SimpleXMLElement` и `DOMDocument` загружают дерево целиком в оперативную память.
3. **Как избежать некорректного экранирования кириллицы при `json_encode()`?**
   *Ответ*: Использовать флаг `JSON_UNESCAPED_UNICODE`.
