# ЧАСТЬ 1. COMPONENT (КОМПОНЕНТЫ)

## 1.1 Что такое компонент

>**Компонент** — независимый, переиспользуемый кусок UI, который принимает входные данные (`props`) и возвращает описание того, что должно быть отрисовано (JSX). React-приложение — это дерево вложенных компонентов.

## 1.2 Функциональные vs классовые компоненты

```jsx
// Функциональный (современный стандарт)
function Greeting({ name }) {
  return <h1>Привет, {name}!</h1>;
}

// Классовый (легаси, но всё ещё встречается в старых проектах)
class Greeting extends React.Component {
  render() {
    return <h1>Привет, {this.props.name}!</h1>;
  }
}
```

С появлением хуков (React 16.8, 2019) функциональные компоненты получили доступ к состоянию и жизненному циклу — сегодня классовые компоненты пишутся крайне редко для нового кода (но остаются обязательными для Error Boundary, о котором говорили ранее — хук-эквивалента для этого механизма до сих пор нет).

## 1.3 Props — входные данные компонента

```jsx
function Button({ label, onClick, variant = "primary" }) {
  return <button className={`btn btn-${variant}`} onClick={onClick}>{label}</button>;
}

<Button label="Отправить" onClick={() => console.log("клик")} />
```

**Особенности:**

- `props` — **только для чтения (read-only)**, компонент никогда не должен изменять свои `props` напрямую — это нарушает **однонаправленный поток данных (one-way data flow)**, ключевой принцип React.
- `props.children` — специальное свойство, содержащее всё, что передано **между** открывающим и закрывающим тегом компонента:

```jsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}
<Card><p>Содержимое карточки</p></Card>
```

## 1.4 Composition (композиция) — предпочтительнее наследования

React рекомендует собирать сложный UI через **композицию** компонентов (вложенность, `children`, render props), а не через наследование классов — это прямое применение принципа "composition over inheritance", который разбирали в контексте ООП.

```jsx
function Layout({ header, sidebar, children }) {
  return (
    <div className="layout">
      <header>{header}</header>
      <aside>{sidebar}</aside>
      <main>{children}</main>
    </div>
  );
}

<Layout header={<Header />} sidebar={<Sidebar />}>
  <MainContent />
</Layout>
```

## 1.5 Условный и списочный рендеринг

```jsx
function UserStatus({ isOnline }) {
  return <p>{isOnline ? "В сети" : "Не в сети"}</p>;
}

function Notifications({ count }) {
  return count > 0 && <span className="badge">{count}</span>; // && для условного рендера
}

function UserList({ users }) {
  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li> // key ОБЯЗАТЕЛЕН для списков
      ))}
    </ul>
  );
}
```

⚠️ **`key` в списках** — не просто формальность, а критически важная подсказка для Reconciliation (см. Часть 3) — React использует `key`, чтобы понять, какие элементы списка добавлены/удалены/переставлены местами между рендерами, вместо того чтобы пересоздавать весь список заново. `key` должен быть **стабильным и уникальным** (обычно `id` из данных), а не индекс массива (индекс ломается при вставке/удалении элементов в середине списка — вызывает лишние перерисовки и баги состояния).