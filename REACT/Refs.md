# ЧАСТЬ 4. REFS

## 4.1 Зачем нужны Refs

>**Ref (reference)** — способ получить **прямой доступ** к DOM-узлу или к "изменяемому значению", которое **не должно вызывать перерендер** при изменении — то есть выход за пределы обычного декларативного потока React (`props`/`state`), к императивному доступу.

## 4.2 `useRef` — доступ к DOM-элементу

```jsx
import { useRef, useEffect } from 'react';

function TextInput() {
  const inputRef = useRef(null);

  useEffect(() => {
    inputRef.current.focus(); // прямой доступ к DOM-методу после монтирования
  }, []);

  return <input ref={inputRef} />;
}
```

- `useRef(initialValue)` возвращает объект вида `{ current: initialValue }`.
- React сам "подставляет" в `.current` реальный DOM-узел, когда элемент с атрибутом `ref` смонтирован.
- Типичные применения: фокус на поле ввода, измерение размеров элемента, интеграция со сторонними non-React библиотеками (например, инициализация карты, графика на canvas), скролл к элементу.

## 4.3 `useRef` для хранения изменяемого значения (не только DOM)

```jsx
function Timer() {
  const countRef = useRef(0);
  const intervalRef = useRef(null);

  useEffect(() => {
    intervalRef.current = setInterval(() => {
      countRef.current += 1; // изменение НЕ вызывает перерендер компонента!
      console.log(countRef.current);
    }, 1000);

    return () => clearInterval(intervalRef.current); // очистка при размонтировании
  }, []);

  return <p>Смотри консоль</p>;
}
```

**Ключевое отличие `useRef` от `useState`:** изменение `.current` у ref **не вызывает перерендер** компонента, в отличие от `setState`. Это делает `useRef` подходящим для хранения значений, которые нужно "помнить" между рендерами, но которые не должны влиять на визуальный вывод (ID таймера, предыдущее значение пропса, счётчик рендеров для отладки и т.д.).

## 4.4 `forwardRef` (до React 19) — передача ref через кастомный компонент

По умолчанию `ref` **нельзя** передать как обычный `prop` кастомному функциональному компоненту — React обрабатывает `ref` особым образом. До React 19 требовался специальный API:

```jsx
import { forwardRef } from 'react';

const FancyInput = forwardRef((props, ref) => {
  return <input ref={ref} className="fancy-input" {...props} />;
});

function Form() {
  const inputRef = useRef(null);
  return <FancyInput ref={inputRef} />;
}
```

_(В React 19 это изменилось — `ref` можно передавать как обычный prop без `forwardRef`, см. Часть 8.)_

## 4.5 `useImperativeHandle` — контроль над тем, что именно "открывается" через ref

```jsx
import { forwardRef, useImperativeHandle, useRef } from 'react';

const CustomInput = forwardRef((props, ref) => {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => ({
    // наружу через ref доступны ТОЛЬКО эти методы, а не весь DOM-узел целиком
    focus: () => inputRef.current.focus(),
    clear: () => { inputRef.current.value = ""; },
  }));

  return <input ref={inputRef} {...props} />;
});
```

Позволяет ограничить и кастомизировать "публичный API" компонента, доступный через `ref`, вместо того чтобы раскрывать наружу весь реальный DOM-узел.

## 4.6 Когда НЕ стоит использовать Refs

Refs — это осознанный "побег" из декларативной модели React. React-документация явно рекомендует **не использовать refs** для того, что можно выразить через обычные `props`/`state`: например, не стоит через ref напрямую менять текст другого компонента вместо передачи новых пропсов — это ломает предсказуемость однонаправленного потока данных.