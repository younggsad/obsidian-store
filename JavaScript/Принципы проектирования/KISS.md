# ЧАСТЬ 3. KISS (Keep It Simple, Stupid)

**Формулировка:** "Делай проще" — из нескольких работающих решений выбирай самое простое и понятное, избегай ненужного усложнения там, где оно не приносит реальной пользы.

### ❌ Плохо — искусственно усложнённое решение простой задачи

```javascript
function isEven(number) {
  return number
    .toString()
    .split("")
    .reverse()
    .join("")
    .split("")
    .map(Number)
    .reduce((acc, digit, index) => {
      if (index === 0) {
        return digit % 2 === 0;
      }
      return acc;
    }, false);
}
```

**Проблема:** это чудовищно избыточный, трудночитаемый способ проверить чётность числа — через превращение в строку, разворот, повторное разбиение на массив, `reduce` с логикой, которую с ходу не поймёшь. Такой код замедляет любую последующую работу с ним — его сложно читать, дебажить, объяснять коллегам.

### ✅ Хорошо — простое, очевидное решение

```javascript
function isEven(number) {
  return number % 2 === 0;
}
```

Одна строка, мгновенно понятная любому, кто хоть немного знает JS. KISS не означает "пиши примитивный код" — он означает "не усложняй без необходимости". Если задача действительно сложная — код может и должен быть сложнее, но не более, чем того требует сама задача.

### Ещё пример — избыточная абстракция там, где она не нужна

```javascript
// ❌ Плохо: фабрика для создания фабрики для создания простого объекта
class ConfigFactoryProviderBuilder {
  static createConfigFactoryProvider() {
    return new ConfigFactoryProvider();
  }
}
class ConfigFactoryProvider {
  provideFactory() {
    return new ConfigFactory();
  }
}
class ConfigFactory {
  createConfig() {
    return { theme: "dark" };
  }
}

const config = ConfigFactoryProviderBuilder
  .createConfigFactoryProvider()
  .provideFactory()
  .createConfig();
```

```javascript
// ✅ Хорошо: то же самое, без искусственной многослойности
const config = { theme: "dark" };
```