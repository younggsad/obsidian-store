---
custom-width: 80
---
## 3.1 Что такое Zod и зачем он нужен

>**Zod** — библиотека для **валидации и типизации данных** во время выполнения (runtime validation), с полной интеграцией в TypeScript — типы **выводятся автоматически** из схемы валидации, без необходимости писать их отдельно вручную.

>**Проблема, которую решает Zod:** TypeScript проверяет типы только **на этапе компиляции** — данные, пришедшие извне (тело HTTP-запроса, ответ стороннего API, содержимое файла) для TypeScript **всегда** имеют тип `any`/`unknown` на момент получения — компилятор не может гарантировать, что реальные данные соответствуют ожидаемой структуре. Zod даёт **runtime-проверку**, которая реально исполняется в момент получения данных, и одновременно выводит из этой проверки точный TypeScript-тип.

## 3.2 Установка

```bash
npm install zod
```

## 3.3 Базовые схемы

```typescript
import { z } from 'zod';

const stringSchema = z.string();
const numberSchema = z.number();
const booleanSchema = z.boolean();

stringSchema.parse("привет");    // "привет" — успешно, возвращает значение
stringSchema.parse(42);              // ❌ выбрасывает ZodError — не строка
```

## 3.4 `parse` vs `safeParse`

```typescript
// parse — выбрасывает исключение при невалидных данных
try {
  const result = stringSchema.parse(42);
} catch (error) {
  console.log(error.errors);
}

// safeParse — НЕ выбрасывает исключение, возвращает объект-результат
const result = stringSchema.safeParse(42);
if (result.success) {
  console.log(result.data);
} else {
  console.log(result.error.errors); // подробности всех ошибок валидации
}
```

`safeParse` предпочтительнее в местах, где ошибка валидации — **ожидаемая** ситуация (например, проверка пользовательского ввода из формы), а не исключительная — избегает необходимости оборачивать в `try/catch` для обычного, предсказуемого сценария.

## 3.5 Объектные схемы — типичный случай для валидации запросов

```typescript
const userSchema = z.object({
  name: z.string().min(2, "Имя должно быть минимум 2 символа"),
  email: z.string().email("Некорректный email"),
  age: z.number().int().positive().optional(),  // optional — необязательное поле
  role: z.enum(["admin", "user", "guest"]),        // только одно из перечисленных значений
});

type User = z.infer<typeof userSchema>; // TypeScript-тип ВЫВОДИТСЯ автоматически из схемы!
// эквивалентно:
// type User = {
//   name: string;
//   email: string;
//   age?: number;
//   role: "admin" | "user" | "guest";
// }
```

`z.infer<typeof schema>` — ключевая возможность: пишешь схему **один раз**, а TypeScript-тип получаешь "бесплатно", без дублирования одной и той же структуры в двух местах (сама схема + отдельный `interface`/`type`) — прямое применение принципа DRY, который разбирали ранее.

## 3.6 Цепочки валидаторов (модификаторы)

```typescript
z.string()
  .min(3, "Минимум 3 символа")
  .max(20, "Максимум 20 символов")
  .trim()                              // убирает пробелы по краям ПЕРЕД проверкой длины
  .toLowerCase();                        // приводит к нижнему регистру

z.number()
  .int("Должно быть целым числом")
  .positive("Должно быть положительным")
  .max(150, "Некорректный возраст");

z.string().email();
z.string().url();
z.string().uuid();
z.string().regex(/^\d{10}$/, "Номер телефона должен состоять из 10 цифр");

z.string().optional();      // string | undefined
z.string().nullable();        // string | null
z.string().nullish();          // string | null | undefined
z.string().default("гость");     // если значение не передано — подставляется значение по умолчанию
```

## 3.7 Массивы и вложенные объекты

```typescript
const orderSchema = z.object({
  id: z.number(),
  items: z.array(                          // массив объектов
    z.object({
      productId: z.number(),
      quantity: z.number().positive(),
    })
  ).min(1, "Заказ должен содержать хотя бы один товар"),
  shippingAddress: z.object({                 // вложенный объект
    city: z.string(),
    zipCode: z.string(),
  }),
});
```

