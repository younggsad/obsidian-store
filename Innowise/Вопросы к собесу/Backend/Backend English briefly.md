---
custom-width: 80
---
# 1. Backend Development Fundamentals

### What is Backend Development?

Backend Development is the server-side part of an application. It handles business logic, databases, authentication, authorization, and communication with other services.

### What is Frontend?

Frontend is the client-side part of an application. It is responsible for the user interface and user interactions.

### What is the difference between Frontend and Backend?

- Frontend handles the user interface.
    
- Backend handles business logic, data processing, and server-side operations.
    
- They communicate through APIs, usually using HTTP.
    

### What is a Client-Server Architecture?

It is an architecture where a client sends requests to a server, and the server processes them and returns responses.

Example:

1. A user opens a website.
    
2. The browser sends an HTTP request.
    
3. The backend processes the request.
    
4. The backend communicates with the database.
    
5. The server returns a response.
    
6. The browser displays the result.
    

### What is a Web Server?

A Web Server receives HTTP requests and returns HTTP responses.

Examples:

- Nginx.
    
- Apache.
    

### What is an Application Server?

An Application Server executes application logic and processes requests.

For example, PHP-FPM executes PHP scripts, while Nginx can act as a reverse proxy and serve static files.

### What is the difference between Nginx and PHP-FPM?

- Nginx handles HTTP connections, static files, and reverse proxying.
    
- PHP-FPM manages PHP worker processes that execute PHP code.
    
- Nginx forwards PHP requests to PHP-FPM through FastCGI.
    

### What is Middleware?

Middleware is a component that processes requests or responses before they reach the main application logic or before the response is returned.

Examples:

- Authentication.
    
- Logging.
    
- Request validation.
    
- Rate limiting.
    

### What is Business Logic?

Business Logic represents the rules and operations of an application.

Example: when creating an order, the application checks product availability, calculates the total price, and validates the payment.

### What is an Application Lifecycle?

It describes how an application starts, receives requests, processes them, and returns responses.

---

# 2. HTTP Protocol

### What is HTTP?

HTTP (Hypertext Transfer Protocol) is an application-layer protocol used for communication between clients and servers.

It follows a request-response model.

### What is HTTPS?

HTTPS is HTTP secured with TLS encryption.

It provides:

- Confidentiality.
    
- Data integrity.
    
- Server authentication.
    

### What is the difference between HTTP and HTTPS?

HTTP sends data without transport encryption.

HTTPS uses TLS to encrypt communication and verify the server's identity.

### What is an HTTP Request?

An HTTP request is a message sent from a client to a server.

It contains:

- Method.
    
- URL.
    
- Headers.
    
- Optional body.
    

Example:

```
POST /api/users HTTP/1.1
Host: example.com
Content-Type: application/json
Authorization: Bearer token

{
    "name": "John",
    "email": "john@example.com"
}
```

### What is an HTTP Response?

An HTTP response is a message returned by the server.

It contains:

- Status code.
    
- Headers.
    
- Optional body.
    

### What are HTTP Methods?

|Method|Purpose|
|---|---|
|GET|Retrieve data|
|POST|Create or process data|
|PUT|Replace a resource|
|PATCH|Partially update a resource|
|DELETE|Delete a resource|
|HEAD|Retrieve headers only|
|OPTIONS|Describe communication options|

### What is the difference between GET and POST?

- GET retrieves data and should not change server state.
    
- POST submits data and can create resources or trigger operations.
    
- GET parameters are commonly placed in the URL.
    
- POST data is commonly sent in the request body.
    

### What is the difference between PUT and PATCH?

- PUT replaces the complete resource representation.
    
- PATCH applies partial modifications.
    

### What is an Idempotent HTTP Method?

An operation is idempotent if repeating the same request produces the same intended server state.

Examples:

- GET.
    
- PUT.
    
- DELETE.
    

POST is not idempotent by default.

### What are HTTP Headers?

Headers provide additional information about requests and responses.

Examples:

|Header|Purpose|
|---|---|
|Content-Type|Describes the body format|
|Accept|Specifies acceptable response formats|
|Authorization|Contains authentication credentials|
|Cache-Control|Defines caching behavior|
|Cookie|Sends cookies to the server|
|Set-Cookie|Sets a cookie|
|Origin|Identifies the request origin|
|User-Agent|Identifies the client|

### What is Content-Type?

Content-Type specifies the media type of the request or response body.

Examples:

- `application/json`
    
- `text/html`
    
- `application/xml`
    
- `multipart/form-data`
    

### What is a Cookie?

A Cookie is a small piece of data stored by the browser and sent to the server with matching requests.

Cookies are commonly used for:

- Session management.
    
- Authentication.
    
- User preferences.
    

### What is a Session?

A Session stores user-specific data on the server between HTTP requests.

Usually, the browser stores a session identifier in a cookie.

### What is the difference between Cookies and Sessions?

- Cookies are stored on the client side.
    
- Session data is usually stored on the server side.
    
- A cookie can contain a session identifier that allows the server to find the corresponding session.
    

### What is Stateless HTTP?

HTTP is stateless because each request is independent by default.

The server does not automatically remember previous requests.

Sessions, cookies, and tokens can be used to maintain application state.

### What are HTTP Status Codes?

**1xx — Informational**

- 100 Continue.
    

**2xx — Success**

- 200 OK.
    
- 201 Created.
    
- 202 Accepted.
    
- 204 No Content.
    

**3xx — Redirection**

- 301 Moved Permanently.
    
- 302 Found.
    
- 304 Not Modified.
    

**4xx — Client Errors**

- 400 Bad Request.
    
- 401 Unauthorized.
    
- 403 Forbidden.
    
- 404 Not Found.
    
- 405 Method Not Allowed.
    
- 409 Conflict.
    
- 422 Unprocessable Content.
    
- 429 Too Many Requests.
    

**5xx — Server Errors**

- 500 Internal Server Error.
    
- 502 Bad Gateway.
    
- 503 Service Unavailable.
    
- 504 Gateway Timeout.
    

### What is the difference between 401 and 403?

- 401 means authentication is required or has failed.
    
- 403 means the server understood the request but refuses access.
    

### What is the difference between 502 and 504?

- 502 Bad Gateway means a gateway received an invalid response from an upstream server.
    
- 504 Gateway Timeout means the upstream server did not respond in time.
    

### What is HTTP Caching?

HTTP caching stores responses in browsers or intermediary caches to reduce network traffic and server load.

Important headers:

- Cache-Control.
    
- ETag.
    
- Last-Modified.
    
- Expires.
    

---

# 3. Networking Fundamentals

### What is an IP Address?

An IP Address identifies a device or network interface in an IP network.

There are two main versions:

- IPv4.
    
- IPv6.
    

