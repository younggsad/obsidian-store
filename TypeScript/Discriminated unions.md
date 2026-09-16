### Проблема, которую решают discriminated unions

Представим union из нескольких типов, описывающих разные "состояния" — например, результат сетевого запроса:

```ts
type RequestState =
  | { status: 'loading' }
  | { status: 'success'; data: string }
  | { status: 'error'; message: string };
```

Если попробовать обратиться к `data` без проверки — TypeScript не даст, потому что не все варианты union имеют это поле:

```ts
function render(state: RequestState) {
  console.log(state.data); // Ошибка: свойство 'data' не существует на типе '{ status: "loading" }'
}
```

### Как это решается через `status` (discriminant / "тег")

Идея discriminated union — у **каждого варианта** union есть **общее поле** (обычно называют `type`, `kind`, `status`) с **буквальным (literal) значением**, уникальным для каждого варианта. TypeScript умеет использовать проверку этого поля через `if`/`switch`, чтобы **сузить (narrow)** тип внутри блока — то есть точно понять, с каким конкретно вариантом union он сейчас работает:

```ts
function render(state: RequestState) {
  if (state.status === 'loading') {
    console.log('Loading...'); // тут TS знает: state — только { status: 'loading' }
  } else if (state.status === 'success') {
    console.log(state.data); // TS точно знает, что здесь есть data — ошибки нет!
  } else if (state.status === 'error') {
    console.log(state.message); // TS точно знает, что здесь есть message
  }
}
```

TypeScript отслеживает проверку `state.status === 'success'` и **автоматически сужает** тип `state` внутри этого блока именно до варианта `{ status: 'success'; data: string }` — поэтому обращение к `state.data` становится безопасным именно **внутри** этого блока, а не во всей функции целиком.

### Ещё удобнее — через `switch` с проверкой полноты (exhaustiveness check)

```ts
function render(state: RequestState) {
  switch (state.status) {
    case 'loading':
      return 'Loading...';
    case 'success':
      return state.data;
    case 'error':
      return state.message;
    default:
      const _exhaustive: never = state; // если забыли обработать вариант — ошибка компиляции!
      return _exhaustive;
  }
}
```

Трюк с `never` в `default` — мощная практическая штука: если позже добавить новый вариант в `RequestState` (например, `{ status: 'idle' }`) и забыть обработать его в `switch`, TypeScript **сам укажет на ошибку компиляции** в этой строке, потому что необработанный вариант больше не сможет присвоиться типу `never`. Это защищает от забытых кейсов при расширении union в будущем.

### Почему это лучше, чем обычный union без общего поля

Без "тега"-дискриминанта TypeScript вынужден был бы проверять **наличие конкретного поля** (`'data' in state`), что работает, но менее надёжно и хуже читается:

```ts
// Работает, но менее явно и легче ошибиться
if ('data' in state) {
  console.log(state.data);
}
```

С discriminated union у тебя **один явный, читаемый признак** (`status`), по которому сразу понятно, какой вариант перед тобой — и это именно тот паттерн, который повсеместно используется в реальных проектах: описание состояний загрузки данных (`loading`/`success`/`error`), состояний форм, событий Redux-экшенов (`{ type: 'ADD_TODO', payload: ... } | { type: 'REMOVE_TODO', payload: ... }`), результатов валидации и так далее.