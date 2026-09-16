---
custom-width: 80
---
## Что такое DTO

>**DTO (Data Transfer Object)** — это объект, который описывает структуру данных, передаваемых между слоями приложения.

Главная задача DTO:

> Контролировать форму входящих и исходящих данных и отделять внешний API контракт от внутренних моделей приложения.

---

# Проблема без DTO

Например:

```ts
const user = {
    id: 1,
    name: "Alex",
    password: "12345",
    role: "ADMIN"
};
```

Если вернуть его напрямую:

```ts
res.json(user);
```

мы случайно отправим пароль клиенту.

---

С DTO:

Создаем объект ответа:

```ts
class UserResponseDto {

    id: number;
    name: string;

    constructor(user) {

        this.id = user.id;
        this.name = user.name;

    }

}
```

Теперь:

```ts
res.json(
    new UserResponseDto(user)
);
```

Ответ:

```json
{
    "id":1,
    "name":"Alex"
}
```

---

# Где используется DTO

Основные места:

## 1. Request DTO

Описывает входящие данные.

Например:

POST /users


Клиент отправляет:

```json
{
    "name":"Alex",
    "email":"alex@mail.com"
}
```


DTO:

```ts
class CreateUserDto {

    name:string;

    email:string;

}
```

---

## 2. Response DTO

Описывает данные ответа.

Пример:

```ts
class UserResponseDto {

    id:string;

    name:string;

}
```

---

# DTO и Validation

Часто DTO используют вместе с валидацией.

Например Zod:

```ts
const createUserSchema = z.object({

    name:z.string(),

    email:z.string().email()

});
```

Middleware проверяет:

```ts
const result =
createUserSchema.safeParse(req.body);
```

Если данные неправильные:

```
Request
   |
   v
Validation Middleware
   |
   X
400 Bad Request
```

---

# DTO vs Entity

Важно различать.


## Entity

Это объект базы данных.


Например Prisma:

```ts
User {
    id
    email
    password
    createdAt
}
```


## DTO

Это объект для передачи данных.


Например:

```ts
CreateUserDto {

    email

}
```


Разница:

| Entity | DTO |
|-|-|
| Модель БД | Контракт API |
| Хранит все поля | Только нужные поля |
| Связан с ORM | Не зависит от БД |
| Внутренний объект | Внешний объект |


---

# DTO vs Interface / TypeScript Type

DTO:

```ts
class CreateUserDto {

    email:string;

}
```

может иметь:

- методы
- трансформацию
- runtime поведение


Interface:

```ts
interface CreateUser {

    email:string;

}
```

существует только во время компиляции.

После сборки TypeScript удаляет interface.


---

# DTO в архитектуре приложения

Поток данных:

```
Client

 |
 v

Request DTO

 |
 v

Controller

 |
 v

Service

 |
 v

Entity

 |
 v

Database
```


Ответ:

```
Database

 |
 v

Entity

 |
 v

Response DTO

 |
 v

Client
```

---

# Почему DTO важны

Преимущества:

## Безопасность

Не отправляем лишние данные:

```
password
internalId
database fields
```

---

## Независимость

Можно поменять БД:

```
PostgreSQL
      |
      v
MongoDB
```

а API контракт останется прежним.


---

## Контроль API

DTO определяет:

что клиент может отправить

и

что клиент может получить.


---

# Что сказать на собеседовании про DTO

> DTO — это объект передачи данных между слоями приложения. Он нужен для отделения внешнего API контракта от внутренних моделей, например Entity или моделей базы данных. DTO позволяет контролировать какие данные принимаются и возвращаются, повышает безопасность и упрощает изменение внутренней архитектуры без изменения API.