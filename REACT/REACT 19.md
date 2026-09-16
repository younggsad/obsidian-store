# ЧАСТЬ 8. REACT 19

## 8.1 Actions — упрощение работы с асинхронными операциями и формами

>**Action** — функция (обычная или помеченная `async`), которую можно передать напрямую в `<form action={...}>` или использовать с новыми хуками для работы с состоянием отправки/ошибок/пендинга без ручного управления через `useState`.

```jsx
function ChangeName() {
  async function updateName(formData) {
    const name = formData.get("name");
    await saveNameToServer(name); // может быть Server Action при использовании React Server Components
  }

  return (
    <form action={updateName}>
      <input name="name" />
      <button type="submit">Сохранить</button>
    </form>
  );
}
```

React автоматически обрабатывает pending-состояние, ошибки и оптимистичные обновления форм, обёрнутых в Actions — существенно меньше "ручного" boilerplate-кода по сравнению с классическим `useState` + `onSubmit` + `preventDefault` + `try/catch`.

## 8.2 `useActionState` — состояние формы вместе с Action

```jsx
import { useActionState } from 'react';

function ChangeName() {
  const [error, submitAction, isPending] = useActionState(
    async (previousState, formData) => {
      const name = formData.get("name");
      const result = await updateName(name);
      if (result.error) {
        return result.error; // становится новым "previousState"/error при следующем вызове
      }
      return null;
    },
    null // начальное состояние
  );

  return (
    <form action={submitAction}>
      <input name="name" />
      <button type="submit" disabled={isPending}>Сохранить</button>
      {error && <p>{error}</p>}
    </form>
  );
}
```

Объединяет паттерн `useReducer` + отслеживание pending-состояния специально под сценарий отправки форм/Actions.

## 8.3 `useFormStatus` — статус ближайшей родительской формы

```jsx
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending } = useFormStatus(); // читает статус БЛИЖАЙШЕЙ родительской <form>, без явного prop drilling
  return <button disabled={pending}>{pending ? "Отправка..." : "Отправить"}</button>;
}

function Form() {
  return (
    <form action={someAction}>
      <input name="email" />
      <SubmitButton /> {/* дочерний компонент "знает" о состоянии формы напрямую, без пропсов */}
    </form>
  );
}
```

Полезно именно для переиспользуемых дочерних компонентов формы (кнопка отправки, индикатор), которым не нужно прокидывать `isSubmitting` вручную через `props`.

## 8.4 `useOptimistic` — оптимистичные обновления UI

```jsx
import { useOptimistic } from 'react';

function MessageList({ messages, sendMessage }) {
  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    (state, newMessage) => [...state, { ...newMessage, sending: true }]
  );

  async function formAction(formData) {
    const text = formData.get("message");
    addOptimisticMessage({ text }); // UI обновляется МГНОВЕННО, ещё до ответа сервера
    await sendMessage(text);          // реальный запрос идёт в фоне
  }

  return (
    <>
      {optimisticMessages.map((m, i) => (
        <p key={i} style={{ opacity: m.sending ? 0.5 : 1 }}>{m.text}</p>
      ))}
      <form action={formAction}>
        <input name="message" />
      </form>
    </>
  );
}
```

Позволяет показать пользователю результат **сразу**, до подтверждения сервера (характерный паттерн для чатов, лайков, комментариев) — если запрос в итоге упадёт с ошибкой, React автоматически откатывает оптимистичное состояние обратно к реальному.

## 8.5 `use()` — универсальное чтение промисов и контекста

```jsx
import { use } from 'react';

function UserProfile({ userPromise }) {
  const user = use(userPromise); // "распаковывает" промис прямо в теле компонента, работает с Suspense
  return <p>{user.name}</p>;
}
```

Отличие от `await` в `async`-компонентах (доступно только в Server Components) — `use()` можно вызывать **условно** (внутри `if`, циклов), в отличие от обычных хуков, для которых действует строгое правило "хуки нельзя вызывать условно". `use()` также умеет читать `Context`, как альтернатива `useContext` в определённых сценариях.

## 8.6 `ref` как обычный prop — больше не нужен `forwardRef`

```jsx
// React 19 — ref передаётся как ОБЫЧНЫЙ prop, без forwardRef
function FancyInput({ ref, ...props }) {
  return <input ref={ref} {...props} />;
}
```

Убирает необходимость в `forwardRef` для подавляющего большинства случаев простой передачи ref внутрь кастомного компонента — заметное упрощение API по сравнению с прошлыми версиями (хотя `forwardRef` остаётся доступным для обратной совместимости).

## 8.7 Улучшенная обработка ошибок гидратации

React 19 значительно улучшил читаемость сообщений об ошибках **гидратации** (несовпадение серверного и клиентского рендера при SSR) — вместо общего "Text content does not match" React 19 показывает конкретный diff, что именно не совпало, существенно упрощая отладку SSR-проблем.

## 8.8 Document Metadata — управление `<title>`/`<meta>` прямо из компонентов

```jsx
function BlogPost({ post }) {
  return (
    <article>
      <title>{post.title}</title>          {/* React 19 сам поднимет это в <head> */}
      <meta name="description" content={post.excerpt} />
      <p>{post.content}</p>
    </article>
  );
}
```

Раньше для управления мета-тегами страницы из глубины дерева компонентов требовались сторонние библиотеки (`react-helmet`) — React 19 поддерживает это нативно: теги `<title>`/`<meta>`/`<link>`, отрендеренные где угодно в дереве, React автоматически "поднимает" в `<head>` документа.

## 8.9 React Server Components (RSC) — стабилизация

Компоненты, выполняющиеся **только на сервере**, никогда не попадающие в клиентский JS-бандл — могут напрямую обращаться к базе данных, файловой системе, секретным API-ключам, без создания отдельного backend-эндпоинта. React 19 (в связке с фреймворками вроде Next.js App Router) сделал этот подход основной, официально поддерживаемой моделью, а не экспериментальной.

```jsx
// Server Component (выполняется только на сервере, по умолчанию в поддерживающих фреймворках)
async function ProductList() {
  const products = await db.query("SELECT * FROM products"); // прямой доступ к БД, без API-роута
  return (
    <ul>
      {products.map((p) => <li key={p.id}>{p.name}</li>)}
    </ul>
  );
}
```

## 8.10 Сводка нового в React 19

```
Actions (form action={...})
useActionState — состояние формы + Action
useFormStatus — статус родительской формы без prop drilling
useOptimistic — мгновенный UI-отклик до подтверждения сервера
use() — условное чтение промисов/контекста
ref как обычный prop — без forwardRef
Улучшенные ошибки гидратации
Нативная поддержка Document Metadata (<title>, <meta> из любого компонента)
React Server Components — стабильный статус
```