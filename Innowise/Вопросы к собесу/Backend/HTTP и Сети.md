---
custom-width: 80
tags: [http, networks, interview]
---
# HTTP и Сети — конспект для собеседования

> [!info] Как пользоваться конспектом
> Структура каждого раздела: **Коротко** → **Теория (RU)** → **🇬🇧 English** → **Схема / пример** → **Вопросы и ответы**. Перед собеседованием читай «Коротко», «Say it in English» и раздел 12.

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
> HTTP — протокол **прикладного уровня** для обмена данными по схеме «клиент — сервер». Он **stateless** (не помнит прошлые запросы) и работает поверх транспорта: TCP (HTTP/1.1, 2) или QUIC поверх UDP (HTTP/3).

### Теория (🇷🇺 Русский)

- **HTTP (HyperText Transfer Protocol)**: протокол запросов и ответов. В версии 1.1 сообщения текстовые, в 2 и 3 — бинарные (но семантика методов, кодов и заголовков одна и та же).
- **Модель «клиент — сервер»**: инициатор всегда клиент (браузер, мобильное приложение, другой сервис). Сервер слушает порт (`80` для HTTP, `443` для HTTPS) и возвращает ответ.
- **Stateless (без состояния)**: каждый запрос содержит всё необходимое для обработки. Состояние (сессия, авторизация) передаётся через cookies/токены. Это позволяет легко масштабировать бэкенд горизонтально.
- **URI, URL, URN**:
  - **URI** (Uniform Resource **Identifier**): общее понятие, идентификатор ресурса. URL и URN — его частные случаи.
  - **URL** (Uniform Resource **Locator**): говорит *где* ресурс и *как* его получить.
  - **URN** (Uniform Resource **Name**): постоянное имя без указания местоположения (`urn:isbn:0451450523`).

```text
https://user:pass@api.example.com:8443/v1/users?active=1#top
└─┬──┘   └───┬───┘ └──────┬──────┘└─┬─┘└───┬───┘└───┬───┘└┬─┘
scheme   userinfo        host    port   path    query  fragment
```

  - `https`: схема (протокол); `api.example.com`: хост; `:8443`: порт; `/v1/users`: путь; `?active=1`: query string; `#top`: фрагмент (якорь), который **не отправляется на сервер**, его обрабатывает только браузер.

- **Модель TCP/IP (4 уровня)**:

| Уровень | Протоколы / примеры | Что делает |
| :--- | :--- | :--- |
| **Прикладной (Application)** | HTTP, DNS, TLS, SMTP | Логика приложений |
| **Транспортный (Transport)** | **TCP**, **UDP**, QUIC | TCP: надёжно, по порядку, с повторной отправкой. UDP: быстро, без гарантий |
| **Сетевой (Internet)** | IP | Адресация и маршрутизация пакетов между сетями |
| **Канальный (Link)** | Ethernet, Wi-Fi | Передача кадров в локальной сети (MAC-адреса) |

*Модель OSI (7 уровней) — теоретическая, TCP/IP — практическая; на собеседовании часто просят назвать, на каком уровне работает протокол.*

### 🇬🇧 English

| Term | Meaning |
| :--- | :--- |
| **HTTP** | An application-layer, request-response protocol between a client and a server. |
| **Stateless** | The server doesn't keep client state between requests at the protocol level. |
| **URI** | Identifier of a resource (the umbrella term). |
| **URL** | A URI that tells you *where* the resource is and *how* to get it. |
| **URN** | A persistent name that doesn't depend on location. |
| **Fragment (`#top`)** | Handled only by the browser; never sent to the server. |
| **TCP / UDP** | Reliable, ordered transport / fast transport without guarantees. |

> [!quote] Say it in English
> «HTTP is a stateless request-response protocol. Because it's stateless, every request must contain all the information the server needs. That makes it easy to scale horizontally, because any server can handle any request.»

### Вопросы и ответы

> [!question] Чем URL отличается от URI?
>
> URI — любой идентификатор ресурса. URL — URI, который ещё и указывает, **где** ресурс находится и как его получить (схема + хост + путь). Любой URL — это URI, но не наоборот.

> [!question] Что значит «HTTP stateless» и как тогда работает авторизация?
>
> Сервер не помнит предыдущих запросов. Состояние хранится на клиенте и передаётся в каждом запросе: cookie с ID сессии или заголовок `Authorization` с токеном.

> [!question] Приходит ли `#fragment` на сервер?
>
> Нет. Он остаётся в браузере (якоря, роутинг в SPA).

---

## 2. Жизненный цикл HTTP-запроса: DNS и TCP Handshake

> [!abstract] Коротко
> Перед HTTP-запросом клиент находит IP-адрес через **DNS**, затем устанавливает **TCP-соединение** (3-way handshake), затем (для HTTPS) договаривается о шифровании по **TLS**, и только потом отправляет HTTP-запрос.

### Теория (🇷🇺 Русский)

1. **DNS Lookup (разрешение имени)**:
   - Сначала клиент ищет в кэшах: **кэш браузера → кэш ОС (и `/etc/hosts`) → кэш роутера → DNS-резолвер провайдера**.
   - Если записи нет, резолвер опрашивает иерархию: **Root (`.`) → TLD (`.com`, `.ru`) → Authoritative DNS** (хранит записи конкретного домена).
   - Основные записи: `A` (IPv4), `AAAA` (IPv6), `CNAME` (псевдоним), `MX` (почта), `TXT` (служебные данные, верификация), `NS` (серверы имён). У записей есть **TTL**, поэтому изменения DNS распространяются не мгновенно.
2. **TCP 3-Way Handshake (трёхстороннее рукопожатие)**:
   - **`SYN`**: клиент отправляет начальный номер `seq = x`.
   - **`SYN-ACK`**: сервер подтверждает `ack = x + 1` и присылает свой `seq = y`.
   - **`ACK`**: клиент подтверждает `ack = y + 1`. Канал установлен. Занимает **1 RTT** (Round-Trip Time).
3. **TLS Handshake** (для HTTPS): см. раздел 3.
4. **HTTP-запрос и ответ**.
5. **Keep-Alive (`Connection: keep-alive`)**: в HTTP/1.1 соединение по умолчанию переиспользуется для следующих запросов, экономя время на TCP/TLS handshake.
6. **Закрытие соединения**: `FIN` / `ACK` (4-way termination).

