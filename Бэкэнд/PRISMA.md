---
custom-width: 80
---
## 2.1 Что такое Prisma

>**Prisma** — современный **ORM (Object-Relational Mapping)** для Node.js/TypeScript — инструмент, позволяющий работать с базой данных через JS/TS-объекты и методы, вместо написания "сырого" SQL вручную. Поддерживает PostgreSQL, MySQL, SQLite, MongoDB и другие БД.

**Три основных компонента Prisma:**

1. **Prisma Schema** — файл, описывающий модели данных (таблицы) декларативным образом.
2. **Prisma Client** — автоматически сгенерированный, полностью типизированный клиент для запросов к БД.
3. **Prisma Migrate** — инструмент управления миграциями (изменениями структуры БД со временем).

## 2.2 Установка и инициализация

```bash
npm install prisma --save-dev
npm install @prisma/client
npx prisma init
```

Создаёт файл `prisma/schema.prisma` и `.env` с переменной `DATABASE_URL`.

## 2.3 Prisma Schema — описание моделей

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  createdAt DateTime @default(now())
  posts     Post[]      // связь один-ко-многим: у пользователя много постов
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  Int
}
```

### Разбор атрибутов

|Атрибут|Значение|
|---|---|
|`@id`|первичный ключ|
|`@default(autoincrement())`|автоинкремент (для числовых ID)|
|`@default(now())`|значение по умолчанию — текущее время|
|`@unique`|уникальное значение (нельзя дублировать)|
|`String?`|вопросительный знак — поле НЕОБЯЗАТЕЛЬНО (nullable)|
|`@relation`|описывает связь между моделями|

## 2.4 Миграции — синхронизация схемы с реальной БД

```bash
npx prisma migrate dev --name init      # создать и применить миграцию (для разработки)
npx prisma migrate deploy                  # применить накопленные миграции (для production)
npx prisma generate                           # сгенерировать/обновить Prisma Client после изменений схемы
```

`migrate dev` — сравнивает текущую `schema.prisma` с фактическим состоянием БД, генерирует SQL-файл миграции (в `prisma/migrations/`) и применяет его. Каждая миграция — это версионируемый, воспроизводимый шаг изменения структуры БД, который можно закоммитить в Git и применить на любом другом окружении (staging, production) в том же порядке.

## 2.5 Prisma Client — CRUD-операции

```javascript
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();
```

### Create

```javascript
const user = await prisma.user.create({
  data: {
    email: 'anna@example.com',
    name: 'Аня',
  },
});
```

### Read

```javascript
const user = await prisma.user.findUnique({
  where: { id: 1 },
});

const users = await prisma.user.findMany({
  where: { name: { contains: 'Ан' } },  // поиск по подстроке
  orderBy: { createdAt: 'desc' },
  take: 10,                                  // LIMIT
  skip: 0,                                      // OFFSET
});

const firstUser = await prisma.user.findFirst({
  where: { email: 'anna@example.com' },
});
```

### Update

```javascript
const updated = await prisma.user.update({
  where: { id: 1 },
  data: { name: 'Максим' },
});

await prisma.user.updateMany({
  where: { published: false },
  data: { published: true },
});
```

### Delete

```javascript
await prisma.user.delete({ where: { id: 1 } });
await prisma.user.deleteMany({ where: { published: false } });
```

## 2.6 Связи (relations) — `include`

```javascript
const userWithPosts = await prisma.user.findUnique({
  where: { id: 1 },
  include: { posts: true }, // подтягивает связанные записи ОДНИМ запросом (JOIN "под капотом")
});

console.log(userWithPosts.posts); // массив постов пользователя
```

Без `include` Prisma по умолчанию **не** подгружает связанные данные — это осознанный выбор (аналог "ленивой загрузки" в других ORM), защищающий от случайного выполнения избыточных запросов, если связанные данные реально не нужны.

## 2.7 Транзакции

```javascript
const [user, post] = await prisma.$transaction([
  prisma.user.create({ data: { email: 'test@test.com', name: 'Тест' } }),
  prisma.post.create({ data: { title: 'Пост', authorId: 1 } }),
]);
```

**Транзакция** гарантирует, что **все** операции внутри неё выполнятся успешно **вместе**, либо (при ошибке в любой из них) **ни одна не применится** — классический принцип атомарности (ACID). Важно для сценариев вроде "перевести деньги" (списать у одного, зачислить другому — обе операции должны либо обе пройти, либо обе откатиться).

```javascript
// Интерактивная транзакция — для сложной логики между шагами
await prisma.$transaction(async (tx) => {
  const account = await tx.account.findUnique({ where: { id: 1 } });
  if (account.balance < 100) {
    throw new Error("Недостаточно средств"); // выброс ошибки откатывает ВСЮ транзакцию
  }
  await tx.account.update({ where: { id: 1 }, data: { balance: account.balance - 100 } });
  await tx.account.update({ where: { id: 2 }, data: { balance: { increment: 100 } } });
});
```

## 2.8 Prisma Studio — визуальный интерфейс БД

```bash
npx prisma studio
```

Запускает браузерный GUI для просмотра и ручного редактирования данных в БД — полезно для отладки без написания SQL-запросов вручную.

## 2.9 Практический пример — полноценный Express + Prisma эндпоинт

```javascript
import express from 'express';
import { PrismaClient } from '@prisma/client';

const app = express();
const prisma = new PrismaClient();
app.use(express.json());

app.post('/api/users', async (req, res) => {
  try {
    const { email, name } = req.body;
    const user = await prisma.user.create({ data: { email, name } });
    res.status(201).json(user);
  } catch (error) {
    if (error.code === 'P2002') { // код ошибки Prisma для нарушения @unique
      return res.status(409).json({ error: 'Email уже зарегистрирован' });
    }
    res.status(500).json({ error: 'Ошибка сервера' });
  }
});
```