### What is the difference between IPv4 and IPv6?

- IPv4 uses 32-bit addresses.
    
- IPv6 uses 128-bit addresses.
    
- IPv6 provides a much larger address space.
    

### What is DNS?

DNS (Domain Name System) translates domain names into IP addresses.

Example:

`example.com` → `93.184.216.34`

### What is a Port?

A Port identifies a specific service or application endpoint on a device.

Common ports:

|Port|Service|
|---|---|
|80|HTTP|
|443|HTTPS|
|22|SSH|
|3306|MySQL|
|5432|PostgreSQL|
|6379|Redis|
|25|SMTP|
|53|DNS|

### What is TCP?

TCP is a connection-oriented transport protocol that provides reliable and ordered data delivery.

### What is UDP?

UDP is a connectionless transport protocol with lower overhead but without guaranteed delivery or ordering.

### What is the difference between TCP and UDP?

- TCP provides reliable delivery and ordering.
    
- UDP has lower overhead and does not guarantee delivery.
    
- TCP is commonly used for HTTP communication.
    
- UDP is commonly used for DNS queries, streaming, and real-time communication.
    

### What is the TCP Three-Way Handshake?

It is the process used to establish a TCP connection.

1. SYN — client requests a connection.
    
2. SYN-ACK — server acknowledges and responds.
    
3. ACK — client confirms the connection.
    

### What is TLS?

TLS (Transport Layer Security) protects communication between two systems.

It provides encryption, integrity, and authentication.

### What is a Firewall?

A Firewall controls network traffic based on configured security rules.

### What is a Reverse Proxy?

A Reverse Proxy receives requests from clients and forwards them to backend servers.

Examples:

- Nginx.
    
- HAProxy.
    
- Traefik.
    

Benefits:

- Load balancing.
    
- SSL/TLS termination.
    
- Request routing.
    
- Caching.
    
- Security.
    

### What is Load Balancing?

Load Balancing distributes incoming traffic across multiple servers.

It improves availability and helps handle higher traffic.

Common algorithms:

- Round Robin.
    
- Least Connections.
    
- Weighted Round Robin.
    
- IP Hash.
    

---

# 4. Database Fundamentals

### What is a Database?

A Database is an organized collection of data that can be stored, retrieved, and modified efficiently.

### What is a DBMS?

A Database Management System is software used to create, manage, and interact with databases.

Examples:

- MySQL.
    
- PostgreSQL.
    
- MariaDB.
    
- MongoDB.
    

### What is SQL?

SQL (Structured Query Language) is a language used to manage and query relational databases.

### What is a Relational Database?

A Relational Database stores data in tables consisting of rows and columns.

Tables can be connected using relationships.

### What is a Table?

A Table is a collection of related records.

### What is a Row?

A Row represents one record in a table.

### What is a Column?

A Column represents a specific attribute of a record.

### What is a Primary Key?

A Primary Key uniquely identifies each row in a table.

A primary key must be unique and cannot contain NULL values.

### What is a Foreign Key?

A Foreign Key creates a relationship between tables by referencing a primary key or another unique key.

### What is a Unique Constraint?

A Unique Constraint ensures that values in a column or a combination of columns are unique.

### What is a Composite Key?

A Composite Key consists of two or more columns that together uniquely identify a record.

### What is an Index?

An Index is a data structure that improves data retrieval speed.

It usually requires additional storage and can slow down INSERT, UPDATE, and DELETE operations.

### What types of indexes exist?

Common index types:

- B-tree.
    
- Hash.
    
- Full-text.
    
- Composite indexes.
    

The available types depend on the database engine.

### What is a Composite Index?

A Composite Index is an index created using multiple columns.

Example:

```
CREATE INDEX idx_status_created
ON orders (status, created_at);
```

Column order matters because databases can use the leftmost prefix of a composite index.

### What is a Clustered Index?

A Clustered Index determines how table records are organized in the underlying storage structure.

For example, InnoDB uses the primary key as the clustered index.

### What is a Non-Clustered Index?

A Non-Clustered Index is a separate index structure that points to the corresponding table records.

### What is Database Normalization?

Normalization organizes database data to reduce duplication and improve consistency.

### What are the main Normal Forms?

**First Normal Form (1NF):**

- Each column contains atomic values.
    
- No repeating groups.
    

**Second Normal Form (2NF):**

- Must satisfy 1NF.
    
- Non-key attributes must depend on the whole candidate key.
    

**Third Normal Form (3NF):**

- Must satisfy 2NF.
    
- Non-key attributes should not depend transitively on candidate keys.
    

### What is Denormalization?

Denormalization intentionally introduces data duplication to improve read performance or simplify queries.

It requires careful consistency management.

### What are Database Relationships?

**One-to-One:**  
One record is related to one record.

**One-to-Many:**  
One record can be related to multiple records.

**Many-to-Many:**  
Multiple records can be related to multiple records through a junction table.

Example:

A user can have multiple roles, and a role can belong to multiple users.

### What is a JOIN?

JOIN combines rows from multiple tables based on a related column.

Types:

- INNER JOIN — matching records only.
    
- LEFT JOIN — all records from the left table and matching records from the right.
    
- RIGHT JOIN — all records from the right table and matching records from the left.
    
- CROSS JOIN — Cartesian product.
    

### What is a Subquery?

A Subquery is a query nested inside another SQL query.

### What is the difference between WHERE and HAVING?

- WHERE filters rows before grouping.
    
- HAVING filters groups after aggregation.
    

### What is GROUP BY?

GROUP BY groups rows with matching values, usually for aggregate calculations.

### What are Aggregate Functions?

Functions that calculate a result from multiple rows.

Examples:

- COUNT().
    
- SUM().
    
- AVG().
    
- MIN().
    
- MAX().
    

### What is the difference between DELETE, TRUNCATE, and DROP?

- DELETE removes selected rows and supports WHERE.
    
- TRUNCATE removes all rows from a table.
    
- DROP removes the table itself.
    

Their transaction and identity-reset behavior depends on the database engine.

---

# 5. SQL Transactions and Concurrency

### What is a Database Transaction?

A Transaction is a group of database operations treated as one logical unit.

Either all operations are committed, or changes are rolled back.

### What are ACID Properties?

**Atomicity:** All operations succeed or none of them do.

**Consistency:** A transaction preserves database constraints and valid states.

**Isolation:** Concurrent transactions are controlled to prevent incorrect interference.

**Durability:** Committed changes survive failures according to the database's durability guarantees.

### What is COMMIT?

COMMIT permanently applies the changes made by a transaction.

### What is ROLLBACK?

ROLLBACK cancels uncommitted changes made by a transaction.

### What is Isolation Level?

Isolation Level defines how much one transaction can see the changes made by other concurrent transactions.