```mermaid
sequenceDiagram
    participant C as Клиент
    participant D as DNS
    participant S as Сервер
    C->>D: Какой IP у example.com?
    D-->>C: 93.184.216.34
    C->>S: SYN (seq=x)
    S-->>C: SYN-ACK (seq=y, ack=x+1)
    C->>S: ACK (ack=y+1)
    Note over C,S: TCP-соединение установлено (дальше TLS для HTTPS)
    C->>S: HTTP-запрос
    S-->>C: HTTP-ответ
```

### 🇬🇧 English

| Term | Meaning |
| :--- | :--- |
| **DNS** | The system that translates domain names into IP addresses. |
| **Resolver** | A server that performs DNS lookups on behalf of the client. |
| **Authoritative DNS** | The server that holds the actual DNS records for a domain. |
| **TCP 3-way handshake** | `SYN` → `SYN-ACK` → `ACK`; establishes a reliable connection. |
| **RTT (Round-Trip Time)** | Time for a packet to go to the server and back. |
| **Keep-Alive** | Reusing one TCP connection for several requests. |

> [!quote] Say it in English
> «Before the browser sends an HTTP request, it resolves the domain name through DNS, opens a TCP connection with a three-way handshake, and for HTTPS it also does a TLS handshake. Keep-alive lets it reuse the connection, so the next requests are faster.»

### Вопросы и ответы

> [!question] Зачем нужен именно трёхстороннее рукопожатие?
>
> Чтобы обе стороны подтвердили, что готовы принимать и отправлять данные, и синхронизировали начальные порядковые номера (`seq`), которые нужны для надёжной доставки и упорядочивания.

> [!question] Почему после смены DNS-записи сайт открывается по-старому адресу ещё какое-то время?
>
> Из-за кэширования: записи хранятся у резолверов, в ОС и браузере в течение TTL. Перед миграцией TTL заранее уменьшают.

> [!question] Чем TCP отличается от UDP?
>
> TCP: соединение, гарантия доставки и порядка, повторная отправка, контроль перегрузки. UDP: без соединения и гарантий, зато быстро и с малыми задержками (DNS, видео, игры, QUIC).

---

## 3. HTTPS и TLS: Как работает шифрование, сертификаты и HSTS

> [!abstract] Коротко
> HTTPS = HTTP внутри зашифрованного канала **TLS**. Даёт **конфиденциальность** (шифрование), **целостность** (данные не подменили) и **аутентификацию** (это настоящий сервер).

### Теория (🇷🇺 Русский)

- **Три гарантии HTTPS**:
  1. *Конфиденциальность*: перехваченный трафик нечитаем.
  2. *Целостность*: MITM (Man-in-the-Middle) не может незаметно изменить данные.
  3. *Аутентификация*: клиент уверен, что говорит с настоящим сервером, а не с фишинговой копией.
- **Гибридное шифрование**:
  - *Асимметричное* (RSA, ECDSA, ECDHE): пара ключей (публичный/приватный). Надёжно для обмена ключами и подписи, но **медленно**.
  - *Симметричное* (AES-GCM, ChaCha20): один общий ключ. **Намного быстрее**, шифрует весь трафик после рукопожатия.
