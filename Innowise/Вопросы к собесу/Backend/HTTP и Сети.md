---
custom-width: 80
---
## Оглавление

1. [[#1. HTTP Basics: Что такое HTTP, URI/URL/URN и стек TCP/IP]]
2. [[#2. Жизненный цикл HTTP-запроса: DNS и TCP Handshake]]
3. [[#3. HTTPS и TLS: Как работает шифрование, сертификаты и HSTS]]
4. [[#4. Структура HTTP-сообщения, Content-Type и заголовки]]
5. [[#5. HTTP Request Methods: Методы, безопасность и идемпотентность]]
6. [[#6. Response Codes: Коды состояния HTTP]]
7. [[#7. HTTP Cookies, Сессии и JWT]]
8. [[#8. Эволюция протокола: HTTP/1.1, HTTP/2 и HTTP/3]]
9. [[#9. HTTP-кэширование и условные запросы]]
10. [[#10. Безопасность HTTP: SOP, CORS и Security-заголовки]]
11. [[#11. Проектирование API: REST, GraphQL, gRPC, WebSockets]]
12. [[#12. Шпаргалка перед собеседованием]]

---

## 1. HTTP Basics: Что такое HTTP, URI/URL/URN и стек TCP/IP

> [!abstract] Коротко
> HTTP — протокол прикладного уровня для передачи данных по схеме «клиент — сервер». Он **stateless** (не помнит прошлые запросы) и работает поверх транспортного протокола (TCP или QUIC).

### Теория (🇷🇺 Русский)

- **Что такое HTTP (HyperText Transfer Protocol)**: текстовый (в 1.1) или бинарный (в 2/3) протокол прикладного уровня, определяющий формат сообщений, которыми обмениваются клиент (браузер, мобильное приложение, сервис) и веб-сервер.
- **Модель «Клиент — Сервер»**: инициатором всегда выступает клиент, отправляющий запрос (Request). Сервер слушает порт (`80` для HTTP, `443` для HTTPS) и возвращает ответ (Response).
- **Stateless (без сохранения состояния)**: протокол не хранит историю запросов. Каждый запрос содержит всю необходимую информацию для его обработки. Это позволяет легко масштабировать бэкенд горизонтально (добавляя новые серверы за балансировщиком нагрузки).
- **Разница между URI, URL и URN**:
  - **URI (Uniform Resource Identifier)** — унифицированный идентификатор ресурса (общее понятие, включающее в себя URL и URN).
  - **URL (Uniform Resource Locator)** — указатель местоположения ресурса, описывающий *как* и *где* его найти.
    - Структура URL: `https://user:pass@api.example.com:8443/v1/users?active=1#top`
      - `https` — схема (протокол);
      - `api.example.com` — хост (доменное имя или IP);
      - `:8443` — порт;
      - `/v1/users` — путь (path);
      - `?active=1` — параметры запроса (query string);
      - `#top` — фрагмент (якорь, **не отправляется на сервер**, обрабатывается только браузером!).
  - **URN (Uniform Resource Name)** — имя ресурса без указания его местоположения (например, `urn:isbn:0451450523`).
- **Место в сетевом стеке (TCP/IP)**:
  - *Канальный (Link)*: Ethernet, Wi-Fi (MAC-адреса).
  - *Сетевой (Internet)*: IP (маршрутизация пакетов между сетями).
  - *Транспортный (Transport)*: **TCP** (надёжная доставка с гарантией порядка и повторной отправкой потерянных пакетов) или **UDP** (быстрая доставка без гарантий и установки соединения).
  - *Прикладной (Application)*: HTTP, DNS, TLS.

### Theory (🇬🇧 English)

- **HTTP Basics**: An application-layer protocol following the Request-Response client-server model. It is inherently **stateless**, meaning the server retains no session state between requests at the protocol level, enabling seamless horizontal scaling.
- **URI vs URL vs URN**:
  - **URI** (*Identifier*): Umbrella term identifying a resource by location, name, or both.
  - **URL** (*Locator*): Specifies both the resource identifier and the protocol/location to access it (`scheme://host:port/path?query#fragment`). Note: the `#fragment` is never sent to the server.
  - **URN** (*Name*): Persistent, location-independent resource name (e.g., `urn:isbn:0451450523`).

---

## 2. Жизненный цикл HTTP-запроса: DNS и TCP Handshake

> [!abstract] Коротко
> Перед отправкой HTTP-запроса клиент находит IP-адрес сервера через DNS и устанавливает надёжное TCP-соединение через трёхстороннее рукопожатие (3-way handshake).

### Теория (🇷🇺 Русский & 🇬🇧 English)

1. **DNS Lookup (Разрешение доменного имени)**:
   - Браузер ищет IP-адрес домена по цепочке кэшей: **Кэш браузера** → **Кэш ОС (`/etc/hosts`)** → **Кэш роутера** → **DNS-резолвер провайдера**.
   - Если записи нет, резолвер опрашивает иерархию DNS-серверов: **Root DNS (`.`)** → **TLD DNS (`.com`, `.ru`)** → **Authoritative DNS** (сервер, хранящий A/AAAA-записи конкретного домена).
2. **TCP 3-Way Handshake (Трёхстороннее рукопожатие)**:
   - **`SYN`**: клиент отправляет пакет с начальным порядковым номером `seq = x`.
   - **`SYN-ACK`**: сервер отвечает подтверждением `ack = x + 1` и своим номером `seq = y`.
   - **`ACK`**: клиент подтверждает `ack = y + 1`. Надёжный дуплексный канал установлен.
3. **Keep-Alive (`Connection: keep-alive`)**: в HTTP/1.1+ TCP-соединение не разрывается после одного запроса, а переиспользуется для следующих запросов, экономя время на TCP/TLS handshake.

---

## 3. HTTPS и TLS: Как работает шифрование, сертификаты и HSTS

> [!abstract] Коротко
> HTTPS — это обычный HTTP, передаваемый внутри зашифрованного криптографического канала **TLS (Transport Layer Security)**. Он обеспечивает конфиденциальность, целостность данных и подлинность сервера.

### Теория (🇷🇺 Русский)

- **Зачем нужен HTTPS (3 главные гарантии)**:
  1. *Конфиденциальность (Encryption)*: перехваченный трафик (в публичном Wi-Fi или у провайдера) невозможно прочитать.
  2. *Целостность (Integrity)*: злоумышленник (MITM — Man-in-the-Middle) не может незаметно подменить данные в ответе (например, вшить рекламу или подменить реквизиты оплаты).
  3. *Аутентификация (Authentication)*: клиент точно знает, что общается с настоящим сервером банка, а не с фишинговым двойником.
- **Почему используются два вида шифрования (Гибридное шифрование)**:
  - *Асимметричное шифрование (RSA, ECDSA, ECDHE)*: пара ключей (публичный и приватный). Очень надёжно для обмена ключами и проверки подписи, но **очень медленное** для больших объёмов данных.
  - *Симметричное шифрование (AES-GCM, ChaCha20)*: один общий секретный ключ у клиента и сервера. Работает **в сотни раз быстрее**, используется для шифрования всего HTTP-трафика после рукопожатия.
- **Как проходит TLS Handshake (TLS 1.2 / 1.3)**:
  1. **ClientHello**: клиент отправляет поддерживаемые версии TLS, список шифров (Cipher Suites) и случайное число (`Client Random`).
  2. **ServerHello + Certificate**: сервер выбирает шифр, отправляет своё случайное число (`Server Random`) и свой **SSL/TLS-сертификат** (в котором содержится публичный ключ сервера, домен и цифровая подпись центра сертификации).
  3. **Проверка сертификата клиентом**: браузер проверяет:
     - не истёк ли срок действия сертификата;
     - совпадает ли домен в сертификате с адресом в строке браузера;
     - подписан ли сертификат доверенным **Центром Сертификации (CA — Certificate Authority)** (Let's Encrypt, DigiCert и др.), чей корневой сертификат вшит в ОС/браузер.
  4. **Генерация сессионного ключа**: стороны с помощью алгоритма Диффи — Хеллмана на эллиптических кривых (**ECDHE**) вырабатывают общий симметричный ключ (обеспечивая **Perfect Forward Secrecy** — даже если приватный ключ сервера украдут в будущем, расшифровать старый записанный трафик не получится).
  5. **Finished**: стороны обмениваются зашифрованными тестовыми сообщениями. Далее идёт обычный HTTP внутри TLS.
- **SSL Termination (Терминация SSL)**: на практике PHP-FPM не занимается расшифровкой HTTPS. Этим занимается обратный прокси (**Nginx**, Cloudflare или Load Balancer), который расшифровывает трафик и передаёт чистый HTTP в PHP-FPM по внутренней сети, добавляя заголовок `X-Forwarded-Proto: https`.
- **HSTS (HTTP Strict Transport Security)**: заголовок `Strict-Transport-Security: max-age=31536000; includeSubDomains`, который запрещает браузеру делать незашифрованные `http://` запросы к сайту (защита от атаки SSL Stripping).

### Theory (🇬🇧 English)

- **HTTPS = HTTP over TLS**: Provides **Encryption** (privacy), **Integrity** (tamper detection), and **Authentication** (server identity verification via X.509 certificates signed by trusted Certificate Authorities).
- **Hybrid Encryption**: Uses asymmetric cryptography (ECDHE/RSA) during the **TLS Handshake** to verify identity and securely negotiate a shared key, then switches to fast symmetric encryption (AES-GCM / ChaCha20) for bulk HTTP payload transfer.
- **Perfect Forward Secrecy (PFS)**: Achieved via ephemeral key exchange (ECDHE), ensuring past recorded sessions cannot be decrypted even if the server's long-term private key is later compromised.
- **SSL Termination**: Reverse proxies (Nginx/ALB) handle TLS decryption and forward plain HTTP to backend workers (PHP-FPM) with `X-Forwarded-Proto: https`.

### Схема TLS Handshake

```mermaid
sequenceDiagram
    participant C as Клиент (Browser)
    participant S as Сервер (Nginx)

    C->>S: 1. ClientHello (TLS 1.3, Cipher Suites, Client Random)
    S-->>C: 2. ServerHello (Chosen Cipher, Server Random)
    S-->>C: 3. Certificate (Public Key + CA Signature)
    Note over C: Проверка цепочки CA, домена и срока действия
    C->>S: 4. Key Exchange (ECDHE) + ChangeCipherSpec
    S-->>C: 5. ChangeCipherSpec + Finished
    Note over C,S: Симметричный сессионный ключ готов (AES)
    C->>S: Зашифрованный HTTP-запрос
    S-->>C: Зашифрованный HTTP-ответ
````

## 4. Структура HTTP-сообщения, Content-Type и заголовки

> [!abstract] Коротко
> 
> HTTP-сообщение состоит из стартовой строки, набора заголовков (`Key: Value`), пустой строки-разделителя и опционального тела (body).

### Теория (🇷🇺 Русский & 🇬🇧 English)

- **Формат Request (Запроса)**:
    
    
    ```HTTP
    POST /api/v1/users HTTP/1.1
    Host: example.com
    Content-Type: application/json
    Accept: application/json
    Authorization: Bearer eyJhbGciOiJIUzI1Ni...
    
    {"name": "Alex", "email": "alex@example.com"}
    ```
    
- **Формат Response (Ответа)**:
    
    
    ```HTTP
    HTTP/1.1 201 Created
    Content-Type: application/json; charset=utf-8
    Location: /api/v1/users/42
    Cache-Control: no-store
    
    {"id": 42, "name": "Alex"}
    ```
    
- **Основные типы `Content-Type` при отправке данных**:
    
    - `application/json`: стандарт для REST API (в PHP читается из `php://input`, в Laravel — через `$request->all()` / `$request->input()`).
        
    - `application/x-www-form-urlencoded`: стандартная HTML-форма без файлов (`key1=val1&key2=val2`, попадает в `$_POST`).
        
    - `multipart/form-data`: форма с загрузкой бинарных файлов (разделяется строкой `boundary`, файлы попадают в `$_FILES`).
        
- **Ключевые заголовки**:
    
    - `Host`: обязателен в HTTP/1.1 (позволяет одному IP обслуживать много сайтов).
        
    - `User-Agent`: информация о клиенте (браузер, ОС, версия приложения).
        
    - `Accept` / `Accept-Language` / `Accept-Encoding`: согласование контента (формат ответа, язык, сжатие `gzip`/`br`).
        
    - `X-Forwarded-For` / `X-Real-IP`: реальный IP-адрес клиента за балансировщиком/Nginx.
        

## 5. HTTP Request Methods: Методы, безопасность и идемпотентность

> [!abstract] Коротко 
> Методы задают действие над ресурсом. На собеседованиях обязательно спрашивают различия между `PUT` и `PATCH`, а также понятия **безопасности** и **идемпотентности**.

### Теория (🇷🇺 Русский)

- **Безопасный метод (Safe)**: метод, который предназначен только для чтения и не изменяет состояние сервера.
    
- **Идемпотентный метод (Idempotent)**: многократное выполнение одного и того же запроса даёт **точно такой же эффект на состояние сервера**, что и однократное выполнение. Критично для безопасных повторных попыток (retries) при сбоях сети.
    

|**Метод**|**Описание**|**Безопасный**|**Идемпотентный**|**Кэшируемый**|**Тело запроса**|
|---|---|---|---|---|---|
|**GET**|Запрос представления ресурса|Да|Да|Да|Нет|
|**HEAD**|Запрос только заголовков ответа (без тела)|Да|Да|Да|Нет|
|**OPTIONS**|Запрос поддерживаемых методов и CORS-политик|Да|Да|Нет|Нет|
|**PUT**|Полная замена ресурса по указанному URI|Нет|Да|Нет|Да|
|**DELETE**|Удаление указанного ресурса|Нет|Да|Нет|Опционально|
|**POST**|Создание нового ресурса или запуск операции|Нет|**Нет**|Почти нет|Да|
|**PATCH**|Частичное изменение полей ресурса|Нет|**Нет***|Нет|Да|

### Theory (🇬🇧 English)

- **Safe Method**: Read-only operation that does not modify server state (`GET`, `HEAD`, `OPTIONS`).
    
- **Idempotent Method**: Repeating the exact same request multiple times produces the **same server state** as executing it once (`GET`, `HEAD`, `OPTIONS`, `PUT`, `DELETE`).
    
- **`PUT` vs `PATCH`**: `PUT` replaces the entire resource representation (idempotent), whereas `PATCH` applies partial updates (not strictly idempotent if applying relative deltas like `{"increment": 1}`).
    
- **Making `POST` Idempotent**: Achieved via an `Idempotency-Key` header (UUID) stored in Redis/DB so duplicate network retries return the cached response of the first execution without duplicating side effects (e.g., double charging).
    

## 6. Response Codes: Коды состояния HTTP

> [!abstract] Коротко 
> Трёхзначный код в первой строке ответа сервера, сообщающий клиенту результат обработки запроса: `1xx` (информация), `2xx` (успех), `3xx` (перенаправление), `4xx` (ошибка клиента), `5xx` (ошибка сервера).

### Теория (🇷🇺 Русский & 🇬🇧 English)

|**Код**|**Название**|**Смысл (🇷🇺)**|**Meaning (🇬🇧)**|
|---|---|---|---|
|**101**|Switching Protocols|Переключение протокола (например, на WebSockets)|Protocol upgrade (e.g., HTTP to WebSockets)|
|**200**|OK|Успешный запрос|Standard successful request|
|**201**|Created|Ресурс успешно создан (обычно на `POST`)|Resource created (typically after `POST`)|
|**202**|Accepted|Запрос принят в фоновую обработку (очередь)|Accepted for asynchronous processing|
|**204**|No Content|Успешно, тело ответа отсутствует (обычно на `DELETE`)|Success with empty response body|
|**301**|Moved Permanently|Постоянный редирект (кэшируется, может сменить `POST` на `GET`)|Permanent redirect (cached; may change `POST` to `GET`)|
|**302**|Found|Временный редирект (исторически меняет `POST` на `GET`)|Temporary redirect|
|**304**|Not Modified|Ресурс не изменился (клиент берёт из кэша)|Resource unchanged; use cached copy|
|**307 / 308**|Temp / Perm Redirect|Временный / постоянный редирект **с сохранением метода и тела**|Redirect strictly preserving HTTP method and body|
|**400**|Bad Request|Синтаксическая ошибка в запросе (битый JSON)|Malformed request syntax|
|**401**|Unauthorized|Не аутентифицирован (нет или неверный токен/сессия)|Unauthenticated (missing/invalid credentials)|
|**403**|Forbidden|Аутентифицирован, но нет прав на это действие|Authenticated, but forbidden (authorization error)|
|**404**|Not Found|Ресурс не найден|Resource not found|
|**405**|Method Not Allowed|Метод не разрешён для этого URL|HTTP method unsupported for endpoint|
|**409**|Conflict|Конфликт состояния (дубликат уникального поля, конфликт версий)|State conflict (e.g., duplicate record)|
|**422**|Unprocessable Entity|Синтаксис верный, но не прошла валидация данных|Valid syntax, but semantic/validation errors|
|**429**|Too Many Requests|Превышен лимит запросов (Rate Limit)|Rate limit exceeded|
|**500**|Internal Server Error|Необработанная ошибка/исключение на сервере|Unhandled server exception|
|**502**|Bad Gateway|Прокси (Nginx) получил плохой ответ от PHP-FPM (воркер упал)|Reverse proxy received invalid response from upstream|
|**503**|Service Unavailable|Сервер перегружен или на техобслуживании|Server overloaded or in maintenance mode|
|**504**|Gateway Timeout|Прокси не дождался ответа от PHP-FPM за отведённый таймаут|Upstream timed out|

## 7. HTTP Cookies, Сессии и JWT

> [!abstract] Коротко 
> Cookies — механизм хранения небольших данных на стороне браузера (до ~4 КБ), который автоматически прикрепляется к запросам. Используется для идентификации сессий и хранения токенов.

### Теория (🇷🇺 Русский)

- **Как работают HTTP Cookies**:
    
    1. Сервер присылает заголовок `Set-Cookie: session_id=xyz123; Path=/; HttpOnly; Secure; SameSite=Lax`.
        
    2. Браузер сохраняет куку и при каждом последующем запросе на подходящий домен и путь автоматически добавляет заголовок `Cookie: session_id=xyz123`.
        
- **Все атрибуты `Set-Cookie`**:
    
    - `Expires` / `Max-Age`: срок жизни куки. Если не указаны — кука является **сессионной (Session Cookie)** и удаляется при закрытии браузера.
        
    - `Domain`: для каких доменов/поддоменов отправлять куку (например, `Domain=.example.com` сделает куку доступной на всех поддоменах).
        
    - `Path`: ограничивает отправку куки конкретным путём (например, `Path=/admin`).
        
    - `HttpOnly`: запрещает чтение/запись куки через JavaScript (`document.cookie`) — **защита от XSS**.
        
    - `Secure`: передавать куку только по HTTPS.
        
    - `SameSite`: защита от **CSRF**:
        
        - `Strict`: не отправлять при любых переходах со сторонних сайтов.
            
        - `Lax` (дефолт в браузерах): отправлять при обычном переходе по ссылке (`GET`), но блокировать при кросс-сайтовых `POST`/AJAX-запросах.
            
        - `None`: отправлять всегда (обязательно требует флаг `Secure`).
            
- **Сессии (Stateful) vs JWT (Stateless)**:
    
    - _Сессии_: в куке только случайный ID, данные лежат в Redis/БД на сервере. Легко мгновенно разлогинить пользователя, но нужен запрос в Redis на каждый HTTP-запрос.
        
    - _JWT (`Header.Payload.Signature`)_: вся информация внутри подписанного токена. Payload только закодирован в Base64 (не зашифрован!), поэтому секреты туда не кладём. Отзыв токена до истечения срока (`exp`) требует чёрного списка в Redis.
        

### Theory (🇬🇧 English)

- **HTTP Cookies Lifecycle**: Server sets a cookie via `Set-Cookie`; browser automatically includes it in subsequent matching requests via `Cookie`.
    
- **Cookie Attributes**:
    
    - `Max-Age` / `Expires`: Persistence duration (omitting both creates a session cookie cleared on browser close).
        
    - `Domain` & `Path`: Scope restrictions defining which hosts and URL paths receive the cookie.
        
    - `HttpOnly`: Blocks `document.cookie` access in JS (mitigates XSS session hijacking).
        
    - `Secure`: Restricts transmission to HTTPS only.
        
    - `SameSite` (`Strict` / `Lax` / `None`): Controls cross-site cookie sending (mitigates CSRF).
        

## 8. Эволюция протокола: HTTP/1.1, HTTP/2 и HTTP/3

> [!abstract] Коротко
> 
> HTTP/1.1 — текстовый и последовательный; HTTP/2 — бинарный с мультиплексированием поверх одного TCP; HTTP/3 — работает поверх QUIC (UDP), убирая блокировки на уровне TCP.

### Теория (🇷🇺 Русский & 🇬🇧 English)

|**Характеристика**|**HTTP/1.1**|**HTTP/2**|**HTTP/3**|
|---|---|---|---|
|**Формат данных**|Текстовый|Бинарный (фреймы)|Бинарный (фреймы)|
|**Транспорт**|TCP|TCP|**QUIC (поверх UDP)**|
|**Мультиплексирование**|Нет (браузер открывает до 6 TCP-соединений на домен)|Да (много потоков в 1 TCP-соединении)|Да (независимые QUIC-потоки)|
|**HOL Blocking**|На уровне HTTP и TCP|Только на уровне TCP (потеря 1 пакета тормозит все потоки)|**Полностью отсутствует**|
|**Сжатие заголовков**|Нет|HPACK|QPACK|
|**Рукопожатие (HTTPS)**|2–3 RTT (TCP + TLS)|2–3 RTT|**0–1 RTT** (TLS 1.3 встроен в QUIC)|

## 9. HTTP-кэширование и условные запросы

> [!abstract] Коротко
> 
> Позволяет не гонять неизменившиеся данные по сети через директивы `Cache-Control` и условные запросы (`ETag` / `Last-Modified` -> `304 Not Modified`).

### Теория (🇷🇺 Русский & 🇬🇧 English)

- **`Cache-Control`**:
    
    - `max-age=3600`: ресурс свежий 1 час; браузер берёт его локально без запроса на сервер.
        
    - `public` / `private`: `public` — можно кэшировать в CDN/прокси; `private` — только в браузере пользователя.
        
    - `no-cache`: кэшировать можно, но перед использованием обязательно провалидировать на сервере.
        
    - `no-store`: полный запрет на кэширование (для конфиденциальных данных).
        
- **Валидация (Conditional Requests)**:
    
    - Сервер отдаёт `ETag: "hash123"` (или `Last-Modified`).
        
    - Клиент при повторном обращении шлёт `If-None-Match: "hash123"` (или `If-Modified-Since`).
        
    - Если данные не менялись, сервер возвращает `304 Not Modified` без тела ответа.
        

## 10. Безопасность HTTP: SOP, CORS и Security-заголовки

> [!abstract] Коротко
> 
> **SOP** запрещает кросс-доменные запросы в браузере; **CORS** разрешает их через заголовки сервера (с предварительным `OPTIONS` для сложных запросов).

### Теория (🇷🇺 Русский & 🇬🇧 English)

- **Same-Origin Policy (SOP)**: запрещает скрипту с одного Origin (**Схема + Хост + Порт**) читать ответы с другого Origin.
    
- **CORS (Cross-Origin Resource Sharing)**:
    
    - Для сложных запросов (`PUT`, `DELETE`, `application/json`, `Authorization`) браузер сначала отправляет **Preflight-запрос `OPTIONS`**.
        
    - Сервер должен вернуть `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers`.
        
    - Если передаются куки (`credentials: 'include'`), `Access-Control-Allow-Origin` нельзя ставить в `*` — нужно указывать точный домен и `Access-Control-Allow-Credentials: true`.
        
- **Security-заголовки**: `Strict-Transport-Security` (HSTS), `Content-Security-Policy` (CSP против XSS), `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY` (против Clickjacking).
    

## 11. Проектирование API: REST, GraphQL, gRPC, WebSockets

> [!abstract] Коротко
> 
> Выбор архитектурного стиля зависит от задачи: REST — для стандартных публичных API, GraphQL — для гибких фронтендов, gRPC — для быстрых внутренних микросервисов, WebSockets — для real-time.

|**Стиль**|**Протокол и формат**|**Плюсы**|**Минусы**|
|---|---|---|---|
|**REST**|HTTP + JSON, ресурсы в URL|Простота, кэширование HTTP, понятные коды|Over-fetching / Under-fetching|
|**GraphQL**|HTTP (`POST /graphql`) + JSON|Клиент запрашивает только нужные поля|Сложное кэширование, риск N+1 запросов|
|**gRPC**|HTTP/2 + Protobuf (бинарный)|Максимальная скорость, строгие контракты|Нужен gRPC-Web для браузеров|
|**WebSockets**|TCP (после HTTP `101`)|Двусторонний real-time обмен без polling|Сложность масштабирования stateful-соединений|

## 12. Шпаргалка перед собеседованием

> [!important] Чек-лист по всем темам HTTP:
> 
> 1. **HTTP Basics**: Stateless-протокол; URL включает схему, хост, порт, путь, query и `#fragment` (который не уходит на сервер).
>     
> 2. **HTTPS / TLS**: Асимметричное шифрование (ECDHE + сертификат CA) нужно только для проверки подлинности и согласования ключа; сам трафик шифруется быстрым симметричным шифром (AES).
>     
> 3. **Request Methods**: Безопасные (`GET, HEAD, OPTIONS`) не меняют данные; идемпотентные (`GET, PUT, DELETE`) дают одинаковый результат при повторе[cite: 2]; `POST` делают идемпотентным через `Idempotency-Key`[cite: 2].
>     
> 4. **Response Codes**: `201` (создано), `204` (успешно без тела), `304` (из кэша), `401` (не залогинен), `403` (нет прав), `422` (ошибка валидации), `502` (упал PHP-FPM), `504` (таймаут PHP-FPM)[cite: 2].
>     
> 5. **HTTP Cookies**: `HttpOnly` защищает от XSS, `Secure` — только HTTPS, `SameSite=Lax/Strict` — от CSRF; отсутствие `Max-Age`/`Expires` делает куку сессионной[cite: 2].
>