Common levels:

- Read Uncommitted.
    
- Read Committed.
    
- Repeatable Read.
    
- Serializable.
    

### What is a Dirty Read?

A Dirty Read happens when a transaction reads data that another transaction has modified but not committed.

### What is a Non-Repeatable Read?

It happens when the same row is read twice in one transaction and returns different values because another transaction committed an update.

### What is a Phantom Read?

It happens when repeating a query returns additional or missing rows because another transaction changed the matching dataset.

### What is a Deadlock?

A Deadlock occurs when transactions wait for resources locked by each other.

Example:

- Transaction A locks row 1 and waits for row 2.
    
- Transaction B locks row 2 and waits for row 1.
    

Neither transaction can continue until the database detects and resolves the deadlock.

### How do you prevent Deadlocks?

- Keep transactions short.
    
- Access resources in a consistent order.
    
- Use appropriate indexes.
    
- Avoid unnecessary locks.
    
- Implement retry logic.
    

### What is Optimistic Locking?

Optimistic Locking assumes conflicts are rare.

It checks whether a record has changed before updating it, often using a version column.

### What is Pessimistic Locking?

Pessimistic Locking locks records to prevent conflicting operations.

Example:

```
SELECT *
FROM accounts
WHERE id = 10
FOR UPDATE;
```

### What is a Race Condition?

A Race Condition occurs when the result depends on the timing or order of concurrent operations.

Example: two requests try to purchase the last available product at the same time.

### How do you prevent a Race Condition?

Use transactions, row-level locking, atomic operations, unique constraints, and proper concurrency control.

---

# 6. Database Performance

### What is Query Optimization?

Query Optimization means improving SQL queries to reduce execution time and resource consumption.

### How do you optimize SQL queries?

- Add appropriate indexes.
    
- Avoid unnecessary columns in SELECT.
    
- Analyze query execution plans.
    
- Avoid unnecessary JOINs.
    
- Reduce repeated queries.
    
- Use pagination.
    
- Avoid N+1 queries.
    

### What is EXPLAIN?

EXPLAIN shows how the database plans to execute a query.

It helps identify:

- Table scans.
    
- Index usage.
    
- Join operations.
    
- Estimated rows.
    
- Query execution strategy.
    

### What is EXPLAIN ANALYZE?

EXPLAIN ANALYZE executes the query and provides actual execution statistics, where supported.

### What is a Full Table Scan?

A Full Table Scan occurs when the database reads the entire table to find matching records.

It can be expensive for large tables.

### What is the N+1 Query Problem?

It happens when one query retrieves a collection and then one additional query is executed for each item.

Example:

```
$users = getUsers();

foreach ($users as $user) {
    $orders = getOrdersByUser($user['id']);
}
```

If there are 100 users, this can produce 101 queries.

Solution:

- Use JOINs.
    
- Use eager loading.
    
- Fetch related records in batches.
    

### What is Connection Pooling?

Connection Pooling reuses database connections instead of creating a new connection for every operation.

It reduces connection overhead and helps manage database resources.

### What is Database Replication?

Replication copies data from one database server to another.

Common architecture:

- Primary — handles writes.
    
- Replicas — handle reads.
    

### What is Database Sharding?

Sharding splits data across multiple database servers.

Each server stores a subset of the total dataset.

### What is the difference between Replication and Sharding?

- Replication creates copies of data.
    
- Sharding divides data into separate parts.
    

### What is a Database Backup?

A Database Backup is a copy of database data used for recovery.

Types:

- Full backup.
    
- Incremental backup.
    
- Differential backup.
    

A backup strategy must include regular restoration testing.

---

# 7. REST API and Web Services

### What is an API?

An API (Application Programming Interface) defines how different software components communicate.

### What is a REST API?

A REST API exposes resources through HTTP endpoints and standard HTTP methods.

Example:

```
GET    /api/users
GET    /api/users/10
POST   /api/users
PUT    /api/users/10
PATCH  /api/users/10
DELETE /api/users/10
```

### What are REST Principles?

Important REST constraints include:

- Client-server architecture.
    
- Stateless communication.
    
- Cacheability.
    
- Uniform interface.
    
- Layered system.
    
- Optional code-on-demand.
    

### What is JSON?

JSON is a lightweight data format used to exchange information between systems.

Example:

```
{
    "id": 10,
    "name": "John",
    "active": true
}
```

### What is Serialization?

Serialization converts an object or data structure into a format suitable for storage or transmission.

### What is Deserialization?

Deserialization converts serialized data back into an object or data structure.

### What is API Versioning?

API Versioning allows an API to evolve without breaking existing clients.

Common approaches:

```
/api/v1/users
/api/v2/users
```

Other approaches include version headers and media types.

### What is Pagination?

Pagination splits a large dataset into smaller responses.

Common approaches:

- Offset pagination.
    
- Cursor pagination.
    

### What is Rate Limiting?

Rate Limiting restricts how many requests a client can make within a specific time period.

It helps prevent abuse and protects server resources.

### What is Throttling?

Throttling controls the rate at which requests or operations are processed.

It can slow down or reject excessive traffic.

### What is API Idempotency?

Idempotency ensures that repeating an operation does not produce duplicate side effects.

Example: a payment API can use an idempotency key to prevent charging a customer twice.

### What is Webhook?

A Webhook is an HTTP callback sent by one system to another when a specific event occurs.

Example: a payment provider sends a webhook when a payment is completed.

### What is the difference between Polling and Webhooks?

- Polling means periodically asking another service for updates.
    
- Webhooks send notifications when an event happens.
    

Webhooks can reduce unnecessary requests.

### What is GraphQL?

GraphQL is a query language and runtime for APIs.

Clients can request exactly the data they need.

### What is the difference between REST and GraphQL?

- REST commonly exposes multiple resource endpoints.
    
- GraphQL usually exposes one endpoint with a flexible query language.
    
- REST uses HTTP methods and resource representations.
    
- GraphQL allows clients to specify the requested data structure.
    

### What is gRPC?

gRPC is a high-performance RPC framework that commonly uses Protocol Buffers and HTTP/2.

It is often used for communication between backend services.

---

# 8. Authentication and Authorization

### What is Authentication?

Authentication verifies the identity of a user or system.

Example: username and password verification.

### What is Authorization?

Authorization determines what an authenticated user is allowed to do.

Example: an administrator can delete users, while a regular user cannot.

### What is the difference between Authentication and Authorization?

Authentication answers: "Who are you?"

Authorization answers: "What are you allowed to do?"

### What is Session-Based Authentication?

The server creates a session after successful authentication.

The browser stores a session identifier, usually in a cookie.

### What is Token-Based Authentication?

The server issues a token that the client sends with subsequent requests.

