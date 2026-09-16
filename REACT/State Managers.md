# ЧАСТЬ 2. STATE MANAGERS (МЕНЕДЖЕРЫ СОСТОЯНИЯ)

## 2.1 Зачем нужны менеджеры состояния

React (и подобные библиотеки) из коробки даёт `useState`/`useContext` для управления состоянием — но при росте приложения возникают проблемы:

1. **Prop drilling** — глубокая передача пропсов через много уровней (Context частично решает, но не идеально — см. ниже).
2. **Избыточные перерендеры** — Context перерендеривает **всех** подписчиков при любом изменении `value`, даже если конкретному компоненту нужен только маленький кусочек состояния.
3. **Сложная логика обновлений**, завязанная на несколько источников данных одновременно.
4. **Синхронизация с сервером** — кэширование, повторные запросы, инвалидация — отдельный класс задач (см. 2.7).

>**State manager** — библиотека, предоставляющая более структурированный, оптимизированный и предсказуемый способ хранения и обновления состояния приложения, часто вне дерева React-компонентов ("снаружи" React).

## 2.2 Redux — классика индустрии

### Основные концепции

```javascript
// 1. Store — единое хранилище всего состояния приложения
// 2. Action — обычный объект, описывающий, ЧТО произошло
{ type: 'counter/incremented', payload: 1 }

// 3. Reducer — чистая функция, вычисляющая НОВОЕ состояние на основе старого и action
function counterReducer(state = { value: 0 }, action) {
  switch (action.type) {
    case 'counter/incremented':
      return { value: state.value + action.payload };
    default:
      return state;
  }
}

// 4. Dispatch — единственный способ ИЗМЕНИТЬ состояние — отправить action
store.dispatch({ type: 'counter/incremented', payload: 1 });
```

### Redux Toolkit (RTK) — современный стандартный способ писать Redux

Классический Redux требовал очень много "шаблонного" (boilerplate) кода — RTK значительно упрощает работу:

```javascript
// counterSlice.js
import { createSlice } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    incremented: (state, action) => {
      state.value += action.payload; // выглядит как МУТАЦИЯ, но Immer под капотом делает immutable-обновление
    },
    reset: (state) => {
      state.value = 0;
    },
  },
});

export const { incremented, reset } = counterSlice.actions;
export default counterSlice.reducer;
```

```javascript
// store.js
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './counterSlice';

export const store = configureStore({
  reducer: { counter: counterReducer },
});
```

```jsx
// App.jsx
import { Provider, useSelector, useDispatch } from 'react-redux';
import { store } from './store';
import { incremented } from './counterSlice';

function Counter() {
  const count = useSelector((state) => state.counter.value); // подписка ТОЛЬКО на нужный кусок состояния
  const dispatch = useDispatch();

  return (
    <button onClick={() => dispatch(incremented(1))}>
      Счёт: {count}
    </button>
  );
}

function App() {
  return (
    <Provider store={store}> {/* оборачивает всё приложение, даёт доступ к store через хуки */}
      <Counter />
    </Provider>
  );
}
```

**Ключевое отличие от React Context по производительности:** `useSelector` подписывает компонент только на **конкретную часть** состояния — компонент перерендерится, только если именно **эта** часть изменилась, а не при любом изменении всего store (в отличие от базового Context, где меняется весь `value` целиком).

### Принципы Redux

- **Единый источник истины (Single Source of Truth)** — всё состояние приложения хранится в одном объекте store.
- **Состояние доступно только для чтения (Read-Only State)** — единственный способ изменить его — задиспатчить action, никогда не мутировать напрямую.
- **Изменения выполняются чистыми функциями (Reducers)** — reducer не должен иметь побочных эффектов, обращаться к сети/времени/случайным числам — только чистое вычисление нового состояния из старого + action.

### Middleware — обработка побочных эффектов (например, асинхронных запросов)

```javascript
// thunk (встроен в Redux Toolkit по умолчанию)
export const fetchUser = (id) => async (dispatch) => {
  dispatch({ type: 'user/loading' });
  const response = await fetch(`/api/users/${id}`);
  const data = await response.json();
  dispatch({ type: 'user/loaded', payload: data });
};
```

Reducer'ы обязаны быть чистыми — вся асинхронная логика (запросы к серверу) выносится в **middleware** (thunk — самый распространённый вариант, также существует Redux Saga для более сложных сценариев побочных эффектов через генераторы).

## 2.3 Zustand — минималистичная альтернатива

```javascript
import { create } from 'zustand';

const useCounterStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  reset: () => set({ count: 0 }),
}));

function Counter() {
  const count = useCounterStore((state) => state.count); // подписка только на count
  const increment = useCounterStore((state) => state.increment);

  return <button onClick={increment}>Счёт: {count}</button>;
}
```

**Особенности:**

