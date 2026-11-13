---
custom-width: 80
---
## Оглавление

1. [[#1. Что такое HTTP, модель клиент-сервер и стек TCP/IP]]
2. [[#2. Жизненный цикл HTTP-запроса: DNS, TCP, TLS]]
3. [[#3. Структура HTTP-сообщения и заголовки]]
4. [[#4. Методы HTTP, безопасность и идемпотентность]]
5. [[#5. Коды состояния HTTP (Status Codes)]]
6. [[#6. Эволюция протокола: HTTP/1.1, HTTP/2 и HTTP/3]]
7. [[#7. Состояние в Stateless-протоколе: Cookies, Сессии и JWT]]
8. [[#8. HTTP-кэширование и условные запросы]]
9. [[#9. Безопасность HTTP: HTTPS, CORS, CSP, CSRF]]
10. [[#10. Проектирование API: REST, GraphQL, gRPC, WebSockets]]
11. [[#11. Шпаргалка перед собеседованием]]

---

## 1. Что такое HTTP, модель клиент-сервер и стек TCP/IP

> [!abstract] Коротко
> HTTP — протокол прикладного уровня для передачи гипертекста и данных, работающий по схеме «запрос — ответ» поверх транспортного протокола (TCP в HTTP/1.1–2, UDP/QUIC в HTTP/3). Главная особенность — **stateless** (сервер не помнит предыдущие запросы).

### Теория (🇷🇺 Русский)

- **Что такое HTTP (HyperText Transfer Protocol)**: протокол прикладного уровня (7-й уровень модели OSI / 4-й уровень TCP/IP), определяющий правила, по которым клиент (браузер, мобильное приложение, другой сервис) запрашивает ресурсы, а сервер их отдаёт.
- **Модель «Клиент — Сервер»**: инициатором общения всегда выступает клиент. Сервер пассивно слушает порт (обычно `80` для HTTP и `443` для HTTPS) и отвечает на входящие запросы.
- **Stateless (без сохранения состояния)**: каждый HTTP-запрос абсолютно независим от предыдущего. Сервер не хранит контекст между запросами на уровне самого протокола. Это упрощает горизонтальное масштабирование (любой сервер за балансировщиком может обработать любой запрос), но требует искусственных механизмов для «узнавания» пользователя (Cookies, сессии, токены).
- **Место в стеке протоколов**:
  - *Канальный уровень (Link)*: Ethernet, Wi-Fi (MAC-адреса, передача кадров).
  - *Сетевой уровень (Internet)*: IP (маршрутизация пакетов между сетями по IP-адресам).
  - *Транспортный уровень (Transport)*: TCP (надёжная доставка с гарантией порядка и повторной отправкой) или UDP (быстрая доставка без гарантий и установки соединения).
  - *Прикладной уровень (Application)*: HTTP, DNS, TLS, SSH.

### Theory (🇬🇧 English)

- **What is HTTP (HyperText Transfer Protocol)**: An application-layer protocol (Layer 7 in OSI / Layer 4 in TCP/IP) defining how clients request resources and how servers respond.
- **Client-Server Model**: Communication is always initiated by the client. The server passively listens on a port (`80` for HTTP, `443` for HTTPS) and responds to incoming requests.
- **Stateless Nature**: Every HTTP request is completely independent; the protocol itself retains no memory of previous requests. While this makes horizontal scaling easy (any backend node behind a load balancer can handle any request), it requires mechanisms like Cookies, Sessions, or Tokens to maintain user state.
- **Protocol Stack**:
  - *Link Layer*: Ethernet, Wi-Fi (MAC addressing).
  - *Internet Layer*: IP (packet routing via IP addresses).
  - *Transport Layer*: TCP (reliable, ordered byte-stream delivery) or UDP (connectionless, low-latency datagrams).
  - *Application Layer*: HTTP, DNS, TLS.

### Вопросы и ответы

> [!question] Почему HTTP сделали stateless и какие у этого плюсы и минусы?
> 
> **Зачем**: протокол создавался для передачи статических документов, где хранить состояние на сервере было избыточно и дорого.
> 
> | Плюсы | Минусы |
> | :--- | :--- |
> | Простое горизонтальное масштабирование (любой сервер обработает запрос) | Приходится передавать идентификатор состояния (куку или токен) в **каждом** запросе |
> | Отказоустойчивость: падение одного узла не теряет контекст протокола | Для хранения сессий в кластере нужно внешнее общее хранилище (Redis/БД) |
> | Упрощение кэширования на прокси-серверах и CDN | Увеличение размера заголовков при передаче больших токенов |

---

## 2. Жизненный цикл HTTP-запроса: DNS, TCP, TLS

> [!abstract] Коротко
> Прежде чем отправится первый байт HTTP, клиент должен узнать IP-адрес (DNS), открыть надёжный канал (TCP 3-way handshake) и зашифровать его (TLS handshake).

### Теория (🇷🇺 Русский)

1. **DNS Lookup (Разрешение доменного имени)**: перевод человекочитаемого домена (`api.example.com`) в IP-адрес (`93.184.216.34`).
   - Проверка кэша браузера → кэша ОС (`/etc/hosts`) → кэша роутера → DNS-резолвера провайдера.
   - Если в кэше нет, резолвер делает рекурсивный обход: **Root DNS (`.`)** → **TLD DNS (`.com`)** → **Authoritative DNS (`example.com`)**.
2. **TCP Handshake (Трёхстороннее рукопожатие)**: установка надёжного соединения:
   - `SYN`: клиент отправляет пакет с начальным порядковым номером (ISN).
   - `SYN-ACK`: сервер подтверждает получение и отправляет свой ISN.
   - `ACK`: клиент подтверждает получение ISN сервера. Соединение открыто.
3. **TLS Handshake (для HTTPS)**: создание защищённого канала поверх TCP:
   - *ClientHello*: клиент шлёт поддерживаемую версию TLS, список шифров (Cipher Suites) и случайное число.
   - *ServerHello*: сервер выбирает шифр, отправляет своё случайное число и **SSL/TLS-сертификат** (с публичным ключом).
   - *Проверка сертификата*: клиент проверяет цепочку доверия до корневого центра сертификации (CA), срок действия и совпадение домена.
   - *Обмен ключами*: стороны вырабатывают общий **симметричный сессионный ключ** (через алгоритм Диффи — Хеллмана / ECDHE). Симметричное шифрование используется дальше, так как оно в сотни раз быстрее асимметричного.
4. **HTTP Request & Response**: отправка запроса и получение ответа.
5. **Keep-Alive или закрытие (`FIN`)**: в HTTP/1.1+ TCP-соединение не закрывается сразу (`Connection: keep-alive`), чтобы не тратить время на повторные TCP/TLS рукопожатия.

### Theory (🇬🇧 English)

1. **DNS Lookup**: Resolving a domain name (`api.example.com`) into an IP address:
   - Checks Browser cache → OS cache (`/etc/hosts`) → Router cache → ISP Recursive Resolver.
   - On cache miss, the resolver queries: **Root DNS** → **TLD DNS (`.com`)** → **Authoritative DNS (`example.com`)**.
2. **TCP 3-Way Handshake**: Establishing a reliable connection:
   - `SYN`: Client sends a synchronize packet with its Initial Sequence Number (ISN).
   - `SYN-ACK`: Server acknowledges client's ISN and sends its own ISN.
   - `ACK`: Client acknowledges server's ISN. Connection is established.
3. **TLS Handshake (for HTTPS)**: Securing the channel:
   - *ClientHello*: Client sends supported TLS versions, cipher suites, and client random.
   - *ServerHello*: Server selects cipher suite, sends server random, and provides its **SSL/TLS Certificate** (containing the public key).
   - *Verification & Key Exchange*: Client validates the certificate chain against trusted Certificate Authorities (CAs). Both sides derive a shared **symmetric session key** (via ECDHE) for fast bulk encryption.
4. **HTTP Exchange & Keep-Alive**: Request is transmitted, response is returned, and the TCP socket is kept open (`keep-alive`) for subsequent requests.

### Схема рукопожатия (HTTPS)

```mermaid
sequenceDiagram
    participant C as Клиент (Browser)
    participant D as DNS Server
    participant S as Сервер (Nginx)

    C->>D: DNS Query (example.com)
    D-->>C: IP Address (93.184.216.34)
    Note over C,S: TCP 3-Way Handshake
    C->>S: SYN
    S-->>C: SYN-ACK
    C->>S: ACK
    Note over C,S: TLS 1.3 Handshake (1-RTT)
    C->>S: ClientHello + Key Share
    S-->>C: ServerHello + Certificate + Key Share
    Note over C,S: Зашифрованный канал готов
    C->>S: HTTP Request (GET /api/users)
    S-->>C: HTTP Response (200 OK + JSON)
````

## 3. Структура HTTP-сообщения и заголовки

> [!abstract] Коротко
> 
> HTTP-сообщение состоит из стартовой строки, заголовков (`Key: Value`), пустой строки-разделителя и опционального тела (payload).

### Теория (🇷🇺 Русский)

- **Структура запроса (Request)**:
    
    1. _Стартовая строка (Request Line)_: `POST /api/v1/orders HTTP/1.1` (Метод, URI, версия протокола).
        
    2. _Заголовки (Headers)_: метаданные запроса (`Host`, `User-Agent`, `Accept`, `Authorization`, `Content-Type`).
        
    3. _Пустая строка (`\r\n`)_: отделяет заголовки от тела.
        
    4. _Тело (Body)_: передаваемые данные (JSON, form-data, бинарные файлы).
        
- **Структура ответа (Response)**:
    
    1. _Строка состояния (Status Line)_: `HTTP/1.1 201 Created` (Версия, статус-код, текстовая расшифровка).
        
    2. _Заголовки ответа_: `Content-Type`, `Content-Length`, `Set-Cookie`, `Cache-Control`, `Location`.
        
    3. _Пустая строка_ и _Тело ответа_.
        
- **Важные заголовки для Backend-разработчика**:
    
    - `Host`: обязателен в HTTP/1.1. Позволяет одному серверу (Nginx) обслуживать десятки разных доменов на одном IP (Virtual Hosting).
        
    - `Content-Type`: формат тела (`application/json`, `application/x-www-form-urlencoded`, `multipart/form-data`).
        
    - `Accept`: какие форматы ответа готов принять клиент (`application/json`).
        
    - `Authorization`: передача учётных данных (`Bearer <token>`, `Basic <base64>`).
        
    - `X-Forwarded-For` / `X-Real-IP`: реальный IP клиента, который добавляет обратный прокси (Nginx/Cloudflare), так как для PHP-FPM прямым клиентом является сам прокси.
        
    - `X-Request-ID` / `X-Correlation-ID`: сквозной идентификатор запроса для трассировки в логах микросервисов.
        

### Theory (🇬🇧 English)

- **Request Structure**:
    
    1. _Request Line_: `POST /api/v1/orders HTTP/1.1` (Method, Request Target, HTTP Version).
        
    2. _Headers_: Key-value metadata (`Host`, `Accept`, `Authorization`, `Content-Type`).
        
    3. _Empty Line (`\r\n`)_: Separates headers from the body.
        
    4. _Body_: Optional payload (JSON, form data, binary stream).
        
- **Response Structure**:
    
    1. _Status Line_: `HTTP/1.1 201 Created` (HTTP Version, Status Code, Reason Phrase).
        
    2. _Headers_: `Content-Type`, `Set-Cookie`, `Cache-Control`, `Location`.
        
    3. _Empty Line_ and _Response Body_.
        
- **Critical Backend Headers**:
    
    - `Host`: Mandatory in HTTP/1.1; enables virtual hosting (multiple domains sharing a single IP).
        
    - `Content-Type` / `Accept`: Content negotiation headers defining payload media types.
        
    - `Authorization`: Carries credentials (`Bearer <token>`).
        
    - `X-Forwarded-For` / `X-Forwarded-Proto`: Added by reverse proxies (Nginx, Load Balancers) to preserve the original client IP and protocol (`https`).
        

## 4. Методы HTTP, безопасность и идемпотентность

> [!abstract] Коротко
> 
> Методы определяют семантику действия над ресурсом. Критически важно различать **безопасные** (не меняют данные) и **идемпотентные** (повторный вызов даёт тот же результат) методы.

### Теория (🇷🇺 Русский)

- **Безопасный метод (Safe)**: выполнение запроса не изменяет состояние сервера (операция только для чтения). Допустимы логирование и кэширование, но не изменение записей в БД.
    
- **Идемпотентный метод (Idempotent)**: многократное выполнение одного и того же запроса приводит к **точно такому же состоянию сервера**, как и его однократное выполнение. Это критично для надёжности сетей: если клиент отправил запрос и не получил ответ из-за обрыва связи, идемпотентный запрос можно безопасно повторить (retry).
    

|**Метод**|**Назначение**|**Безопасный**|**Идемпотентный**|**Кэшируемый**|**Тело запроса**|
|---|---|---|---|---|---|
|**GET**|Получение представления ресурса|Да|Да|Да|Нет (не реком.)|
|**HEAD**|Получение только заголовков (проверка размера/существования)|Да|Да|Да|Нет|
|**OPTIONS**|Запрос поддерживаемых методов и политик CORS|Да|Да|Нет|Нет|
|**PUT**|Полная замена ресурса по известному URI|Нет|Да|Нет|Да|
|**DELETE**|Удаление ресурса|Нет|Да|Нет|Опционально|
|**POST**|Создание нового ресурса или запуск процесса|Нет|**Нет**|Почти нет|Да|
|**PATCH**|Частичное обновление ресурса|Нет|**Нет***|Нет|Да|

### Theory (🇬🇧 English)

- **Safe Method**: Read-only operation that does not alter server state.
    
- **Idempotent Method**: Making multiple identical requests leaves the server in the **exact same state** as making a single request. Crucial for network reliability, allowing clients to safely retry failed requests without unintended side effects.
    

|**Method**|**Purpose**|**Safe**|**Idempotent**|**Cacheable**|**Request Body**|
|---|---|---|---|---|---|
|**GET**|Retrieve a resource representation|Yes|Yes|Yes|No|
|**HEAD**|Retrieve response headers only|Yes|Yes|Yes|No|
|**OPTIONS**|Describe communication options (CORS)|Yes|Yes|No|No|
|**PUT**|Full replacement of a resource|No|Yes|No|Yes|
|**DELETE**|Remove a resource|No|Yes|No|Optional|
|**POST**|Create a subordinate resource / trigger action|No|**No**|Rarely|Yes|
|**PATCH**|Partial modification of a resource|No|**No***|No|Yes|

### Вопросы и ответы

> [!question] Почему `DELETE` считается идемпотентным, если первый запрос вернёт `200 OK`, а второй — `404 Not Found`?
> 
> **Объяснение**: идемпотентность определяется **состоянием сервера**, а не кодом ответа. После первого `DELETE /users/5` пользователя с ID 5 в базе нет. После второго `DELETE /users/5` пользователя в базе по-прежнему нет — состояние сервера не изменилось.

> [!question] Как сделать операцию создания платежа (`POST /payments`) идемпотентной?
> 
> **Решение**: использовать **ключ идемпотентности (Idempotency Key)**.
> 
> 1. Клиент генерирует UUID и передаёт его в заголовке `Idempotency-Key: 550e8400-e29b-...`.
>     
> 2. Сервер проверяет наличие этого ключа в Redis или таблице БД (с уникальным индексом).
>     
> 3. Если ключ новый — сервер проводит платёж, сохраняет результат ответа под этим ключом и возвращает клиенту.
>     
> 4. Если запрос пришёл повторно с тем же ключом — сервер не списывает деньги повторно, а сразу возвращает сохранённый ответ первого запроса.
>     

## 5. Коды состояния HTTP (Status Codes)

> [!abstract] Коротко
> 
> Трёхзначные числа, разделённые на 5 классов: `1xx` (информация), `2xx` (успех), `3xx` (редирект), `4xx` (ошибка клиента), `5xx` (ошибка сервера).

### Теория (🇷🇺 Русский & 🇬🇧 English)

|**Код**|**Название**|**Описание (🇷🇺)**|**Description (🇬🇧)**|
|---|---|---|---|
|**200**|OK|Стандартный успешный ответ|Standard successful response|
|**201**|Created|Ресурс создан (в ответ на `POST`), часто с заголовком `Location`|Resource created; often includes `Location` header|
|**202**|Accepted|Запрос принят в обработку (асинхронно через очередь), но ещё не выполнен|Request accepted for async processing|
|**204**|No Content|Успешно выполнено, тело ответа отсутствует (типично для `DELETE`)|Success with no response payload|
|**301**|Moved Permanently|Постоянный редирект. Кэшируется браузером, может менять `POST` на `GET`|Permanent redirect; cached, may downgrade `POST` to `GET`|
|**302**|Found|Временный редирект (исторически также меняет `POST` на `GET`)|Temporary redirect (historically changes method to `GET`)|
|**304**|Not Modified|Ресурс не изменился, клиент может взять его из своего кэша|Resource unchanged; use cached version|
|**307 / 308**|Temporary / Permanent Redirect|Временный / постоянный редирект с **строгим сохранением** метода и тела|Redirect strictly preserving HTTP method and body|
|**400**|Bad Request|Синтаксическая ошибка (битый JSON, неверный формат заголовка)|Malformed request syntax (e.g., invalid JSON)|
|**401**|Unauthorized|Клиент не аутентифицирован (нет токена, токен протух)|Unauthenticated (missing or invalid credentials)|
|**403**|Forbidden|Клиент аутентифицирован, но не имеет прав доступа (ошибка авторизации)|Authenticated, but lacks permissions (authorization failure)|
|**404**|Not Found|Ресурс не найден|Resource does not exist|
|**405**|Method Not Allowed|URI существует, но метод не поддерживается (например, `DELETE` на `/login`)|Endpoint exists, but HTTP method is unsupported|
|**409**|Conflict|Конфликт состояния (например, дубликат email или конфликт версий)|State conflict (e.g., duplicate unique key)|
|**422**|Unprocessable Entity|Синтаксис верный (валидный JSON), но не прошла бизнес-валидация полей|Valid syntax, but semantic/validation errors|
|**429**|Too Many Requests|Превышен лимит запросов (Rate Limiting), желателен заголовок `Retry-After`|Rate limit exceeded; often paired with `Retry-After`|
|**500**|Internal Server Error|Необработанное исключение или фатальная ошибка в коде сервера|Unhandled server exception or crash|
|**502**|Bad Gateway|Reverse proxy (Nginx) получил невалидный ответ от upstream (PHP-FPM упал)|Reverse proxy received an invalid response from upstream|
|**503**|Service Unavailable|Сервер временно перегружен или закрыт на техобслуживание|Server overloaded or in maintenance mode|
|**504**|Gateway Timeout|Reverse proxy не дождался ответа от upstream за установленный таймаут|Reverse proxy timed out waiting for upstream|

### Вопросы и ответы

> [!question] В чём разница между `400 Bad Request` и `422 Unprocessable Entity`?
> 
> - **`400`**: сервер не смог распарсить сам запрос (например, невалидный синтаксис JSON `{name: }` или повреждённый multipart-запрос).
>     
> - **`422`**: запрос распарсен корректно, структура понятна, но данные не прошли правила валидации (например, `email` имеет неверный формат или `age` отрицательный). В Laravel `FormRequest` по умолчанию возвращает именно `422`.
>     

> [!question] Как диагностировать `502 Bad Gateway` и `504 Gateway Timeout` в связке Nginx + PHP-FPM?
> 
> - **`502 Bad Gateway`**: Nginx попытался связаться с PHP-FPM (через сокет или порт), но соединение оборвалось или вернуло мусор. Причины: PHP-FPM не запущен, упал по `segfault` или превышению памяти, неверный путь к `.sock` файлу, исчерпана очередь `listen.backlog`.
>     
> - **`504 Gateway Timeout`**: PHP-FPM принял запрос, но выполнял его слишком долго (например, тяжёлый SQL-запрос или зависший внешний API), и Nginx разорвал ожидание по директиве `fastcgi_read_timeout`.
>     

## 6. Эволюция протокола: HTTP/1.1, HTTP/2 и HTTP/3

> [!abstract] Коротко
> 
> HTTP/1.1 — текстовый и последовательный; HTTP/2 — бинарный с мультиплексированием поверх одного TCP; HTTP/3 — работает поверх QUIC (UDP), решая проблему блокировки на уровне TCP.

### Теория (🇷🇺 Русский)

- **HTTP/1.1 (1997)**:
    
    - Текстовый формат (легко читать глазами, но избыточен).
        
    - **Persistent Connections (`keep-alive`)**: переиспользование TCP-соединения.
        
    - **Проблема Head-of-Line (HOL) Blocking**: в рамках одного TCP-соединения запросы и ответы идут строго по очереди (FIFO). Если первый запрос «завис» на 5 секунд, остальные ждут. Чтобы обойти это, браузеры открывают по **6 параллельных TCP-соединений** на один домен.
        
    - Заголовки передаются в каждом запросе открытым текстом без сжатия.
        
- **HTTP/2 (2015)**:
    
    - **Бинарный протокол**: данные разбиваются на бинарные фреймы (frames).
        
    - **Мультиплексирование (Multiplexing)**: множество независимых потоков (streams) передаются параллельно внутри **одного TCP-соединения**. Решает проблему HOL Blocking на уровне HTTP.
        
    - **Сжатие заголовков (HPACK)**: динамическая таблица заголовков устраняет дублирование.
        
    - **Ограничение**: остаётся **TCP HOL Blocking**. Поскольку всё идёт через один TCP-поток, потеря хотя бы одного TCP-пакета заставляет ОС приостановить доставку всех потоков до повторной передачи потерянного пакета.
        
- **HTTP/3 (2022)**:
    
    - Работает поверх **QUIC (поверх UDP)** вместо TCP.
        
    - **Независимые потоки на уровне транспорта**: потеря пакета в одном потоке не блокирует остальные потоки.
        
    - **Быстрое рукопожатие (0-RTT / 1-RTT)**: TLS 1.3 встроен прямо в QUIC, установка соединения происходит за 1 шаг вместо 2–3 (TCP + TLS).
        
    - **Connection Migration**: при переключении телефона с Wi-Fi на 4G соединение не рвётся, так как идентифицируется по `Connection ID`, а не по паре IP:Port.
        

### Theory (🇬🇧 English)

- **HTTP/1.1**: Text-based protocol. Introduced `keep-alive`, but suffers from **HTTP Head-of-Line (HOL) Blocking** (requests on a single connection are processed sequentially, forcing browsers to open up to 6 parallel TCP connections per domain). Headers are uncompressed.
    
- **HTTP/2**: Binary framing layer. Introduces **Multiplexing** (multiple concurrent streams over a single TCP connection) and **HPACK** header compression. Eliminates HTTP-level HOL blocking, but still suffers from **TCP-level HOL blocking** (one dropped TCP packet stalls all multiplexed streams).
    
- **HTTP/3**: Runs over **QUIC (UDP)** instead of TCP. Solves TCP HOL blocking (packet loss affects only the individual stream), integrates TLS 1.3 natively for **0-RTT/1-RTT** handshakes, and supports **Connection Migration** across network changes.
    

### Сравнительная таблица

|**Характеристика**|**HTTP/1.1**|**HTTP/2**|**HTTP/3**|
|---|---|---|---|
|**Формат данных**|Текстовый|Бинарный|Бинарный|
|**Транспорт**|TCP|TCP|QUIC (UDP)|
|**Мультиплексирование**|Нет (6 TCP-соединений)|Да (1 TCP-соединение)|Да (независимые QUIC-потоки)|
|**HOL Blocking**|На уровне HTTP и TCP|Только на уровне TCP|Отсутствует|
|**Сжатие заголовков**|Нет|HPACK|QPACK|
|**Рукопожатие (HTTPS)**|2–3 RTT (TCP + TLS)|2–3 RTT|0–1 RTT (встроенный TLS 1.3)|

## 7. Состояние в Stateless-протоколе: Cookies, Сессии и JWT

> [!abstract] Коротко
> 
> Так как HTTP не хранит состояние, идентификация пользователя строится либо через **Stateful-сессии** (ID в Cookie + данные в Redis/БД), либо через **Stateless-токены** (подписанный JWT).

### Теория (🇷🇺 Русский)

- **HTTP Cookies**: небольшие фрагменты данных (до ~4 КБ), которые сервер отправляет в заголовке `Set-Cookie`, а браузер автоматически прикрепляет ко всем последующим запросам на этот домен в заголовке `Cookie`.
    
    - `HttpOnly`: запрещает доступ к куке из JavaScript (`document.cookie`) — **защита от кражи сессии при XSS**.
        
    - `Secure`: передавать куку только по зашифрованному HTTPS.
        
    - `SameSite`: управляет отправкой куки при кросс-сайтовых запросах — **защита от CSRF**:
        
        - `Strict`: кука не шлётся ни при каких внешних переходах.
            
        - `Lax` (дефолт в современных браузерах): шлётся при безопасной навигации (`GET` по ссылке), но блокируется для кросс-доменных `POST`/AJAX.
            
        - `None`: шлётся всегда (требует флага `Secure`).
            
- **Серверные сессии (Stateful)**: в куке лежит только случайный `session_id`, а сами данные пользователя (ID, роли) хранятся на сервере (в Redis, Memcached или БД).
    
- **JWT (JSON Web Token — Stateless)**: самодостаточный подписанный токен формата `Header.Payload.Signature` (три части в Base64Url).
    
    - _Header_: алгоритм подписи (`HS256`, `RS256`) и тип токена.
        
    - _Payload_: полезная нагрузка (claims: `sub`, `exp`, `iat`, роли). **Важно**: Payload не зашифрован, а только закодирован в Base64 — класть туда пароли или секреты нельзя!
        
    - _Signature_: криптографическая подпись `HMACSHA256(base64(header) + "." + base64(payload), secret)`. Гарантирует, что токен не был подделан.
        
    - **Схема Access + Refresh токенов**: Access-токен живёт мало (5–15 минут), Refresh-токен живёт долго (дни/недели), хранится в БД/Redis и в `HttpOnly` cookie, позволяя выпустить новую пару токенов и при необходимости отозвать доступ.
        

### Theory (🇬🇧 English)

- **Cookies**: Small key-value pairs (`Set-Cookie` / `Cookie` headers) automatically sent by browsers.
    
    - `HttpOnly`: Blocks `document.cookie` access in JS (mitigates XSS token theft).
        
    - `Secure`: Transmits only over HTTPS.
        
    - `SameSite` (`Strict` / `Lax` / `None`): Controls cross-site cookie transmission (mitigates CSRF).
        
- **Server-Side Sessions (Stateful)**: Cookie holds only an opaque `session_id`; user state resides on the server (Redis/DB). Instant revocation, minimal payload overhead.
    
- **JWT (JSON Web Token — Stateless)**: Signed token structured as `Header.Payload.Signature`.
    
    - Payload is Base64Url-encoded (readable by anyone), **not encrypted**.
        
    - Signature prevents tampering.
        
    - Uses short-lived **Access Tokens** (5–15m) paired with revocable long-lived **Refresh Tokens** stored in `HttpOnly` cookies.
        

### Вопросы и ответы

> [!question] Что выбрать для SPA/Mobile API: Cookies + Сессии или JWT?
> 
> |**Критерий**|**Сессии (Cookie + Redis)**|**JWT (Access + Refresh)**|
> |---|---|---|
> |**Отзыв доступа (Logout / Бан)**|Мгновенный (удаляем ключ в Redis)|Сложный (Access-токен валиден до истечения `exp`, нужен blacklist)|
> |**Нагрузка на хранилище**|Запрос в Redis на каждый HTTP-запрос|Проверка подписи в CPU без обращения к БД|
> |**Микросервисы**|Нужен общий Redis или сервис авторизации|Любой сервис сам проверяет подпись публичным ключом (`RS256`)|
> |**Мобильные клиенты**|Менее удобно, чем заголовки|Стандарт (`Authorization: Bearer ...`)|

## 8. HTTP-кэширование и условные запросы

> [!abstract] Коротко
> 
> HTTP-кэш позволяет не передавать неизменившиеся данные по сети. Бывает **свежим** (клиент даже не обращается к серверу) и **валидируемым** (сервер отвечает `304 Not Modified` без тела).

### Теория (🇷🇺 Русский)

- **Управляющий заголовок `Cache-Control`**:
    
    - `max-age=3600`: ресурс считается свежим 3600 секунд; в течение этого времени браузер берёт его из локального кэша без обращения к серверу.
        
    - `public` / `private`: `public` разрешает кэшировать промежуточным прокси и CDN; `private` — только в браузере конкретного пользователя (для персональных данных).
        
    - `no-cache`: **не запрещает кэшировать!** Означает: «сохрани в кэш, но перед каждым использованием обязательно спроси у сервера, не устарел ли ресурс».
        
    - `no-store`: строгий запрет на сохранение где-либо (для банковских выписок, секретов).
        
- **Валидация кэша (Условные запросы)**:
    
    - **По хэшу (`ETag`)**: сервер отдаёт заголовок `ETag: "v1-a8f9c"`. При следующем запросе клиент отправляет `If-None-Match: "v1-a8f9c"`. Если данные не изменились, сервер возвращает `304 Not Modified` с пустым телом.
        
    - **По времени (`Last-Modified`)**: сервер отдаёт дату изменения, клиент присылает `If-Modified-Since`.
        

### Theory (🇬🇧 English)

- **`Cache-Control` Directives**:
    
    - `max-age=<seconds>`: Defines freshness lifetime; browser serves directly from cache without contacting the server.
        
    - `public` vs `private`: `public` allows shared caches (CDNs/proxies); `private` restricts caching to the end-user's browser.
        
    - `no-cache`: Caches the response, but requires **revalidation** with the origin server before reuse.
        
    - `no-store`: Completely prohibits storing the response anywhere.
        
- **Cache Revalidation (Conditional Requests)**:
    
    - `ETag` + `If-None-Match`: Content-hash validation. Returns `304 Not Modified` (empty body) if unchanged.
        
    - `Last-Modified` + `If-Modified-Since`: Timestamp-based validation.
        

## 9. Безопасность HTTP: HTTPS, CORS, CSP, CSRF

> [!abstract] Коротко
> 
> Безопасность на уровне HTTP строится на шифровании трафика (HTTPS/HSTS), ограничении кросс-доменных запросов (SOP/CORS) и защите браузера через security-заголовки.

### Теория (🇷🇺 Русский)

- **SOP (Same-Origin Policy)**: базовое правило безопасности браузера. Скрипт со страницы `https://site-a.com` не может читать ответы от `https://site-b.com`, если отличается хотя бы одна часть **Origin**: **схема (протокол)**, **домен (хост)** или **порт**.
    
- **CORS (Cross-Origin Resource Sharing)**: механизм контролируемого ослабления SOP через HTTP-заголовки со стороны сервера.
    
    - _Простые запросы_ (`GET`, `HEAD`, `POST` со стандартными заголовками): браузер отправляет сразу, но блокирует чтение ответа в JS, если в ответе нет `Access-Control-Allow-Origin`.
        
    - _Сложные запросы (Preflight)_ (`PUT`, `DELETE`, `Content-Type: application/json`, заголовок `Authorization`): браузер сначала шлёт предварительный запрос **`OPTIONS`**. Сервер должен ответить заголовками `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers` (и `Access-Control-Allow-Credentials: true`, если передаются куки).
        
- **Важные Security-заголовки**:
    
    - `Strict-Transport-Security (HSTS)`: приказывает браузеру всегда обращаться к сайту только по HTTPS.
        
    - `Content-Security-Policy (CSP)`: белый список источников, откуда разрешено загружать JS, CSS, картинки (главный барьер против XSS).
        
    - `X-Content-Type-Options: nosniff`: запрещает браузеру угадывать MIME-тип файла (защита от исполнения скрипта, замаскированного под картинку).
        
    - `X-Frame-Options: DENY`: запрещает встраивание сайта в `<iframe>` (защита от Clickjacking).
        

### Theory (🇬🇧 English)

- **Same-Origin Policy (SOP)**: Browser security model restricting scripts on one origin from reading data from another origin (where Origin = **Scheme + Host + Port**).
    
- **CORS (Cross-Origin Resource Sharing)**: Server-driven headers relaxing SOP.
    
    - Non-simple requests (`PUT`, `DELETE`, `application/json`, `Authorization` header) trigger a **Preflight `OPTIONS`** request checking `Access-Control-Allow-Origin`, `-Methods`, and `-Headers`.
        
- **Key HTTP Security Headers**:
    
    - `Strict-Transport-Security (HSTS)`: Enforces HTTPS-only connections.
        
    - `Content-Security-Policy (CSP)`: Whitelists trusted sources for scripts/styles to mitigate XSS.
        
    - `X-Content-Type-Options: nosniff`: Prevents MIME-sniffing attacks.
        
    - `X-Frame-Options: DENY`: Prevents Clickjacking via `<iframe>` embedding.
        

### Вопросы и ответы

> [!question] Защищает ли CORS мой API от запросов из Postman или cURL?
> 
> **Нет.** CORS — это политика, которую исполняет **только браузер** для защиты пользователя. Любой серверный скрипт, cURL или Postman игнорирует заголовки CORS и отправляет запросы напрямую. Для защиты API нужны аутентификация, авторизация и rate limiting.

## 10. Проектирование API: REST, GraphQL, gRPC, WebSockets

> [!abstract] Коротко
> 
> REST — стандарт для публичных API (ресурсы + HTTP-методы); GraphQL решает проблему over-fetching/under-fetching; gRPC идеален для быстрых внутренних микросервисов; WebSockets — для двустороннего real-time общения.

### Теория (🇷🇺 Русский & 🇬🇧 English)

|**Подход**|**Суть и транспорт**|**Плюсы**|**Минусы**|**Когда выбирать**|
|---|---|---|---|---|
|**REST**|Ресурсы (`/users/1`), HTTP-методы, JSON поверх HTTP/1.1–3|Простота, встроенное HTTP-кэширование, понятные статус-коды|Over-fetching (лишние поля) и Under-fetching (нужно много запросов)|Публичные API, стандартные веб- и мобильные бэкенды|
|**GraphQL**|Один эндпоинт (`POST /graphql`), клиент сам описывает схему нужных полей|Нет лишних данных, строгая типизация схемы, удобно для сложных фронтендов|Сложное кэширование (всегда `200 OK`), риск тяжелых вложенных запросов (N+1)|Сложные UI с разными клиентами (Web, iOS, Android)|
|**gRPC**|Бинарный протокол **Protobuf** поверх **HTTP/2**, генерация кода по `.proto` контракту|Максимальная скорость, строгий контракт, двусторонний стриминг|Не читается глазами в браузере, нужен gRPC-Web для фронтенда|Внутреннее общение между микросервисами|
|**WebSockets**|Постоянное двустороннее (full-duplex) TCP-соединение после HTTP `101 Switching Protocols`|Мгновенная доставка событий от сервера к клиенту без polling|Stateful-соединения сложнее балансировать и масштабировать|Чаты, биржевые котировки, онлайн-игры, уведомления|

## 11. Шпаргалка перед собеседованием

> [!important] Ключевые тезисы по HTTP:
> 
> 1. **Stateless**: HTTP не хранит состояние; состояние обеспечивается через `HttpOnly; Secure; SameSite` Cookies (сессии) или `Authorization: Bearer` (JWT).
>     
> 2. **Безопасность и идемпотентность**: `GET/HEAD/OPTIONS` — безопасны и идемпотентны; `PUT/DELETE` — небезопасны, но идемпотентны; `POST` — не безопасен и не идемпотентен (лечится `Idempotency-Key`).
>     
> 3. **Статусы**: `401` (кто ты?), `403` (я знаю кто ты, но тебе нельзя), `422` (JSON валидный, но данные не прошли валидацию), `502` (упал upstream/PHP-FPM), `504` (PHP-FPM не уложился в таймаут).
>     
> 4. **HTTP/2 vs HTTP/3**: HTTP/2 убрал HTTP HOL Blocking через мультиплексирование в одном TCP, но оставил TCP HOL Blocking; HTTP/3 перешёл на QUIC (UDP) и решил проблему полностью.
>     
> 5. **CORS**: браузерный механизм; `OPTIONS` (preflight) отправляется перед сложными запросами; `Access-Control-Allow-Origin: *` несовместим с передачей кук (`Allow-Credentials: true`).
>