The server validates the token to identify the client or user.

### What is JWT?

JWT (JSON Web Token) is a compact token format used to transmit claims between parties.

It consists of:

- Header.
    
- Payload.
    
- Signature.
    

The payload is Base64URL-encoded, not encrypted by default.

### What is the difference between JWT and Sessions?

**Sessions:**

- State is stored on the server.
    
- The client usually stores a session identifier.
    
- Session invalidation is straightforward with centralized storage.
    

**JWT:**

- Claims are stored in the token.
    
- The server verifies the token signature.
    
- Revocation and permission changes require additional design.
    

### What is OAuth 2.0?

OAuth 2.0 is an authorization framework that allows applications to access resources on behalf of a user or another application.

### What is OpenID Connect?

OpenID Connect (OIDC) is an identity layer built on top of OAuth 2.0.

It adds authentication and standardized identity information.

### What is Multi-Factor Authentication?

MFA requires more than one authentication factor.

Examples:

- Password.
    
- Authentication application.
    
- Hardware security key.
    

### What is Role-Based Access Control?

RBAC assigns permissions to roles, and users receive permissions through their assigned roles.

### What is Attribute-Based Access Control?

ABAC makes authorization decisions based on attributes such as user properties, resource properties, and environment conditions.

---

# 9. Web Security

### What is SQL Injection?

SQL Injection happens when untrusted input is inserted into an SQL query in an unsafe way.

Prevention:

- Prepared statements.
    
- Parameterized queries.
    
- Input validation.
    
- Least-privilege database accounts.
    

### What is XSS?

Cross-Site Scripting allows attackers to inject malicious scripts into webpages viewed by other users.

Prevention:

- Context-aware output escaping.
    
- Input validation.
    
- Content Security Policy.
    
- Avoid unsafe HTML rendering.
    

### What is CSRF?

Cross-Site Request Forgery tricks an authenticated browser into making an unwanted request.

Prevention:

- CSRF tokens.
    
- SameSite cookies.
    
- Origin checks.
    
- Appropriate authorization checks.
    

### What is CORS?

Cross-Origin Resource Sharing is a browser security mechanism that controls whether a webpage can access resources from another origin.

### What is the Same-Origin Policy?

The Same-Origin Policy restricts how documents and scripts from one origin can interact with resources from another origin.

An origin consists of:

- Scheme.
    
- Host.
    
- Port.
    

### What is an XSS vs CSRF?

- XSS injects malicious scripts into a trusted webpage.
    
- CSRF abuses an authenticated user's browser to perform unwanted actions.
    

### What is Password Hashing?

Password Hashing transforms a password into a one-way hash.

Passwords should be stored using dedicated password hashing algorithms such as Argon2id or bcrypt.

### What is the difference between Hashing and Encryption?

- Hashing is designed to be one-way.
    
- Encryption is reversible using the appropriate key.
    

### What is Brute Force?

A Brute Force Attack repeatedly tries possible passwords or credentials.

Prevention:

- Rate limiting.
    
- Account lockout or progressive delays.
    
- MFA.
    
- Monitoring suspicious activity.
    

### What is CSRF Token?

A CSRF Token is an unpredictable value used to verify that a state-changing request originated from a legitimate application interaction.

### What is Content Security Policy?

CSP is an HTTP security policy that restricts which sources can load scripts, styles, images, and other resources.

### What is the Principle of Least Privilege?

Every user, application, or service should have only the permissions required to perform its tasks.

### What is Security Misconfiguration?

Security Misconfiguration happens when systems use unsafe settings, exposed debug tools, default credentials, or unnecessary permissions.

---

# 10. Caching

### What is Caching?

Caching stores frequently accessed data so that it can be retrieved faster without repeating expensive operations.

### Why do we need caching?

- Reduce database load.
    
- Improve response time.
    
- Reduce CPU usage.
    
- Handle higher traffic.
    

### What types of caching exist?

- Browser Cache.
    
- HTTP Cache.
    
- Application Cache.
    
- Database Cache.
    
- Distributed Cache.
    
- CDN Cache.
    

### What is Redis?

Redis is an in-memory data store commonly used for caching, session storage, queues, counters, and other fast data operations.

### What is Memcached?

Memcached is an in-memory key-value caching system designed for simple and fast data caching.

### What is the difference between Redis and Memcached?

- Redis supports richer data structures and additional capabilities.
    
- Memcached focuses on simple key-value caching.
    
- Redis supports persistence options and replication.
    
- Both can be used to reduce database load.
    

### What is Cache Invalidation?

Cache Invalidation is the process of marking cached data as outdated or removing it when the underlying data changes.

### What is TTL?

TTL (Time To Live) defines how long a cached item remains valid.

Example:

```
user:10 → cached data → TTL: 300 seconds
```

### What is Cache-Aside?

Cache-Aside is a caching pattern where the application checks the cache first.

If data is missing, it retrieves it from the database and stores it in the cache.

### What is Write-Through Caching?

Write-Through means data is written to the cache and the underlying storage as part of the write operation.

### What is Write-Behind Caching?

Write-Behind means data is written to the cache first and persisted to the database asynchronously.

It requires careful consistency and durability handling.

### What is Cache Stampede?

A Cache Stampede happens when many requests attempt to rebuild the same expired cache entry simultaneously.

Prevention:

- Request locking.
    
- Stale-while-revalidate.
    
- Randomized TTL.
    
- Early cache refresh.
    

### What is a CDN?

A Content Delivery Network distributes content across geographically distributed servers.

It reduces latency and decreases load on the origin server.

---

# 11. Message Queues and Asynchronous Processing

### What is Asynchronous Processing?

Asynchronous Processing allows a task to execute independently of the main request.

The application does not need to wait for the entire operation to finish.

### What is a Message Queue?

A Message Queue stores messages that are processed by consumers later.

Examples:

- RabbitMQ.
    
- Apache Kafka.
    
- Redis-based queues.
    
- Amazon SQS.
    

### What is a Producer?

A Producer creates and sends messages to a queue or messaging system.

### What is a Consumer?

A Consumer receives and processes messages.

### What is a Worker?

A Worker is a background process that executes queued tasks.

### What is RabbitMQ?

RabbitMQ is a message broker that supports queues, exchanges, routing, acknowledgements, and delivery mechanisms.

### What is Apache Kafka?

Kafka is a distributed event streaming platform designed for high-throughput, durable event streams.

### What is the difference between RabbitMQ and Kafka?

- RabbitMQ is commonly used for task queues and message routing.
    
- Kafka is commonly used for event streaming and high-throughput data pipelines.
    
- Kafka stores events in durable ordered partitions.
    
- RabbitMQ focuses on message delivery and routing.
    

### What is Message Acknowledgement?