- Никакого `Provider`, никакого boilerplate с actions/reducers/dispatch — состояние и логика его изменения определяются в одном месте.
- Компонент подписывается на нужный "срез" состояния через селектор-функцию — перерендер происходит только при изменении именно этого среза.
- Store существует **вне** React-дерева — к нему можно обращаться и из обычного (не-React) JS-кода.
- Значительно меньше кода и концепций для изучения по сравнению с Redux — популярный выбор для средних проектов, где полноценный Redux избыточен.

## 2.4 MobX — реактивный подход

```javascript
import { makeAutoObservable } from 'mobx';
import { observer } from 'mobx-react-lite';

class CounterStore {
  count = 0;
  constructor() {
    makeAutoObservable(this); // автоматически делает свойства "наблюдаемыми"
  }
  increment() {
    this.count += 1; // прямая мутация — MobX сам отслеживает изменение
  }
}

const counterStore = new CounterStore();

const Counter = observer(() => { // observer автоматически подписывает компонент на используемые observable
  return <button onClick={() => counterStore.increment()}>Счёт: {counterStore.count}</button>;
});
```

**Философия отличается от Redux:** MobX построен на **реактивности** и разрешает **прямую мутацию** состояния (в отличие от Redux, где мутация запрещена принципиально) — под капотом MobX отслеживает, какие именно observable-свойства **реально читаются** внутри каждого `observer`-компонента, и перерендеривает только те компоненты, чьи зависимости изменились, автоматически, без ручного написания селекторов.

## 2.5 Jotai / Recoil — атомарный подход

```javascript
import { atom, useAtom } from 'jotai';

const countAtom = atom(0); // "атом" — минимальная единица состояния

function Counter() {
  const [count, setCount] = useAtom(countAtom);
  return <button onClick={() => setCount((c) => c + 1)}>Счёт: {count}</button>;
}
```

**Идея атомарного подхода:** вместо одного большого объекта состояния (как в Redux) состояние разбивается на множество маленьких, независимых "атомов" — компонент подписывается только на конкретные атомы, которые ему нужны, что даёт естественную, мелкогранулированную оптимизацию перерендеров без ручных селекторов.

```javascript
// Производные (derived) атомы — вычисляются на основе других атомов
const doubledCountAtom = atom((get) => get(countAtom) * 2);
```

## 2.6 Effector — событийно-ориентированный реактивный подход

**Effector** — стейт-менеджер, построенный вокруг явных **событий (events)** и **эффектов (effects)** как основных строительных блоков, с полностью реактивным графом вычислений, где связи между данными объявляются декларативно и разрешаются автоматически.

### Основные примитивы

```javascript
import { createStore, createEvent, createEffect } from 'effector';
import { useUnit } from 'effector-react';

// Store — контейнер состояния (аналог store в Redux, но их может быть много, каждый независим)
const $count = createStore(0); // соглашение об именовании: сторы начинаются с $

// Event — описывает НАМЕРЕНИЕ что-то изменить (аналог action в Redux, но вызывается напрямую как функция)
const incremented = createEvent();
const reset = createEvent();

// Связываем store с событиями через .on() — декларативное описание, КАК store реагирует на событие
$count
  .on(incremented, (state) => state + 1)
  .reset(reset);
```

```jsx
function Counter() {
  const count = useUnit($count);            // подписка на store
  const onIncrement = useUnit(incremented);    // получаем функцию-диспетчер события

  return <button onClick={onIncrement}>Счёт: {count}</button>;
}
```

### Effect — для асинхронных операций (аналог thunk в Redux, но с встроенным управлением состоянием запроса)

```javascript
const fetchUserFx = createEffect(async (id) => {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
});

const $user = createStore(null).on(fetchUserFx.doneData, (_, user) => user);
const $isLoading = fetchUserFx.pending; // готовый store состояния загрузки — не нужно писать вручную!

fetchUserFx(1); // запуск эффекта
```

`createEffect` автоматически создаёт связанные с ним события жизненного цикла (`.pending`, `.done`, `.fail`, `.doneData`, `.failData`) — не нужно вручную диспатчить `loading`/`success`/`error` actions, как в классическом Redux thunk-паттерне.

### `sample` — декларативное связывание событий и сторов между собой

```javascript
import { sample } from 'effector';

sample({
  clock: incremented,        // когда произошло ЭТО событие...
  source: $count,               // ...взять текущее значение ЭТОГО стора...
  filter: (count) => count < 10,  // ...если условие истинно...
  target: someOtherEvent,           // ...вызвать вот это событие/эффект
});
```

`sample` — центральный механизм Effector для построения сложных связей между разными частями состояния без ручной подписки через `useEffect`-подобные конструкции — весь граф зависимостей между событиями/сторами/эффектами описывается декларативно при инициализации приложения.

### Философия Effector — в чём отличие от Redux/Zustand/MobX

