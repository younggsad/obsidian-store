---
custom-width: 80
---
## Оглавление

1. [[#1. Архитектура Laravel и Жизненный цикл запроса (Request Lifecycle)]]
2. [[#2. Service Container, Dependency Injection и Service Providers]]
3. [[#3. Facades (Фасады) и Contracts (Контракты)]]
4. [[#4. Routing, Middleware и Контроллеры]]
5. [[#5. Валидация (FormRequest) и API Resources]]
6. [[#6. Eloquent ORM: Active Record, Связи, Scopes и Аксессоры]]
7. [[#7. Оптимизация Eloquent: Проблема N+1, Chunking и Транзакции]]
8. [[#8. Аутентификация (Sanctum/Passport) и Авторизация (Gates & Policies)]]
9. [[#9. Очереди (Queues), Jobs, Events и Планировщик (Scheduler)]]
10. [[#10. Кэширование, Логирование и Обработка ошибок]]
11. [[#11. Laravel Octane, Swoole/RoadRunner и производительность]]
12. [[#12. Тестирование в Laravel и Шпаргалка перед собеседованием]]

---

## 1. Архитектура Laravel и Жизненный цикл запроса (Request Lifecycle)

> [!abstract] Коротко
> Все запросы входят через единую точку `public/index.php`, поднимают приложение через `bootstrap/app.php`, регистрируют и загружают **Service Providers**, проходят конвейер **Middleware**, попадают в **Router → Controller**, формируют `Response` и отправляют его клиенту.

### Теория (🇷🇺 Русский)

1. **Единая точка входа (`public/index.php`)**: веб-сервер (Nginx) перенаправляет все запросы (кроме статических файлов) на `public/index.php`. Здесь подключается автозагрузчик Composer (`vendor/autoload.php`).
2. **Создание экземпляра приложения (`bootstrap/app.php`)**: инициализируется `Illuminate\Foundation\Application` (наследник Service Container), настраиваются роуты, middleware и обработчик исключений (в Laravel 11/12 это делается через fluent-методы `Application::configure()`).
3. **HTTP Kernel (`Illuminate\Foundation\Http\Kernel`)**:
   - Запускает массив системных `bootstrappers`: загрузка переменных окружения (`.env`), чтение конфигурации (`config/*`), регистрация обработчика ошибок, регистрация фасадов и провайдеров.
4. **Service Providers (Сервис-провайдеры)** — сердце загрузки фреймворка:
   - *Фаза 1 (`register()`)*: Laravel проходит по всем провайдерам и вызывает только `register()`. На этом этапе разрешено **только связывать классы в контейнере**. Обращаться к другим сервисам запрещено, так как они ещё могут быть не зарегистрированы.
   - *Фаза 2 (`boot()`)*: когда все провайдеры зарегистрированы, вызывается `boot()`. Здесь уже можно безопасно использовать любые зависимости, регистрировать слушателей событий, роуты, директивы Blade и правила валидации.
5. **Routing & Middleware Pipeline**: запрос проходит через глобальные middleware, затем роутер находит совпадение по URI и HTTP-методу, прогоняет запрос через route-middleware (`auth`, `throttle`, `can`) и вызывает действие контроллера (или замыкание).
6. **Формирование и отправка `Response`**: контроллер возвращает объект `Response` (или данные, которые Laravel сам преобразует в JSON/HTML). Ответ проходит обратно через слои middleware (например, шифрование и добавление cookies) и отправляется клиенту методом `$response->send()`.

### Theory (🇬🇧 English)

1. **Entry Point (`public/index.php`)**: Web server routes all requests here; loads Composer's autoloader (`vendor/autoload.php`).
2. **Application Bootstrap (`bootstrap/app.php`)**: Instantiates the `Application` (Service Container) and configures routing, middleware, and exception handling.
3. **HTTP Kernel**: Runs core bootstrappers (environment variables, configuration loading, exception handling, Facades, and Service Providers).
4. **Service Providers**:
   - `register()` phase: Strictly binds interfaces/classes into the Service Container. Do not resolve other services here.
   - `boot()` phase: Executed after all providers are registered; safe to resolve dependencies, register event listeners, routes, and macros.
5. **Middleware Pipeline & Routing**: Request traverses global middleware, matches a route, executes route-specific middleware, and hits the controller.
6. **Response Lifecycle**: Controller returns a `Response`, which travels back out through the middleware stack before being sent to the client.

### Схема жизненного цикла

```mermaid
flowchart LR
    A[public/index.php] --> B[bootstrap/app.php]
    B --> C[HTTP Kernel]
    C --> D[Providers: register]
    D --> E[Providers: boot]
    E --> F[Global & Route Middleware]
    F --> G[Router -> Controller]
    G --> H[Middleware Outbound]
    H --> I[Response -> Client]
````

## 2. Service Container, Dependency Injection и Service Providers

> [!abstract] Коротко
> 
> **Service Container** — мощный IoC-контейнер (Inversion of Control), который автоматически разрешает зависимости классов через PHP Reflection API. **Service Providers** — центральное место настройки этих связей.

### Теория (🇷🇺 Русский)

- **Как работает авторазрешение (Auto-wiring)**: когда Laravel нужно создать контроллер или сервис, он через `ReflectionClass` смотрит на типы аргументов конструктора (`__construct`). Если аргумент — конкретный класс, контейнер рекурсивно создаёт его и все его зависимости. Если аргумент — интерфейс, контейнер ищет привязанную реализацию.
    
- **Типы связывания (Binding) в контейнере**:
    
    - `$this->app->bind(PaymentInterface::class, StripePayment::class)`: при каждом запросе зависимости создаётся **новый** экземпляр класса.
        
    - `$this->app->singleton(PaymentInterface::class, StripePayment::class)`: создаётся **один** экземпляр при первом обращении, а при всех последующих обращениях в рамках этого HTTP-запроса возвращается тот же самый объект.
        
    - `$this->app->scoped(TenantContext::class, ...)`: работает как `singleton` в течение одного запроса, но автоматически сбрасывается между запросами в **Laravel Octane** или между задачами в **Queue Worker**.
        
    - `$this->app->instance(LoggerInterface::class, $customLogger)`: привязка уже готового, созданного вручную объекта.
        
- **Контекстное связывание (Contextual Binding)**: позволяет подставлять разные реализации одного интерфейса в разные классы:
    
    
    ```PHP
    $this->app->when(OrderController::class)
        ->needs(Filesystem::class)
        ->give(fn () => Storage::disk('s3'));
    ```
    
- **Deferred Providers (Отложенные провайдеры)**: реализуют интерфейс `DeferrableProvider` и метод `provides()`. Такие провайдеры не загружаются на каждый HTTP-запрос, а инициализируются только тогда, когда код реально запросит из контейнера один из указанных в `provides()` сервисов.
    

### Theory (🇬🇧 English)

- **Service Container (IoC)**: Manages class dependencies and performs Dependency Injection (DI) automatically via PHP's Reflection API.
    
- **Binding Lifecycles**:
    
    - `bind()`: Resolves a fresh instance on every injection.
        
    - `singleton()`: Resolves a single shared instance throughout the request lifecycle.
        
    - `scoped()`: Like `singleton`, but automatically flushed between requests in **Laravel Octane** or jobs in **Queue Workers** to prevent state pollution.
        
    - `instance()`: Binds an existing object instance into the container.
        
- **Deferred Providers (`DeferrableProvider`)**: Lazy-loaded only when one of the services declared in `provides()` is actually requested from the container.
    

## 3. Facades (Фасады) и Contracts (Контракты)

> [!abstract] Коротко
> 
> **Facades** дают лаконичный статический синтаксис (`Cache::get()`), но под капотом резолвят реальный объект из Service Container. **Contracts** — это интерфейсы ядра, обеспечивающие слабую связанность.

### Теория (🇷🇺 Русский & 🇬🇧 English)

- **Как устроен Фасад под капотом**:
    
    1. Все фасады наследуются от `Illuminate\Support\Facades\Facade` и переопределяют один защищённый метод `getFacadeAccessor()`, возвращающий строковый ключ или имя класса в контейнере (например, `'cache'`).
        
    2. Когда вы вызываете `Cache::get('key')`, срабатывает магический метод `__callStatic()` базового класса `Facade`.
        
    3. Он достаёт реальный объект из Service Container (`static::$app['cache']`) и вызывает на нём обычный динамический метод `->get('key')`.
        
- **Плюсы и минусы Фасадов**:
    
    - _Плюсы_: лаконичный код, идеальная поддержка тестирования из коробки (`Cache::shouldReceive('get')`, `Event::fake()`, `Queue::fake()`, `Http::fake()`).
        
    - _Минусы_: скрывают реальные зависимости класса (не видны в конструкторе), из-за чего класс сложнее переносить вне Laravel и можно незаметно нарушить принцип единой ответственности (SRP), напихав 10 разных фасадов в один сервис.
        
- **Contracts (Контракты — `Illuminate\Contracts\...`)**: набор PHP-интерфейсов, определяющих сервисы фреймворка (`Cache\Repository`, `Queue\Queue`, `Mail\Mailer`). Внедрение контракта в конструктор сервиса делает код независимым от реализации и полностью соответствует принципу **Dependency Inversion (SOLID)**.
    

## 4. Routing, Middleware и Контроллеры

> [!abstract] Коротко
> 
> Роутер сопоставляет HTTP-запрос с контроллером, поддерживает неявное связывание моделей (Route Model Binding) и кэширование маршрутов, а Middleware оборачивают запрос по паттерну «Луковица» (Decorator / Chain of Responsibility).

### Теория (🇷🇺 Русский)

- **Route Model Binding (Связывание моделей с маршрутом)**:
    
    - _Implicit (Неявное)_: `Route::get('/posts/{post:slug}', ...)` — Laravel автоматически выполнит `Post::where('slug', $value)->firstOrFail()` и либо передаст модель в контроллер, либо выбросит `404 Not Found`.
        
    - _Explicit (Явное)_: настраивается через `Route::model()` или `Route::bind()` для сложной логики поиска.
        
- **Кэширование роутов (`php artisan route:cache`)**: компилирует все маршруты в один оптимизированный PHP-массив. Обязательно использовать на продакшене! _Важно_: не работает, если в файлах роутов используются замыкания (Closures) вместо контроллеров.
    
- **Middleware (Посредники)**: реализация паттерна **Chain of Responsibility / Decorator** («луковица»).
    
    - Код **до** `$next($request)` выполняется при входе запроса (проверка токена, локали, rate limit).
        
    - Код **после** `$response = $next($request)` выполняется при возврате ответа (добавление заголовков CORS, логирование времени ответа).
        
    - **Terminable Middleware**: если у middleware есть метод `terminate($request, $response)`, Laravel вызовет его **уже после того**, как HTTP-ответ физически отправлен клиенту (при использовании PHP-FPM вызывается `fastcgi_finish_request()`), что позволяет писать аналитику без задержки для пользователя.
        

### Код

```PHP
<?php
declare(strict_types=1);

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

final class MeasureRequestTimeMiddleware
{
    public function handle(Request $request, Closure$next): Response
    {
        $start = microtime(true);

        // Передаём запрос дальше по конвейеру к контроллеру
        $response = $next($request);

        // Модифицируем ответ на обратном пути
        $duration = round((microtime(true) -$start) * 1000, 2);
        $response->headers->set('X-Response-Time-Ms', (string)$duration);

        return $response;
    }
}
```

## 5. Валидация (FormRequest) и API Resources

> [!abstract] Коротко
> 
> Никогда не валидируйте данные в контроллере — выносите правила в `FormRequest`. Никогда не возвращайте Eloquent-модели напрямую в JSON — оборачивайте их в `JsonResource` (слой трансформации).

### Теория (🇷🇺 Русский & 🇬🇧 English)

- **FormRequest**: специальный класс запроса, который автоматически запускается **до** вызова метода контроллера:
    
    1. `authorize(): bool` — проверяет, имеет ли текущий пользователь право выполнить этот запрос (если `false`, вернёт `403 Forbidden`).
        
    2. `rules(): array` — массив правил валидации (если не пройдено, для API вернёт `422 Unprocessable Entity` с JSON-ошибками, для Web — редирект назад с ошибками в сессии).
        
    3. В контроллере всегда используйте `$request->validated()` вместо `$request->all()`, чтобы получить только проверенные поля и защититься от Mass Assignment.
        
- **API Resources (`JsonResource` / `ResourceCollection`)**: слой сериализации между Eloquent-моделями и JSON-ответом API.
    
    - Позволяет скрывать внутренние поля БД, форматировать даты, переименовывать ключи и безопасно включать связи только тогда, когда они были загружены (`$this->whenLoaded('author')`), предотвращая случайный запуск N+1 при сериализации.
        

### Код

```PHP
// FormRequest
final class StoreOrderRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user()->can('create', Order::class);
    }

    public function rules(): array
    {
        return [
            'items'            => ['required', 'array', 'min:1'],
            'items.*.id'       => ['required', 'integer', 'exists:products,id'],
            'items.*.quantity' => ['required', 'integer', 'min:1', 'max:100'],
            'promo_code'       => ['nullable', 'string', 'max:32'],
        ];
    }
}

// API Resource
final class OrderResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id'         => $this->id,
            'status'     => $this->status->value,
            'total'      => $this->total_cents / 100, 'customer'   => new UserResource($this->whenLoaded('user')),
            'created_at' => $this->created_at->toIso8601String(),
        ];
    }
}
```

## 6. Eloquent ORM: Active Record, Связи, Scopes и Аксессоры

> [!abstract] Коротко
> 
> Eloquent реализует паттерн **Active Record**: класс модели одновременно описывает структуру данных, связи и умеет сам сохранять/удалять себя в БД.

### Теория (🇷🇺 Русский)

- **Active Record vs Data Mapper**:
    
    - _Active Record (Eloquent)_: модель привязана к таблице БД, хранит состояние и методы сохранения (`$user->save()`). Очень быстрая разработка, но доменная логика смешивается с инфраструктурой БД.
        
    - _Data Mapper (Doctrine)_: сущности — чистые PHP-объекты (POPO), ничего не знающие о БД; за сохранение отвечает отдельный `EntityManager`. Лучше для сложного DDD, но много шаблонного кода.
        
- **Виды связей (Relationships)**:
    
    - `hasOne` / `belongsTo` (один к одному; `belongsTo` всегда ставится на таблицу, в которой лежит `foreign_key`).
        
    - `hasMany` / `belongsTo` (один ко многим).
        
    - `belongsToMany` (многие ко многим через промежуточную pivot-таблицу; дополнительные поля pivot-таблицы достаются через `->withPivot(...)`).
        
    - `hasOneThrough` / `hasManyThrough` (связь через промежуточную модель: Страна → Пользователи → Посты).
        
    - **Полиморфные связи (`morphOne`, `morphMany`, `morphTo`, `morphToMany`)**: позволяют одной модели (например, `Comment` или `Image`) принадлежать сразу нескольким разным моделям (`Post`, `Video`) через пару колонок `commentable_id` и `commentable_type`. _Минус_: на уровне БД нельзя повесить строгий `FOREIGN KEY`.
        
- **Scopes (Заготовки запросов)**:
    
    - _Local Scopes_: методы модели с префиксом `scopeActive(Builder $query)` (или атрибутом `#[Scope]` в Laravel 12), позволяющие писать переиспользуемые цепочки `User::active()->verified()->get()`.
        
    - _Global Scopes_: автоматически добавляют условия ко **всем** запросам модели (так под капотом работает `SoftDeletes`, добавляя `WHERE deleted_at IS NULL`).
        
- **Casts, Accessors и Mutators**:
    
    - `$casts`: автоматическое приведение типов колонок при чтении/записи (`'settings' => 'array'`, `'status' => OrderStatus::class`, `'password' => 'hashed'`, `'secret' => 'encrypted'`).
        
    - `Attribute::make(get: ..., set: ...)`: современный синтаксис аксессоров и мутаторов для вычисляемых или форматируемых атрибутов.
        
- **Mass Assignment (Массовое присваивание)**: защита от подмены нежелательных полей (например, `is_admin = 1` из запроса) через белый список `$fillable` или чёрный список `$guarded`.
    

### Theory (🇬🇧 English)

- **Active Record Pattern**: Eloquent models encapsulate both domain data and persistence logic (`save()`, `delete()`), contrasting with Data Mapper (Doctrine) which isolates domain entities from database persistence.
    
- **Polymorphic Relations**: Allow a model (e.g., `Comment`) to belong to multiple model types (`Post`, `Video`) using `{name}_id` and `{name}_type` columns (note: cannot enforce DB-level foreign key constraints).
    
- **Scopes & Casts**: Global scopes automatically constrain all queries (e.g., `SoftDeletes`); `$casts` convert raw DB values into PHP enums, arrays, DTOs, or encrypted strings.
    

## 7. Оптимизация Eloquent: Проблема N+1, Chunking и Транзакции

> [!abstract] Коротко
> 
> Главные враги производительности в Laravel — ленивая загрузка связей в цикле (**N+1**), выгрузка тысяч моделей в RAM через `->get()` и нехватка транзакций/блокировок при изменении балансов.

### Теория (🇷🇺 Русский & 🇬🇧 English)

- **Проблема N+1 и Eager Loading**:
    
    - _Проблема_: `Post::all()` делает 1 запрос, а обращение `$post->author->name` в цикле `foreach` делает ещё N запросов.
        
    - _Решение_: `Post::with(['author:id,name', 'tags'])->get()` — жадная загрузка делает ровно 2 запроса (`SELECT * FROM posts` и `SELECT id, name FROM users WHERE id IN (...)`).
        
    - _Защита от дураков_: в `AppServiceProvider::boot()` включить `Model::preventLazyLoading(! $this->app->isProduction());` — на этапе разработки любая попытка ленивой загрузки выбросит исключение `LazyLoadingViolationException`.
        
- **Обработка больших объёмов данных без переполнения памяти**:
    
    - `chunk(1000, fn ($users) => ...)`: загружает по 1000 записей через `LIMIT / OFFSET`. _Ловушка_: если внутри цикла вы обновляете или удаляете записи, попадающие в условие выборки, `OFFSET` сдвинется и часть записей будет пропущена!
        
    - `chunkById(1000, fn ($users) => ...)`: использует курсорную пагинацию `WHERE id > $lastId ORDER BY id ASC LIMIT 1000`. Работает намного быстрее и **не пропускает записи** при изменении данных.
        
    - `lazy()` / `lazyById()`: возвращает `LazyCollection` на базе PHP-генераторов (удобно для красивых цепочек `->filter()->each()` с расходом памяти $O(1)$).
        
    - `cursor()`: использует один курсор PDO и гидрирует ровно по 1 модели за раз (минимальный расход RAM, но нельзя использовать eager loading `with()`).
        
- **Транзакции и пессимистические блокировки**:
    
    
    ```PHP
    DB::transaction(function () use ($fromId, $toId,$amount) {
        // lockForUpdate() генерирует SELECT ... FOR UPDATE (пессимистическая блокировка)
        $sender = Wallet::where('id',$fromId)->lockForUpdate()->firstOrFail();
        $sender->decrement('balance',$amount);
    
        Wallet::where('id', $toId)->increment('balance',$amount);
    }, attempts: 3); // автоматически повторит транзакцию до 3 раз при Deadlock!
    ```
    

## 8. Аутентификация (Sanctum/Passport) и Авторизация (Gates & Policies)

> [!abstract] Коротко
> 
> **Аутентификация** проверяет личность пользователя (Sanctum для SPA/Mobile, Passport для полноценного OAuth2). **Авторизация** проверяет права доступа к действию или конкретной модели (Gates и Policies).

### Теория (🇷🇺 Русский & 🇬🇧 English)

- **Laravel Sanctum vs Laravel Passport**:
    
    - **Sanctum**: лёгкий пакет для двух задач:
        
        1. _SPA Authentication_: использует стандартные stateful-сессии и `HttpOnly` cookies (с защитой через CSRF-токен `/sanctum/csrf-cookie`). Никаких JWT в `localStorage`!
            
        2. _API Tokens_: выдаёт простые случайные токены (хэш SHA-256 хранится в таблице `personal_access_tokens`), идеально для мобильных приложений и внешних интеграций.
            
    - **Passport**: полная реализация сервера **OAuth 2.0** (Authorization Code, Client Credentials, JWT-токены). Нужен только тогда, когда вы строите свой аналог «Войти через Google/GitHub» для сторонних разработчиков.
        
- **Gates vs Policies (Авторизация)**:
    
    - **Gates (Шлюзы)**: замыкания для общих проверок, не привязанных к конкретной Eloquent-модели (например, `Gate::define('access-admin-panel', fn (User $user) => $user->isAdmin())`).
        
    - **Policies (Политики)**: классы, группирующие правила доступа вокруг конкретной модели (`PostPolicy` с методами `view`, `create`, `update`, `delete`). Вызываются через `$this->authorize('update', $post)` или `$user->can('update', $post)`.
        

## 9. Очереди (Queues), Jobs, Events и Планировщик (Scheduler)

> [!abstract] Коротко
> 
> Всё медленное (отправка писем, генерация отчётов, вебхуки, парсинг) выносится из HTTP-запроса в **Queues**, а связь между модулями строится через **Events & Listeners**.

### Теория (🇷🇺 Русский & 🇬🇧 English)

- **Архитектура очередей**:
    
    - Продюсер отправляет задачу в брокер (`Redis`, `Database`, `SQS`, `RabbitMQ`): `SendInvoiceJob::dispatch($order)`.
        
    - Фоновые воркеры (`php artisan queue:work` под управлением **Supervisor** или **Laravel Horizon** для Redis) забирают задачи и выполняют метод `handle()`.
        
- **Критически важные настройки Job на продакшене**:
    
    - `$tries = 3`: количество попыток выполнения перед отправкой в таблицу `failed_jobs`.
        
    - `$backoff = [10, 60, 300]`: экспоненциальная задержка (в секундах) перед повторными попытками.
        
    - `ShouldBeUnique`: интерфейс, гарантирующий (через атомарные блокировки в кэше), что в очереди одновременно не будет двух одинаковых задач (например, дважды запущенный пересчёт баланса).
        
    - `afterCommit()` (или конфиг `'after_commit' => true`): гарантирует, что задача попадёт в Redis **только после успешного `COMMIT` транзакции БД**. Иначе быстрый воркер может схватить задачу раньше, чем БД зафиксирует запись, и упасть с `ModelNotFoundException`!
        
- **Events & Listeners (События и Слушатели)**:
    
    - Паттерн **Observer** для развязывания бизнес-логики. При событии `OrderPaid` один слушатель отправляет чек на почту, второй начисляет бонусы, третий шлёт уведомление в Telegram. Если слушатель реализует `ShouldQueue`, он выполнится асинхронно в очереди.
        
- **Task Scheduler (Планировщик)**: вместо десятков записей в системном `crontab` на сервере добавляется одна запись `* * * * * php /path/artisan schedule:run`, а всё расписание описывается в коде (`routes/console.php`). Для кластера из нескольких серверов используется `->onOneServer()` (координация через Redis-лок).
    

## 10. Кэширование, Логирование и Обработка ошибок

> [!abstract] Коротко
> 
> Кэширование в Redis ускоряет ответы и снижает нагрузку на БД; атомарные блокировки (`Cache::lock`) спасают от гонок и Cache Stampede.

### Теория (🇷🇺 Русский & 🇬🇧 English)

- **Паттерн Cache-Aside в Laravel**:
    
    - `Cache::remember("user:{$id}", 3600, fn () => User::findOrFail($id))` — ищет в кэше; если нет, выполняет замыкание, сохраняет результат на 3600 секунд и возвращает его.
        
    - `Cache::flexible("stats", [60, 300], fn () => ...)` (Laravel 11+): паттерн **Stale-While-Revalidate** — первые 60 сек данные считаются свежими; с 60 по 300 сек отдаются устаревшие данные мгновенно, а обновление запускается в фоне после отправки ответа (`defer`), полностью предотвращая **Cache Stampede**!
        
- **Атомарные блокировки (Distributed Locks)**:
    
    
    ```PHP
    // Защита от одновременного нажатия кнопки оплаты двумя запросами
    Cache::lock("pay-order-{$order->id}", 10)->block(5, function () use ($order) {
        // Критическая секция: только один процесс выполняется здесь одновременно
    });
    ```
    
- **Обработка исключений**: настраивается в `bootstrap/app.php` через `->withExceptions(function (Exceptions $exceptions) { ... })`. Позволяет кастомизировать рендер ошибок (например, приводить все исключения к единому JSON-формату для API) и настраивать репортинг в Sentry/Bugsnag.
    

## 11. Laravel Octane, Swoole/RoadRunner и производительность

> [!abstract] Коротко
> 
> В классическом PHP-FPM фреймворк загружается заново на каждый запрос. **Laravel Octane** держит загруженное приложение в RAM между запросами поверх **Swoole** или **RoadRunner**, ускоряя ответы в 5–10 раз.

### Теория (🇷🇺 Русский & 🇬🇧 English)

- **Как работает Octane**: при старте сервера поднимается пул воркеров, каждый воркер один раз загружает ядро Laravel, конфиги, роуты и провайдеры в оперативную память и обрабатывает тысячи HTTP-запросов без повторного бутстрапа.
    
- **Главные риски при работе с Octane (Обязательно знать на собеседовании!)**:
    
    1. **Загрязнение состояния (State Pollution)**: любые статические свойства классов (`public static $currentUser`), глобальные переменные или `singleton`-сервисы, внутри которых сохраняется объект текущего `Request` или `User`, **переживают запрос** и утекают к следующему пользователю!
        
    2. **Решение**: использовать `$app->scoped(...)` вместо `singleton()`, не инжектить `Request` в конструктор синглтон-сервиса (инжектить в метод или использовать `fn () => request()`), не хранить пользовательские данные в `static`-свойствах.
        
    3. **Утечки памяти (Memory Leaks)**: если массив в синглтоне постоянно растёт от запроса к запросу, воркер упрётся в лимит памяти. Лечится перезапуском воркеров каждые N запросов (`--max-requests=500`).
        
- **Чек-лист оптимизации Laravel для продакшена**:
    
    - `php artisan optimize` (кэширует конфиги, роуты, события и Blade-шаблоны).
        
    - `composer install --no-dev --optimize-autoloader`.
        
    - Включённый PHP OPcache (`opcache.validate_timestamps=0`).
        
    - Драйверы сессий и кэша переведены с `file`/`database` на `redis`.
        

## 12. Тестирование в Laravel и Шпаргалка перед собеседованием

> [!abstract] Коротко
> 
> Laravel предоставляет мощный инструментарий для Feature-тестов (HTTP-запросы, проверка БД) и удобные фейки (`Queue::fake()`, `Http::fake()`).

### Теория и Код

- **Unit vs Feature тесты**:
    
    - _Unit-тесты_: проверяют изолированный класс/метод (например, расчёт скидки или DTO) без поднятия БД и HTTP.
        
    - _Feature-тесты_: имитируют полноценный HTTP-запрос (`$this->postJson('/api/orders', ...)`), прогоняют его через middleware, валидацию, контроллер, БД (с откатом через трейт `RefreshDatabase`) и проверяют JSON-ответ.
        
- **Фейки (Fakes) для изоляции внешних систем**:
    
    
    ```PHP
    public function test_user_can_register(): void
    {
        Event::fake([UserRegistered::class]);
        Queue::fake();
        Http::fake(['[api.crm.com/](https://api.crm.com/)*' => Http::response(['status' => 'ok'], 200)]);
    
        $response =$this->postJson('/api/register', [
            'email' => 'dev@example.com',
            'password' => 'StrongPass123!',
        ]);
    
        $response->assertCreated();$this->assertDatabaseHas('users', ['email' => 'dev@example.com']);
        Queue::assertPushed(SendWelcomeEmailJob::class);
    }
    ```

### Шпаргалка перед собеседованием

> [!important] Ключевые тезисы по Laravel:
> 
> 1. **Service Providers**: сначала у всех вызывается `register()` (только биндинг в контейнер), затем у всех `boot()` (можно использовать сервисы).
>     
> 2. **Bind vs Singleton vs Scoped**: `bind` — каждый раз новый объект; `singleton` — один на всё время жизни приложения; `scoped` — один на запрос (сбрасывается в Octane и Queue Worker).
>     
> 3. **Facades**: работают через `__callStatic()`, резолвя реальный объект из Service Container по ключу из `getFacadeAccessor()`.
>     
> 4. **Eloquent N+1**: решается через `with()` (Eager Loading); в dev-среде страхуемся через `Model::preventLazyLoading()`. Для больших выборок используем `chunkById()` или `lazyById()`.
>     
> 5. **Queues + Transactions**: используем `->afterCommit()`, чтобы воркер не запустил задачу до коммита данных в БД, и `ShouldBeUnique` от дублей.
>     
> 6. **Laravel Octane**: держит приложение в RAM; главная опасность — утечка состояния между пользователями через `static` свойства и `singleton`-сервисы.
>