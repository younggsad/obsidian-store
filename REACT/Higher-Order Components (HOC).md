## Определение

>**Higher-Order Component (HOC)** — это функция, которая принимает компонент и возвращает **новый** компонент с дополнительной функциональностью, "обёрнутой" вокруг исходного.

```typescript
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```

Это не часть React API как таковая — это **паттерн**, который вытекает из того, что React-компоненты — это обычные функции, а функции в JS можно комбинировать друг с другом. Термин пришёл из функционального программирования: **higher-order function** — это функция, которая принимает функцию и/или возвращает функцию. HOC — это higher-order function применительно к React-компонентам.

---

## Зачем нужен — какую проблему решает

HOC решает задачу **переиспользования логики** между разными компонентами, когда эта логика не сводится к переиспользованию UI (для этого хватило бы обычной композиции компонентов).

Типичные примеры такой переиспользуемой логики:

- Подписка на внешний источник данных
- Проверка авторизации перед рендером
- Логирование, аналитика (трекинг рендеров/действий)
- Внедрение дополнительных пропсов (theme, локализация)
- Обработка загрузки/ошибок (loading/error states)

Без HOC пришлось бы **дублировать** одну и ту же логику в каждом компоненте, которому она нужна.

---

## Базовый пример

```jsx
function withLoading(WrappedComponent) {
  return function WithLoadingComponent({ isLoading, ...props }) {
    if (isLoading) {
      return <div>Loading...</div>;
    }
    return <WrappedComponent {...props} />;
  };
}

function UserProfile({ user }) {
  return <div>{user.name}</div>;
}

const UserProfileWithLoading = withLoading(UserProfile);

// Использование:
<UserProfileWithLoading isLoading={false} user={{ name: 'Alice' }} />
```

`withLoading` — это HOC: он принимает `UserProfile` и возвращает новый компонент, который добавляет логику проверки `isLoading` **перед** тем как отрендерить оригинальный компонент.

---

## Пример посложнее — подписка на данные (классический use case до хуков)

```jsx
function withUserData(WrappedComponent) {
  return class extends React.Component {
    state = { user: null, loading: true };

    componentDidMount() {
      fetchUser(this.props.userId).then(user => {
        this.setState({ user, loading: false });
      });
    }

    render() {
      return (
        <WrappedComponent
          {...this.props}
          user={this.state.user}
          loading={this.state.loading}
        />
      );
    }
  };
}

function Profile({ user, loading }) {
  if (loading) return <div>Loading...</div>;
  return <div>{user.name}</div>;
}

const ProfileWithData = withUserData(Profile);
```

Логика загрузки данных инкапсулирована внутри HOC — сам компонент `Profile` остаётся "чистым" и не знает, откуда пришли данные, просто получает их как пропсы.

---

## Ключевые правила и соглашения

### 1. Никогда не мутировать оригинальный компонент внутри HOC

```js
// Неправильно — мутация переданного компонента
function withLog(WrappedComponent) {
  WrappedComponent.prototype.componentDidMount = function() { ... }; // плохо!
  return WrappedComponent;
}

// Правильно — композиция через оборачивание
function withLog(WrappedComponent) {
  return class extends React.Component {
    componentDidMount() { ... }
    render() {
      return <WrappedComponent {...this.props} />;
    }
  };
}
```

### 2. Пропускать (forward) все пропсы, не относящиеся к HOC

```jsx
function withExtra(WrappedComponent) {
  return function (props) {
    return <WrappedComponent {...props} extra="value" />; // спред всех входящих пропсов
  };
}
```

Если HOC "съедает" какой-то проп молча или не прокидывает остальные — это ломает компонент непредсказуемым образом для того, кто его использует.

### 3. Называть отображаемое имя для дебага (displayName)

```jsx
function withLoading(WrappedComponent) {
  function WithLoading(props) { /* ... */ }
  WithLoading.displayName = `WithLoading(${WrappedComponent.displayName || WrappedComponent.name})`;
  return WithLoading;
}
```

Без этого в React DevTools все обёрнутые компоненты будут отображаться как безликие `Component` или `Anonymous`, что усложняет дебаг.

### 4. Не использовать HOC внутри `render()` другого компонента