Acknowledgement confirms that a consumer has successfully processed a message.

### What is Retry Mechanism?

A Retry Mechanism attempts to process a failed task again.

A common strategy is exponential backoff.

### What is Dead Letter Queue?

A Dead Letter Queue stores messages that cannot be successfully processed after the configured retry policy.

### What is At-Least-Once Delivery?

A message is delivered one or more times.

Consumers must handle possible duplicates.

### What is At-Most-Once Delivery?

A message is delivered zero or one time.

Messages can be lost, but duplicates are avoided by the delivery strategy.

### What is Exactly-Once Processing?

Exactly-once processing aims to ensure that an operation has one effective result despite retries or failures.

In distributed systems, this requires careful coordination, idempotency, or transactional mechanisms.

### What is Event-Driven Architecture?

Event-Driven Architecture allows components to communicate through events.

Example:

When an order is created:

1. The order service publishes an event.
    
2. The notification service sends an email.
    
3. The analytics service updates statistics.
    
4. The inventory service updates stock.
    

### What is the difference between Synchronous and Asynchronous Processing?

- Synchronous processing waits for an operation to finish.
    
- Asynchronous processing allows the caller to continue while the operation is handled separately.
    

---

# 12. Backend Architecture

### What is Monolithic Architecture?

A Monolithic Architecture is an application where major components are deployed as one unit.

Advantages:

- Simple deployment.
    
- Easier local development.
    
- Straightforward communication between components.
    

Disadvantages:

- Scaling individual components is difficult.
    
- Large codebases can become harder to maintain.
    
- Changes can affect the whole application.
    

### What is Microservices Architecture?

Microservices Architecture divides an application into independently deployable services.

Each service usually owns a specific business capability.

Advantages:

- Independent deployment.
    
- Independent scaling.
    
- Technology flexibility.
    
- Team autonomy.
    

Disadvantages:

- Distributed system complexity.
    
- Network failures.
    
- Data consistency challenges.
    
- More complicated monitoring and deployment.
    

### What is the difference between Monolith and Microservices?

A Monolith is deployed as one application, while Microservices consist of multiple independently deployed services.

Microservices are not automatically better. A modular monolith is often a better starting point for many projects.

### What is a Modular Monolith?

A Modular Monolith is one deployable application divided into clearly separated modules or business domains.

It combines simple deployment with better internal boundaries.

### What is MVC?

MVC (Model-View-Controller) is a design pattern that separates an application into:

- Model — data and domain logic.
    
- View — presentation.
    
- Controller — request handling and coordination.
    

### What is Layered Architecture?

Layered Architecture separates an application into layers with different responsibilities.

Common layers:

- Presentation.
    
- Application.
    
- Domain.
    
- Infrastructure.
    

### What is Clean Architecture?

Clean Architecture separates business rules from external technologies.

The core business logic should not depend directly on databases, frameworks, or external services.

### What is Hexagonal Architecture?

Hexagonal Architecture (Ports and Adapters) isolates application logic from external systems through interfaces.

Examples:

- Database adapter.
    
- HTTP adapter.
    
- Message queue adapter.
    

### What is the Repository Pattern?

Repository Pattern provides an abstraction over data access.

It separates persistence logic from business logic.

### What is the Service Layer?

A Service Layer contains application operations and coordinates domain objects and external dependencies.

### What is a DTO?

A Data Transfer Object is a simple object used to transfer structured data between application layers or services.

### What is an Entity?

An Entity is an object with a unique identity that remains important even when its attributes change.

### What is a Value Object?

A Value Object is defined by its values rather than identity.

Examples:

- Money.
    
- Email Address.
    
- Date Range.
    

### What is Domain-Driven Design?

DDD is an approach to designing software around business domains and business rules.

Important concepts:

- Entity.
    
- Value Object.
    
- Aggregate.
    
- Aggregate Root.
    
- Repository.
    
- Domain Service.
    
- Bounded Context.
    

### What is an Aggregate?

An Aggregate is a group of related domain objects treated as one consistency boundary.

The Aggregate Root controls access to the objects inside the aggregate.

### What is CQRS?

CQRS (Command Query Responsibility Segregation) separates operations that modify data from operations that read data.

### What is Event Sourcing?

Event Sourcing stores state changes as a sequence of events instead of storing only the current state.

The current state can be reconstructed by replaying those events.

### What is the difference between CQRS and Event Sourcing?

- CQRS separates read and write operations.
    
- Event Sourcing stores changes as events.
    
- They can be used together but are independent architectural concepts.
    

---

# 13. Distributed Systems

### What is a Distributed System?

A Distributed System consists of multiple independent computers or services that communicate over a network and work together.

### What is Network Latency?

Network Latency is the time required for data to travel between two endpoints.

### What is Throughput?

Throughput is the amount of work or data processed within a given time.

### What is Scalability?

Scalability is the ability of a system to handle increasing workloads.

### What is Availability?

Availability measures how often a system is operational and accessible.

### What is Fault Tolerance?

Fault Tolerance is the ability of a system to continue operating when some components fail.

### What is High Availability?

High Availability aims to minimize downtime through redundancy, monitoring, failover, and recovery mechanisms.

### What is the CAP Theorem?

The CAP Theorem states that during a network partition, a distributed data system must choose between:

- Consistency.
    
- Availability.
    

Partition Tolerance is required when network partitions are possible.

### What is Eventual Consistency?

Eventual Consistency means that distributed replicas may temporarily contain different data but are expected to converge if no new updates occur.

### What is Strong Consistency?

Strong Consistency means that after a successful write, subsequent reads observe that write according to the system's consistency guarantees.

### What is a Circuit Breaker?

A Circuit Breaker prevents repeated calls to a failing external service.

It usually has three states:

- Closed — requests are allowed.
    
- Open — requests are rejected temporarily.
    
- Half-Open — a limited number of test requests are allowed.
    

### What is a Retry Strategy?

A Retry Strategy defines when and how failed operations should be attempted again.

Exponential Backoff increases the delay between retries.

### What is a Timeout?

A Timeout limits how long an application waits for an operation to complete.

It prevents requests from waiting indefinitely.

### What is Idempotency in Distributed Systems?

Idempotency ensures that repeating an operation does not produce additional unwanted side effects.

It is especially important when messages or HTTP requests can be delivered more than once.

### What is the Saga Pattern?

The Saga Pattern manages a distributed business transaction through a sequence of local transactions.

If one step fails, compensating actions can reverse the effects of previous steps.

### What is the Outbox Pattern?

The Outbox Pattern stores business changes and outgoing events in the same database transaction.

A separate process publishes the events to a message broker.

It helps prevent losing events between database commits and message publishing.

---

# 14. Linux and Command Line

### What is Linux?

Linux is an open-source operating system widely used for servers and backend infrastructure.