## 3.8 Union и Discriminated Union — валидация "одно из нескольких"

```typescript
const responseSchema = z.union([
  z.object({ status: z.literal("success"), data: z.string() }),
  z.object({ status: z.literal("error"), message: z.string() }),
]);

// Discriminated union — более эффективная и точная версия, когда есть общее "поле-дискриминатор"
const responseSchema2 = z.discriminatedUnion("status", [
  z.object({ status: z.literal("success"), data: z.string() }),
  z.object({ status: z.literal("error"), message: z.string() }),
]);
```

## 3.9 Кастомная валидация — `.refine()`

```typescript
const passwordSchema = z.object({
  password: z.string().min(8),
  confirmPassword: z.string(),
}).refine((data) => data.password === data.confirmPassword, {
  message: "Пароли не совпадают",
  path: ["confirmPassword"], // указывает, к какому конкретно полю относится ошибка
});
```

`.refine()` позволяет добавить **любую** произвольную логику валидации, выходящую за рамки встроенных проверок (например, сравнение двух полей друг с другом, проверка через внешний запрос — есть и асинхронный вариант `.refine()` с `async`).

## 3.10 Трансформация данных — `.transform()`

```typescript
const dateSchema = z.string().transform((str) => new Date(str));
// входные данные — строка, но результат ПОСЛЕ парсинга — уже объект Date

const trimmedNumberSchema = z.string().transform((val) => Number(val.trim()));
```

`.transform()` не просто проверяет, но и **преобразует** данные в процессе валидации — полезно для нормализации ввода (строка из формы → число, строка даты → объект `Date`).

## 3.11 Практический пример — валидация тела запроса в Express

```typescript
import express from 'express';
import { z } from 'zod';

const app = express();
app.use(express.json());

const createUserSchema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
  age: z.number().int().positive().optional(),
});

app.post('/api/users', (req, res) => {
  const result = createUserSchema.safeParse(req.body);

  if (!result.success) {
    return res.status(400).json({
      error: 'Некорректные данные',
      details: result.error.flatten().fieldErrors, // удобный, структурированный формат ошибок по полям
    });
  }

  const validData = result.data; // ПОЛНОСТЬЮ типизированные данные, гарантированно соответствующие схеме
  // ...сохранение в БД через Prisma...
  res.status(201).json(validData);
});
```

### Middleware-обёртка для переиспользуемой валидации

```typescript
function validateBody(schema) {
  return (req, res, next) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      return res.status(400).json({ error: result.error.flatten().fieldErrors });
    }
    req.body = result.data; // заменяем на провалидированные (и, возможно, трансформированные) данные
    next();
  };
}

app.post('/api/users', validateBody(createUserSchema), (req, res) => {
  // здесь req.body уже гарантированно валиден
  res.status(201).json(req.body);
});
```

Этот паттерн — прямое применение идеи **middleware** из Части 1: валидация вынесена в переиспользуемую функцию, а не дублируется в каждом обработчике маршрута вручную (снова DRY).

## 3.12 Zod + Prisma — типичная связка в реальном проекте

```typescript
// Zod валидирует ВХОДНЫЕ данные от клиента (до похода в БД)
const createPostSchema = z.object({
  title: z.string().min(1).max(200),
  content: z.string().optional(),
  authorId: z.number().int(),
});

app.post('/api/posts', validateBody(createPostSchema), async (req, res) => {
  const post = await prisma.post.create({ data: req.body }); // Prisma сохраняет уже ПРОВЕРЕННЫЕ данные
  res.status(201).json(post);
});
```

**Разделение ответственности** (снова принцип SRP из SOLID): Zod отвечает за то, что данные **синтаксически корректны и соответствуют ожидаемой форме**, ещё до того, как они вообще доберутся до слоя работы с базой данных — Prisma в этом случае отвечает только за само сохранение уже проверенных данных, не занимаясь их валидацией самостоятельно.