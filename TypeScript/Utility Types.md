
## Что это такое

>**Utility types** — это встроенные generic-типы TypeScript, которые позволяют трансформировать уже существующие типы, не описывая новый тип с нуля. Они работают поверх **mapped types** и **conditional types** — то есть под капотом это обычные TS-конструкции, просто вынесенные в стандартную библиотеку типов для удобства.

Все они принимают тип как generic-параметр: `UtilityType<SomeType>`.

---

## `Partial<T>`

Делает **все поля опциональными** (необязательными) — добавляет `?` к каждому свойству.

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

type PartialUser = Partial<User>;
// эквивалентно:
// { id?: number; name?: string; email?: string; }
```

**Где применяется:** функции обновления объекта (`updateUser`), где не все поля обязательно передавать:

```ts
function updateUser(id: number, changes: Partial<User>) {
  // changes может содержать любое подмножество полей User
}

updateUser(1, { name: "Alice" }); // OK, остальные поля не нужны
```

---

## `Required<T>`

Полная противоположность `Partial` — делает **все поля обязательными**, убирает `?`.

```ts
interface Config {
  host?: string;
  port?: number;
}

type FullConfig = Required<Config>;
// { host: string; port: number; }
```

**Где применяется:** когда есть тип с опциональными полями (например, входные настройки от пользователя), а внутри функции нужно гарантировать, что все поля уже заполнены (обычно после применения значений по умолчанию).

---

## `Readonly<T>`

Делает **все поля доступными только для чтения** — попытка присвоить значение свойству вызовет ошибку компиляции.

```ts
interface Point {
  x: number;
  y: number;
}

const p: Readonly<Point> = { x: 1, y: 2 };
p.x = 10; // Ошибка: Cannot assign to 'x' because it is a read-only property
```

**Где применяется:** защита от случайной мутации объекта — часто используется для пропсов в React, конфигов, констант, любых данных, которые не должны меняться после создания.

**Важный нюанс:** `Readonly` работает только на **верхнем уровне** — это не делает вложенные объекты неизменяемыми (не рекурсивно):

```ts
interface Nested {
  inner: { value: number };
}

const n: Readonly<Nested> = { inner: { value: 1 } };
n.inner = { value: 2 }; // Ошибка
n.inner.value = 99;     // OK! Вложенное свойство менять можно
```

---

## Pick<T, Keys>

Создаёт новый тип, **выбирая только указанные поля** из исходного типа.

```ts
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

type PublicUser = Pick<User, 'id' | 'name'>;
// { id: number; name: string; }
```

**Где применяется:** когда нужен "урезанный" вариант типа для конкретного случая — например, публичное API не должно возвращать `password`, поэтому явно выбираем только безопасные поля.

---

## Omit<T, Keys>

Противоположность `Pick` — создаёт новый тип, **исключая указанные поля**, оставляя всё остальное.

```ts
type UserWithoutPassword = Omit<User, 'password'>;
// { id: number; name: string; email: string; }
```

**Где применяется:** тот же сценарий, что и `Pick`, но удобнее, когда исключаемых полей мало, а оставшихся — много (не нужно перечислять всё, что оставляем).

**Практическое правило выбора между Pick и Omit:** если нужных полей **меньше** — используй `Pick` (перечисли, что оставить). Если исключаемых полей **меньше** — используй `Omit` (перечисли, что убрать).

---

## Record<Keys, Type>

Создаёт тип объекта, где **ключи** — это одно из перечисленных значений (обычно union-тип или `string`/`number`), а **значения** — заданного типа. По сути — типизированный словарь/map.

```ts
type Role = 'admin' | 'editor' | 'viewer';

type Permissions = Record<Role, boolean>;
// { admin: boolean; editor: boolean; viewer: boolean; }

const perms: Permissions = {
  admin: true,
  editor: true,
  viewer: false
};
```

**Где применяется:** маппинг из фиксированного набора ключей (enum, union строк) на значения — конфиги, словари переводов, таблицы соответствий status → цвет и т.д.

```ts
type Status = 'success' | 'error' | 'pending';

const statusColors: Record<Status, string> = {
  success: 'green',
  error: 'red',
  pending: 'yellow'
};
```

---

## Другие полезные utility types (кратко)

|Тип|Что делает|
|---|---|
|`Exclude<T, U>`|Убирает из union-типа `T` все типы, входящие в `U`|
|`Extract<T, U>`|Оставляет из union-типа `T` только типы, входящие в `U`|
|`NonNullable<T>`|Убирает `null` и `undefined` из типа|
|`ReturnType<T>`|Возвращает тип, который возвращает функция `T`|
|`Parameters<T>`|Возвращает tuple-тип с параметрами функции `T`|
|`Awaited<T>`|Разворачивает тип из `Promise<T>` (например, для async-функций)|

Пример `Exclude`/`Extract`:

```ts
type AllTypes = 'a' | 'b' | 'c';

type WithoutA = Exclude<AllTypes, 'a'>; // 'b' | 'c'
type OnlyA = Extract<AllTypes, 'a'>;    // 'a'
```

Пример `ReturnType`:

```ts
function createUser() {
  return { id: 1, name: "Alice" };
}

type User = ReturnType<typeof createUser>;
// { id: number; name: string; }
```

---

## Частые вопросы на собеседовании

**В: Как реализован `Partial<T>` под капотом?** О: Через mapped type:

```ts
type Partial<T> = {
  [P in keyof T]?: T[P];
};
```

`keyof T` — получает union всех ключей типа `T`, `in` — проходит по каждому ключу, `?` — делает опциональным.

**В: Можно ли комбинировать utility types?** О: Да, это частая практика:

```ts
type UpdatableUser = Partial<Omit<User, 'id'>>;
// все поля опциональны, кроме того что id вообще убрали из типа
```

**В: В чём разница между `Readonly<T>` и `const`?** О: `const` защищает **переменную** от переприсваивания (нельзя сделать `x = ...` заново), но не защищает от мутации объекта, на который она ссылается. `Readonly<T>` защищает именно **свойства объекта** от изменения — можно использовать вместе: `const p: Readonly<Point> = {...}` защищает и от переприсваивания `p`, и от мутации `p.x`.