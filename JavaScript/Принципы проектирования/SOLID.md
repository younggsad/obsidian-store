# ЧАСТЬ 1. SOLID

>**SOLID** — пять принципов объектно-ориентированного дизайна, сформулированные Робертом Мартином. Аббревиатура:

- **S** — Single Responsibility Principle (принцип единственной ответственности)
- **O** — Open/Closed Principle (принцип открытости/закрытости)
- **L** — Liskov Substitution Principle (принцип подстановки Барбары Лисков)
- **I** — Interface Segregation Principle (принцип разделения интерфейса)
- **D** — Dependency Inversion Principle (принцип инверсии зависимостей)

---

## 1.1 S — Single Responsibility Principle (SRP)

**Формулировка:** у класса/модуля/функции должна быть только **одна причина для изменения** — то есть он должен отвечать только за одну конкретную задачу.

### ❌ Плохо — один класс делает слишком много

```javascript
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }

  // Ответственность 1: хранение данных пользователя
  getName() {
    return this.name;
  }

  // Ответственность 2: валидация
  validateEmail() {
    return /\S+@\S+\.\S+/.test(this.email);
  }

  // Ответственность 3: сохранение в базу
  saveToDatabase() {
    console.log(`Сохраняем ${this.name} в БД...`);
    // логика работы с базой данных
  }

  // Ответственность 4: отправка письма
  sendWelcomeEmail() {
    console.log(`Отправляем письмо на ${this.email}`);
    // логика отправки email
  }
}
```

**Проблема:** если понадобится поменять способ хранения (БД на другую), логику отправки писем (другой email-сервис) или правила валидации — придётся лезть в **один и тот же** класс `User` по совершенно разным причинам. Это увеличивает риск сломать что-то не связанное с текущей задачей, и класс тяжело тестировать изолированно.

### ✅ Хорошо — ответственность разделена

```javascript
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }
}

class UserValidator {
  static validateEmail(user) {
    return /\S+@\S+\.\S+/.test(user.email);
  }
}

class UserRepository {
  save(user) {
    console.log(`Сохраняем ${user.name} в БД...`);
  }
}

class EmailService {
  sendWelcomeEmail(user) {
    console.log(`Отправляем письмо на ${user.email}`);
  }
}

// Использование:
const user = new User("Аня", "anna@example.com");
if (UserValidator.validateEmail(user)) {
  new UserRepository().save(user);
  new EmailService().sendWelcomeEmail(user);
}
```

Теперь каждый класс меняется по **своей отдельной** причине: изменилась логика БД → трогаем только `UserRepository`; поменялся email-провайдер → только `EmailService`. `User` вообще остаётся простым хранилищем данных.

---

## 1.2 O — Open/Closed Principle (OCP)

**Формулировка:** код должен быть **открыт для расширения**, но **закрыт для изменения** — новую функциональность нужно добавлять, не трогая уже написанный и протестированный код.

### ❌ Плохо — приходится редактировать функцию при каждом новом случае

```javascript
function calculateArea(shape) {
  if (shape.type === "circle") {
    return Math.PI * shape.radius ** 2;
  } else if (shape.type === "square") {
    return shape.side ** 2;
  }
  // Появится новая фигура — придётся снова лезть сюда и добавлять else if
}
```

**Проблема:** каждый раз, когда появляется новая фигура (треугольник, прямоугольник...), нужно **редактировать** уже существующую, рабочую функцию `calculateArea` — риск случайно что-то сломать в уже отлаженной логике.

### ✅ Хорошо — расширяем через новые классы, не трогая старые

```javascript
class Circle {
  constructor(radius) {
    this.radius = radius;
  }
  area() {
    return Math.PI * this.radius ** 2;
  }
}

class Square {
  constructor(side) {
    this.side = side;
  }
  area() {
    return this.side ** 2;
  }
}

// Новая фигура — просто добавляем НОВЫЙ класс, ничего не редактируя
class Triangle {
  constructor(base, height) {
    this.base = base;
    this.height = height;
  }
  area() {
    return (this.base * this.height) / 2;
  }
}

function calculateArea(shape) {
  return shape.area(); // работает с ЛЮБОЙ фигурой, у которой есть area()
}

console.log(calculateArea(new Circle(5)));
console.log(calculateArea(new Triangle(4, 6)));
```

Функция `calculateArea` больше никогда не меняется — она "закрыта" для модификации, но система в целом "открыта" для расширения новыми фигурами (это тот же принцип полиморфизма, который разбирали в конспекте по ООП).

---

## 1.3 L — Liskov Substitution Principle (LSP)

**Формулировка:** объекты дочернего класса должны иметь возможность **заменить** объекты родительского класса без нарушения корректности программы — то есть наследник не должен "ломать" ожидания, заложенные в родителе.

### ❌ Плохо — наследник нарушает поведение родителя

```javascript
class Bird {
  fly() {
    console.log("Летит");
  }
}

class Sparrow extends Bird {
  fly() {
    console.log("Воробей летит");
  }
}

class Penguin extends Bird {
  fly() {
    throw new Error("Пингвины не летают!"); // ломает ожидание родителя!
  }
}

function makeBirdFly(bird) {
  bird.fly(); // код рассчитывает, что ЛЮБАЯ Bird умеет летать
}

makeBirdFly(new Sparrow());  // Воробей летит
makeBirdFly(new Penguin());  // ❌ Ошибка! Хотя формально Penguin — это Bird
```

