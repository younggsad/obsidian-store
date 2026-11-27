---
custom-width: 80
tags: [http, networks, interview]
---
---
## Contents

- [[#1. HTTP basics, URI, URL, URN and the TCP/IP stack]]
- [[#2. Life of an HTTP request, DNS and TCP handshake]]
- [[#3. HTTPS and TLS, encryption, certificates and HSTS]]
- [[#4. HTTP message structure, Content-Type and headers]]
- [[#5. HTTP methods, safety and idempotency]]
- [[#6. HTTP status codes]]
- [[#7. Cookies, sessions and JWT]]
- [[#8. Evolution of the protocol, HTTP/1.1, HTTP/2 and HTTP/3]]
- [[#9. HTTP caching and conditional requests]]
- [[#10. HTTP security, SOP, CORS and security headers]]
- [[#11. API design, REST, GraphQL, gRPC, WebSockets]]
- [[#12. Cheat sheet before the interview]]

---

# 1. HTTP basics, URI, URL, URN and the TCP/IP stack

## Main idea

HTTP это протокол прикладного уровня для обмена данными по схеме «клиент и сервер». Он stateless, то есть не помнит прошлые запросы. Он работает поверх транспорта: TCP (HTTP/1.1 и HTTP/2) или QUIC поверх UDP (HTTP/3).

HTTP is an application-layer protocol for exchanging data in the "client and server" model. It is stateless, which means it does not remember previous requests. It works on top of a transport: TCP (HTTP/1.1 and HTTP/2) or QUIC over UDP (HTTP/3).

## HTTP and the client-server model

HTTP (HyperText Transfer Protocol) это протокол запросов и ответов. В версии 1.1 сообщения текстовые, а в версиях 2 и 3 они бинарные, но семантика методов, кодов и заголовков остаётся одной и той же. Инициатор всегда клиент: браузер, мобильное приложение или другой сервис. Сервер слушает порт (80 для HTTP и 443 для HTTPS) и возвращает ответ.

HTTP (HyperText Transfer Protocol) is a request-response protocol. In version 1.1 the messages are text, and in versions 2 and 3 they are binary, but the semantics of methods, codes and headers stays the same. The client always starts the communication: a browser, a mobile app or another service. The server listens on a port (80 for HTTP and 443 for HTTPS) and returns a response.

## Stateless

Каждый запрос содержит всё, что нужно для его обработки. Состояние, например сессия или авторизация, передаётся через cookies или токены. Благодаря этому бэкенд легко масштабировать горизонтально: любой сервер может обработать любой запрос.

Every request contains everything that is needed to process it. The state, for example a session or authorization, is passed through cookies or tokens. Thanks to this, it is easy to scale the backend horizontally, because any server can handle any request.

## URI, URL and URN

URI (Uniform Resource Identifier) это общее понятие, идентификатор ресурса. URL и URN это его частные случаи. URL (Uniform Resource Locator) говорит, где находится ресурс и как его получить. URN (Uniform Resource Name) это постоянное имя без указания местоположения, например urn:isbn:0451450523. Любой URL является URI, но не наоборот.

URI (Uniform Resource Identifier) is the general term, an identifier of a resource. URL and URN are its special cases. URL (Uniform Resource Locator) tells where the resource is and how to get it. URN (Uniform Resource Name) is a persistent name without the location, for example urn:isbn:0451450523. Every URL is a URI, but not the other way around.

```text
https://user:pass@api.example.com:8443/v1/users?active=1#top
└─┬──┘   └───┬───┘ └──────┬──────┘└─┬─┘└───┬───┘└───┬───┘└┬─┘
scheme   userinfo        host    port   path    query  fragment
```

В этом примере https это схема (протокол), api.example.com это хост, 8443 это порт, /v1/users это путь, active=1 это query string, а #top это фрагмент (якорь). Фрагмент не отправляется на сервер, его обрабатывает только браузер.

In this example, https is the scheme (the protocol), api.example.com is the host, 8443 is the port, /v1/users is the path, active=1 is the query string, and #top is the fragment (the anchor). The fragment is not sent to the server, and only the browser handles it.

## The TCP/IP model

В модели TCP/IP четыре уровня.

- Прикладной уровень (Application): HTTP, DNS, TLS, SMTP. Здесь живёт логика приложений.
- Транспортный уровень (Transport): TCP, UDP, QUIC. TCP работает надёжно, по порядку и с повторной отправкой. UDP быстрый, но без гарантий.
- Сетевой уровень (Internet): IP. Он отвечает за адресацию и маршрутизацию пакетов между сетями.
- Канальный уровень (Link): Ethernet, Wi-Fi. Он передаёт кадры в локальной сети по MAC-адресам.

Модель OSI с семью уровнями теоретическая, а TCP/IP практическая. На собеседовании часто просят назвать, на каком уровне работает протокол.

The TCP/IP model has four layers.

- The Application layer: HTTP, DNS, TLS, SMTP. The logic of applications lives here.
- The Transport layer: TCP, UDP, QUIC. TCP works reliably, in order and with retransmission. UDP is fast, but it gives no guarantees.
- The Internet layer: IP. It is responsible for addressing and routing packets between networks.
- The Link layer: Ethernet, Wi-Fi. It transfers frames in a local network by MAC addresses.

The OSI model with seven layers is theoretical, and TCP/IP is practical. In an interview, people often ask us to say which layer a protocol works on.

> [!question] Чем URL отличается от URI? / What is the difference between URL and URI?
> 
> URI это любой идентификатор ресурса. URL это URI, который ещё и указывает, где находится ресурс и как его получить (схема, хост и путь). Любой URL это URI, но не наоборот.
> 
> A URI is any identifier of a resource. A URL is a URI that also tells where the resource is and how to get it (the scheme, the host and the path). Every URL is a URI, but not the other way around.

> [!question] Что значит «HTTP stateless» и как тогда работает авторизация? / What does "HTTP is stateless" mean, and how does authorization work then?
> 
> Сервер не помнит предыдущие запросы. Состояние хранится на клиенте и передаётся в каждом запросе: это cookie с ID сессии или заголовок Authorization с токеном.
> 
> The server does not remember previous requests. The state is stored on the client and sent with every request: it is a cookie with a session ID, or an Authorization header with a token.

> [!question] Приходит ли #fragment на сервер? / Does the #fragment reach the server?
> 
> Нет. Он остаётся в браузере и используется для якорей и для роутинга в SPA.
> 
> No. It stays in the browser and is used for anchors and for routing in a SPA.

---

# 2. Life of an HTTP request, DNS and TCP handshake

## Main idea

Перед HTTP-запросом клиент находит IP-адрес через DNS, потом устанавливает TCP-соединение (three-way handshake), потом для HTTPS договаривается о шифровании по TLS, и только после этого отправляет HTTP-запрос.

Before an HTTP request, the client finds the IP address through DNS, then it opens a TCP connection (the three-way handshake), then for HTTPS it agrees on encryption with TLS, and only after that does it send the HTTP request.

## DNS lookup

Сначала клиент ищет адрес в кэшах: в кэше браузера, потом в кэше ОС (и в файле /etc/hosts), потом в кэше роутера и в DNS-резолвере провайдера. Если записи нет, резолвер опрашивает иерархию: Root (.), потом TLD (.com, .ru), потом Authoritative DNS, который хранит записи конкретного домена.

Основные DNS-записи такие: A (IPv4), AAAA (IPv6), CNAME (псевдоним), MX (почта), TXT (служебные данные и верификация) и NS (серверы имён). У записей есть TTL, поэтому изменения DNS распространяются не мгновенно.

First, the client looks for the address in caches: the browser cache, then the OS cache (and the /etc/hosts file), then the router cache and the DNS resolver of the provider. If there is no record, the resolver asks the hierarchy: Root (.), then TLD (.com, .ru), then the Authoritative DNS, which holds the records of the specific domain.

The main DNS records are: A (IPv4), AAAA (IPv6), CNAME (an alias), MX (mail), TXT (service data and verification) and NS (name servers). Records have a TTL, so DNS changes do not spread instantly.

## TCP three-way handshake

Трёхстороннее рукопожатие состоит из трёх шагов. Сначала клиент отправляет SYN с начальным номером seq = x. Потом сервер отвечает SYN-ACK: он подтверждает ack = x + 1 и присылает свой seq = y. Потом клиент отправляет ACK с ack = y + 1. После этого канал установлен. Это занимает один RTT (Round-Trip Time).

The three-way handshake has three steps. First, the client sends SYN with the initial number seq = x. Then the server answers with SYN-ACK: it confirms ack = x + 1 and sends its own seq = y. Then the client sends ACK with ack = y + 1. After this, the channel is established. It takes one RTT (Round-Trip Time).

## Keep-Alive and closing the connection

Потом идёт TLS handshake (для HTTPS, см. раздел 3), а затем HTTP-запрос и ответ. В HTTP/1.1 соединение по умолчанию переиспользуется для следующих запросов (Connection: keep-alive), и это экономит время на TCP и TLS handshake. Соединение закрывается обменом FIN и ACK (четырёхстороннее закрытие).

Then comes the TLS handshake (for HTTPS, see section 3), and then the HTTP request and response. In HTTP/1.1, the connection is reused for the next requests by default (Connection: keep-alive), and this saves the time of the TCP and TLS handshakes. The connection is closed by an exchange of FIN and ACK (a four-way termination).

```mermaid
sequenceDiagram
    participant C as Client
    participant D as DNS
    participant S as Server
    C->>D: What is the IP of example.com?
    D-->>C: 93.184.216.34
    C->>S: SYN (seq=x)
    S-->>C: SYN-ACK (seq=y, ack=x+1)
    C->>S: ACK (ack=y+1)
    Note over C,S: TCP connection is established (then TLS for HTTPS)
    C->>S: HTTP request
    S-->>C: HTTP response
```

> [!question] Зачем нужно именно трёхстороннее рукопожатие? / Why do we need exactly a three-way handshake?
> 
> Обе стороны должны подтвердить, что готовы принимать и отправлять данные, и синхронизировать начальные порядковые номера (seq). Эти номера нужны для надёжной доставки и для упорядочивания данных.
> 
> Both sides must confirm that they are ready to receive and send data, and they must synchronize the initial sequence numbers (seq). These numbers are needed for reliable delivery and for ordering the data.

> [!question] Почему после смены DNS-записи сайт ещё какое-то время открывается по старому адресу? / Why does a site open at the old address for some time after we change a DNS record?
> 
> Причина в кэшировании. Записи хранятся у резолверов, в ОС и в браузере в течение TTL. Перед миграцией TTL заранее уменьшают.
> 
> The reason is caching. Records are stored in resolvers, in the OS and in the browser for the duration of the TTL. Before a migration, we reduce the TTL in advance.

> [!question] Чем TCP отличается от UDP? / What is the difference between TCP and UDP?
> 
> TCP устанавливает соединение, гарантирует доставку и порядок, повторно отправляет потерянные данные и контролирует перегрузку. UDP работает без соединения и без гарантий, зато быстро и с малой задержкой. Его используют для DNS, видео, игр и QUIC.
> 
> TCP opens a connection, guarantees delivery and order, resends lost data and controls congestion. UDP works without a connection and without guarantees, but it is fast and has low latency. It is used for DNS, video, games and QUIC.

---

# 3. HTTPS and TLS, encryption, certificates and HSTS

## Main idea

HTTPS это HTTP внутри зашифрованного канала TLS. Он даёт конфиденциальность (шифрование), целостность (данные не подменили) и аутентификацию (мы говорим с настоящим сервером).

HTTPS is HTTP inside an encrypted TLS channel. It gives confidentiality (encryption), integrity (the data was not changed) and authentication (we talk to the real server).

## Three guarantees of HTTPS

Конфиденциальность означает, что перехваченный трафик нельзя прочитать. Целостность означает, что MITM (Man-in-the-Middle) не может незаметно изменить данные. Аутентификация означает, что клиент уверен, что говорит с настоящим сервером, а не с фишинговой копией.

Confidentiality means that intercepted traffic cannot be read. Integrity means that a MITM (Man-in-the-Middle) cannot change the data without being noticed. Authentication means that the client is sure that it talks to the real server and not to a phishing copy.

## Hybrid encryption

В HTTPS используется гибридное шифрование. Асимметричное шифрование (RSA, ECDSA, ECDHE) использует пару ключей, публичный и приватный. Оно надёжно подходит для обмена ключами и для подписи, но работает медленно. Симметричное шифрование (AES-GCM, ChaCha20) использует один общий ключ. Оно намного быстрее, поэтому шифрует весь трафик после рукопожатия.

HTTPS uses hybrid encryption. Asymmetric encryption (RSA, ECDSA, ECDHE) uses a pair of keys, a public one and a private one. It is reliable for key exchange and for signatures, but it is slow. Symmetric encryption (AES-GCM, ChaCha20) uses one shared key. It is much faster, so it encrypts all the traffic after the handshake.

## TLS handshake

Упрощённо рукопожатие TLS выглядит так.

1. ClientHello. Клиент отправляет версии TLS, список шифров (cipher suites) и Client Random.
2. ServerHello и Certificate. Сервер выбирает шифр, отправляет Server Random и сертификат (публичный ключ, домен и подпись центра сертификации).
3. Проверка сертификата. Клиент проверяет срок действия, совпадение домена и цепочку доверия до корневого CA (Certificate Authority), например Let's Encrypt или DigiCert, чей корневой сертификат есть в ОС или браузере. Также он проверяет, что сертификат не отозван.
4. Обмен ключами. ECDHE (Диффи-Хеллман на эллиптических кривых) вырабатывает общий сессионный ключ. Это даёт Perfect Forward Secrecy (PFS): даже если приватный ключ сервера украдут позже, ранее записанный трафик расшифровать нельзя.
5. Finished. Стороны подтверждают рукопожатие, и дальше идёт обычный HTTP внутри TLS.

In a simplified form, the TLS handshake looks like this.

1. ClientHello. The client sends the TLS versions, the list of ciphers (cipher suites) and the Client Random.
2. ServerHello and Certificate. The server chooses a cipher, sends the Server Random and the certificate (the public key, the domain and the signature of the certificate authority).
3. Certificate check. The client checks the expiry date, the domain match and the chain of trust up to a root CA (Certificate Authority), for example Let's Encrypt or DigiCert, whose root certificate is in the OS or the browser. It also checks that the certificate is not revoked.
4. Key exchange. ECDHE (Diffie-Hellman on elliptic curves) creates a shared session key. This gives Perfect Forward Secrecy (PFS): even if someone steals the server's private key later, the previously recorded traffic cannot be decrypted.
5. Finished. Both sides confirm the handshake, and then normal HTTP goes inside TLS.

```mermaid
sequenceDiagram
    participant C as Client (Browser)
    participant S as Server (Nginx)
    C->>S: 1. ClientHello (TLS 1.3, ciphers, Client Random)
    S-->>C: 2. ServerHello (chosen cipher, Server Random)
    S-->>C: 3. Certificate (public key + CA signature)
    Note over C: Check the CA chain, the domain and the expiry date
    C->>S: 4. Key exchange (ECDHE)
    S-->>C: 5. Finished
    Note over C,S: Symmetric session key is ready (AES)
    C->>S: Encrypted HTTP request
    S-->>C: Encrypted HTTP response
```

## TLS 1.3, SSL termination, HSTS and mTLS

В TLS 1.3 рукопожатие короче: оно занимает 1 RTT, а для повторных соединений возможен 0-RTT. В нём убраны устаревшие небезопасные шифры, а PFS обязателен.

SSL Termination означает, что расшифровку берёт на себя обратный прокси (Nginx, Cloudflare, балансировщик), а в PHP-FPM по внутренней сети передаётся чистый HTTP с заголовком X-Forwarded-Proto: https. Важно, что приложение должно доверять этому заголовку только от своих прокси (в Laravel для этого есть TrustProxies).

HSTS (HTTP Strict Transport Security) это заголовок Strict-Transport-Security: max-age=31536000; includeSubDomains. Благодаря ему браузер сам открывает сайт только по HTTPS, и это защищает от SSL Stripping.

mTLS (mutual TLS) означает, что аутентифицируются обе стороны, то есть клиент тоже показывает сертификат. Это используется между микросервисами.

In TLS 1.3, the handshake is shorter: it takes 1 RTT, and 0-RTT is possible for repeated connections. Old unsafe ciphers are removed, and PFS is mandatory.

SSL Termination means that a reverse proxy (Nginx, Cloudflare, a load balancer) does the decryption, and plain HTTP goes to PHP-FPM through the internal network with the header X-Forwarded-Proto: https. It is important that the application trusts this header only from its own proxies (in Laravel there is TrustProxies for this).

HSTS (HTTP Strict Transport Security) is the header Strict-Transport-Security: max-age=31536000; includeSubDomains. Thanks to it, the browser opens the site only over HTTPS by itself, and this protects against SSL Stripping.

mTLS (mutual TLS) means that both sides are authenticated, so the client also shows a certificate. It is used between microservices.

> [!question] Зачем два вида шифрования, если можно использовать только асимметричное? / Why do we need two kinds of encryption if we can use only asymmetric?
> 
> Асимметричное шифрование надёжно для обмена ключами, но на порядки медленнее. Поэтому с его помощью согласуют общий ключ, а сами данные шифруют быстрым симметричным алгоритмом.
> 
> Asymmetric encryption is reliable for key exchange, but it is many times slower. So we use it to agree on a shared key, and we encrypt the data itself with a fast symmetric algorithm.

> [!question] Как браузер понимает, что сертификату можно доверять? / How does the browser know that it can trust a certificate?
> 
> Он проверяет цепочку подписей до корневого CA из своего хранилища доверенных корней, срок действия, совпадение домена (CN или SAN) и статус отзыва (CRL или OCSP).
> 
> It checks the chain of signatures up to a root CA from its store of trusted roots, the expiry date, the domain match (CN or SAN) and the revocation status (CRL or OCSP).

> [!question] Что такое Perfect Forward Secrecy? / What is Perfect Forward Secrecy?
> 
> Сессионные ключи генерируются временными (ephemeral, ECDHE) и не выводятся из приватного ключа сервера. Поэтому утечка этого ключа в будущем не позволяет расшифровать ранее записанный трафик.
> 
> The session keys are generated as temporary ones (ephemeral, ECDHE) and are not derived from the server's private key. So a leak of this key in the future does not allow us to decrypt traffic that was recorded earlier.

> [!question] Что защищает HSTS и в чём его слабость? / What does HSTS protect, and what is its weakness?
> 
> Он защищает от понижения до HTTP (SSL Stripping). Его слабость в том, что самый первый запрос пользователя может пойти по HTTP. Здесь помогает HSTS Preload List.
> 
> It protects against a downgrade to HTTP (SSL Stripping). Its weakness is that the very first request of the user may go over HTTP. The HSTS Preload List helps here.

---

# 4. HTTP message structure, Content-Type and headers

## Main idea

HTTP-сообщение состоит из стартовой строки, заголовков (Key: Value), пустой строки и тела (оно необязательно).

An HTTP message consists of a start line, headers (Key: Value), an empty line and a body (it is optional).

## Request and response

Запрос (Request) состоит из метода, пути и версии, затем идут заголовки и тело.

A request consists of the method, the path and the version, then the headers and the body.

```http
POST /api/v1/users HTTP/1.1
Host: example.com
Content-Type: application/json
Accept: application/json
Authorization: Bearer eyJhbGciOiJIUzI1Ni...

{"name": "Alex", "email": "alex@example.com"}
```

Ответ (Response) состоит из версии, кода и причины, затем идут заголовки и тело.

A response consists of the version, the code and the reason, then the headers and the body.

```http
HTTP/1.1 201 Created
Content-Type: application/json; charset=utf-8
Location: /api/v1/users/42
Cache-Control: no-store

{"id": 42, "name": "Alex"}
```

## Main Content-Type values

- application/json это стандарт для REST API. В PHP сырой JSON читается из php://input, а в Laravel данные доступны через $request->all() или $request->input().
    
- application/x-www-form-urlencoded используется для обычной HTML-формы (key1=val1&key2=val2). В PHP эти данные лежат в $_POST.
    
- multipart/form-data используется для форм с файлами. Части тела разделены строкой boundary. В PHP данные лежат в $_POST и $_FILES.
    
- application/json is the standard for REST APIs. In PHP, the raw JSON is read from php://input, and in Laravel the data is available through $request->all() or $request->input().
    
- application/x-www-form-urlencoded is used for a normal HTML form (key1=val1&key2=val2). In PHP, this data is in $_POST.
    
- multipart/form-data is used for forms with files. The parts of the body are separated by a boundary string. In PHP, the data is in $_POST and $_FILES.
    

## Key headers

- Host обязателен в HTTP/1.1, потому что один IP-адрес обслуживает много сайтов (виртуальные хосты).
    
- User-Agent содержит информацию о клиенте: браузер, ОС, версию приложения.
    
- Accept, Accept-Language и Accept-Encoding нужны для согласования контента: формат, язык и сжатие (gzip, br).
    
- Content-Type и Content-Length показывают тип и размер тела.
    
- Authorization содержит учётные данные (Bearer <token> или Basic ...).
    
- Location указывает, куда перенаправить клиента (3xx), или адрес созданного ресурса (201).
    
- X-Forwarded-For и X-Real-IP показывают реальный IP клиента за прокси или балансировщиком. Их можно подделать, поэтому доверять им можно только от своих прокси.
    
- Retry-After говорит, через сколько секунд повторить запрос (коды 429 и 503).
    
- Host is mandatory in HTTP/1.1, because one IP address serves many sites (virtual hosts).
    
- User-Agent contains information about the client: the browser, the OS and the app version.
    
- Accept, Accept-Language and Accept-Encoding are used for content negotiation: the format, the language and the compression (gzip, br).
    
- Content-Type and Content-Length show the type and the size of the body.
    
- Authorization contains credentials (Bearer <token> or Basic ...).
    
- Location says where to redirect the client (3xx), or the address of the created resource (201).
    
- X-Forwarded-For and X-Real-IP show the real IP of the client behind a proxy or a load balancer. They can be faked, so we can trust them only from our own proxies.
    
- Retry-After says after how many seconds to repeat the request (codes 429 and 503).
    

> [!question] Чем application/x-www-form-urlencoded отличается от multipart/form-data? / What is the difference between application/x-www-form-urlencoded and multipart/form-data?
> 
> Первый это компактный формат key=value&... для простых форм. Второй разбивает тело на части с boundary и позволяет передавать файлы и бинарные данные.
> 
> The first is a compact key=value&... format for simple forms. The second splits the body into parts with a boundary and allows us to send files and binary data.

> [!question] Как узнать реальный IP клиента за Nginx или балансировщиком? / How do we find the real client IP behind Nginx or a load balancer?
> 
> Его берут из X-Forwarded-For или X-Real-IP, но этим заголовкам можно доверять, только если запрос пришёл от известного прокси. Иначе клиент может подделать заголовок.
> 
> We take it from X-Forwarded-For or X-Real-IP, but we can trust these headers only if the request came from a known proxy. Otherwise the client can fake the header.

> [!question] Зачем нужен заголовок Host? / Why do we need the Host header?
> 
> Он нужен, чтобы сервер понял, к какому из сайтов на одном IP обращается клиент (виртуальные хосты).
> 
> It is needed so that the server understands which of the sites on one IP the client wants (virtual hosts).

---

# 5. HTTP methods, safety and idempotency

## Main idea

Метод задаёт действие над ресурсом. Нужно обязательно знать разницу между PUT и PATCH, а также понимать безопасность (safe) и идемпотентность (idempotent).

The method defines the action on a resource. We must know the difference between PUT and PATCH, and understand safety (safe) and idempotency (idempotent).

## Safe and idempotent

Безопасный (Safe) метод только читает данные и не меняет состояние сервера. Идемпотентный (Idempotent) метод при многократном повторе даёт тот же эффект на сервере, что и при одном вызове. Это важно для безопасных повторов (retry) при сбоях сети. Идемпотентность говорит об эффекте на состояние, а не о том, что ответы одинаковые. Например, повторный DELETE вернёт 404, но состояние сервера будет то же.

A safe method only reads data and does not change the server state. An idempotent method, when repeated many times, has the same effect on the server as one call. This is important for safe retries after network failures. Idempotency is about the effect on the state, and not about the responses being the same. For example, a repeated DELETE returns 404, but the server state is the same.

## Methods one by one

- GET получает представление ресурса. Он безопасный, идемпотентный и кэшируемый, и у него нет тела запроса.
    
- HEAD работает как GET, но возвращает только заголовки. Он безопасный, идемпотентный и кэшируемый.
    
- OPTIONS возвращает поддерживаемые методы и используется для CORS preflight. Он безопасный и идемпотентный.
    
- PUT полностью заменяет ресурс по URI. Он небезопасный, но идемпотентный, и у него есть тело.
    
- DELETE удаляет ресурс. Он небезопасный, но идемпотентный.
    
- POST создаёт ресурс или запускает операцию. Он небезопасный и не идемпотентный, и почти не кэшируется.
    
- PATCH частично изменяет ресурс. Он небезопасный и не обязан быть идемпотентным: запрос {"op": "increment"} при повторе изменит данные ещё раз, а запрос {"status": "paid"} нет.
    
- GET gets a representation of a resource. It is safe, idempotent and cacheable, and it has no request body.
    
- HEAD works like GET, but returns only the headers. It is safe, idempotent and cacheable.
    
- OPTIONS returns the supported methods and is used for CORS preflight. It is safe and idempotent.
    
- PUT fully replaces a resource by its URI. It is not safe, but it is idempotent, and it has a body.
    
- DELETE removes a resource. It is not safe, but it is idempotent.
    
- POST creates a resource or starts an operation. It is not safe and not idempotent, and it is almost never cached.
    
- PATCH partially changes a resource. It is not safe and does not have to be idempotent: a request {"op": "increment"} changes the data again when repeated, but a request {"status": "paid"} does not.
    

## PUT vs PATCH, POST vs PUT and Idempotency-Key

PUT присылает весь ресурс целиком, и недостающие поля обнуляются или заменяются. PATCH присылает только изменяемые поля. При создании ресурса через POST /users сервер сам выбирает ID, а при PUT /users/42 URI указывает клиент.

Чтобы сделать POST идемпотентным, клиент присылает заголовок Idempotency-Key (UUID). Сервер сохраняет ключ и результат первого выполнения в Redis или в базе. Если приходит повторный запрос с тем же ключом, сервер возвращает сохранённый ответ и не выполняет действие заново, например не списывает деньги второй раз.

Нельзя использовать GET для изменений. GET кэшируется, браузеры и краулеры могут предзагружать его, и он попадает в логи и историю.

PUT sends the whole resource, and missing fields are cleared or replaced. PATCH sends only the fields that change. When we create a resource with POST /users, the server chooses the ID, and with PUT /users/42 the client specifies the URI.

To make POST idempotent, the client sends the header Idempotency-Key (a UUID). The server saves the key and the result of the first execution in Redis or in the database. If a repeated request comes with the same key, the server returns the saved response and does not run the action again, for example it does not charge the money a second time.

We must not use GET for changes. GET is cached, browsers and crawlers can prefetch it, and it ends up in logs and history.

> [!question] Идемпотентность и безопасность это одно и то же? / Are idempotency and safety the same thing?
> 
> Нет. DELETE и PUT идемпотентны, но небезопасны, потому что меняют данные. Все безопасные методы идемпотентны, но не наоборот.
> 
> No. DELETE and PUT are idempotent but not safe, because they change data. All safe methods are idempotent, but not the other way around.

> [!question] Когда использовать PUT, а когда PATCH? / When do we use PUT and when PATCH?
> 
> PUT нужен, когда клиент отправляет полное представление ресурса. PATCH нужен, когда меняется только часть полей. На практике многие API используют PATCH для частичных обновлений.
> 
> We use PUT when the client sends the full representation of the resource. We use PATCH when only some fields change. In practice, many APIs use PATCH for partial updates.

> [!question] Почему идемпотентность важна при повторных запросах? / Why is idempotency important for retries?
> 
> Если ответ потерялся, клиент не знает, выполнилась ли операция, и повторяет запрос. Идемпотентные операции безопасно повторять, а POST без защиты может создать дубликат, например двойную оплату.
> 
> If the response is lost, the client does not know whether the operation was done, and it repeats the request. Idempotent operations are safe to repeat, but an unprotected POST can create a duplicate, for example a double payment.

---

# 6. HTTP status codes

## Main idea

Код состояния это трёхзначное число в стартовой строке ответа. 1xx означает информацию, 2xx успех, 3xx перенаправление, 4xx ошибку клиента, а 5xx ошибку сервера.

The status code is a three-digit number in the start line of the response. 1xx means information, 2xx success, 3xx redirection, 4xx a client error and 5xx a server error.

## Codes one by one

- 101 Switching Protocols: сервер переключает протокол, например на WebSocket.
    
- 200 OK: стандартный успех.
    
- 201 Created: ресурс создан, обычно после POST, и часто с заголовком Location.
    
- 202 Accepted: запрос принят в фоновую обработку (очередь), результат будет позже.
    
- 204 No Content: успех без тела ответа, часто после DELETE или PUT.
    
- 301 Moved Permanently: постоянный редирект, он кэшируется, и POST может превратиться в GET.
    
- 302 Found: временный редирект, исторически POST превращается в GET.
    
- 304 Not Modified: ресурс не изменился, нужно взять копию из кэша.
    
- 307 и 308 Temporary и Permanent Redirect: редирект с сохранением метода и тела запроса.
    
- 400 Bad Request: некорректный запрос, например битый JSON.
    
- 401 Unauthorized: клиент не аутентифицирован, то есть токена нет или он неверный.
    
- 403 Forbidden: клиент аутентифицирован, но прав у него нет.
    
- 404 Not Found: ресурс не найден.
    
- 405 Method Not Allowed: метод не разрешён для этого URL.
    
- 409 Conflict: конфликт состояния, например дубликат или конфликт версий.
    
- 410 Gone: ресурс удалён навсегда.
    
- 415 Unsupported Media Type: сервер не поддерживает этот Content-Type.
    
- 422 Unprocessable Entity: синтаксис верный, но валидация не пройдена.
    
- 429 Too Many Requests: превышен лимит запросов, часто с заголовком Retry-After.
    
- 500 Internal Server Error: необработанная ошибка на сервере.
    
- 502 Bad Gateway: прокси получил некорректный ответ от upstream, например упал PHP-FPM.
    
- 503 Service Unavailable: сервер перегружен или идут технические работы.
    
- 504 Gateway Timeout: прокси не дождался ответа от upstream.
    
- 101 Switching Protocols: the server switches the protocol, for example to WebSocket.
    
- 200 OK: standard success.
    
- 201 Created: the resource was created, usually after POST, and often with a Location header.
    
- 202 Accepted: the request was accepted for background processing (a queue), and the result comes later.
    
- 204 No Content: success with an empty response body, often after DELETE or PUT.
    
- 301 Moved Permanently: a permanent redirect, it is cached, and POST may become GET.
    
- 302 Found: a temporary redirect, and historically POST becomes GET.
    
- 304 Not Modified: the resource did not change, so use the cached copy.
    
- 307 and 308 Temporary and Permanent Redirect: a redirect that keeps the method and the body of the request.
    
- 400 Bad Request: the request is malformed, for example broken JSON.
    
- 401 Unauthorized: the client is not authenticated, which means the token is missing or wrong.
    
- 403 Forbidden: the client is authenticated, but it has no permission.
    
- 404 Not Found: the resource was not found.
    
- 405 Method Not Allowed: the method is not allowed for this URL.
    
- 409 Conflict: a state conflict, for example a duplicate or a version conflict.
    
- 410 Gone: the resource was removed permanently.
    
- 415 Unsupported Media Type: the server does not support this Content-Type.
    
- 422 Unprocessable Entity: the syntax is valid, but validation failed.
    
- 429 Too Many Requests: the rate limit was exceeded, often with a Retry-After header.
    
- 500 Internal Server Error: an unhandled error on the server.
    
- 502 Bad Gateway: the proxy got an invalid response from the upstream, for example PHP-FPM crashed.
    
- 503 Service Unavailable: the server is overloaded or under maintenance.
    
- 504 Gateway Timeout: the proxy did not get a response from the upstream in time.
    

## Two things to remember

401 означает «я не знаю, кто ты», и нужно войти. 403 означает «я знаю, кто ты, но тебе нельзя». Если не нужно раскрывать существование ресурса, иногда вместо 403 отдают 404.

Для кодов 301 и 302 браузеры могут сменить POST на GET. Если нужно сохранить метод и тело запроса, используют 307 (временный редирект) и 308 (постоянный).

401 means "I do not know who you are", and you need to log in. 403 means "I know who you are, but you are not allowed". If we do not want to reveal that a resource exists, we sometimes return 404 instead of 403.

For the codes 301 and 302, browsers may change POST to GET. If we need to keep the method and the body, we use 307 (temporary redirect) and 308 (permanent redirect).

> [!question] Чем 401 отличается от 403? / What is the difference between 401 and 403?
> 
> При 401 аутентификация отсутствует или неверна. При 403 пользователь известен, но у него нет прав на это действие.
> 
> With 401, the authentication is missing or wrong. With 403, the user is known, but they have no permission for this action.

> [!question] Какой код вернуть при ошибке валидации? / Which code do we return for a validation error?
> 
> Обычно это 422 (синтаксис верный, но данные не прошли проверку) или 400. Главное, чтобы API был единообразным, а в теле ответа был список ошибок по полям.
> 
> Usually it is 422 (the syntax is valid, but the data failed the check) or 400. The main thing is that the API is consistent and the response body has a list of errors by field.

> [!question] В чём разница между 502 и 504? / What is the difference between 502 and 504?
> 
> При 502 upstream ответил некорректно или соединение сброшено, например процесс упал. При 504 upstream не ответил за отведённое время, то есть произошёл таймаут.
> 
> With 502, the upstream answered incorrectly or the connection was reset, for example the process crashed. With 504, the upstream did not answer in the given time, which means a timeout.

> [!question] Какой код вернуть, если запрос принят в очередь? / Which code do we return if the request was accepted into a queue?
> 
> Нужно вернуть 202 Accepted: результат будет позже, и можно вернуть ссылку на статус задачи.
> 
> We return 202 Accepted: the result comes later, and we can return a link to the task status.

---

# 7. Cookies, sessions and JWT

## Main idea

Cookie это небольшие данные (до примерно 4 КБ), которые браузер сохраняет и автоматически отправляет с запросами на подходящий домен и путь. Их используют для сессий и токенов.

A cookie is a small piece of data (up to about 4 KB) that the browser stores and sends automatically with requests to a matching domain and path. Cookies are used for sessions and tokens.

## How cookies work

Сначала сервер присылает заголовок Set-Cookie: session_id=xyz123; Path=/; HttpOnly; Secure; SameSite=Lax. Браузер сохраняет cookie, а при каждом следующем запросе на подходящий домен и путь добавляет заголовок Cookie: session_id=xyz123.

First, the server sends the header Set-Cookie: session_id=xyz123; Path=/; HttpOnly; Secure; SameSite=Lax. The browser saves the cookie, and with every next request to a matching domain and path it adds the header Cookie: session_id=xyz123.

## Set-Cookie attributes

- Expires и Max-Age задают срок жизни. Без них cookie сессионная и исчезает при закрытии браузера. Max-Age приоритетнее, чем Expires.
- Domain определяет, для каких доменов отправлять cookie. Запись .example.com включает поддомены.
- Path определяет, для каких путей отправлять cookie, например /admin.
- HttpOnly делает cookie недоступной из JavaScript (document.cookie). Это защищает от кражи сессии через XSS.
- Secure означает, что cookie передаётся только по HTTPS.
- SameSite защищает от CSRF. Значение Strict запрещает отправку при любых кросс-сайтовых запросах. Значение Lax (по умолчанию в современных браузерах) разрешает отправку при переходе по ссылке GET, но не при кросс-сайтовых POST и AJAX. Значение None разрешает всегда, но требует Secure.

Префиксы __Host- и __Secure- в имени cookie заставляют браузер проверять Secure, Path=/ и отсутствие Domain.

- Expires and Max-Age set the lifetime. Without them, the cookie is a session cookie and disappears when the browser closes. Max-Age has priority over Expires.
- Domain defines the domains where the cookie is sent. The value .example.com includes subdomains.
- Path defines the paths where the cookie is sent, for example /admin.
- HttpOnly makes the cookie unavailable to JavaScript (document.cookie). This protects against session theft through XSS.
- Secure means that the cookie is sent only over HTTPS.
- SameSite protects against CSRF. The value Strict forbids sending in any cross-site requests. The value Lax (the default in modern browsers) allows sending on a GET link navigation, but not in cross-site POST and AJAX requests. The value None allows always, but it requires Secure.

The prefixes __Host- and __Secure- in a cookie name make the browser check Secure, Path=/ and the absence of Domain.

## Sessions vs JWT

При использовании сессий данные хранятся на сервере (в Redis или в базе), а в cookie лежит только ID. Сессию можно отозвать мгновенно, удалив её. Минусы: на каждый запрос нужно обращаться в хранилище, есть риск CSRF (если используется cookie) и риск кражи ID.

JWT (Header.Payload.Signature) хранит данные внутри подписанного токена. Для проверки нужна только проверка подписи, без запроса в хранилище. Но отозвать токен до срока exp сложно: нужен чёрный список или короткий срок жизни. Payload только закодирован в Base64 и не зашифрован, поэтому секреты в него класть нельзя. Ещё есть риск кражи токена.

With sessions, the data is stored on the server (in Redis or in a database), and only the ID is in the cookie. We can revoke a session instantly by deleting it. The downsides are that every request needs a call to the storage, there is a CSRF risk (if a cookie is used) and a risk of ID theft.

A JWT (Header.Payload.Signature) keeps the data inside a signed token. We need only a signature check, without a call to a storage. But it is hard to revoke a token before its exp time: we need a blacklist or a short lifetime. The payload is only encoded in Base64 and not encrypted, so we must not put secrets into it. There is also a risk of token theft.

## Access and refresh tokens, storage and OAuth

Схема Access plus Refresh Token работает так: короткоживущий access-токен (5–15 минут) и долгоживущий refresh-токен, который хранится безопасно, например в HttpOnly cookie, и обменивается на новый access-токен.

Где хранить JWT в браузере? localStorage уязвим к XSS. HttpOnly cookie уязвима к CSRF, поэтому нужен SameSite или CSRF-токен. Компромисс выбирают под задачу. Для API-клиентов используют заголовок Authorization: Bearer <token>.

OAuth 2.0 это протокол делегирования доступа (например, «войти через Google» и токены доступа). OIDC это слой аутентификации поверх OAuth 2.0, который добавляет ID-токен.

The Access plus Refresh Token scheme works like this: a short-lived access token (5 to 15 minutes) and a long-lived refresh token, which is stored safely, for example in an HttpOnly cookie, and is exchanged for a new access token.

Where do we store a JWT in the browser? localStorage is vulnerable to XSS. An HttpOnly cookie is vulnerable to CSRF, so we need SameSite or a CSRF token. We choose the compromise for the task. For API clients, we use the header Authorization: Bearer <token>.

OAuth 2.0 is a protocol of access delegation (for example "sign in with Google" and access tokens). OIDC is an authentication layer on top of OAuth 2.0 that adds an ID token.

> [!question] Что защищает HttpOnly, а что SameSite? / What does HttpOnly protect, and what does SameSite protect?
> 
> HttpOnly не даёт JavaScript прочитать cookie, и это смягчает последствия XSS. SameSite ограничивает отправку cookie при кросс-сайтовых запросах, и это защита от CSRF. Это разные угрозы, поэтому обычно используют оба флага.
> 
> HttpOnly does not let JavaScript read the cookie, and this reduces the impact of XSS. SameSite limits sending the cookie in cross-site requests, and this protects against CSRF. These are different threats, so we usually use both flags.

> [!question] Сессии или JWT? / Sessions or JWT?
> 
> Сессии легко отзывать, но им нужно общее хранилище (Redis). JWT не требует запроса в хранилище и удобен для микросервисов, но его сложнее отзывать. Для классического веб-приложения сессии часто проще и безопаснее.
> 
> Sessions are easy to revoke, but they need a shared storage (Redis). A JWT does not need a call to a storage and is convenient for microservices, but it is harder to revoke. For a classic web application, sessions are often simpler and safer.

> [!question] JWT зашифрован? / Is a JWT encrypted?
> 
> Нет. Он подписан, то есть проверяется целостность, но payload может прочитать любой, потому что это Base64. Шифрованный вариант называется JWE, и он используется редко.
> 
> No. It is signed, which means integrity is checked, but anyone can read the payload, because it is Base64. The encrypted variant is called JWE, and it is rarely used.

---

# 8. Evolution of the protocol, HTTP/1.1, HTTP/2 and HTTP/3

## Main idea

HTTP/1.1 текстовый, и запросы идут по очереди в одном соединении. HTTP/2 бинарный и использует мультиплексирование в одном TCP-соединении. HTTP/3 работает поверх QUIC (UDP) и не имеет блокировок на уровне TCP.

HTTP/1.1 is text-based, and requests go one by one in a connection. HTTP/2 is binary and uses multiplexing in one TCP connection. HTTP/3 works over QUIC (UDP) and has no blocking at the TCP level.

## Differences between versions

HTTP/1.1 использует текстовый формат и транспорт TCP. Мультиплексирования в нём нет, поэтому браузер открывает до шести TCP-соединений на домен. Head-of-Line (HOL) blocking есть на уровне HTTP и TCP. Сжатия заголовков нет.

HTTP/2 использует бинарный формат (фреймы) и тоже транспорт TCP. В нём есть мультиплексирование: много потоков в одном соединении. HOL blocking снят на уровне HTTP, но остался на уровне TCP, потому что потеря пакета тормозит все потоки. Заголовки сжимаются через HPACK. Рукопожатие HTTPS занимает несколько RTT (TCP плюс TLS). Server Push есть, но на практике почти не используется и удалён из Chrome.

HTTP/3 использует бинарный формат и транспорт QUIC поверх UDP. Потоки QUIC независимы, поэтому потеря пакета тормозит только свой поток, и HOL blocking нет. Заголовки сжимаются через QPACK. Рукопожатие занимает 1 RTT, а для повторных соединений возможен 0-RTT, потому что TLS 1.3 встроен в QUIC. Server Push в HTTP/3 нет.

HTTP/1.1 uses a text format and the TCP transport. It has no multiplexing, so the browser opens up to six TCP connections per domain. Head-of-Line (HOL) blocking exists at the HTTP and the TCP level. There is no header compression.

HTTP/2 uses a binary format (frames) and also the TCP transport. It has multiplexing: many streams in one connection. HOL blocking is removed at the HTTP level, but it remains at the TCP level, because a lost packet slows down all the streams. Headers are compressed with HPACK. The HTTPS handshake takes several RTTs (TCP plus TLS). Server Push exists, but in practice it is almost never used and was removed from Chrome.

HTTP/3 uses a binary format and the QUIC transport over UDP. QUIC streams are independent, so a lost packet slows down only its own stream, and there is no HOL blocking. Headers are compressed with QPACK. The handshake takes 1 RTT, and 0-RTT is possible for repeated connections, because TLS 1.3 is built into QUIC. HTTP/3 has no Server Push.

## Other important points

HOL (Head-of-Line) blocking это ситуация, когда первый «застрявший» элемент очереди задерживает все остальные.

Для разработчика важно, что при HTTP/2 и HTTP/3 старые приёмы оптимизации HTTP/1.1 (спрайты, склейка CSS и JS, domain sharding) теряют смысл или даже вредят.

QUIC также поддерживает миграцию соединения: можно сменить сеть, например с Wi-Fi на мобильную, и соединение не разорвётся. ALPN и Alt-Svc это механизмы, через которые клиент и сервер договариваются о версии протокола (h2, h3).

HOL (Head-of-Line) blocking is a situation where the first "stuck" element in a queue delays all the others.

For a developer, it is important that with HTTP/2 and HTTP/3 the old optimization tricks of HTTP/1.1 (sprites, bundling CSS and JS, domain sharding) lose their sense or even do harm.

QUIC also supports connection migration: we can change the network, for example from Wi-Fi to mobile, and the connection does not break. ALPN and Alt-Svc are the mechanisms that the client and the server use to agree on the protocol version (h2, h3).

> [!question] Что такое мультиплексирование и чем оно лучше нескольких соединений? / What is multiplexing and why is it better than several connections?
> 
> Несколько запросов и ответов одновременно чередуются в одном соединении. Это экономит handshake и ресурсы, и контроль перегрузки работает лучше.
> 
> Several requests and responses are interleaved in one connection at the same time. This saves handshakes and resources, and congestion control works better.

> [!question] Почему HTTP/2 не избавился от HOL blocking полностью? / Why did HTTP/2 not remove HOL blocking completely?
> 
> HTTP/2 работает поверх TCP, а TCP гарантирует порядок байтов. Поэтому потерянный пакет задерживает все потоки в соединении. HTTP/3 решает эту проблему с помощью независимых потоков QUIC.
> 
> HTTP/2 works on top of TCP, and TCP guarantees the order of bytes. So a lost packet delays all the streams in the connection. HTTP/3 solves this problem with independent QUIC streams.

---

# 9. HTTP caching and conditional requests

## Main idea

Кэширование убирает лишние передачи данных. Оно работает через заголовок Cache-Control (свежесть) и через условные запросы (ETag или Last-Modified и ответ 304 Not Modified).

Caching removes unnecessary data transfers. It works through the Cache-Control header (freshness) and through conditional requests (ETag or Last-Modified and the 304 Not Modified response).

## Cache-Control directives

- max-age=3600 означает, что ресурс свежий один час, и браузер берёт его из кэша без запроса.
    
- public разрешает хранить ответ в CDN и прокси, а private разрешает хранить только в браузере пользователя.
    
- no-cache разрешает хранить ответ, но перед использованием его нужно проверить на сервере (revalidate).
    
- no-store запрещает хранить ответ вообще, и это нужно для конфиденциальных данных.
    
- must-revalidate требует после истечения срока обязательно проверить ответ на сервере.
    
- immutable означает, что ресурс никогда не изменится, например файл с хэшем в имени.
    
- stale-while-revalidate=N разрешает отдать устаревший ответ, пока в фоне идёт обновление.
    
- max-age=3600 means that the resource is fresh for one hour, and the browser takes it from the cache without a request.
    
- public allows storing the response in a CDN and proxies, and private allows storing it only in the user's browser.
    
- no-cache allows storing the response, but before using it we must check it on the server (revalidate).
    
- no-store forbids storing the response at all, and this is for confidential data.
    
- must-revalidate requires that after expiry the response must be checked on the server.
    
- immutable means that the resource will never change, for example a file with a hash in its name.
    
- stale-while-revalidate=N allows giving a stale response while an update runs in the background.
    

## Validation and other tools

Валидация работает так. Сервер отдаёт ETag: "hash123" (или Last-Modified). При следующем обращении клиент отправляет If-None-Match: "hash123" (или If-Modified-Since). Если ресурс не менялся, сервер отвечает 304 Not Modified без тела, и это экономит трафик.

Заголовок Vary перечисляет заголовки, от которых зависит ответ (например, Accept-Encoding или Accept-Language), чтобы кэш не отдал не ту версию.

Cache busting это приём для статики с долгим max-age: в имя файла добавляют хэш (app.3f9a1c.js), и новая версия получает новый URL.

Условные запросы также защищают от потерянных обновлений: заголовок If-Match вместе с ETag даёт оптимистичную блокировку в PUT и PATCH.

Validation works like this. The server gives ETag: "hash123" (or Last-Modified). With the next request, the client sends If-None-Match: "hash123" (or If-Modified-Since). If the resource did not change, the server answers 304 Not Modified without a body, and this saves traffic.

The Vary header lists the headers that the response depends on (for example Accept-Encoding or Accept-Language), so that the cache does not give the wrong version.

Cache busting is a technique for static files with a long max-age: we add a hash to the file name (app.3f9a1c.js), and the new version gets a new URL.

Conditional requests also protect against lost updates: the If-Match header together with ETag gives optimistic locking in PUT and PATCH.

```mermaid
sequenceDiagram
    participant B as Browser
    participant S as Server
    B->>S: GET /logo.png
    S-->>B: 200 OK, ETag: "abc", Cache-Control: max-age=60
    Note over B: After 60 seconds the cache is stale
    B->>S: GET /logo.png, If-None-Match: "abc"
    S-->>B: 304 Not Modified (no body)
```

> [!question] Чем no-cache отличается от no-store? / What is the difference between no-cache and no-store?
> 
> При no-cache хранить ответ можно, но перед каждым использованием нужна проверка на сервере. При no-store хранить ответ запрещено совсем.
> 
> With no-cache, we can store the response, but before every use we need a check on the server. With no-store, storing the response is forbidden completely.

> [!question] Как обновить JS-файл, закэшированный на год? / How do we update a JS file that is cached for a year?
> 
> Нужно изменить его URL, например имя с хэшем содержимого. Старый кэш остаётся, но страница ссылается на новый файл.
> 
> We need to change its URL, for example a name with a content hash. The old cache stays, but the page links to the new file.

> [!question] Чем ETag лучше, чем Last-Modified? / Why is ETag better than Last-Modified?
> 
> ETag точнее: он не зависит от секундной точности времени и от «ложных» изменений даты. Но генерация хэша может быть дороже.
> 
> ETag is more precise: it does not depend on the one-second precision of time or on "false" date changes. But generating a hash can be more expensive.

---

# 10. HTTP security, SOP, CORS and security headers

## Main idea

SOP запрещает скрипту с одного origin читать ответы другого origin. CORS это механизм, с помощью которого сервер явно разрешает такие запросы. CORS не защищает сервер от запросов, он защищает пользователя в браузере.

SOP forbids a script from one origin to read responses from another origin. CORS is the mechanism with which the server explicitly allows such requests. CORS does not protect the server from requests, it protects the user in the browser.

## Origin, SOP and CORS

Origin это схема, хост и порт вместе. Например, https://a.com и https://a.com:8443 это разные origin. По Same-Origin Policy (SOP) скрипт с одного origin не может читать ответы с другого.

В CORS (Cross-Origin Resource Sharing) простые запросы (GET, POST или HEAD с «безопасными» заголовками и типами) уходят сразу, а браузер проверяет заголовок ответа. Сложные запросы (PUT, DELETE, application/json, Authorization) сначала вызывают preflight: это запрос OPTIONS с заголовками Origin, Access-Control-Request-Method и Access-Control-Request-Headers. Сервер отвечает заголовками Access-Control-Allow-Origin, Access-Control-Allow-Methods и Access-Control-Allow-Headers (и Access-Control-Max-Age для кэширования preflight). Если запрос идёт с cookie (credentials: 'include'), то в Access-Control-Allow-Origin нельзя указывать *. Нужно указать конкретный origin и добавить Access-Control-Allow-Credentials: true.

Origin is the scheme, the host and the port together. For example, https://a.com and https://a.com:8443 are different origins. By the Same-Origin Policy (SOP), a script from one origin cannot read responses from another.

In CORS (Cross-Origin Resource Sharing), simple requests (GET, POST or HEAD with "safe" headers and types) go out immediately, and the browser checks the response header. Complex requests (PUT, DELETE, application/json, Authorization) first cause a preflight: this is an OPTIONS request with the headers Origin, Access-Control-Request-Method and Access-Control-Request-Headers. The server answers with the headers Access-Control-Allow-Origin, Access-Control-Allow-Methods and Access-Control-Allow-Headers (and Access-Control-Max-Age to cache the preflight). If the request goes with cookies (credentials: 'include'), we cannot use * in Access-Control-Allow-Origin. We must specify a concrete origin and add Access-Control-Allow-Credentials: true.

## Typical attacks

- XSS (Cross-Site Scripting) это внедрение чужого JavaScript на страницу. Защита: экранирование вывода, CSP и HttpOnly cookie.
    
- CSRF (Cross-Site Request Forgery) это ситуация, когда чужой сайт заставляет браузер отправить запрос с вашими cookie. Защита: SameSite, CSRF-токены и проверка Origin.
    
- Clickjacking это невидимый iframe поверх страницы. Защита: X-Frame-Options или frame-ancestors в CSP.
    
- SSL Stripping и MITM. Защита: HTTPS и HSTS.
    
- XSS (Cross-Site Scripting) is the injection of foreign JavaScript into a page. Protection: output escaping, CSP and HttpOnly cookies.
    
- CSRF (Cross-Site Request Forgery) is a situation where a foreign site makes the browser send a request with your cookies. Protection: SameSite, CSRF tokens and an Origin check.
    
- Clickjacking is an invisible iframe on top of the page. Protection: X-Frame-Options or frame-ancestors in CSP.
    
- SSL Stripping and MITM. Protection: HTTPS and HSTS.
    

## Security headers

- Strict-Transport-Security это HSTS, он разрешает только HTTPS.
    
- Content-Security-Policy задаёт белый список источников скриптов, стилей и картинок, и это защита от XSS.
    
- X-Content-Type-Options: nosniff запрещает браузеру угадывать MIME-тип.
    
- X-Frame-Options: DENY запрещает встраивание страницы в iframe, и это защита от Clickjacking.
    
- Referrer-Policy определяет, что отдавать в заголовке Referer.
    
- Permissions-Policy ограничивает доступ к камере, геолокации и другим функциям.
    
- Strict-Transport-Security is HSTS, and it allows only HTTPS.
    
- Content-Security-Policy sets a whitelist of sources for scripts, styles and images, and it protects against XSS.
    
- X-Content-Type-Options: nosniff forbids the browser to guess the MIME type.
    
- X-Frame-Options: DENY forbids embedding the page in an iframe, and it protects against Clickjacking.
    
- Referrer-Policy defines what to send in the Referer header.
    
- Permissions-Policy limits access to the camera, geolocation and other features.
    

> [!question] CORS защищает мой API от чужих сайтов? / Does CORS protect my API from other sites?
> 
> Нет. CORS работает только в браузере и ограничивает чтение ответа чужим скриптом. Запрос от curl или другого сервера уйдёт без ограничений. Поэтому API нужно защищать аутентификацией и авторизацией.
> 
> No. CORS works only in the browser and limits reading the response by a foreign script. A request from curl or another server goes out without limits. So we must protect the API with authentication and authorization.

> [!question] Зачем нужен preflight-запрос? / Why do we need a preflight request?
> 
> Браузер заранее проверяет, что сервер разрешает «потенциально опасный» запрос (с нестандартным методом или заголовками), прежде чем отправить его.
> 
> The browser checks in advance that the server allows a "potentially dangerous" request (with a non-standard method or headers) before it sends it.

> [!question] Чем XSS отличается от CSRF? / What is the difference between XSS and CSRF?
> 
> При XSS злоумышленник исполняет свой код в контексте вашего сайта. При CSRF злоумышленник заставляет браузер жертвы отправить запрос на ваш сайт с её cookie, но ответ он не читает.
> 
> With XSS, the attacker runs their own code in the context of your site. With CSRF, the attacker makes the victim's browser send a request to your site with the victim's cookies, but the attacker does not read the response.

> [!question] Почему нельзя Access-Control-Allow-Origin: * вместе с cookie? / Why can we not use Access-Control-Allow-Origin: * together with cookies?
> 
> Спецификация это запрещает, иначе любой сайт мог бы делать запросы от имени пользователя и читать ответы. Нужно указать конкретный origin.
> 
> The specification forbids it, because otherwise any site could make requests on behalf of the user and read the responses. We must specify a concrete origin.

---

# 11. API design, REST, GraphQL, gRPC, WebSockets

## Main idea

Выбор стиля зависит от задачи. REST подходит для стандартных публичных API, GraphQL для гибких клиентов, gRPC для быстрых внутренних микросервисов, а WebSockets и SSE для real-time.

The choice of style depends on the task. REST is for standard public APIs, GraphQL is for flexible clients, gRPC is for fast internal microservices, and WebSockets and SSE are for real-time.

## Styles of API

- REST использует HTTP и JSON, а ресурсы указываются в URL. Плюсы: простота, HTTP-кэширование и понятные коды. Минус: over-fetching и under-fetching.
    
- GraphQL использует POST /graphql и JSON. Плюсы: клиент запрашивает только нужные поля, и хватает одного запроса. Минусы: кэширование сложнее, есть риск N+1, и нужны лимиты глубины и сложности запросов.
    
- gRPC использует HTTP/2 и бинарный Protobuf. Плюсы: скорость, строгие контракты и стриминг. Минусы: для браузеров нужен gRPC-Web, и отладка сложнее.
    
- WebSockets работают по TCP после ответа HTTP 101. Плюс: двусторонний real-time без polling. Минус: stateful-соединения сложнее масштабировать.
    
- SSE (Server-Sent Events) это поток от сервера к клиенту по HTTP. Плюсы: это просто, и есть автопереподключение. Минус: данные идут только от сервера к клиенту.
    
- REST uses HTTP and JSON, and resources are specified in the URL. The advantages are simplicity, HTTP caching and clear codes. The disadvantage is over-fetching and under-fetching.
    
- GraphQL uses POST /graphql and JSON. The advantages are that the client asks only for the fields it needs, and one request is enough. The disadvantages are that caching is harder, there is an N+1 risk, and we need limits on query depth and complexity.
    
- gRPC uses HTTP/2 and binary Protobuf. The advantages are speed, strict contracts and streaming. The disadvantages are that browsers need gRPC-Web, and debugging is harder.
    
- WebSockets work over TCP after an HTTP 101 response. The advantage is two-way real-time without polling. The disadvantage is that stateful connections are harder to scale.
    
- SSE (Server-Sent Events) is a stream from the server to the client over HTTP. The advantages are that it is simple and has automatic reconnection. The disadvantage is that data goes only from the server to the client.
    

## REST principles and good practices

Принципы REST такие: ресурсы идентифицируются URL (существительные во множественном числе, например /users/42/orders), действия выражаются методами HTTP, взаимодействие stateless, коды ответов единообразны, а ответы кэшируемы.

Хороший REST API использует версионирование (/v1/... или заголовок), пагинацию (limit и offset или cursor) и фильтрацию и сортировку через query-параметры. У него единый формат ошибок (код, сообщение, поля) и корректные коды ответов. Также он поддерживает идемпотентность (Idempotency-Key), rate limiting (429) и аутентификацию (Bearer). Его документируют через OpenAPI или Swagger. HATEOAS (ссылки в ответах) это высший уровень зрелости по Ричардсону, но на практике он используется редко.

The principles of REST are these: resources are identified by URLs (nouns in the plural, for example /users/42/orders), actions are expressed by HTTP methods, the interaction is stateless, response codes are consistent, and responses are cacheable.

A good REST API uses versioning (/v1/... or a header), pagination (limit and offset or a cursor), and filtering and sorting through query parameters. It has one error format (code, message, fields) and correct response codes. It also supports idempotency (Idempotency-Key), rate limiting (429) and authentication (Bearer). It is documented with OpenAPI or Swagger. HATEOAS (links in responses) is the highest maturity level by Richardson, but in practice it is rarely used.

## Polling, long polling, WebSockets, SSE and webhooks

Polling означает, что клиент периодически спрашивает сервер. Это просто, но расточительно. Long polling означает, что сервер держит запрос, пока не появятся данные. WebSockets это постоянное двустороннее соединение для чатов, игр и торговли. SSE это односторонний поток событий для уведомлений и лент.

Webhooks работают наоборот: сервер сам вызывает URL клиента, когда происходит событие (например, прошла оплата). Для них нужны подписи, повторные попытки и идемпотентная обработка.

Polling means that the client asks the server from time to time. It is simple, but wasteful. Long polling means that the server holds the request until data appears. WebSockets are a permanent two-way connection for chats, games and trading. SSE is a one-way stream of events for notifications and feeds.

Webhooks work the other way around: the server itself calls the client's URL when an event happens (for example, a payment was successful). They need signatures, retries and idempotent processing.

> [!question] REST или GraphQL? / REST or GraphQL?
> 
> REST подходит для простых публичных API и кэширования на уровне HTTP. GraphQL подходит, когда есть много разных клиентов (web и mobile) с разными потребностями в данных. Платой за это становятся сложное кэширование и необходимость защиты от тяжёлых запросов.
> 
> REST is good for simple public APIs and for caching at the HTTP level. GraphQL is good when there are many different clients (web and mobile) with different data needs. The price for this is complex caching and the need to protect against heavy queries.

> [!question] Как масштабировать WebSockets? / How do we scale WebSockets?
> 
> Соединения stateful, поэтому нужен балансировщик с поддержкой WebSocket, общий брокер (Redis Pub/Sub или Kafka) для рассылки сообщений между узлами и учёт лимитов соединений на узел.
> 
> The connections are stateful, so we need a load balancer with WebSocket support, a shared broker (Redis Pub/Sub or Kafka) to send messages between nodes, and we must take the connection limits per node into account.

> [!question] Как выбрать между limit/offset и cursor-пагинацией? / How do we choose between limit/offset and cursor pagination?
> 
> Offset прост, но медленный на больших страницах и нестабилен при вставках. Cursor (keyset) быстрее и стабильнее, но не позволяет перейти на произвольную страницу.
> 
> Offset is simple, but slow on large pages and unstable when rows are inserted. Cursor (keyset) is faster and more stable, but it does not allow jumping to an arbitrary page.

---

# 12. Cheat sheet before the interview

## Checklist for HTTP and networks

1. Основы. HTTP stateless и работает на прикладном уровне. URL это схема, хост, порт, путь, query и #fragment. Фрагмент не уходит на сервер.
    
2. Жизненный цикл. DNS, потом TCP three-way handshake, потом TLS, потом HTTP-запрос. Keep-alive переиспользует соединение.
    
3. HTTPS и TLS. Асимметричная криптография (ECDHE и сертификат CA) нужна только для проверки подлинности и согласования ключа. Данные шифруются быстрым симметричным шифром (AES-GCM). Запомни PFS, HSTS и терминацию TLS на прокси.
    
4. Методы. Безопасные методы (GET, HEAD, OPTIONS) ничего не меняют. Идемпотентные методы (GET, PUT, DELETE) дают тот же эффект при повторе. POST делают идемпотентным через Idempotency-Key. PUT это полная замена, PATCH это частичное изменение.
    
5. Коды. 201 создано, 202 принято в очередь, 204 без тела, 304 из кэша, 401 не вошёл, 403 нет прав, 409 конфликт, 422 валидация, 429 лимит, 502 упал upstream, 504 таймаут upstream.
    
6. Cookies. HttpOnly защищает от XSS, Secure означает только HTTPS, SameSite защищает от CSRF. Без Max-Age и Expires cookie сессионная. JWT подписан, но не зашифрован.
    
7. HTTP/1.1, 2 и 3. Текст, потом бинарный формат и мультиплексирование, потом QUIC поверх UDP без HOL blocking на уровне транспорта.
    
8. Кэширование. max-age, no-cache (проверять), no-store (не хранить), ETag и If-None-Match дают 304.
    
9. Безопасность. SOP и CORS (preflight OPTIONS), XSS, CSRF и Clickjacking, заголовки CSP, HSTS, nosniff и X-Frame-Options.
    
10. API. REST по умолчанию, GraphQL для гибкости, gRPC для быстрого общения внутри, WebSockets и SSE для real-time.
    
11. Basics. HTTP is stateless and works at the application layer. A URL is the scheme, host, port, path, query and #fragment. The fragment does not go to the server.
    
12. Life cycle. DNS, then the TCP three-way handshake, then TLS, then the HTTP request. Keep-alive reuses the connection.
    
13. HTTPS and TLS. Asymmetric cryptography (ECDHE and a CA certificate) is used only to verify identity and to agree on a key. The data is encrypted with a fast symmetric cipher (AES-GCM). Remember PFS, HSTS and TLS termination on a proxy.
    
14. Methods. Safe methods (GET, HEAD, OPTIONS) change nothing. Idempotent methods (GET, PUT, DELETE) give the same effect when repeated. We make POST idempotent with an Idempotency-Key. PUT is a full replacement, and PATCH is a partial change.
    
15. Codes. 201 created, 202 accepted into a queue, 204 no body, 304 from cache, 401 not logged in, 403 no permission, 409 conflict, 422 validation, 429 rate limit, 502 upstream crashed, 504 upstream timeout.
    
16. Cookies. HttpOnly protects against XSS, Secure means HTTPS only, SameSite protects against CSRF. Without Max-Age and Expires, a cookie is a session cookie. A JWT is signed, but not encrypted.
    
17. HTTP/1.1, 2 and 3. Text, then binary format and multiplexing, then QUIC over UDP without HOL blocking at the transport level.
    
18. Caching. max-age, no-cache (revalidate), no-store (do not store), ETag and If-None-Match give 304.
    
19. Security. SOP and CORS (the OPTIONS preflight), XSS, CSRF and Clickjacking, the headers CSP, HSTS, nosniff and X-Frame-Options.
    
20. API. REST by default, GraphQL for flexibility, gRPC for fast communication inside, WebSockets and SSE for real-time.
    

## Useful phrases for answers

Эти английские фразы помогают структурировать ответ на собеседовании.

These English phrases help to structure an answer in the interview.

- To give a definition: "X is a ... that is used to ..."
- To explain a difference: "The key difference is that ... while ..."
- To give an example: "For example, when a user logs in, ..."
- To describe a sequence: "First, ..., then ..., and finally ..."
- To admit that you do not know: "I'm not sure about the exact details, but as far as I know..."

## Questions that are often asked

- Что происходит, когда ты вводишь URL в браузере и нажимаешь Enter? / What happens when you type a URL in the browser and press Enter?
- Чем HTTP отличается от HTTPS? Как работает TLS handshake? / What is the difference between HTTP and HTTPS, and how does the TLS handshake work?
- PUT vs PATCH. Что такое идемпотентность и как сделать POST идемпотентным? / PUT vs PATCH. What is idempotency, and how do we make POST idempotent?
- 401 vs 403, 502 vs 504, 301 vs 302 vs 307 и 308? / 401 vs 403, 502 vs 504, 301 vs 302 vs 307 and 308?
- Как работают cookies? Для чего HttpOnly, Secure и SameSite? Сессии или JWT? / How do cookies work, what are HttpOnly, Secure and SameSite for, and sessions or JWT?
- Чем HTTP/2 отличается от HTTP/1.1, а HTTP/3 от HTTP/2? / How is HTTP/2 different from HTTP/1.1, and HTTP/3 from HTTP/2?
- Что такое CORS и preflight? Чем XSS отличается от CSRF? / What are CORS and preflight, and what is the difference between XSS and CSRF?
- REST, GraphQL или gRPC: когда что выбрать? / REST, GraphQL or gRPC: when do we choose which?

## The classic question: what happens when you type a URL and press Enter

Это классический вопрос. Ответ состоит из восьми шагов.

This is a classic question. The answer has eight steps.

1. Браузер разбирает URL и проверяет HSTS и кэш.
    
2. DNS переводит домен в IP: сначала кэш браузера, потом ОС, потом резолвер, потом иерархия DNS.
    
3. TCP: three-way handshake (или QUIC для HTTP/3).
    
4. TLS: рукопожатие, проверка сертификата и выработка общего ключа.
    
5. HTTP-запрос: GET / HTTP/1.1, заголовок Host, cookies и другие заголовки.
    
6. Балансировщик или Nginx принимает запрос, терминирует TLS и передаёт запрос в PHP-FPM.
    
7. Приложение обрабатывает запрос (middleware, контроллер, база данных или кэш) и формирует ответ.
    
8. Браузер получает ответ, разбирает HTML, запрашивает CSS, JS и изображения (с учётом кэша), строит DOM и рисует страницу.


9. The browser parses the URL and checks HSTS and the cache.
    
10. DNS translates the domain into an IP: first the browser cache, then the OS, then the resolver, then the DNS hierarchy.
    
11. TCP: the three-way handshake (or QUIC for HTTP/3).
    
12. TLS: the handshake, the certificate check and the creation of a shared key.
    
13. HTTP request: GET / HTTP/1.1, the Host header, cookies and other headers.
    
14. The load balancer or Nginx receives the request, terminates TLS and passes the request to PHP-FPM.
    
15. The application handles the request (middleware, controller, database or cache) and builds a response.
    
16. The browser receives the response, parses the HTML, requests CSS, JS and images (taking the cache into account), builds the DOM and renders the page.