### Mapped Types

**Идея:** взять существующий тип и **пройтись по каждому его ключу**, применяя одно и то же преобразование — вместо того чтобы писать новый тип с нуля вручную.

Синтаксис строится на `keyof` (получить union всех ключей типа) и `in` (пройтись по каждому значению union):

```ts
interface User {
  id: number;
  name: string;
}

// Ручная реализация Partial
type MyPartial<T> = {
  [K in keyof T]?: T[K];
};

type PartialUser = MyPartial<User>;
// { id?: number; name?: string; }
```

Разберём по частям:

- `keyof T` → `'id' | 'name'` (union строковых литералов — ключей типа `T`)
- `[K in keyof T]` → для каждого `K` из этого union создаём новое свойство с именем `K`
- `T[K]` → **indexed access type** — берёт тип значения по ключу `K` (например, `T['id']` → `number`)
- `?` → делает свойство опциональным

**Модификаторы, которые можно добавлять/убирать в mapped types:**

```ts
// Убрать readonly и опциональность (через минус)
type MyRequired<T> = {
  [K in keyof T]-?: T[K]; // убирает "?", если он был
};

type MyMutable<T> = {
  -readonly [K in keyof T]: T[K]; // убирает readonly, если был
};

// Переименовать ключи через "as" (key remapping, TS 4.1+)
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}``]: () => T[K];
};

interface Person {
  name: string;
  age: number;
}

type PersonGetters = Getters<Person>;
// { getName: () => string; getAge: () => number; }
```

---

### Conditional Types

**Идея:** тип выбирается **условно**, в зависимости от того, "подходит" ли один тип под другой — работает по аналогии с тернарным оператором, но на уровне типов, а не значений.

Синтаксис: `T extends U ? X : Y` — "если тип `T` совместим с `U`, результат `X`, иначе `Y`".

```ts
type IsString<T> = T extends string ? true : false;

type A = IsString<'hello'>; // true
type B = IsString<42>;      // false
```

**Ключевое слово `infer`** — позволяет "вытащить" (вывести) тип изнутри другой конструкции прямо внутри условия:

```ts
// Как устроен встроенный ReturnType<T>
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function createUser() {
  return { id: 1, name: "Alice" };
}

type User = MyReturnType<typeof createUser>;
// { id: number; name: string; }
```

Здесь `infer R` говорит: "если `T` — это функция, выведи тип её возвращаемого значения и назови его `R`, а дальше верни именно `R`".

**Distributive conditional types** — важный, часто спрашиваемый нюанс: если `T` — это **union-тип**, а не одиночный тип, conditional type автоматически **применяется к каждому члену union по отдельности**, а результаты объединяются обратно в union:

```ts
type ToArray<T> = T extends any ? T[] : never;

type Result = ToArray<string | number>;
// Не (string | number)[], а string[] | number[]
// TS применяет ToArray отдельно к string и отдельно к number, потом объединяет
```

Это и есть механизм, на котором построены `Exclude`/`Extract`:

```ts
// Как устроен встроенный Exclude<T, U>
type MyExclude<T, U> = T extends U ? never : T;

type Result = MyExclude<'a' | 'b' | 'c', 'a'>;
// Проверяется по одному: 'a' extends 'a' ? never : 'a' → never
//                        'b' extends 'a' ? never : 'b' → 'b'
//                        'c' extends 'a' ? never : 'c' → 'c'
// Итог: never | 'b' | 'c' → 'b' | 'c' (never выпадает из union автоматически)
```

---

### Зачем это знать практически

Mapped и conditional types — это не просто теория "для собеса", это фундамент, на котором построена вся стандартная библиотека `lib.d.ts` в TypeScript (все utility types из прошлого конспекта реализованы именно так). Понимание этого механизма нужно, когда:

- Стандартных utility types не хватает, и нужно написать свой.
- Нужно разобраться в чужом сложном типе в библиотеке (например, в типах React или сложных API-клиентов).
- На собеседовании senior-уровня почти гарантированно спросят "как устроен `Partial` изнутри" или "напиши свой `DeepReadonly`".

**Частый вопрос на собесе**, который стоит запомнить: _"Напиши тип `DeepReadonly<T>`, который рекурсивно делает все поля объекта readonly, включая вложенные объекты."_

```ts
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object ? DeepReadonly<T[K]> : T[K];
};
```

Тут комбинируются оба механизма сразу: mapped type (`[K in keyof T]`) и conditional type (`T[K] extends object ? ... : ...`) — плюс рекурсия внутри самого type alias.