- **Не привязан к React** — Effector-сторы и события существуют полностью независимо от UI-фреймворка; `effector-react` — лишь один из адаптеров (есть и для Vue, Solid, vanilla JS).
- **Никаких reducers с `switch/case`** — обновления описываются точечно, через `.on()` для каждой пары "store + событие" отдельно, а не одной большой функцией-редьюсером на весь стор.
- **Явное разделение "намерения" (event) и "эффекта" (effect)** — синхронные и асинхронные операции принципиально разные типы сущностей, каждая со своим набором вспомогательных инструментов.
- **Строгая типизация "из коробки"** — библиотека изначально проектировалась с прицелом на TypeScript, выведение типов работает без дополнительной ручной аннотации в большинстве случаев.

**Когда выбирают Effector:** проекты, где важна максимальная предсказуемость и тестируемость бизнес-логики в отрыве от UI, большие enterprise-приложения (популярен в частности в русскоязычном сообществе, ряд крупных финтех/e-commerce проектов используют его как основной стейт-менеджер), сценарии со сложными, ветвящимися цепочками асинхронных операций, где `sample` даёт более декларативное описание, чем вложенные thunk'и.
## 2.7 Отдельная категория — Server State менеджеры (TanStack Query, SWR)

**Важное концептуальное разделение**, о котором часто забывают: **Client State** (тема/язык интерфейса, открыта ли модалка, значение поля формы) и **Server State** (данные, полученные с сервера — список пользователей, товары) — это **разные по природе** виды состояния, и использовать Redux/Zustand для второго — часто избыточно и приводит к ручному написанию логики кэширования, инвалидации, повторных запросов, которая уже решена специализированными инструментами.

### TanStack Query (ранее React Query) — подробнее

**TanStack Query** — библиотека, специализирующаяся конкретно на управлении **асинхронным серверным состоянием**: запрос, кэширование, фоновое обновление, инвалидация. Название "React Query" осталось в народе, но библиотека фреймворк-агностична — есть официальные адаптеры под React, Vue, Solid, Svelte.

```jsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

function UserProfile({ userId }) {
  const { data, isLoading, error } = useQuery({
    queryKey: ['user', userId],       // уникальный ключ запроса — используется для кэша и инвалидации
    queryFn: () => fetch(`/api/users/${userId}`).then((r) => r.json()),
    staleTime: 60_000,                    // сколько мс данные считаются "свежими" (не будут перезапрошены автоматически)
  });

  if (isLoading) return <p>Загрузка...</p>;
  if (error) return <p>Ошибка</p>;
  return <p>{data.name}</p>;
}
```

### Мутации — изменение данных на сервере

```jsx
function EditUserForm({ userId }) {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: (newData) =>
      fetch(`/api/users/${userId}`, {
        method: 'PATCH',
        body: JSON.stringify(newData),
      }),
    onSuccess: () => {
      // инвалидируем кэш — TanStack Query автоматически ЗАНОВО запросит актуальные данные
      queryClient.invalidateQueries({ queryKey: ['user', userId] });
    },
  });

  return (
    <button onClick={() => mutation.mutate({ name: 'Новое имя' })}>
      {mutation.isPending ? 'Сохранение...' : 'Сохранить'}
    </button>
  );
}
```

**Что TanStack Query берёт на себя автоматически:**

- Кэширование результатов по `queryKey`, переиспользование между компонентами без дублирования запросов
- Дедупликация — если два компонента одновременно запрашивают одни и те же данные, реальный сетевой запрос уходит только один раз
- Фоновое обновление "устаревших" данных (`stale-while-revalidate` стратегия) — показать закэшированное сразу, попутно проверить актуальность на сервере
- Автоматические повторные попытки при сетевых сбоях с экспоненциальной задержкой
- Повторный запрос при возврате фокуса на вкладку браузера (`refetchOnWindowFocus`)
- Оптимистичные обновления, отмена устаревших запросов при размонтировании компонента

Весь этот функционал пришлось бы писать вручную поверх Redux/Zustand/Effector, если хранить серверные данные там же, где обычное клиентское состояние.

**Практическая рекомендация, часто встречающаяся в современных проектах:** использовать **TanStack Query/SWR специально для серверных данных**, а Zustand/Redux/Effector (или просто `useState`/Context) — только для настоящего клиентского состояния (UI-состояние, не связанное напрямую с сервером). Комбинация "TanStack Query для сервера + лёгкий клиентский стейт-менеджер (или просто Context) для UI" — частый современный паттерн, вытеснивший практику "тащить абсолютно всё, включая серверные данные, в единый Redux store", распространённую в более ранних подходах.

## 2.8 Когда вообще не нужен отдельный state manager

Если приложение небольшое, состояние в основном локально для конкретных компонентов, и prop drilling не превышает 2-3 уровня — стандартных `useState`/`useReducer`/`useContext` часто вполне достаточно (в духе принципа YAGNI, который разбирали ранее) — добавление полноценного state manager "на будущее", без реальной текущей необходимости, добавляет сложность без соразмерной выгоды.