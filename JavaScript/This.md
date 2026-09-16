# Маленькая памятка

>`this` — это специальное ключевое слово. Оно указывает на объект, в контексте которого была вызвана функция.

Обычная функция:

> `this` определяется **способом вызова**.

Стрелочная функция:

> `this` определяется **местом создания**.

Если эту разницу ты запомнишь, то 90% вопросов про `this` перестанут быть сложными.

## Разберем первую часть

```javascript
const user = {
  name: "Alex",

  greet() {
    return () => {
      console.log(this.name);
    };
  },
};

const fn = user.greet();

fn();
```

### Что происходит?

Когда вызывается:

```javascript
user.greet();
```

у обычной функции `greet`:

```javascript
this === user
```

Теперь внутри нее создается стрелочная функция:

```javascript
() => {
  console.log(this.name);
}
```

Но мы уже знаем правило:

> **Стрелочная функция не имеет собственного `this`. Она берет его из внешнего лексического окружения в момент создания.**

А внешнее окружение — это выполнение `user.greet()`, где:

```javascript
this === user
```

Поэтому стрелочная функция навсегда "запомнила":

```javascript
this === user
```

Даже если потом:

```javascript
const fn = user.greet();

fn();
```

она выведет:

```javascript
Alex
```

Потому что `this` уже лексически захвачен.

---

## Вторая часть

Теперь меняем код:

```javascript
greet() {
  return function () {
    console.log(this.name);
  };
}
```

Теперь `greet()` возвращает **обычную функцию**.

Когда ты делаешь:

```javascript
const fn = user.greet();

fn();
```

это обычный вызов функции.

Следовательно:

```javascript
this === undefined   // strict mode / ES-модули
```

или глобальный объект в нестрогом режиме.

То есть результат будет:

```javascript
undefined
```

# Что делает `new`?

Допустим есть код:

```javascript
function User(name) {
    this.name = name;
}

const user = new User("Alex");
```

Когда выполняется

```javascript
new User("Alex");
```

движок делает следующее.

### Шаг 1

Создает новый пустой объект.

Условно:

```javascript
const obj = {};
```

---

### Шаг 2

Связывает этот объект с прототипом функции.

Условно:

```javascript
obj.__proto__ = User.prototype;
```

Именно поэтому потом работает:

```javascript
user instanceof User // true
```

и

```javascript
user.sayHello();
```

если метод находится в `User.prototype`.

---

### Шаг 3

Вызывает функцию, подставляя новый объект в качестве `this`.

То есть примерно так:

```javascript
User.call(obj, "Alex");
```

Поэтому внутри конструктора:

```javascript
this.name = name;
```

работает именно с новым объектом.

---

### Шаг 4

Если функция **не вернула объект явно**, оператор `new` возвращает созданный объект.

Получается:

```javascript
return obj;
```

---

## Что будет без `new`?

```javascript
function User(name) {
    this.name = name;
}

const user = User("Alex");
```

В современном JavaScript (ES-модули, strict mode):

```javascript
this === undefined
```

И строка

```javascript
this.name = name;
```

выбросит

```javascript
TypeError
```

Потому что нельзя записывать свойства в `undefined`.

В старом нестрогом режиме `this` стал бы `window` (или `globalThis`), что приводило к случайному созданию глобальных переменных. Именно поэтому сейчас почти весь современный код работает в строгом режиме.

# Первая задача

```javascript
function User(name) {
  this.name = name;

  return {
    name: "Bob",
  };
}

const user = new User("Alex");

console.log(user.name);
```

Ты сказал:

> создался новый объект, но явно говорится вернуть Bob

✅ Верно.

Вывод:

```javascript
Bob
```

---

Почему?

Оператор `new` обычно возвращает созданный объект:

```javascript
{
  name: "Alex"
}
```

Но есть специальное правило:

> Если конструктор явно возвращает **объект**, оператор `new` использует этот объект вместо созданного.

То есть происходит примерно:

```javascript
const obj = {};

User.call(obj, "Alex");

return {
  name: "Bob"
};
```

Так как вернулся объект:

```javascript
{name: "Bob"}
```

именно он становится результатом:

```javascript
user === {name: "Bob"}
```

---

# Вторая задача

```javascript
function User(name) {
  this.name = name;

  return 100;
}

const user = new User("Alex");

console.log(user.name);
```

Вывод:

```javascript
Alex
```

---

Правило `new`:

Если конструктор возвращает:

- объект → использовать его;
- примитив (`string`, `number`, `boolean`, `null`, `undefined`) → игнорировать.

То есть:

```javascript
return 100;
```

просто не учитывается.

Поэтому остается созданный объект:

```javascript
{
  name: "Alex"
}
```
