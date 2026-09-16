## 1.1 Что такое Promise и зачем он нужен

>**Promise** (промис) — это объект, представляющий результат асинхронной операции, который в момент создания ещё неизвестен, но **обещает** появиться в будущем — либо успешно, либо с ошибкой.

Проблема, которую решают промисы: асинхронные операции (запрос к серверу, таймер, чтение файла) не блокируют выполнение кода — они запускаются и "работают в фоне", а результат приходит позже. Без промисов пришлось бы городить вложенные callback-и ("callback hell"), что плохо читается и сложно поддерживать.

```javascript
// Без промиса — данных ещё нет к моменту вывода
let data = fetch("https://example.com/data");
console.log(data); // не то, что ожидалось

// С промисом — явно ждём результат
fetch("https://example.com/data").then(data => console.log(data));
```

---

## 1.2 Три состояния промиса

|Состояние|Значение|
|---|---|
|**pending** (ожидание)|промис создан, результат ещё не готов|
|**fulfilled** (выполнен)|операция завершилась успешно, есть результат|
|**rejected** (отклонён)|операция завершилась с ошибкой|

Как только промис перешёл в `fulfilled` или `rejected` — он считается **settled** (урегулированным) и **больше никогда** не может сменить состояние. Переход происходит один раз и необратимо.

---

## 1.3 Создание промиса — executor


```javascript
const myPromise = new Promise((resolve, reject) => {
  // это ИСПОЛНИТЕЛЬ (executor)
  // код здесь выполняется СИНХРОННО, сразу в момент создания промиса

  const success = true;

  if (success) {
    resolve("Успех!");   // переводит промис в fulfilled
  } else {
    reject("Ошибка!");   // переводит промис в rejected
  }
});
```

**Особенности executor'а:**

- `resolve` и `reject` — функции, которые создаёт сам JavaScript и передаёт исполнителю как аргументы.
- `resolve(значение)` — сообщает: операция прошла успешно, вот результат.
- `reject(причина)` — сообщает: произошла ошибка, вот причина.
- **Код внутри executor'а выполняется немедленно, синхронно** — это одна из самых частых ловушек на собеседованиях. Сам факт `new Promise(...)` не делает код асинхронным. Асинхронность появляется только там, где сам `resolve`/`reject` специально отложен (например, обёрнут в `setTimeout`).

```javascript
console.log("A");
const p = new Promise((resolve) => {
  console.log("B"); // выполнится СРАЗУ, синхронно
  resolve("C");
});
console.log("D");
// Порядок: A, B, D — и только потом (через микротаск) можно забрать "C" через .then()
```

---

## 1.4 Получение результата: `.then()` и `.catch()`


```javascript
myPromise
  .then((result) => {
    console.log("Успех:", result);
  })
  .catch((error) => {
    console.log("Ошибка:", error);
  });
```

- `.then(onFulfilled, onRejected)` — принимает до двух функций: первая вызывается при `resolve`, вторая (необязательная) — при `reject`.
- `.catch(onRejected)` — синтаксический сахар для `.then(undefined, onRejected)`, вызывается только при отклонении.
- Выполняется **только одна** ветка — либо `.then()`, либо `.catch()`, никогда обе.
- `.then()`/`.catch()`/`.finally()` — это **микротаски**, они выполняются раньше любых макротасков (`setTimeout` и т.д.), но после всего синхронного кода.

### `.finally()`

```javascript
myPromise
  .then((result) => console.log(result))
  .catch((error) => console.log(error))
  .finally(() => console.log("Выполнено в любом случае"));
```

Срабатывает независимо от результата (успех или ошибка) — удобно для действий вроде "скрыть индикатор загрузки".

---

## 1.5 Цепочка промисов (Promise chaining)

Каждый `.then()` **сам возвращает новый промис** — благодаря этому можно строить цепочку из нескольких последовательных шагов.

```javascript
function delay(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

delay(1000)
  .then(() => {
    console.log("Прошла 1 секунда");
    return delay(1000); // возвращаем НОВЫЙ промис
  })
  .then(() => {
    console.log("Прошло ещё 1 секунда (итого 2)");
  });
```

**Ключевое правило:** если из `.then()` вернуть промис — следующий `.then()` в цепочке автоматически дождётся его завершения, прежде чем выполниться сам.

### Частая ошибка — забыли `return`

```javascript
delay(1000).then(() => {
  console.log("Шаг 1");
  delay(1000); // без return!
}).then(() => {
  console.log("Шаг 2"); // выполнится СРАЗУ, не дождавшись второй задержки
});
```

Без `return` следующий `.then()` не знает, что нужно кого-то ждать — он получает `undefined` и продолжает немедленно.

### Передача значений по цепочке

```javascript
getNumber()
  .then((num) => {
    console.log(num);   // 5
    return num * 2;      // обычное значение — автоматически оборачивается в промис
  })
  .then((num) => {
    console.log(num);   // 10
    return num * 2;
  })
  .then((num) => {
    console.log(num);   // 20
  });
```

