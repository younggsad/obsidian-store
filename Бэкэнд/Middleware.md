---
custom-width: 80
---

## Что такое Middleware

>**Middleware (промежуточное ПО)** — это функция, которая находится между получением HTTP-запроса сервером и выполнением основного обработчика (controller).

Middleware получает доступ к:

- объекту запроса `req` (**Request**)
- объекту ответа `res` (**Response**)
- функции `next()`, которая передает управление следующему middleware или контроллеру

Общий вид:

```ts
(req, res, next) => {
    // логика middleware

    next();
}
```

Главная идея:

> Middleware позволяет вынести общую логику из контроллеров и выполнять её до или после обработки запроса.

---

# Зачем нужны Middleware

Middleware используют для решения повторяющихся задач:

- проверка авторизации
- проверка прав доступа
- валидация данных
- обработка ошибок
- логирование запросов
- настройка CORS
- работа с cookies
- rate limiting
- преобразование данных запроса


Пример без middleware:

```ts
app.get("/users", (req, res) => {

    if (!req.headers.authorization) {
        return res.status(401).json({
            message: "Unauthorized"
        });
    }

    // основная логика

});
```

Проблема:

Если таких проверок много, код начинает дублироваться.

С middleware:

```ts
app.get(
    "/users",
    authMiddleware,
    userController.getUsers
);
```

Контроллер отвечает только за бизнес-логику.

---

# Как работает цепочка Middleware

HTTP запрос проходит через цепочку middleware:

```
Request
   |
   v
Middleware 1
   |
   v
Middleware 2
   |
   v
Controller
   |
   v
Response
```


Пример:

```ts
app.use(loggerMiddleware);

app.use(authMiddleware);

app.use("/users", userRoutes);
```

Запрос:

```
GET /users
```

проходит:

```
loggerMiddleware
        |
        v
authMiddleware
        |
        v
UserController
```

---

# Функция next()

`next()` передает управление следующему обработчику.

Пример:

```ts
const loggerMiddleware = (
    req,
    res,
    next
) => {

    console.log(req.method);

    next();
};
```

Если вызвать `next()`:

```
Middleware -> Controller
```

Если не вызвать:

```
Middleware -> остановка запроса
```

Например:

```ts
const authMiddleware = (
    req,
    res,
    next
) => {

    if (!req.user) {
        return res.status(401).json({
            message: "Unauthorized"
        });
    }

    next();
};
```

При отсутствии пользователя запрос дальше не идет.

---

# Виды Middleware

## 1. Application-level Middleware

Подключается через `app.use()`.

Пример:

```ts
app.use(express.json());
```

Используется глобально для всего приложения.


---

## 2. Router-level Middleware

Работает только внутри определенного роутера.

Пример:

```ts
router.use(authMiddleware);

router.get("/", controller.getAll);
```

Теперь middleware применяется только к этому роуту.


---

## 3. Built-in Middleware

Встроенные middleware Express.

Пример:

```ts
app.use(express.json());
```

Разбирает JSON тело запроса:

До:

```json
{
    "name": "Alex"
}
```

После:

```ts
req.body = {
    name: "Alex"
}
```


---

## 4. Third-party Middleware

Сторонние middleware.

Например:

```ts
import cors from "cors";

app.use(cors());
```

или:

```ts
import helmet from "helmet";

app.use(helmet());
```


---

## 5. Error Middleware

Специальный middleware для обработки ошибок.

Отличается четырьмя аргументами:

```ts
(
    error,
    req,
    res,
    next
)
```

Пример:

```ts
export const errorMiddleware = (
    error,
    req,
    res,
    next
) => {

    res.status(500).json({
        message: error.message
    });

};
```

Express понимает, что это error middleware именно из-за четырех аргументов.

---

# Async Middleware

В Express часто используются асинхронные middleware.

Например проверка пользователя:

```ts
const authMiddleware = async (
    req,
    res,
    next
) => {

    const user = await findUser();

    req.user = user;

    next();
};
```

Проблема:

Ошибки в async middleware нужно правильно передавать.

Используют wrapper:

```ts
export const asyncHandler =
(
    fn
) =>
(
    req,
    res,
    next
) => {

    Promise
        .resolve(fn(req,res,next))
        .catch(next);

};
```

Теперь ошибки попадут в error middleware.

---

# Middleware в архитектуре Express приложения

Обычно структура:

```
src
 |
 |-- middleware
 |      |
 |      |-- auth.middleware.ts
 |      |-- validate.middleware.ts
 |      |-- error.middleware.ts
 |
 |-- controllers
 |
 |-- services
 |
 |-- routes
```

Ответственность:

```
Middleware
    |
    |-- проверка запроса
    |-- безопасность
    |-- подготовка данных

Controller
    |
    |-- принимает запрос
    |-- вызывает service

Service
    |
    |-- бизнес-логика

Repository
    |
    |-- работа с БД
```

---

# Что сказать на собеседовании про Middleware

> Middleware — это промежуточный слой обработки HTTP-запроса в Express. Он получает request, response и функцию next. Middleware используется для вынесения общей логики, например авторизации, валидации, логирования и обработки ошибок. Middleware выполняются последовательно в цепочке, а next передает управление следующему обработчику.