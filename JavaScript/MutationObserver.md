## 2.1 Общее определение

>**MutationObserver** — встроенный в браузер API, который позволяет асинхронно отслеживать изменения в дереве DOM и реагировать на них через функцию-обработчик, без постоянного ручного опроса (polling) состояния страницы.

- **"Mutation"** — любое изменение DOM: добавление/удаление элемента, изменение атрибута, изменение текста.
- **"Observer"** — объект, подписывающийся на элемент и получающий уведомление о мутациях в момент их появления.

Смысл: _"следи за этим куском DOM, и как только там что-то изменится (неважно, каким кодом — моим или чужим), сообщи мне"_.

**Важно:** реакция `MutationObserver` — это **микротаск**, а не макротаска. Если DOM меняется во время синхронного кода, callback сработает в ближайшую паузу, вместе с `.then()`, но раньше любого `setTimeout`.

---

## 2.2 Базовый синтаксис

```javascript
const targetNode = document.querySelector("#myElement");

const observer = new MutationObserver((mutationsList, observer) => {
  console.log("Что-то изменилось!", mutationsList);
});

observer.observe(targetNode, {
  childList: true,
  attributes: true,
  subtree: true,
});
```

- `new MutationObserver(callback)` — создаёт наблюдателя; ничего не делает, пока не вызван `.observe()`.
- `observer.observe(target, config)` — запускает наблюдение за `target` с настройками `config`.

---

## 2.3 Опции конфигурации

```javascript
const config = {
  childList: true,             // добавление/удаление детей элемента
  attributes: true,             // изменение атрибутов
  characterData: true,          // изменение текста внутри текстовых узлов
  subtree: true,                 // применять всё вышеперечисленное к ВСЕМ вложенным потомкам

  attributeOldValue: true,       // сохранять предыдущее значение атрибута
  characterDataOldValue: true,   // сохранять предыдущий текст
  attributeFilter: ["class", "data-status"], // следить только за конкретными атрибутами
};
```

⚠️ Нужно указать **хотя бы одну** из `childList`, `attributes`, `characterData` — иначе браузер выдаст ошибку.

---

## 2.4 Объект mutation — что приходит в callback

Callback получает **массив** мутаций (могут "пачковаться", если случилось сразу несколько изменений):

```javascript
const observer = new MutationObserver((mutationsList) => {
  mutationsList.forEach((mutation) => {
    switch (mutation.type) {
      case "childList":
        console.log("Добавлено:", mutation.addedNodes.length);
        console.log("Удалено:", mutation.removedNodes.length);
        break;
      case "attributes":
        console.log("Атрибут:", mutation.attributeName);
        console.log("Было:", mutation.oldValue);
        break;
      case "characterData":
        console.log("Текст был:", mutation.oldValue);
        break;
    }
  });
});
```

|Поле|Значение|
|---|---|
|`mutation.type`|`"childList"`, `"attributes"` или `"characterData"`|
|`mutation.target`|DOM-элемент, где произошло изменение|
|`mutation.addedNodes` / `removedNodes`|добавленные/удалённые узлы (для childList)|
|`mutation.attributeName`|имя изменённого атрибута|
|`mutation.oldValue`|предыдущее значение (только если включены `attributeOldValue`/`characterDataOldValue`)|

⚠️ `addedNodes`/`removedNodes` могут содержать не только элементы, но и текстовые узлы — стоит проверять `node.nodeType === Node.ELEMENT_NODE`.

---

## 2.5 Практический пример — автопрокрутка чата

```javascript
const chat = document.querySelector("#chat");

const observer = new MutationObserver((mutationsList) => {
  for (const mutation of mutationsList) {
    if (mutation.type === "childList" && mutation.addedNodes.length > 0) {
      chat.scrollTop = chat.scrollHeight; // автопрокрутка вниз
    }
  }
});

observer.observe(chat, { childList: true });

// Код добавления сообщения вообще НЕ знает о существовании observer —
// они полностью независимы
chat.appendChild(document.createElement("li"));
```

Главная сила API: можно реагировать на изменения DOM, происходящие из совершенно другого места в коде (в том числе из стороннего скрипта, который нельзя изменить).

---

## 2.6 Остановка наблюдения

```javascript
observer.disconnect(); // полностью прекращает наблюдение
```

Важно вызывать, когда наблюдение больше не нужно (элемент удалён, компонент "размонтирован" в SPA) — иначе наблюдатель продолжит работать впустую и может привести к утечкам памяти в долгоживущих приложениях.

`observer.takeRecords()` — возвращает накопленные, но ещё не обработанные записи мутаций (используется редко, обычно перед `disconnect()`, чтобы не потерять "незабранные" изменения).

---

## 2.7 MutationObserver vs альтернативы

|Подход|Когда использовать|
|---|---|
|`addEventListener`|когда сам создаёшь событие и точно знаешь, где его слушать|
|`MutationObserver`|когда DOM меняется "снаружи" (сторонний код/библиотека) и нужно на это отреагировать|
|`setInterval` + проверка|устаревший, неэффективный способ — избегать, если доступен `MutationObserver`|
