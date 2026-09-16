# ЧАСТЬ 5. ВСТРОЕННЫЕ REACT HOOKS

## 5.1 `useState` — локальное состояние компонента

```jsx
const [count, setCount] = useState(0);

setCount(count + 1);              // прямое новое значение
setCount((prev) => prev + 1);       // функциональное обновление — БЕЗОПАСНЕЕ, если новое значение зависит от старого
```

⚠️ Функциональная форма (`prev => ...`) особенно важна при нескольких быстрых последовательных вызовах `setState` в одном событии — гарантирует использование **актуального** значения, а не "устаревшего" из замыкания текущего рендера.

## 5.2 `useEffect` — побочные эффекты

```jsx
useEffect(() => {
  console.log("Выполняется после каждого рендера, если нет массива зависимостей");
});

useEffect(() => {
  console.log("Выполняется ТОЛЬКО один раз, после первого монтирования");
}, []); // пустой массив зависимостей

useEffect(() => {
  console.log("Выполняется при монтировании И при каждом изменении userId");

  return () => {
    console.log("Функция очистки — выполняется ПЕРЕД следующим эффектом и при размонтировании");
  };
}, [userId]);
```

**Массив зависимостей** — определяет, после каких изменений эффект должен перезапуститься. Частая ошибка — забыть указать реально используемую внутри эффекта переменную в массиве зависимостей (ESLint-плагин `eslint-plugin-react-hooks` специально ловит такие случаи — упоминали ранее в конспекте про ESLint).

## 5.3 `useLayoutEffect` — синхронная версия `useEffect`

```jsx
useLayoutEffect(() => {
  // выполняется СИНХРОННО, ПОСЛЕ обновления DOM, но ДО того, как браузер отрисует изменения на экране
}, []);
```

Отличие от `useEffect`: `useEffect` выполняется **асинхронно**, уже после того, как браузер отрисовал кадр — `useLayoutEffect` выполняется **до** отрисовки, блокируя её. Используется редко, в основном когда нужно **синхронно** измерить/поменять DOM (например, позиционирование тултипа по реальным размерам элемента), чтобы избежать заметного "мигания" (visual flicker) между старым и новым положением.

## 5.4 `useContext` — чтение контекста (разбирали в Части 2)

```jsx
const theme = useContext(ThemeContext);
```

## 5.5 `useReducer` — состояние со сложной логикой обновления

```jsx
function reducer(state, action) {
  switch (action.type) {
    case "increment": return { count: state.count + 1 };
    case "decrement": return { count: state.count - 1 };
    case "reset": return { count: 0 };
    default: throw new Error("Неизвестное действие");
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  return (
    <>
      <p>{state.count}</p>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
    </>
  );
}
```

Предпочтительнее `useState`, когда: следующее состояние сложным образом зависит от предыдущего, обновлений состояния много и они логически связаны, или нужно передать функцию обновления глубоко вниз без прокидывания множества отдельных `setState`-функций (часто комбинируется с Context — паттерн "reducer + context" вместо тяжёлых внешних стейт-менеджеров).

## 5.6 `useMemo` — мемоизация вычисленного значения

```jsx
const expensiveResult = useMemo(() => {
  return computeExpensiveValue(a, b); // пересчитается ТОЛЬКО если a или b изменились
}, [a, b]);
```

Кэширует результат **вычисления**, чтобы не пересчитывать его на каждом рендере, если входные данные не изменились. Используется для действительно затратных вычислений или для сохранения **ссылочной стабильности** объекта/массива между рендерами (важно, например, для `value` в Context Provider, как разбирали в Части 2).

## 5.7 `useCallback` — мемоизация функции

```jsx
const handleClick = useCallback(() => {
  console.log(count);
}, [count]); // новая функция создаётся только при изменении count
```

По сути `useCallback(fn, deps)` эквивалентен `useMemo(() => fn, deps)` — просто мемоизирует саму **функцию-ссылку**, а не результат её выполнения. Критически важен, когда функция передаётся как `prop` дочернему компоненту, обёрнутому в `React.memo` (см. Часть 6) — без мемоизации функция создавалась бы заново на каждом рендере родителя, и `React.memo` не смог бы предотвратить лишний перерендер ребёнка (ссылка на функцию всегда была бы "новой").

## 5.8 `useRef` (разбирали в Части 4 [[Refs]]})

## 5.9 `useId` — генерация уникальных ID (React 18+)

```jsx
function FormField() {
  const id = useId();
  return (
    <>
      <label htmlFor={id}>Имя</label>
      <input id={id} />
    </>
  );
}
```

Генерирует стабильный уникальный ID, безопасный для использования в связке с SSR (серверным рендерингом) — обычный `Math.random()` дал бы разные значения на сервере и на клиенте, что приводило бы к ошибкам **гидратации** (hydration mismatch).

## 5.10 `useTransition` и `useDeferredValue` — React 18, приоритизация обновлений

```jsx
import { useState, useTransition } from 'react';

function SearchPage() {
  const [query, setQuery] = useState("");
  const [isPending, startTransition] = useTransition();

  function handleChange(e) {
    setQuery(e.target.value); // СРОЧНОЕ обновление — поле ввода должно реагировать мгновенно

    startTransition(() => {
      // НЕсрочное обновление — можно прервать, если придёт более важное
      searchAndSetResults(e.target.value);
    });
  }

  return (
    <>
      <input value={query} onChange={handleChange} />
      {isPending && <Spinner />}
      <SearchResults query={query} />
    </>
  );
}
```

`useTransition` помечает обновление состояния как **низкоприоритетное (non-urgent)** — React может отрисовать более срочные обновления (например, саму печать в поле ввода) первыми, не дожидаясь завершения "тяжёлого" обновления (например, перерисовки большого списка результатов поиска). `isPending` — булев флаг, показывающий, что переход ещё в процессе.

```jsx
import { useDeferredValue } from 'react';

function SearchResults({ query }) {
  const deferredQuery = useDeferredValue(query); // "отстающая" версия значения
  const results = useMemo(() => search(deferredQuery), [deferredQuery]);
  return <List items={results} />;
}
```

`useDeferredValue` — похожая идея, но без явного `startTransition`: React может отобразить компонент со **старым** значением чуть дольше, пока не будет готово более срочное обновление, а затем "догнать" актуальное значение, когда будет время.

## 5.11 `useSyncExternalStore` — подписка на внешние (не-React) источники состояния

```jsx
function useOnlineStatus() {
  return useSyncExternalStore(
    (callback) => {
      window.addEventListener('online', callback);
      window.addEventListener('offline', callback);
      return () => {
        window.removeEventListener('online', callback);
        window.removeEventListener('offline', callback);
      };
    },
    () => navigator.onLine // getSnapshot — как получить текущее значение
  );
}
```

Официальный, безопасный способ подписаться на изменения источника данных **вне** React (браузерный API, сторонняя библиотека состояния) с гарантией корректной работы при Concurrent Rendering (React 18) — обычная подписка через `useEffect` + `useState` в конкурентном режиме теоретически может привести к рассинхронизации (tearing).

## 5.12 Хуки React 19 — кратко (подробнее в Части 8)

```
useActionState  — управление состоянием формы вместе с Server/Client Actions
useFormStatus    — статус ближайшей родительской формы (отправляется ли сейчас)
useOptimistic     — оптимистичное обновление UI до подтверждения от сервера
use               — универсальное чтение промисов/контекста, в том числе условно
```