**Проблема:** `Penguin` — технически `Bird` (наследует от него), но нарушает контракт, заложенный в родительском классе ("любая птица умеет летать"). Код, написанный для работы с `Bird`, ломается на конкретном наследнике — это прямое нарушение LSP.

### ✅ Хорошо — правильная иерархия, отражающая реальные возможности

```javascript
class Bird {
  eat() {
    console.log("Птица ест");
  }
}

class FlyingBird extends Bird {
  fly() {
    console.log("Летит");
  }
}

class Sparrow extends FlyingBird {
  fly() {
    console.log("Воробей летит");
  }
}

class Penguin extends Bird { // Penguin НЕ наследует FlyingBird — и это честно
  swim() {
    console.log("Пингвин плывёт");
  }
}

function makeBirdFly(bird) {
  bird.fly();
}

makeBirdFly(new Sparrow()); // работает
// makeBirdFly(new Penguin()); — теперь такой код даже не скомпилируется/
// не будет вызван, потому что Penguin просто не имеет метода fly — ошибка на уровне дизайна, а не в рантайме
```

Иерархия классов теперь честно отражает реальность: `FlyingBird` гарантирует умение летать, а `Penguin`, не наследуя этот класс, не создаёт ложных ожиданий.

---

## 1.4 I — Interface Segregation Principle (ISP)

**Формулировка:** не стоит заставлять объект зависеть от методов, которые он не использует — лучше несколько **маленьких, специализированных** интерфейсов, чем один большой "универсальный".

_(В JS формальных `interface` нет, как в TypeScript/Java — принцип применяется к дизайну объектов/классов и тому, какие методы объект обязан реализовать)_

### ❌ Плохо — один "толстый" интерфейс на все случаи

```javascript
class Worker {
  work() { throw new Error("Не реализовано"); }
  eat() { throw new Error("Не реализовано"); }
  sleep() { throw new Error("Не реализовано"); }
}

class HumanWorker extends Worker {
  work() { console.log("Человек работает"); }
  eat() { console.log("Человек ест"); }
  sleep() { console.log("Человек спит"); }
}

class RobotWorker extends Worker {
  work() { console.log("Робот работает"); }
  eat() { throw new Error("Роботы не едят!"); }   // вынужден реализовать ненужное
  sleep() { throw new Error("Роботы не спят!"); }  // и это тоже
}
```

**Проблема:** `RobotWorker` вынужден "реализовывать" методы `eat()`/`sleep()`, которые ему физически не нужны — просто потому, что так устроен общий родительский класс `Worker`. Это создаёт мусорный, вводящий в заблуждение код.

### ✅ Хорошо — разделённые, специализированные "интерфейсы" (миксины/маленькие классы)

```javascript
const Workable = {
  work() { console.log(`${this.name} работает`); }
};

const Eatable = {
  eat() { console.log(`${this.name} ест`); }
};

const Sleepable = {
  sleep() { console.log(`${this.name} спит`); }
};

class HumanWorker {
  constructor(name) { this.name = name; }
}
Object.assign(HumanWorker.prototype, Workable, Eatable, Sleepable);

class RobotWorker {
  constructor(name) { this.name = name; }
}
Object.assign(RobotWorker.prototype, Workable); // только то, что реально нужно

const robot = new RobotWorker("Робот-1");
robot.work(); // "Робот-1 работает"
// robot.eat() — метода просто нет, и это правильно, а не выброс искусственной ошибки
```

Каждый класс "подключает" только те возможности, которые ему действительно нужны — никаких лишних, фиктивных реализаций.

---

## 1.5 D — Dependency Inversion Principle (DIP)

**Формулировка:** модули верхнего уровня не должны зависеть от модулей нижнего уровня напрямую — оба должны зависеть от **абстракции**. Иными словами: код должен зависеть от "контракта" (что нужно сделать), а не от конкретной реализации (как именно это сделано).

### ❌ Плохо — жёсткая зависимость от конкретной реализации

```javascript
class MySQLDatabase {
  save(data) {
    console.log("Сохраняем в MySQL:", data);
  }
}

class UserService {
  constructor() {
    this.db = new MySQLDatabase(); // жёстко "вшита" конкретная база данных
  }

  createUser(data) {
    this.db.save(data);
  }
}
```

**Проблема:** если понадобится сменить базу данных (например, на MongoDB) или подставить "заглушку" для тестов — придётся редактировать сам класс `UserService`, потому что он **напрямую** создаёт и знает про `MySQLDatabase`.

### ✅ Хорошо — зависимость передаётся извне (Dependency Injection), через общий контракт

```javascript
class MySQLDatabase {
  save(data) {
    console.log("Сохраняем в MySQL:", data);
  }
}

class MongoDatabase {
  save(data) {
    console.log("Сохраняем в MongoDB:", data);
  }
}

class FakeDatabaseForTests {
  save(data) {
    console.log("Тестовое сохранение (ничего реально не пишем):", data);
  }
}

class UserService {
  constructor(database) { // зависимость передаётся СНАРУЖИ, не создаётся внутри
    this.db = database;
  }

  createUser(data) {
    this.db.save(data);
  }
}

// Использование — легко подменить реализацию:
const service1 = new UserService(new MySQLDatabase());
const service2 = new UserService(new MongoDatabase());
const testService = new UserService(new FakeDatabaseForTests());
```

`UserService` теперь не зависит от **конкретной** базы данных — он зависит только от контракта "что-то, у чего есть метод `save()`". Это называется **Dependency Injection (внедрение зависимости)** — практический способ реализации DIP. Такой код гораздо проще тестировать (подставить фейковую БД) и расширять (добавить новый тип хранилища, не трогая `UserService`).