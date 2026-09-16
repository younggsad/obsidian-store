>Execution Context — это внутренняя структура данных, которую движок создает для выполнения кода. Она содержит Lexical Environment, Variable Environment, значение `this` и другую служебную информацию, необходимую для выполнения текущего участка кода.

# Какие бывают Execution Context?

## 1. Global Execution Context

Создается **один раз**.

```
console.log("Hello");
```

Как только файл начал выполняться:

```
Global EC
```

уже существует.

---

## 2. Function Execution Context

Каждый вызов функции создает **новый** контекст.

```
function foo() {}

foo();
foo();
foo();
```

Сколько раз вызвали?
Три.

Сколько контекстов?
Тоже три.

---

## 3. Eval Execution Context

Практически не используется.

```
eval("console.log(1)");
```