Если вернуть **обычное значение** (не промис) — JS сам оборачивает его в `Promise.resolve(значение)` и передаёт дальше. Если вернуть **промис** — следующий `.then()` дождётся именно его.

---

## 1.6 Обработка ошибок в цепочке

```javascript
riskyOperation()
  .then((result) => console.log("Не выполнится:", result))
  .then((result) => console.log("Тоже не выполнится:", result))
  .catch((error) => console.log("Поймали:", error));
```

Если где-то в цепочке произошёл `reject` (или выброшено исключение через `throw`), JavaScript **пропускает все `.then()`** до ближайшего `.catch()` — не обязательно ловить ошибку сразу после проблемного места, один `.catch()` в конце цепочки поймает ошибку из **любого** её участка.

**`throw` внутри `.then()`** тоже приводит к отклонению промиса:

```javascript
Promise.resolve()
  .then(() => {
    throw new Error("Что-то сломалось");
  })
  .catch((error) => console.log(error.message)); // "Что-то сломалось"
```

**Можно продолжить цепочку после `.catch()`** — `.catch()` тоже возвращает промис, и если внутри него нет повторного `throw`/`reject`, цепочка считается "восстановленной" и идёт дальше в `fulfilled`-состоянии:

```javascript
riskyOperation()
  .catch((error) => {
    console.log("Ошибка обработана:", error);
    return "запасное значение"; // цепочка продолжится как успешная
  })
  .then((result) => console.log(result)); // "запасное значение"
```

---

## 1.7 Promise.all — параллельное выполнение, дождаться всех

```javascript
Promise.all([promise1, promise2, promise3])
  .then((results) => {
    console.log(results); // массив результатов в ИСХОДНОМ порядке
  })
  .catch((error) => {
    console.log("Хотя бы один отклонён:", error); // fail-fast
  });
```

**Особенности:**

- Все промисы запускаются **параллельно** (одновременно), а не по очереди.
- `.then()` срабатывает, только когда **все** промисы завершились успешно.
- Результат — массив, где порядок элементов соответствует порядку **в исходном массиве**, а не порядку фактического завершения.
- **Fail-fast**: если хотя бы один промис отклоняется — весь `Promise.all` немедленно переходит в `rejected`, не дожидаясь остальных. Остальные промисы продолжают выполняться "в фоне", но их результат уже никуда не попадёт.

```javascript
function task(id, delay) {
  return new Promise((resolve) => {
    setTimeout(() => resolve(`результат-${id}`), delay);
  });
}

Promise.all([task(1, 300), task(2, 100), task(3, 200)])
  .then((results) => console.log(results));
  // ["результат-1", "результат-2", "результат-3"]
  // порядок как в массиве, а НЕ по скорости завершения (task(2) был быстрее всех)
```

---

## 1.8 Другие методы группы Promise

### `Promise.allSettled()` — дождаться всех, независимо от результата

```javascript

Promise.allSettled([promise1, promise2, promise3]).then((results) => {
  console.log(results);
  // [
  //   { status: "fulfilled", value: ... },
  //   { status: "rejected", reason: ... },
  //   { status: "fulfilled", value: ... }
  // ]
});
```

В отличие от `Promise.all`, **не** прерывается при первой же ошибке — ждёт завершения всех промисов и для каждого возвращает объект со статусом (`fulfilled`/`rejected`) и результатом (`value`/`reason`). Полезно, когда важны результаты всех операций, даже если часть из них упала.

### `Promise.race()` — первый завершившийся побеждает

```javascript
Promise.race([promise1, promise2]).then((result) => {
  console.log("Первый завершившийся:", result);
});
```

Срабатывает, как только **любой** из промисов завершается — неважно, успешно или с ошибкой. Часто используется для реализации таймаутов:

```javascript
function withTimeout(promise, ms) {
  const timeout = new Promise((_, reject) =>
    setTimeout(() => reject("Timeout!"), ms)
  );
  return Promise.race([promise, timeout]);
}
```

### `Promise.any()` — первый УСПЕШНЫЙ побеждает

```javascript
Promise.any([promise1, promise2, promise3])
  .then((result) => console.log("Первый успешный:", result))
  .catch((error) => console.log("Все отклонены:", error));
```

В отличие от `race`, игнорирует отклонённые промисы и ждёт первый **успешный**. Отклоняется (`AggregateError`), только если **все** промисы отклонены.

### Сравнительная таблица

|Метод|Ждёт|Успех при|Отказ при|
|---|---|---|---|
|`Promise.all`|все|все успешны|хотя бы один отклонён|
|`Promise.allSettled`|все|всегда (никогда не reject)|—|
|`Promise.race`|первый settled|первый успешен|первый отклонён|
|`Promise.any`|первый успешный|хотя бы один успешен|все отклонены|