```jsx
// Неправильно — новый компонент создаётся на КАЖДЫЙ рендер
function App() {
  const EnhancedComponent = withLoading(MyComponent); // плохо!
  return <EnhancedComponent />;
}

// Правильно — HOC применяется один раз, снаружи компонента
const EnhancedComponent = withLoading(MyComponent);
function App() {
  return <EnhancedComponent />;
}
```

Если создавать обёрнутый компонент заново при каждом рендере родителя — React считает это **новым типом компонента** каждый раз, из-за чего весь поддерев размонтируется и монтируется заново на каждый рендер — теряется state, теряется производительность.

---

## Известные проблемы HOC (почему появились хуки)

### 1. "Wrapper hell" — обёртки внутри обёрток

```jsx
export default withRouter(connect(mapStateToProps)(withTheme(withAuth(MyComponent))));
```

При использовании нескольких HOC одновременно дерево компонентов в React DevTools превращается в глубоко вложенную структуру оберток, что усложняет отладку и чтение дерева.

### 2. Неявный источник пропсов

Глядя на компонент `MyComponent`, использующий проп `user`, непонятно **откуда** этот проп взялся — он может приходить от родителя напрямую, а может быть внедрён одним из нескольких HOC в цепочке оборачивания. Приходится "прыгать" по всей цепочке HOC, чтобы понять источник данных.

### 3. Конфликт имён пропсов

Если два разных HOC внедряют проп с одинаковым именем (например, оба хотят передать `data`) — один молча перезатирает другой, и это трудно отследить.

### 4. Статические методы теряются при оборачивании

```js
MyComponent.staticMethod = () => {...};
const Enhanced = withHOC(MyComponent);
Enhanced.staticMethod; // undefined — HOC вернул новый компонент без статических методов оригинала
```

Решается через `hoist-non-react-statics`, но это дополнительная библиотека и лишняя забота разработчика.

---

## HOC vs Hooks — почему хуки во многом заменили HOC

С появлением хуков (React 16.8) большая часть задач, которые раньше решались через HOC, стала решаться проще — через **кастомные хуки (custom hooks)**:

```jsx
// HOC-подход (было)
const ProfileWithData = withUserData(Profile);

// Хук-подход (стало)
function useUserData(userId) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchUser(userId).then(user => {
      setUser(user);
      setLoading(false);
    });
  }, [userId]);

  return { user, loading };
}

function Profile({ userId }) {
  const { user, loading } = useUserData(userId); // логика "внедряется" явно, прямо внутри компонента
  if (loading) return <div>Loading...</div>;
  return <div>{user.name}</div>;
}
```

**Преимущества кастомных хуков перед HOC для этой задачи:**

- Нет дополнительной вложенности в дереве компонентов (никакого "wrapper hell").
- Явно видно, откуда взялись данные — прямо в теле компонента, а не в невидимой обёртке снаружи.
- Можно использовать **несколько** хуков одновременно без конфликтов имён (в отличие от нескольких HOC).
- Не нужно беспокоиться о статических методах, displayName, forwarding пропсов.

**HOC всё ещё используется** в некоторых случаях — например, `React.memo` и `React.forwardRef` — это, по сути, встроенные в React HOC. Также HOC иногда применяют, когда логика должна **менять сам рендер** компонента снаружи (например, условно не рендерить компонент вообще — как в примере с `withLoading`), а не просто предоставлять данные — это чуть сложнее выразить через один хук, хотя тоже возможно через условный рендеринг внутри самого компонента.

---

## Частые вопросы на собеседовании

**В: Чем HOC отличается от обычного компонента, принимающего компонент как проп (render props)?** 
О: Render props — альтернативный паттерн, где вместо оборачивания компонент передаёт **функцию** как проп (часто как `children`), которая решает, что рендерить:

```jsx
<DataProvider render={data => <Profile data={data} />} />
```

Это тоже способ переиспользования логики без наследования, решает похожие задачи что и HOC, но не создаёт обёрнутый компонент — используется композиция через сам JSX. Как и HOC, во многом вытеснен хуками.

**В: Можно ли комбинировать несколько HOC?** 
О: Да, через вложенные вызовы (`withA(withB(withC(Component)))`), но это и есть источник "wrapper hell" — именно поэтому современный React рекомендует кастомные хуки для новой логики.

**В: Является ли `connect` из Redux HOC?** 
О: Да, классический пример HOC из реальной библиотеки — `connect(mapStateToProps)(MyComponent)` оборачивает компонент, внедряя пропсы из Redux store.