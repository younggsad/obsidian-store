# ЧАСТЬ 2. CONTEXT

## 2.1 Проблема, которую решает Context — "prop drilling"

>Без Context данные, нужные глубоко вложенному компоненту, приходится "прокидывать" через `props` на каждом промежуточном уровне, даже если этим уровням сами данные не нужны:

```jsx
function App() {
  const [theme, setTheme] = useState("dark");
  return <Page theme={theme} />;
}
function Page({ theme }) {
  return <Sidebar theme={theme} />; // Page сам не использует theme, просто передаёт дальше
}
function Sidebar({ theme }) {
  return <Widget theme={theme} />; // то же самое
}
function Widget({ theme }) {
  return <div className={theme}>...</div>; // а тут наконец реально нужен
}
```

Это и называется **prop drilling** — "прокачка" пропсов через много уровней компонентов, не использующих их напрямую.

## 2.2 Решение — Context API

```jsx
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext(); // создаём "канал" передачи данных

function App() {
  const [theme, setTheme] = useState("dark");
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Page />
    </ThemeContext.Provider>
  );
}

function Page() {
  return <Sidebar />; // НЕ нужно передавать theme через props
}

function Sidebar() {
  return <Widget />;
}

function Widget() {
  const { theme, setTheme } = useContext(ThemeContext); // берём напрямую, минуя промежуточные уровни
  return <div className={theme}>...</div>;
}
```

## 2.3 Как это работает

- `createContext(defaultValue)` создаёт объект контекста; `defaultValue` используется, только если компонент вызывает `useContext` **без** оборачивающего `Provider` выше по дереву.
- `<Context.Provider value={...}>` — оборачивает часть дерева, "публикуя" значение для всех потомков, независимо от глубины вложенности.
- `useContext(Context)` — читает ближайшее значение `Provider` выше по дереву (если Provider'ов несколько вложенных — берётся **ближайший**).

## 2.4 Важная особенность — Context вызывает перерендер всех потребителей при изменении `value`

```jsx
// ❌ Плохо — новый объект создаётся при КАЖДОМ рендере App, даже если theme не менялся
<ThemeContext.Provider value={{ theme, setTheme }}>
```

Каждый рендер `App` создаёт **новый** объект `{ theme, setTheme }` (даже если `theme` не изменился) — и все компоненты, читающие этот контекст через `useContext`, перерендерятся, потому что React сравнивает `value` по ссылке, а новый объект — это всегда "другая" ссылка.

```jsx
// ✅ Лучше — мемоизировать value, чтобы избежать лишних перерендеров
import { useMemo } from 'react';

function App() {
  const [theme, setTheme] = useState("dark");
  const value = useMemo(() => ({ theme, setTheme }), [theme]);
  return <ThemeContext.Provider value={value}>...</ThemeContext.Provider>;
}
```

## 2.5 Когда использовать Context, а когда — нет

**Хорошо подходит для:** темы оформления, текущего аутентифицированного пользователя, языка интерфейса (i18n), настроек — данных, которые действительно нужны **широкому кругу** компонентов на разных уровнях дерева.

**Плохо подходит для:** часто изменяющихся данных с высокой частотой обновления (например, позиция курсора, значение текстового поля в реальном времени) — из-за перерендера всех потребителей при каждом изменении это может стать узким местом производительности. Для сложного, часто меняющегося глобального состояния чаще выбирают специализированные библиотеки управления состоянием (Redux, Zustand, Jotai) — они умеют избирательно уведомлять только реально подписанные на конкретный кусочек состояния компоненты, в отличие от базового Context.