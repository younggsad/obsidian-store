>**TypeScript** — это надстройка над JavaScript ([[Что такое JavaScript]]), которая добавляет статическую типизацию. Код на TS компилируется в обычный JS. Главная задача — ловить ошибки на этапе написания кода, а не во время его выполнения в браузере.

#### 1. Зачем нужен TypeScript (простыми словами)

- **Подсказки в редакторе:** Вы пишете `user.` — и видите все доступные свойства, не заглядывая в документацию.
    
- **Ошибки до запуска:** Забыли передать аргумент в функцию или перепутали тип — редактор подсветит сразу.
    
- **Документация через типы:** Другой разработчик видит сигнатуру функции и понимает, что она принимает и возвращает.
    

#### 2. Базовые типы (самое частое)

```typescript
let age: number = 25;              // число
let name: string = "Alice";        // строка
let isReady: boolean = true;       // true/false
let list: string[] = ["a", "b"];   // массив строк
let tuple: [string, number] = ["Alice", 25]; // кортеж
let nothing: null = null;
let notDefined: undefined = undefined;
```

**Тип `any` (и почему его избегать):**  
`any` отключает проверку. Переменная с `any` ведёт себя как в обычном JS.

```typescript
let x: any = 10;
x = "hello"; // ошибки нет
x.несуществующийМетод(); // ошибки нет, упадёт в рантайме
```

_Вопрос с собеседования:_ «Чем плох any?» — Он убивает весь смысл TypeScript. Лучше использовать `unknown`.

**Тип `unknown` (безопасный `any`):**

```typescript
let data: unknown = "строка";
// data.toUpperCase(); // Ошибка! Нельзя использовать, пока не проверим тип
if (typeof data === "string") {
  data.toUpperCase(); // Вот теперь можно
}
```

#### 3. Интерфейсы и Type (два способа описать объект)

**Interface:**

```typescript
interface User {
  name: string;
  age: number;
  email?: string; // ? — необязательное свойство
}
const user: User = {
  name: "Ivan",
  age: 30
};
```

**Type:**

```typescript
type User = {
  name: string;
  age: number;
};
```

_Главное отличие для ответа на собеседовании:_

- `interface` можно «дополнять» (Declaration Merging), объявив его ещё раз где-то в коде — свойства объединятся.
    
- `type` так нельзя. Зато `type` может описывать объединения (`union`) и кортежи.
    

#### 4. Объединения (Union) и пересечения (Intersection)

**Union — «ИЛИ»:**  
Переменная может быть одного типа из нескольких.

```typescript
let id: string | number;
id = "abc"; // ок
id = 123;   // ок
// id = true; // ошибка
```

**Intersection — «И» (объединение нескольких типов в один):**

```typescript
type Person = { name: string };
type Employee = { id: number };
type Worker = Person & Employee;
const w: Worker = { name: "Petr", id: 1 }; // должны быть все свойства
```

#### 5. Функции — типизация параметров и возврата

```typescript
// Типизируем параметры и возвращаемое значение
function sum(a: number, b: number): number {
  return a + b;
}
// void — функция ничего не возвращает
function log(message: string): void {
  console.log(message);
}
```

#### 6. Дженерики (Generics) — для начинающих

Дженерик позволяет функции или интерфейсу работать с разными типами, сохраняя информацию о них.  
_Без дженерика (плохо):_

```typescript
function identity(arg: any): any {
  return arg; // теряем тип
}
const result = identity("hello"); // result имеет тип any
```

_С дженериком (правильно):_

```typescript
function identity<T>(arg: T): T {
  return arg;
}
const result = identity("hello"); // TS знает, что result — строка
```

#### 7. Работа с объектами (простые Utility Types)

То, что реально пригодится и спросят:

- **`Partial<T>`** — все свойства становятся необязательными.
    
    ```typescript
    interface User { name: string; age: number }
    function updateUser(user: Partial<User>) { ... }
    updateUser({ name: "Bob" }); // age можно не передавать
    ```
    
- **Pick<T, K>** — взять только определённые свойства.
    
    ```typescript
    type UserPreview = Pick<User, "name">;
    // UserPreview = { name: string }
    ```
    
- **`Omit<T, K>`** — исключить свойства.
    
    ```typescript
    type UserWithoutAge = Omit<User, "age">;
    ```
    

#### 8. Конфигурация (tsconfig.json — что нужно знать)

Файл `tsconfig.json` в корне проекта говорит TypeScript, как компилировать код.

- **`target`:** Во что компилируем (`ES5` для старых браузеров, `ES2020` для современных).
    
- **`strict: true`:** Включает все строгие проверки. На проектах всегда должно быть `true`.
    
- **`outDir`:** Куда складывать скомпилированные `.js` файлы.
    

---

**Типовой вопрос с собеседования Junior:**

> «TypeScript — это язык или инструмент?»

**Хороший ответ:**  
>TypeScript — это надмножество JavaScript с системой типов. Его задача — статический анализ кода на этапе разработки. Весь TypeScript-код в итоге превращается в чистый JavaScript, который выполняется в браузере или Node.js. Типы существуют только до компиляции, в рантайме их нет.