### What is a Process?

A Process is an instance of a running program with its own resources and execution context.

### What is a Thread?

A Thread is an execution unit within a process.

Threads within the same process can share memory.

### What is the difference between a Process and a Thread?

- Processes have separate address spaces.
    
- Threads share memory within a process.
    
- Threads are generally lighter than processes.
    

### What is SSH?

SSH (Secure Shell) is a protocol for secure remote access and command execution.

### What is a Linux Permission?

Linux permissions define who can read, write, or execute a file.

Three permission groups:

- Owner.
    
- Group.
    
- Others.
    

### What does chmod do?

`chmod` changes file permissions.

```
chmod 755 script.sh
chmod 644 config.txt
```

### What does chown do?

`chown` changes file ownership.

```
chown user:group file.txt
```

### Important Linux Commands

|Command|Purpose|
|---|---|
|`pwd`|Show current directory|
|`ls -la`|List files|
|`cd`|Change directory|
|`mkdir`|Create directory|
|`rm`|Remove files|
|`cp`|Copy files|
|`mv`|Move or rename files|
|`cat`|Display file contents|
|`less`|Read files|
|`grep`|Search text|
|`find`|Find files|
|`tail -f`|Follow log files|
|`ps aux`|List processes|
|`top`|Monitor processes|
|`kill`|Send a signal to a process|
|`df -h`|Check disk space|
|`free -h`|Check memory|
|`curl`|Make HTTP requests|
|`wget`|Download files|
|`ss -tulpn`|Inspect listening sockets|
|`systemctl`|Manage system services|

### What is an Environment Variable?

An Environment Variable is a value provided to a process through its execution environment.

It is commonly used for configuration such as database credentials, API endpoints, and application modes.

### What is a Log File?

A Log File stores information about application or system events.

Logs help developers investigate errors and monitor system behavior.

### What is a Cron Job?

A Cron Job is a scheduled command executed automatically at configured times.

---

# 15. Docker and Containerization

### What is Docker?

Docker is a platform for building, distributing, and running applications in containers.

### What is a Container?

A Container is an isolated environment that runs an application and its dependencies.

Containers share the host operating system kernel.

### What is an Image?

A Docker Image is an immutable template used to create containers.

### What is a Dockerfile?

A Dockerfile contains instructions for building a Docker Image.

### What is Docker Compose?

Docker Compose defines and manages multiple containers through a YAML configuration file.

Example:

```
services:
  app:
    build: .
    ports:
      - "8080:80"
    depends_on:
      - database

  database:
    image: mysql:8.4
    environment:
      MYSQL_ROOT_PASSWORD: secret
```

### What is the difference between an Image and a Container?

- An Image is a template.
    
- A Container is a running or stopped instance created from an image.
    

### What is a Docker Volume?

A Volume provides persistent storage for container data.

It allows data to survive container replacement.

### What is Docker Networking?

Docker Networking allows containers to communicate with each other and with external systems.

### What is a Multi-Stage Build?

A Multi-Stage Build uses multiple build stages to produce smaller and cleaner production images.

### What is Container Orchestration?

Container Orchestration automates container deployment, scaling, networking, and recovery.

Examples:

- Kubernetes.
    
- Docker Swarm.
    

### What is the difference between Docker and a Virtual Machine?

- Containers share the host OS kernel.
    
- Virtual Machines run their own guest operating systems.
    
- Containers generally require fewer resources and start faster.
    

---

# 16. CI/CD and DevOps

### What is DevOps?

DevOps is a set of practices that combines development and operations to improve collaboration, automation, and software delivery.

### What is CI?

Continuous Integration automatically builds and tests code changes when developers push or merge code.

### What is CD?

Continuous Delivery ensures that software can be released reliably.

Continuous Deployment automatically releases changes to production after successful checks.

### What is a CI/CD Pipeline?

A CI/CD Pipeline is an automated sequence of steps used to build, test, validate, and deploy an application.

Example:

1. Developer pushes code.
    
2. Pipeline installs dependencies.
    
3. Static analysis runs.
    
4. Automated tests run.
    
5. Application is built.
    
6. Deployment is performed.
    

### What is Git?

Git is a distributed version control system used to track changes in source code.

### What is a Branch?

A Branch is an independent line of development.

### What is a Merge?

A Merge combines changes from different branches.

### What is a Pull Request?

A Pull Request is a proposal to merge code changes into another branch.

It allows code review and automated checks before integration.

### What is a Merge Conflict?

A Merge Conflict happens when Git cannot automatically combine changes.

Developers must resolve the conflicting code manually.

### What is Git Rebase?

Rebase moves or reapplies commits onto another base commit.

It can create a cleaner linear history.

### What is the difference between Merge and Rebase?

- Merge combines branches and usually creates a merge commit.
    
- Rebase replays commits onto a new base and rewrites commit history.
    

Avoid rebasing shared branches unless the team explicitly allows it.

### What is Semantic Versioning?

Semantic Versioning uses three numbers:

`MAJOR.MINOR.PATCH`

- MAJOR — breaking changes.
    
- MINOR — backward-compatible features.
    
- PATCH — backward-compatible bug fixes.
    

### What is Rollback?

Rollback restores a previous working version after a failed deployment or release.

### What is Infrastructure as Code?

Infrastructure as Code defines infrastructure using version-controlled configuration files.

Examples:

- Terraform.
    
- Ansible.
    

---

# 17. Software Testing and Quality

### What is Software Testing?

Software Testing verifies that an application behaves as expected and helps detect defects.

### What is Unit Testing?

Unit Testing checks an isolated function, method, or component.

### What is Integration Testing?

Integration Testing checks whether multiple components work correctly together.

### What is Functional Testing?

Functional Testing verifies application behavior against functional requirements.

### What is End-to-End Testing?

E2E Testing verifies complete user workflows across the application.

### What is Regression Testing?

Regression Testing checks that new changes have not broken existing functionality.

### What is Smoke Testing?

Smoke Testing performs basic checks to verify that the most important application functionality works after deployment.

### What is Load Testing?

Load Testing checks application behavior under expected traffic.

### What is Stress Testing?

Stress Testing evaluates application behavior beyond normal capacity to identify breaking points.

### What is a Mock?

A Mock is a test double used to simulate dependencies and verify interactions.

### What is a Stub?

A Stub provides predefined responses to method calls.

### What is Static Analysis?

Static Analysis examines source code without executing it.

Examples:

- PHPStan.
    
- Psalm.
    

### What is Code Coverage?

Code Coverage measures which parts of the code are executed during tests.

High coverage does not guarantee that the tests are meaningful.

### What is TDD?

Test-Driven Development follows three steps:

1. Write a failing test.
    
2. Implement code to pass the test.
    