- **TLS Handshake (упрощённо)**:
  1. **ClientHello**: версии TLS, список шифров (cipher suites), `Client Random`.
  2. **ServerHello + Certificate**: выбранный шифр, `Server Random`, **сертификат** (публичный ключ, домен, подпись центра сертификации).
  3. **Проверка сертификата** клиентом: срок действия; совпадение домена; цепочка доверия до корневого **CA (Certificate Authority)** (Let's Encrypt, DigiCert), чей корневой сертификат есть в ОС/браузере; не отозван ли.
  4. **Обмен ключами**: **ECDHE** (Диффи — Хеллман на эллиптических кривых) вырабатывает общий сессионный ключ. Это даёт **Perfect Forward Secrecy (PFS)**: даже если приватный ключ сервера украдут позже, записанный ранее трафик не расшифровать.
  5. **Finished**: стороны подтверждают рукопожатие, дальше идёт обычный HTTP внутри TLS.
- **TLS 1.3 vs 1.2**: в 1.3 рукопожатие короче (**1 RTT**, возможен **0-RTT** для повторных соединений), убраны устаревшие небезопасные шифры, PFS обязателен.
- **SSL Termination**: расшифровку берёт на себя обратный прокси (Nginx, Cloudflare, балансировщик) и передаёт чистый HTTP в PHP-FPM по внутренней сети с заголовком `X-Forwarded-Proto: https`. *Важно: приложение должно доверять этому заголовку только от своих прокси* (в Laravel — `TrustProxies`).
- **HSTS (HTTP Strict Transport Security)**: заголовок `Strict-Transport-Security: max-age=31536000; includeSubDomains` — браузер сам будет открывать сайт только по HTTPS (защита от **SSL Stripping**).
- **mTLS (mutual TLS)**: аутентифицируются обе стороны (клиент тоже предъявляет сертификат). Используется между микросервисами.

```mermaid
sequenceDiagram
    participant C as Клиент (Browser)
    participant S as Сервер (Nginx)
    C->>S: 1. ClientHello (TLS 1.3, шифры, Client Random)
    S-->>C: 2. ServerHello (выбранный шифр, Server Random)
    S-->>C: 3. Certificate (публичный ключ + подпись CA)
    Note over C: Проверка цепочки CA, домена и срока действия
    C->>S: 4. Обмен ключами (ECDHE)
    S-->>C: 5. Finished
    Note over C,S: Симметричный сессионный ключ готов (AES)
    C->>S: Зашифрованный HTTP-запрос
    S-->>C: Зашифрованный HTTP-ответ
```

### 🇬🇧 English

| Term | Meaning |
| :--- | :--- |
| **HTTPS** | HTTP over TLS: encryption, integrity and server authentication. |
| **TLS handshake** | The setup phase where client and server agree on ciphers and keys. |
| **Certificate** | A document that links a public key to a domain, signed by a CA. |
| **CA (Certificate Authority)** | A trusted organization that signs certificates. |
| **Hybrid encryption** | Asymmetric crypto for the handshake, symmetric crypto for the data. |
| **PFS (Perfect Forward Secrecy)** | Past sessions stay secret even if the server's private key leaks later. |
| **HSTS** | A header that forces browsers to use HTTPS only. |
| **SSL termination** | Decrypting HTTPS at a proxy and forwarding plain HTTP to the backend. |

> [!quote] Say it in English
> «HTTPS uses asymmetric cryptography only for the handshake, to verify the server and agree on a key. After that, the traffic is encrypted with a fast symmetric cipher like AES. In production, Nginx or the load balancer terminates TLS, and PHP-FPM receives plain HTTP.»

### Вопросы и ответы

> [!question] Зачем два вида шифрования, если можно использовать только асимметричное?
>
> Асимметричное шифрование надёжно для обмена ключами, но на порядки медленнее. Поэтому им согласуют общий ключ, а данные шифруют быстрым симметричным алгоритмом.

> [!question] Как браузер понимает, что сертификату можно доверять?
>
> Проверяет цепочку подписей до корневого CA из своего хранилища доверенных корней, срок действия, совпадение домена (CN/SAN) и статус отзыва (CRL/OCSP).

> [!question] Что такое Perfect Forward Secrecy?
>
> Сессионные ключи генерируются временными (ephemeral, ECDHE) и не выводятся из приватного ключа сервера. Утечка этого ключа в будущем не позволяет расшифровать ранее записанный трафик.

> [!question] Что защищает HSTS и в чём его слабость?
>
> Защищает от понижения до HTTP (SSL Stripping). Слабость: самый первый запрос пользователя может быть по HTTP; от этого помогает HSTS Preload List.

---

## 4. Структура HTTP-сообщения, Content-Type и заголовки

> [!abstract] Коротко
> HTTP-сообщение: **стартовая строка** → **заголовки** (`Key: Value`) → **пустая строка** → **тело** (необязательно).

### Теория (🇷🇺 Русский)

- **Запрос (Request)**: *метод + путь + версия*, заголовки, тело:

```http
POST /api/v1/users HTTP/1.1
Host: example.com
Content-Type: application/json
Accept: application/json
Authorization: Bearer eyJhbGciOiJIUzI1Ni...

{"name": "Alex", "email": "alex@example.com"}
```

- **Ответ (Response)**: *версия + код + причина*, заголовки, тело:

```http
HTTP/1.1 201 Created
Content-Type: application/json; charset=utf-8
Location: /api/v1/users/42
Cache-Control: no-store

{"id": 42, "name": "Alex"}
```

- **Основные `Content-Type` для отправки данных**:

| Content-Type | Для чего | В PHP / Laravel |
| :--- | :--- | :--- |
| `application/json` | Стандарт REST API | `php://input` (сырой JSON), `$request->all()` / `$request->input()` |
| `application/x-www-form-urlencoded` | Обычная HTML-форма (`key1=val1&key2=val2`) | `$_POST` |
| `multipart/form-data` | Форма с файлами (части разделены `boundary`) | `$_POST` + `$_FILES` |

- **Ключевые заголовки**:

| Заголовок | Назначение |
| :--- | :--- |
| `Host` | Обязателен в HTTP/1.1: один IP обслуживает много сайтов (виртуальные хосты) |
| `User-Agent` | Информация о клиенте (браузер, ОС, версия приложения) |
| `Accept` / `Accept-Language` / `Accept-Encoding` | Согласование контента: формат, язык, сжатие (`gzip`, `br`) |
| `Content-Type` / `Content-Length` | Тип и размер тела |
| `Authorization` | Учётные данные (`Bearer <token>`, `Basic ...`) |
| `Location` | Куда перенаправлять (3xx) или адрес созданного ресурса (201) |
| `X-Forwarded-For` / `X-Real-IP` | Реальный IP клиента за прокси/балансировщиком (можно подделать, доверять только от своих прокси) |
| `Retry-After` | Через сколько секунд повторить (429, 503) |

### 🇬🇧 English

| Term | Meaning |
| :--- | :--- |
| **Start line** | The first line: method, path and version (request) or version and status (response). |
| **Header** | A `Key: Value` pair with metadata about the message. |
| **Body** | The payload of the message (optional). |
| **`Content-Type`** | Tells the receiver the format of the body. |
| **Content negotiation** | The client states what it accepts (`Accept*`), and the server picks a format. |

> [!quote] Say it in English
> «An HTTP request has a start line with the method and path, then headers, an empty line, and an optional body. For a JSON API, I send `Content-Type: application/json`, and for file uploads, I use `multipart/form-data`.»

### Вопросы и ответы

> [!question] Чем `application/x-www-form-urlencoded` отличается от `multipart/form-data`?
>
> Первый — компактный формат `key=value&...` для простых форм. Второй разбивает тело на части с `boundary` и позволяет передавать файлы и бинарные данные.

> [!question] Как узнать реальный IP клиента за Nginx/балансировщиком?
>
> Из `X-Forwarded-For`/`X-Real-IP`, но доверять этим заголовкам можно только если запрос пришёл от известного прокси. Иначе клиент подделает заголовок.

> [!question] Зачем заголовок `Host`?
>
> Чтобы сервер понял, к какому из сайтов на одном IP обращается клиент (виртуальные хосты).

---

## 5. HTTP Request Methods: Методы, безопасность и идемпотентность

> [!abstract] Коротко
> Метод задаёт действие над ресурсом. Обязательно знать: **PUT vs PATCH**, а также **безопасность (safe)** и **идемпотентность (idempotent)**.

### Теория (🇷🇺 Русский)

- **Безопасный (Safe)**: только чтение, не меняет состояние сервера.
- **Идемпотентный (Idempotent)**: многократный одинаковый запрос даёт **тот же эффект на сервере**, что и один. Важно для безопасных повторов (retry) при сбоях сети. *Идемпотентность говорит об эффекте на состояние, а не о том, что ответы будут одинаковыми (повторный `DELETE` вернёт `404`, но состояние то же).*

| Метод | Описание | Safe | Idempotent | Кэшируемый | Тело запроса |
| :--- | :--- | :---: | :---: | :---: | :---: |
| **GET** | Получить представление ресурса | Да | Да | Да | Нет |
| **HEAD** | То же, что GET, но только заголовки | Да | Да | Да | Нет |
| **OPTIONS** | Поддерживаемые методы, CORS preflight | Да | Да | Нет | Нет |
| **PUT** | **Полная** замена ресурса по URI | Нет | Да | Нет | Да |
| **DELETE** | Удаление ресурса | Нет | Да | Нет | Обычно нет |
| **POST** | Создание ресурса / запуск операции | Нет | **Нет** | Почти нет | Да |
| **PATCH** | **Частичное** изменение | Нет | **Нет*** | Нет | Да |

\* `PATCH` не обязан быть идемпотентным: `{"op": "increment"}` при повторе изменит данные ещё раз, а `{"status": "paid"}` — нет.

- **`PUT` vs `PATCH`**: `PUT` присылает **весь** ресурс (недостающие поля обнулятся/заменятся), `PATCH` — только изменяемые поля.
- **`POST` vs `PUT` при создании**: `POST /users` — сервер сам выбирает ID; `PUT /users/42` — клиент указывает URI.
- **Как сделать `POST` идемпотентным**: клиент присылает заголовок **`Idempotency-Key`** (UUID). Сервер сохраняет (в Redis/БД) ключ и результат первого выполнения; при повторе с тем же ключом возвращает сохранённый ответ и не выполняет действие заново (например, не списывает деньги дважды).
- **Не используй `GET` для изменений**: `GET` кэшируется, предзагружается браузером и краулерами, попадает в логи и историю.

### 🇬🇧 English

| Term | Meaning |
| :--- | :--- |
| **Safe method** | Read-only; doesn't change server state (`GET`, `HEAD`, `OPTIONS`). |
| **Idempotent method** | Repeating it has the same effect as doing it once (`GET`, `PUT`, `DELETE`…). |
| **`PUT`** | Replaces the whole resource. |
| **`PATCH`** | Applies a partial update. |
| **`Idempotency-Key`** | A unique key from the client so a retried `POST` isn't processed twice. |

> [!quote] Say it in English
> «`PUT` replaces the entire resource and is idempotent. `PATCH` updates only some fields and isn't always idempotent. To make `POST` safe to retry, I use an `Idempotency-Key` header and store the result of the first request.»

### Вопросы и ответы

> [!question] Идемпотентность и безопасность — это одно и то же?
>
> Нет. `DELETE` и `PUT` идемпотентны, но **небезопасны** (меняют данные). Все безопасные методы идемпотентны, но не наоборот.

> [!question] Когда использовать `PUT`, а когда `PATCH`?
>
> `PUT` — когда клиент отправляет полное представление ресурса. `PATCH` — когда меняется часть полей. На практике многие API используют `PATCH` для частичных обновлений.

> [!question] Почему идемпотентность важна при ретраях?
>
> Если ответ потерялся, клиент не знает, выполнилась ли операция, и повторяет запрос. Идемпотентные операции безопасны для повтора, `POST` без защиты может создать дубликат (двойная оплата).

---

## 6. Response Codes: Коды состояния HTTP

> [!abstract] Коротко
> Трёхзначный код в стартовой строке ответа: `1xx` — информация, `2xx` — успех, `3xx` — перенаправление, `4xx` — ошибка клиента, `5xx` — ошибка сервера.

### Теория (🇷🇺 Русский & 🇬🇧 English)

| Код | Название | Смысл (🇷🇺) | Meaning (🇬🇧) |
| :--- | :--- | :--- | :--- |
| **101** | Switching Protocols | Переключение протокола (например, на WebSocket) | Protocol upgrade (e.g. to WebSocket) |
| **200** | OK | Успех | Standard success |
| **201** | Created | Ресурс создан (обычно после `POST`), часто с `Location` | Resource created |
| **202** | Accepted | Принято в фоновую обработку (очередь), результат позже | Accepted for async processing |
| **204** | No Content | Успех без тела (часто после `DELETE`/`PUT`) | Success with empty body |
| **301** | Moved Permanently | Постоянный редирект (кэшируется; `POST` может стать `GET`) | Permanent redirect |
| **302** | Found | Временный редирект (исторически `POST` → `GET`) | Temporary redirect |
| **304** | Not Modified | Не изменилось, бери из кэша | Use your cached copy |
| **307 / 308** | Temporary / Permanent Redirect | Редирект **с сохранением метода и тела** | Redirect that keeps method and body |
| **400** | Bad Request | Некорректный запрос (битый JSON) | Malformed request |
| **401** | Unauthorized | Не аутентифицирован (нет/неверный токен) | Not authenticated |
| **403** | Forbidden | Аутентифицирован, но прав нет | Authenticated, but not allowed |
| **404** | Not Found | Ресурс не найден | Resource not found |
| **405** | Method Not Allowed | Метод не разрешён для этого URL | Method not supported here |
| **409** | Conflict | Конфликт состояния (дубликат, конфликт версий) | State conflict |
| **410** | Gone | Ресурс удалён навсегда | Permanently removed |
| **415** | Unsupported Media Type | Неподдерживаемый `Content-Type` | Unsupported content type |
| **422** | Unprocessable Entity | Синтаксис верен, но валидация не пройдена | Valid syntax, failed validation |
| **429** | Too Many Requests | Превышен лимит запросов (`Retry-After`) | Rate limit exceeded |
| **500** | Internal Server Error | Необработанная ошибка на сервере | Unhandled server error |
| **502** | Bad Gateway | Прокси получил некорректный ответ от upstream (упал PHP-FPM) | Invalid response from upstream |
| **503** | Service Unavailable | Перегрузка или техработы | Overloaded or in maintenance |
| **504** | Gateway Timeout | Прокси не дождался ответа upstream | Upstream timed out |

> [!tip] 401 vs 403
> **401** — «я не знаю, кто ты» (нужно войти). **403** — «я знаю, кто ты, но тебе нельзя». Если не хочешь раскрывать существование ресурса, иногда вместо 403 отдают `404`.

> [!tip] 301 / 302 vs 307 / 308
> Для 301/302 браузеры могут сменить `POST` на `GET`. Если нужно сохранить метод и тело, используют **307** (временный) и **308** (постоянный).

### Вопросы и ответы

> [!question] Чем 401 отличается от 403?
>
> 401 — отсутствует или неверна аутентификация. 403 — пользователь известен, но у него нет прав на действие.

> [!question] Какой код вернуть при ошибке валидации?
>
> Обычно `422` (синтаксис верен, данные не прошли проверку) или `400`. Главное — единообразие в API и тело ответа со списком ошибок по полям.

> [!question] В чём разница между 502 и 504?
>
> 502 — upstream ответил, но некорректно, или соединение сброшено (процесс упал). 504 — upstream не ответил за отведённое время (таймаут).

> [!question] Какой код вернуть, если запрос принят в очередь?
>
> `202 Accepted`: результат будет позже (можно вернуть ссылку на статус задачи).

---

## 7. HTTP Cookies, Сессии и JWT

> [!abstract] Коротко
> **Cookie** — небольшие данные (до ~4 КБ), которые браузер сохраняет и автоматически отправляет с запросами на подходящий домен и путь. Используются для сессий и токенов.

### Теория (🇷🇺 Русский)

- **Как работают cookies**:
  1. Сервер присылает `Set-Cookie: session_id=xyz123; Path=/; HttpOnly; Secure; SameSite=Lax`.
  2. Браузер сохраняет и при каждом следующем запросе на подходящий домен/путь добавляет `Cookie: session_id=xyz123`.
- **Атрибуты `Set-Cookie`**:

| Атрибут | Назначение |
| :--- | :--- |
| `Expires` / `Max-Age` | Срок жизни. Без них кука **сессионная**: исчезает при закрытии браузера (`Max-Age` приоритетнее `Expires`) |
| `Domain` | Для каких доменов отправлять (`.example.com` включает поддомены) |
| `Path` | Для каких путей (`/admin`) |
| `HttpOnly` | Недоступна из JavaScript (`document.cookie`): **защита от кражи сессии через XSS** |
| `Secure` | Передаётся только по HTTPS |
| `SameSite` | **Защита от CSRF**: `Strict` (не отправлять при любых кросс-сайтовых запросах), `Lax` (по умолчанию в современных браузерах: отправлять при переходе по ссылке `GET`, но не при кросс-сайтовых `POST`/AJAX), `None` (всегда; **требует `Secure`**) |

  Префиксы `__Host-` и `__Secure-` в имени куки заставляют браузер проверять `Secure`, `Path=/` и отсутствие `Domain`.

- **Сессии (stateful) vs JWT (stateless)**:

| | Сессии (cookie + хранилище) | JWT (`Header.Payload.Signature`) |
| :--- | :--- | :--- |
| **Где данные** | На сервере (Redis/БД), в куке только ID | Внутри подписанного токена |
| **Отзыв** | Мгновенно (удалить сессию) | Сложно до `exp`: нужен чёрный список или короткий срок жизни |
| **Нагрузка** | Запрос в хранилище на каждый запрос | Только проверка подписи |
| **Риски** | CSRF (если cookie), кража ID | Payload лишь **закодирован в Base64, не зашифрован**: секреты не класть; кража токена |

  - **Access + Refresh Token**: короткоживущий access-токен (5–15 минут) + долгоживущий refresh-токен (хранится безопасно, например в `HttpOnly` cookie; обмен на новый access).
  - **Где хранить JWT в браузере**: `localStorage` уязвим к XSS; `HttpOnly` cookie уязвима к CSRF (нужен `SameSite`/CSRF-токен). Компромисс выбирают под задачу.
- **Заголовки для API-клиентов**: `Authorization: Bearer <token>`.
- **OAuth 2.0 / OIDC** (кратко): OAuth 2.0 — протокол **делегирования доступа** («войти через Google» + токены доступа); OIDC — слой аутентификации поверх OAuth 2.0 (ID-токен).

### 🇬🇧 English

| Term | Meaning |
| :--- | :--- |
| **Cookie** | Small data stored by the browser and sent back automatically with requests. |
| **Session cookie** | A cookie without an expiry; deleted when the browser closes. |
| **`HttpOnly`** | JavaScript can't read the cookie, which helps against XSS. |
| **`Secure`** | The cookie is sent only over HTTPS. |
| **`SameSite`** | Controls cross-site sending to help against CSRF. |
| **JWT** | A signed token with three parts: header, payload, signature. |
| **Refresh token** | A long-lived token used to get new short-lived access tokens. |

> [!quote] Say it in English
> «For a session cookie, I set `HttpOnly`, `Secure` and `SameSite=Lax`. With JWT, the payload is only Base64-encoded, so I never put secrets in it. Revoking a JWT before it expires is hard, so I use short-lived access tokens with refresh tokens.»

### Вопросы и ответы

> [!question] Что защищает `HttpOnly`, а что `SameSite`?
>
> `HttpOnly` не даёт JavaScript прочитать куку (смягчает последствия XSS). `SameSite` ограничивает отправку куки при кросс-сайтовых запросах (защита от CSRF). Это разные угрозы, обычно используют оба флага.

> [!question] Сессии или JWT?
>
> Сессии: просто отзывать, но нужен общий стор (Redis). JWT: не нужен запрос в хранилище, удобно для микросервисов, но сложнее отзывать. Для классического веб-приложения сессии часто проще и безопаснее.

> [!question] Зашифрован ли JWT?
>
> Нет: подписан (проверяется целостность), но payload читается любым (Base64). Шифрованный вариант — JWE, он используется редко.

---

## 8. Эволюция протокола: HTTP/1.1, HTTP/2 и HTTP/3

> [!abstract] Коротко
> **HTTP/1.1** — текстовый, запросы по очереди в соединении. **HTTP/2** — бинарный, **мультиплексирование** в одном TCP. **HTTP/3** — поверх **QUIC (UDP)**, без блокировок на уровне TCP.

### Теория (🇷🇺 Русский & 🇬🇧 English)

| Характеристика | HTTP/1.1 | HTTP/2 | HTTP/3 |
| :--- | :--- | :--- | :--- |
| **Формат** | Текстовый | Бинарный (фреймы) | Бинарный (фреймы) |
| **Транспорт** | TCP | TCP | **QUIC (поверх UDP)** |
| **Мультиплексирование** | Нет (браузер открывает до ~6 TCP-соединений на домен) | Да (много потоков в одном соединении) | Да (независимые потоки QUIC) |
| **Head-of-Line (HOL) blocking** | На уровне HTTP и TCP | Снят на уровне HTTP, остался на уровне **TCP** (потеря пакета тормозит все потоки) | **Нет** (потеря пакета тормозит только свой поток) |
| **Сжатие заголовков** | Нет | HPACK | QPACK |
| **Рукопожатие (HTTPS)** | TCP + TLS: несколько RTT | TCP + TLS: несколько RTT | **1 RTT**, 0-RTT для повторных соединений (TLS 1.3 встроен в QUIC) |
| **Server Push** | Нет | Есть (на практике почти не используется, удалён из Chrome) | Нет |

- **HOL (Head-of-Line) blocking**: ситуация, когда первый «застрявший» элемент очереди задерживает все остальные.
- **Что поменялось для разработчика**: при HTTP/2/3 старые приёмы оптимизации HTTP/1.1 (спрайты, склейка CSS/JS, domain sharding) теряют смысл или вредят.
- **QUIC** также поддерживает **миграцию соединения** (смена сети Wi-Fi ↔ мобильная без разрыва).
- **ALPN / Alt-Svc**: механизмы, через которые клиент и сервер договариваются о версии протокола (`h2`, `h3`).

### 🇬🇧 English

| Term | Meaning |
| :--- | :--- |
| **Multiplexing** | Sending several requests and responses at once over one connection. |
| **HOL blocking** | One delayed item blocks everything behind it. |
| **QUIC** | A modern transport over UDP with built-in TLS 1.3 and independent streams. |
| **HPACK / QPACK** | Header compression schemes for HTTP/2 and HTTP/3. |

> [!quote] Say it in English
> «HTTP/2 introduced multiplexing, so many requests share one TCP connection. But TCP packet loss still blocks all streams. HTTP/3 solves it by using QUIC over UDP, where each stream is independent.»

### Вопросы и ответы

> [!question] Что такое мультиплексирование и чем оно лучше нескольких соединений?
>
> Несколько запросов/ответов одновременно «чередуются» в одном соединении. Экономятся handshake'и, ресурсы и лучше работает контроль перегрузки.

> [!question] Почему HTTP/2 не избавился от HOL blocking полностью?
>
> Потому что работает поверх TCP, а TCP гарантирует порядок байтов: потерянный пакет задерживает все потоки в соединении. HTTP/3 решает это, используя независимые потоки QUIC.

---

## 9. HTTP-кэширование и условные запросы

> [!abstract] Коротко
> Кэширование убирает лишние передачи данных. Работает через **`Cache-Control`** (свежесть) и **условные запросы** (`ETag` / `Last-Modified` → `304 Not Modified`).

### Теория (🇷🇺 Русский)

- **`Cache-Control` (директивы)**:

| Директива | Смысл |
| :--- | :--- |
| `max-age=3600` | Ресурс свежий 1 час, браузер берёт из кэша без запроса |
| `public` / `private` | `public`: можно хранить в CDN/прокси; `private`: только в браузере пользователя |
| `no-cache` | Хранить можно, но **перед использованием проверять** на сервере (revalidate) |
| `no-store` | Не хранить вообще (конфиденциальные данные) |
| `must-revalidate` | После истечения срока обязательно проверить на сервере |
| `immutable` | Ресурс никогда не изменится (файлы с хэшем в имени) |
| `stale-while-revalidate=N` | Можно отдать устаревшее, пока в фоне идёт обновление |

- **Валидация (условные запросы)**:
  1. Сервер отдаёт `ETag: "hash123"` (или `Last-Modified`).
  2. Клиент при следующем обращении шлёт `If-None-Match: "hash123"` (или `If-Modified-Since`).
  3. Если не менялось — `304 Not Modified` **без тела** (экономия трафика).
- **`Vary`**: перечисляет заголовки, от которых зависит ответ (`Vary: Accept-Encoding`, `Accept-Language`), чтобы кэш не отдал не ту версию.
- **Cache busting**: для статики с долгим `max-age` в имя файла добавляют хэш (`app.3f9a1c.js`), и новая версия получает новый URL.
- **Условные запросы также защищают от потерянных обновлений** (`If-Match` + `ETag` для оптимистичной блокировки в `PUT`/`PATCH`).

```mermaid
sequenceDiagram
    participant B as Браузер
    participant S as Сервер
    B->>S: GET /logo.png
    S-->>B: 200 OK, ETag: "abc", Cache-Control: max-age=60
    Note over B: Через 60 секунд кэш считается устаревшим
    B->>S: GET /logo.png, If-None-Match: "abc"
    S-->>B: 304 Not Modified (без тела)
```

### 🇬🇧 English

| Term | Meaning |
| :--- | :--- |
| **`Cache-Control`** | Header that defines how and how long a response can be cached. |
| **`no-cache`** | Cached copies must be revalidated before use. |
| **`no-store`** | Nothing is stored at all. |
| **`ETag`** | A version identifier (fingerprint) of a resource. |
| **Conditional request** | A request with `If-None-Match` / `If-Modified-Since`. |
| **`304 Not Modified`** | The cached copy is still valid; no body is sent. |

> [!quote] Say it in English
> «`no-cache` doesn't mean "don't cache". It means the browser must revalidate with the server before using the cached copy. If I need to forbid storing the response, I use `no-store`.»

### Вопросы и ответы

> [!question] Чем `no-cache` отличается от `no-store`?
>
> `no-cache`: хранить можно, но перед каждым использованием нужна проверка на сервере. `no-store`: хранить запрещено совсем.

> [!question] Как обновить закэшированный на год JS-файл?
>
> Изменить его URL (имя с хэшем содержимого). Старый кэш остаётся, но страница ссылается на новый файл.

> [!question] Чем `ETag` лучше `Last-Modified`?
>
> `ETag` точнее (не зависит от секундной точности времени и от «ложных» изменений даты), хотя генерация хэша может быть дороже.

---

## 10. Безопасность HTTP: SOP, CORS и Security-заголовки

> [!abstract] Коротко
> **SOP** запрещает скрипту с одного origin читать ответы другого origin. **CORS** — механизм, которым сервер *явно разрешает* такие запросы. CORS не защищает сервер от запросов, он защищает **пользователя в браузере**.

### Теория (🇷🇺 Русский)

- **Origin** = **схема + хост + порт**. `https://a.com` и `https://a.com:8443` — разные origin.
- **Same-Origin Policy (SOP)**: скрипт с одного origin не может читать ответы с другого.
- **CORS (Cross-Origin Resource Sharing)**:
  - **Простые запросы** (`GET`/`POST`/`HEAD` с «безопасными» заголовками и типами) уходят сразу, а браузер проверяет заголовок ответа.
  - **Сложные запросы** (`PUT`, `DELETE`, `application/json`, `Authorization`) сначала вызывают **preflight** — запрос `OPTIONS` с `Origin`, `Access-Control-Request-Method`, `Access-Control-Request-Headers`.
  - Сервер отвечает: `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers` (и `Access-Control-Max-Age` для кэширования preflight).
  - С куками (`credentials: 'include'`): `Access-Control-Allow-Origin` **нельзя `*`**, нужен конкретный origin и `Access-Control-Allow-Credentials: true`.
- **Типичные атаки**:
  - **XSS (Cross-Site Scripting)**: внедрение чужого JS на страницу. Защита: экранирование вывода, **CSP**, `HttpOnly`-куки.
  - **CSRF (Cross-Site Request Forgery)**: чужой сайт заставляет браузер отправить запрос с вашими куками. Защита: `SameSite`, CSRF-токены, проверка `Origin`.
  - **Clickjacking**: невидимый iframe поверх страницы. Защита: `X-Frame-Options` / `frame-ancestors` в CSP.
  - **SSL Stripping / MITM**: защита HTTPS + HSTS.
- **Security-заголовки**:

| Заголовок | Что делает |
| :--- | :--- |
| `Strict-Transport-Security` | HSTS: только HTTPS |
| `Content-Security-Policy` | Белый список источников скриптов/стилей/картинок (против XSS) |
| `X-Content-Type-Options: nosniff` | Запрещает браузеру «угадывать» MIME-тип |
| `X-Frame-Options: DENY` | Запрещает встраивание в iframe (против Clickjacking) |
| `Referrer-Policy` | Что отдавать в заголовке `Referer` |
| `Permissions-Policy` | Ограничение доступа к камере, геолокации и т.д. |

### 🇬🇧 English

| Term | Meaning |
| :--- | :--- |
| **Origin** | Scheme + host + port. |
| **SOP (Same-Origin Policy)** | Browser rule: a page can't read responses from another origin. |
| **CORS** | A way for servers to allow cross-origin requests via headers. |
| **Preflight** | An automatic `OPTIONS` request that checks permissions before the real request. |
| **XSS** | Injecting malicious script into a page. |
| **CSRF** | Tricking a user's browser into sending an unwanted authenticated request. |
| **CSP** | A header that controls which sources the page may load. |

> [!quote] Say it in English
> «CORS is enforced by the browser, not by the server. If my frontend is on another domain, the API has to send the right `Access-Control-Allow-*` headers and answer the `OPTIONS` preflight request. With cookies, I can't use a wildcard origin.»

### Вопросы и ответы

> [!question] CORS защищает мой API от чужих сайтов?
>
> Нет. CORS работает только в браузере и ограничивает чтение ответа чужим скриптом. `curl` или другой сервер отправят запрос без ограничений. API нужно защищать аутентификацией и авторизацией.

> [!question] Зачем нужен preflight-запрос?
>
> Чтобы браузер заранее убедился, что сервер разрешает «потенциально опасный» запрос (нестандартный метод/заголовки), прежде чем отправлять его.

> [!question] Чем XSS отличается от CSRF?
>
> XSS: злоумышленник исполняет свой код **в контексте вашего сайта**. CSRF: злоумышленник заставляет браузер жертвы отправить запрос **на ваш сайт** со своими куками, не читая ответ.

> [!question] Почему нельзя `Access-Control-Allow-Origin: *` вместе с куками?
>
> Спецификация запрещает: иначе любой сайт мог бы делать запросы от имени пользователя и читать ответы. Нужно указать конкретный origin.

---

## 11. Проектирование API: REST, GraphQL, gRPC, WebSockets

> [!abstract] Коротко
> Выбор стиля зависит от задачи: **REST** — стандартные публичные API; **GraphQL** — гибкие клиенты; **gRPC** — быстрые внутренние микросервисы; **WebSockets/SSE** — real-time.

### Теория (🇷🇺 Русский)

| Стиль | Протокол и формат | Плюсы | Минусы |
| :--- | :--- | :--- | :--- |
| **REST** | HTTP + JSON, ресурсы в URL | Простота, HTTP-кэширование, понятные коды | Over-/under-fetching |
| **GraphQL** | `POST /graphql` + JSON | Клиент запрашивает только нужные поля, один запрос | Сложное кэширование, риск N+1, нужны лимиты глубины/сложности |
| **gRPC** | HTTP/2 + Protobuf (бинарный) | Скорость, строгие контракты, стриминг | Нужен gRPC-Web для браузеров, сложнее отладка |
| **WebSockets** | TCP после HTTP `101` | Двусторонний real-time без polling | Stateful-соединения сложнее масштабировать |
| **SSE (Server-Sent Events)** | HTTP, поток от сервера к клиенту | Просто, автопереподключение | Только сервер → клиент |

- **Принципы REST**: ресурсы идентифицируются URL (существительные во множественном числе: `/users/42/orders`), действия выражаются **методами HTTP**, stateless, единообразные коды ответов, кэшируемость.
- **Практики хорошего REST API**:
  - Версионирование (`/v1/...` или заголовок).
  - **Пагинация** (`limit/offset` или cursor/keyset), **фильтрация и сортировка** через query-параметры.
  - Единый формат ошибок (код, сообщение, поля), корректные коды ответов.
  - **Идемпотентность** (`Idempotency-Key`), **rate limiting** (`429`), аутентификация (`Bearer`).
  - Документация: **OpenAPI / Swagger**.
  - **HATEOAS** (ссылки в ответах) — высший уровень зрелости (по Ричардсону), на практике используется редко.
- **Polling, Long polling, WebSockets, SSE**:
  - *Polling*: клиент периодически спрашивает (просто, расточительно).
  - *Long polling*: сервер держит запрос, пока не появятся данные.
  - *WebSockets*: постоянное двустороннее соединение (чаты, игры, торговля).
  - *SSE*: односторонний поток событий (уведомления, ленты).
- **Webhooks**: сервер сам вызывает URL клиента при событии (оплата прошла). Требуют подписи, ретраев и идемпотентной обработки.

### 🇬🇧 English

| Term | Meaning |
| :--- | :--- |
| **REST** | An architectural style using HTTP methods on resources identified by URLs. |
| **Over-fetching / under-fetching** | Getting more data than needed / too little, requiring extra requests. |
| **GraphQL** | A query language where the client chooses exactly which fields to get. |
| **gRPC** | A fast binary RPC framework over HTTP/2 with Protobuf. |
| **WebSocket** | A persistent two-way connection between client and server. |
| **Webhook** | The server calls your URL when an event happens. |
| **Pagination** | Splitting large results into pages. |

> [!quote] Say it in English
> «For a public API, I'd choose REST: it's simple, cacheable and well understood. For internal service-to-service calls where performance matters, I'd consider gRPC. For real-time features like chat, I'd use WebSockets.»

### Вопросы и ответы

> [!question] REST или GraphQL?
>
> REST: простые публичные API, кэширование на уровне HTTP. GraphQL: много разных клиентов (web/mobile) с разными потребностями в данных; платим сложностью кэширования и защитой от тяжёлых запросов.

> [!question] Как масштабировать WebSockets?
>
> Соединения stateful: нужен балансировщик с поддержкой WebSocket, общий брокер (Redis Pub/Sub, Kafka) для рассылки сообщений между узлами, учёт лимитов соединений на узел.

> [!question] Как выбрать между limit/offset и cursor-пагинацией?
>
> `offset` прост, но медленный на больших страницах и нестабилен при вставках. Cursor/keyset быстрее и стабильнее, но не позволяет прыгнуть на произвольную страницу.

---

## 12. Шпаргалка перед собеседованием

> [!important] Чек-лист по HTTP и сетям
> 1. **Основы**: HTTP — stateless, прикладной уровень; URL = схема + хост + порт + путь + query + `#fragment` (фрагмент не уходит на сервер).
> 2. **Жизненный цикл**: DNS → TCP 3-way handshake → TLS → HTTP-запрос; keep-alive переиспользует соединение.
> 3. **HTTPS/TLS**: асимметричная криптография (ECDHE + сертификат CA) только для проверки подлинности и согласования ключа; данные шифруются быстрым симметричным шифром (AES-GCM). PFS, HSTS, терминация TLS на прокси.
> 4. **Методы**: безопасные (`GET`, `HEAD`, `OPTIONS`) ничего не меняют; идемпотентные (`GET`, `PUT`, `DELETE`) дают тот же эффект при повторе; `POST` делают идемпотентным через `Idempotency-Key`; `PUT` — полная замена, `PATCH` — частичная.
> 5. **Коды**: `201` создано, `202` принято в очередь, `204` без тела, `304` из кэша, `401` не залогинен, `403` нет прав, `409` конфликт, `422` валидация, `429` лимит, `502` упал upstream, `504` таймаут upstream.
> 6. **Cookies**: `HttpOnly` (против XSS), `Secure` (только HTTPS), `SameSite` (против CSRF); нет `Max-Age`/`Expires` → сессионная кука. JWT подписан, но **не зашифрован**.
> 7. **HTTP/1.1 → 2 → 3**: текст → бинарный + мультиплексирование → QUIC поверх UDP без HOL blocking на уровне транспорта.
> 8. **Кэширование**: `max-age`, `no-cache` (проверять), `no-store` (не хранить), `ETag` + `If-None-Match` → `304`.
> 9. **Безопасность**: SOP + CORS (preflight `OPTIONS`), XSS/CSRF/Clickjacking, заголовки CSP, HSTS, `nosniff`, `X-Frame-Options`.
> 10. **API**: REST (по умолчанию), GraphQL (гибкость), gRPC (быстро внутри), WebSockets/SSE (real-time).

### 🇬🇧 Фразы-заготовки для ответов

| Ситуация | Фраза |
| :--- | :--- |
| Дать определение | «X is a … that is used to …» |
| Объяснить разницу | «The key difference is that … while …» |
| Привести пример | «For example, when a user logs in, …» |
| Описать последовательность | «First, …, then …, and finally …» |
| Признать незнание | «I'm not sure about the exact details, but as far as I know…» |

> [!tip] Вопросы, которые часто задают
> - Что происходит, когда ты вводишь URL в браузере и нажимаешь Enter?
> - Чем HTTP отличается от HTTPS? Как работает TLS handshake?
> - `PUT` vs `PATCH`; что такое идемпотентность? Как сделать `POST` идемпотентным?
> - `401` vs `403`; `502` vs `504`; `301` vs `302` vs `307/308`?
> - Как работают cookies? Для чего `HttpOnly`, `Secure`, `SameSite`? Сессии vs JWT?
> - Чем HTTP/2 отличается от HTTP/1.1, а HTTP/3 от HTTP/2?
> - Что такое CORS и preflight? Чем XSS отличается от CSRF?
> - REST vs GraphQL vs gRPC: когда что выбрать?

> [!example] Что происходит, когда вводишь URL и нажимаешь Enter (классический вопрос)
> 1. Браузер разбирает URL и проверяет HSTS/кэш.
> 2. **DNS**: домен → IP (кэш браузера → ОС → резолвер → иерархия DNS).
> 3. **TCP**: 3-way handshake (или QUIC для HTTP/3).
> 4. **TLS**: handshake, проверка сертификата, общий ключ.
> 5. **HTTP-запрос** (`GET / HTTP/1.1`, `Host`, куки, заголовки).
> 6. Балансировщик/Nginx принимает запрос, терминирует TLS и передаёт в PHP-FPM.
> 7. Приложение обрабатывает запрос (middleware → контроллер → БД/кэш) и формирует ответ.
> 8. Браузер получает ответ, разбирает HTML, запрашивает CSS/JS/изображения (с учётом кэша), строит DOM и рисует страницу.