---

## 1.9 async/await — синтаксический сахар поверх промисов

```javascript
async function loadUser() {
  const user = await getUser();       // ждём результат, не блокируя программу
  console.log(user.name);
  const orders = await getOrders(user.id);
  console.log(orders);
}
```

**Ключевые правила:**

- `async` перед функцией — тогда внутри можно использовать `await`.
- `await` "распаковывает" промис — ждёт его завершения и возвращает результат, но не блокирует весь поток выполнения программы (другие задачи продолжают ждать своей очереди через event loop).
- **`async`-функция всегда возвращает промис**, даже если внутри нет явного `return` промиса:

```javascript
async function getValue() { return 42; }
console.log(getValue()); // Promise {<pending>}, а НЕ просто 42
getValue().then((v) => console.log(v)); // 42
```

- **Код внутри `async`-функции выполняется синхронно вплоть до первого `await`** — частая ловушка: думать, что вся функция сразу асинхронна.

### Обработка ошибок — через `try/catch`

```javascript
async function loadUser() {
  try {
    const user = await getUser();
    console.log(user);
  } catch (error) {
    console.log("Ошибка:", error);
  }
}
```

`try/catch` ловит как явные `throw`, так и отклонённые (`reject`) промисы, обёрнутые в `await`.

### async/await + Promise.all — для параллельности

Если использовать `await` последовательно — операции выполнятся одна за другой (медленнее):

```javascript
const a = await taskA(); // ждём A
const b = await taskB(); // только потом начинаем B
```

Чтобы выполнить параллельно, но всё равно дождаться обоих через await:

```javascript
const [a, b] = await Promise.all([taskA(), taskB()]);
```

---

## 1.10 Event Loop и промисы — микротаски

`.then()`, `.catch()`, `.finally()`, а также код после `await` — это всегда **микротаски**. Они выполняются:

1. После того как весь текущий синхронный код закончился
2. **Раньше** любых макротасков (`setTimeout` и т.д.), даже если те были "поставлены в очередь" раньше по тексту кода
3. Очередь микротасков опустошается **полностью**, включая новые микротаски, появившиеся в процессе — только после этого берётся следующая макротаска

```javascript
console.log("1");
setTimeout(() => console.log("2"), 0);
Promise.resolve().then(() => console.log("3"));
console.log("4");
// Порядок: 1, 4, 3, 2
```

---
## Запомни правило:

`catch` **не заканчивает цепочку**.

Он может:

### 1. Бросить ошибку дальше

```javascript
.catch(() => {
    throw new Error();
})
```

цепочка остаётся `rejected`

---

### 2. Вернуть значение

```javascript
.catch(() => {
    return 100;
})
```

цепочка становится `fulfilled`

Идём дальше:

```javascript
.catch(err => {
    console.log(err);
    return 5;
})
```

Ошибка ловится.

Вывод:

```bash
error
```

Но теперь самое важное.

`catch` возвращает значение:

```javascript
return 5;
```

А любой `return` внутри `then/catch` создаёт новый Promise:

```javascript
Promise.resolve(5)
```

То есть цепочка становится снова:

```javascript
fulfilled: 5
```

---
# Теперь разберем цепочку

Код:

```javascript
Promise.resolve(1)
  .then((value) => {
    console.log(value);
    return 2;
  })
  .then((value) => {
    console.log(value);
    throw new Error("error");
  })
  .catch((error) => {
    console.log(error.message);
  })
  .then((value) => {
    console.log(value);
  });
```

Вывод в консоль:

```javascript
1
2
error
undefined
```

---

Почему?

Первый `then`:

```javascript
console.log(value);
```

value:

```javascript
1
```

Возвращаем:

```javascript
return 2;
```

Следующий `then` получает:

```javascript
2
```

Вывод:

```javascript
2
```

Дальше:

```javascript
throw new Error("error")
```

Цепочка переходит в:

```javascript
catch
```

Вывод:

```javascript
error
```

Но `catch` ничего не возвращает.

Значит следующий `then` получает:

```javascript
undefined
```

Поэтому:

```javascript
undefined
```

---
## 1.11 Итоговая карта терминов

|Понятие|Что это|
|---|---|
|`Promise`|объект-обещание будущего результата|
|`resolve(value)`|успешное завершение|
|`reject(error)`|завершение с ошибкой|
|`.then(fn)`|обработка успеха|
|`.catch(fn)`|обработка ошибки|
|`.finally(fn)`|выполняется всегда|
|Цепочка `.then().then()`|следующий `.then()` ждёт `return` из предыдущего|
|`Promise.all`|все параллельно, fail-fast|
|`Promise.allSettled`|все параллельно, без fail-fast|
|`Promise.race`|первый завершившийся (успех или ошибка)|
|`Promise.any`|первый успешный|
|`async function`|всегда возвращает промис|
|`await`|ждёт результат промиса, не блокируя программу|