3. Refactor the implementation.
    

### What is BDD?

Behavior-Driven Development describes application behavior using examples understandable to both technical and non-technical stakeholders.

---

# 18. Software Design Principles

### What is SOLID?

SOLID is a set of five object-oriented design principles.

**S — Single Responsibility Principle**

A class should have one main responsibility and one reason to change.

**O — Open/Closed Principle**

Software entities should be open for extension but closed for modification.

**L — Liskov Substitution Principle**

A child class should be usable wherever its parent type is expected without breaking behavior.

**I — Interface Segregation Principle**

Prefer small, specific interfaces instead of large interfaces.

**D — Dependency Inversion Principle**

High-level modules should depend on abstractions rather than concrete implementations.

### What is DRY?

DRY (Don't Repeat Yourself) means avoiding unnecessary duplication of knowledge or logic.

### What is KISS?

KISS (Keep It Simple, Stupid) means preferring simple solutions over unnecessary complexity.

### What is YAGNI?

YAGNI (You Aren't Gonna Need It) means avoiding implementation of features that are not currently needed.

### What is Separation of Concerns?

It means dividing application responsibilities into independent components.

### What is Loose Coupling?

Loose Coupling means components have minimal dependencies on each other.

### What is High Cohesion?

High Cohesion means a component has closely related responsibilities.

Good software architecture generally aims for loose coupling and high cohesion.

### What is Technical Debt?

Technical Debt is the future cost of choosing a quick or less maintainable implementation.

---

# 19. System Design Fundamentals

### What is System Design?

System Design is the process of defining a system's architecture, components, communication, data storage, and infrastructure.

### What is Scalability?

Scalability is the ability of a system to handle increasing traffic or workload.

### What is Vertical Scaling?

Vertical Scaling means increasing the resources of an existing server.

Example: adding more CPU and RAM.

### What is Horizontal Scaling?

Horizontal Scaling means adding more servers or application instances.

### What is the difference between Vertical and Horizontal Scaling?

- Vertical scaling increases the capacity of one machine.
    
- Horizontal scaling distributes workload across multiple machines.
    

### What is High Availability?

High Availability means designing a system to minimize downtime.

Common techniques:

- Redundant servers.
    
- Load balancing.
    
- Health checks.
    
- Automatic failover.
    

### What is a Single Point of Failure?

A Single Point of Failure is a component whose failure can cause the entire system or a critical part of it to become unavailable.

### What is a CDN?

A CDN distributes content across geographically distributed servers to reduce latency and origin server load.

### What is a Load Balancer?

A Load Balancer distributes traffic across multiple backend instances.

### What is Observability?

Observability is the ability to understand a system's internal state through its external outputs.

Three main pillars:

- Logs.
    
- Metrics.
    
- Traces.
    

### What is Monitoring?

Monitoring continuously tracks system health and performance using metrics, alerts, and dashboards.

### What is Distributed Tracing?

Distributed Tracing follows a request across multiple services to identify latency and failures.

### What is an SLA?

A Service Level Agreement is a formal agreement defining expected service levels.

### What is an SLO?

A Service Level Objective is a measurable reliability target.

### What is an SLI?

A Service Level Indicator is a metric used to measure service performance.

Example:

- SLI — percentage of successful HTTP requests.
    
- SLO — 99.9% successful requests.
    
- SLA — contractual availability commitment.
    

### What is a Bottleneck?

A Bottleneck is a component that limits the performance of the entire system.

Examples:

- Slow database queries.
    
- Insufficient CPU.
    
- Network latency.
    
- Limited worker processes.
    

---

# 20. Important Backend Tools and Technologies

|Technology|Purpose|
|---|---|
|PHP|Backend programming language|
|MySQL|Relational database|
|PostgreSQL|Relational database|
|MariaDB|Relational database|
|MongoDB|Document-oriented database|
|Redis|In-memory data store and cache|
|Memcached|Distributed caching|
|Nginx|Web server and reverse proxy|
|Apache|Web server|
|PHP-FPM|PHP process manager|
|Composer|PHP dependency manager|
|Docker|Containerization|
|Kubernetes|Container orchestration|
|RabbitMQ|Message broker|
|Apache Kafka|Event streaming platform|
|Git|Version control|
|GitHub Actions|CI/CD automation|
|PHPUnit|PHP testing framework|
|PHPStan|Static analysis|
|Postman|API testing|
|cURL|HTTP client|
|Elasticsearch|Search and analytics engine|
|Prometheus|Metrics monitoring|
|Grafana|Monitoring and visualization|
|Sentry|Error tracking|
|Terraform|Infrastructure as Code|

---

# 21. Common Backend Interview Questions

### What happens when you enter a URL in a browser?

1. The browser checks its cache.
    
2. DNS resolves the domain name.
    
3. The browser establishes a TCP connection.
    
4. TLS is negotiated if HTTPS is used.
    
5. The browser sends an HTTP request.
    
6. The web server receives the request.
    
7. The backend processes it.
    
8. The application communicates with the database if necessary.
    
9. The server returns an HTTP response.
    
10. The browser processes and displays the result.
    

### How does a browser communicate with a backend?

The browser sends HTTP requests to backend endpoints. The backend processes requests and returns responses, commonly in JSON or HTML.

### How do you design a secure REST API?

I use authentication, authorization, input validation, parameterized database queries, HTTPS, rate limiting, and appropriate error handling.

### How do you improve database performance?

I analyze slow queries, use appropriate indexes, avoid N+1 queries, optimize JOINs, and use caching when necessary.

### How do you handle large amounts of data?

I use pagination, database-level filtering, batch processing, generators, and asynchronous workers.

### How do you handle long-running operations?

I move expensive operations into background jobs using message queues and workers.

### How do you prevent duplicate operations?

I use idempotency keys, database constraints, transactions, and proper state management.

### How do you handle external API failures?

I configure timeouts, retries with exponential backoff, logging, and circuit breakers where appropriate.

### How do you prevent race conditions?

I use database transactions, locking, atomic operations, and constraints.

### How do you scale a backend application?

I optimize the application first, introduce caching, use load balancing, add application instances, and scale database infrastructure when necessary.

### How do you handle sensitive information?

I use secure secret management, encrypted communication, password hashing, access restrictions, and avoid logging credentials or sensitive data.

### How do you investigate a production issue?

1. Reproduce the problem if possible.
    
2. Check logs and monitoring.
    
3. Inspect metrics and traces.
    
4. Identify the affected component.
    
5. Find the root cause.
    
6. Implement a safe fix.
    
7. Add regression tests.
    
8. Monitor the application after deployment.
    

### How do you choose between SQL and NoSQL?

I choose SQL when relational data, transactions, and complex queries are important.

I consider NoSQL when the data model and access patterns benefit from document storage, key-value access, or distributed data structures.

The decision depends on requirements rather than popularity.

### How do you choose between Monolith and Microservices?

I usually start with a modular monolith because it is simpler to develop and deploy.

I consider microservices when independent scaling, deployment, or clear business boundaries justify the additional distributed-system complexity.

---

# 22. Key Differences — Quick Revision

|Concept A|Concept B|
|---|---|
|HTTP — application protocol|HTTPS — HTTP over TLS|
|TCP — reliable transport|UDP — connectionless transport|
|DNS — resolves domain names|DHCP — assigns network configuration|
|GET — retrieve|POST — submit or create|
|PUT — replace|PATCH — partial update|
|401 — authentication required/failed|403 — access forbidden|
|502 — invalid upstream response|504 — upstream timeout|
|Cookie — client-side data|Session — server-side state|
|Authentication — identity|Authorization — permissions|
|SQL — relational query language|NoSQL — non-relational database systems|
|Primary Key — unique identifier|Foreign Key — relationship|
|WHERE — filter rows|HAVING — filter groups|
|DELETE — remove rows|DROP — remove table|
|Index — faster reads|Constraint — enforce data rules|
|COMMIT — save transaction|ROLLBACK — cancel transaction|
|Optimistic Locking — check for conflicts|Pessimistic Locking — lock resources|
|Replication — copy data|Sharding — split data|
|Cache — temporary data storage|Database — persistent data storage|
|Redis — in-memory data store|MySQL — relational database|
|RabbitMQ — message broker|Kafka — event streaming|
|Polling — periodically request updates|Webhook — receive event notification|
|Monolith — one deployable application|Microservices — independent services|
|Synchronous — wait for result|Asynchronous — process independently|
|Vertical Scaling — stronger machine|Horizontal Scaling — more machines|
|Unit Test — isolated functionality|Integration Test — component interaction|
|CI — automatic integration and testing|CD — automated delivery/deployment|
|Docker Image — application template|Docker Container — running instance|
|Process — independent execution context|Thread — execution unit within a process|
|Hashing — one-way transformation|Encryption — reversible transformation|
|Strong Consistency — immediate consistency guarantee|Eventual Consistency — convergence over time|
|CQRS — separate reads and writes|Event Sourcing — store state changes as events|

---

# 23. Questions You Should Be Able to Answer Without Thinking

1. What is Backend Development?
    
2. What is Client-Server Architecture?
    
3. What happens when you enter a URL in a browser?
    
4. What is HTTP?
    
5. What is HTTPS?
    
6. What is the difference between GET and POST?
    
7. What is the difference between PUT and PATCH?
    
8. What are HTTP status codes?
    
9. What is the difference between 401 and 403?
    
10. What are Cookies and Sessions?
    
11. What is DNS?
    
12. What is TCP?
    
13. What is the difference between TCP and UDP?
    
14. What is a Reverse Proxy?
    
15. What is Load Balancing?
    
16. What is SQL?
    
17. What is a Primary Key?
    
18. What is a Foreign Key?
    
19. What is an Index?
    
20. What is Database Normalization?
    
21. What are JOINs?
    
22. What is the difference between WHERE and HAVING?
    
23. What are ACID properties?
    
24. What is a Database Transaction?
    
25. What is a Deadlock?
    
26. What is a Race Condition?
    
27. What is Optimistic Locking?
    
28. What is the N+1 Query Problem?
    
29. What is Database Replication?
    
30. What is Database Sharding?
    
31. What is a REST API?
    
32. What is JSON?
    
33. What is API Versioning?
    
34. What is Idempotency?
    
35. What is JWT?
    
36. What is OAuth 2.0?
    
37. What is the difference between Authentication and Authorization?
    
38. What is SQL Injection?
    
39. What is XSS?
    
40. What is CSRF?
    
41. What is CORS?
    
42. What is Redis?
    
43. What is Cache Invalidation?
    
44. What is a Message Queue?
    
45. What is the difference between RabbitMQ and Kafka?
    
46. What is Event-Driven Architecture?
    
47. What is Monolithic Architecture?
    
48. What is Microservices Architecture?
    
49. What is CQRS?
    
50. What is Event Sourcing?
    
51. What is Docker?
    
52. What is the difference between an Image and a Container?
    
53. What is CI/CD?
    
54. What is the difference between Unit and Integration Testing?
    
55. What is Horizontal Scaling?
    
56. What is High Availability?
    
57. What is Fault Tolerance?
    
58. What is Observability?
    
59. What is the CAP Theorem?
    
60. How do you investigate a production issue?
    

---

# Final Interview Advice

When answering backend interview questions, use this structure:

1. **Definition** — explain what the concept is.
    
2. **Purpose** — explain why it is needed.
    
3. **Example** — give a practical example.
    
4. **Trade-off** — for Middle/Senior questions, explain the advantages and disadvantages.
    

### Example answers

**Question: What is the difference between SQL and NoSQL?**

"SQL databases store structured data in relational tables and support relationships and transactions. NoSQL databases use different data models, such as documents or key-value pairs. I choose the database based on data structure, consistency requirements, and access patterns."

**Question: What is a Database Transaction?**

"A transaction is a group of database operations treated as one logical unit. It follows ACID principles to ensure data consistency and reliability. For example, transferring money between two accounts should debit one account and credit another within the same transaction."

**Question: What is Redis?**

"Redis is an in-memory data store commonly used for caching, session storage, queues, and fast data operations. It reduces database load and improves application response times."

**Question: What is the difference between Monolith and Microservices?**

"A monolithic application is deployed as one unit, while microservices consist of independently deployable services. Microservices provide independent scaling and deployment but introduce network communication and distributed-system complexity. I usually prefer a modular monolith until there is a clear reason to split it."

**Question: What is a Race Condition?**

"A race condition occurs when the result of an operation depends on the timing of concurrent requests. For example, two users may try to purchase the last available product simultaneously. I prevent this using transactions, locking, and atomic database operations."

**Question: What is the difference between Authentication and Authorization?**

"Authentication verifies the identity of a user, while authorization determines what the user is allowed to access. For example, logging in authenticates a user, while checking whether they can delete an article is authorization."

**Question: What is CI/CD?**

"Continuous Integration automatically builds and tests code changes. Continuous Delivery ensures that changes are ready for release, while Continuous Deployment automatically releases validated changes to production."

**Question: How do you improve backend performance?**

"First, I identify bottlenecks using profiling and monitoring. Then I optimize database queries, introduce caching, reduce unnecessary processing, and move long-running operations to background workers. I measure the results to verify the improvements."

**The main goal:** understand how a complete backend application works, from the HTTP request to the database and back, and be able to explain the technical decisions behind its architecture.