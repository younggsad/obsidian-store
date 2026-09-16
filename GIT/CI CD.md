# ЧАСТЬ 1. ОСНОВЫ — ЧТО ТАКОЕ CI/CD И ИЗ ЧЕГО ОН СОСТОИТ

## 1.1 Определения

- **CI (Continuous Integration)** — практика частой интеграции изменений в общую ветку с автоматической проверкой (сборка, линт, тесты) при каждом изменении.
- **CD (Continuous Delivery)** — код всегда готов к деплою, но финальный запуск в продакшен — вручную (по кнопке).
- **CD (Continuous Deployment)** — деплой в продакшен происходит полностью автоматически после прохождения всех проверок.

## 1.2 Из чего состоит типичный пайплайн

```
Push/PR → Install deps → Lint → Type-check → Unit tests → Build → E2E tests → Deploy (staging) → Deploy (production)
```

Каждый шаг — это **gate** (барьер): если он падает, весь пайплайн останавливается, и разработчик получает сигнал "здесь проблема" раньше, чем код попадёт к другим людям.

## 1.3 Инструменты CI/CD (обзор)

|Инструмент|Особенность|
|---|---|
|**GitHub Actions**|встроен в GitHub, YAML-конфиги в `.github/workflows/`, самый популярный выбор для проектов на GitHub|
|**GitLab CI/CD**|встроен в GitLab, `.gitlab-ci.yml` в корне репозитория|
|**CircleCI**|облачный, независимый от хостинга репозитория|
|**Jenkins**|self-hosted, максимально гибкий, но требует администрирования сервера|
|**Vercel/Netlify**|заточены под фронтенд-деплой, автоматический CI/CD "из коробки" при подключении репозитория|

Дальше в гайде — примеры на **GitHub Actions**, как наиболее распространённом варианте.

---

# ЧАСТЬ 2. ПОШАГОВОЕ ПОДКЛЮЧЕНИЕ ПАЙПЛАЙНА

## Шаг 1 — Подготовка проекта

Прежде чем настраивать CI, в `package.json` должны быть определены нужные скрипты — CI будет просто вызывать их:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "lint": "eslint . --ext .ts,.tsx",
    "type-check": "tsc --noEmit",
    "test": "vitest run",
    "test:coverage": "vitest run --coverage",
    "test:e2e": "playwright test"
  }
}
```

Проверь **локально**, что каждая команда работает и завершается с правильным кодом выхода (0 — успех, не 0 — ошибка) — CI ориентируется именно на код выхода, а не на текст в выводе.

## Шаг 1.2 — Линтеры и ESLint

>**Линтер** — инструмент статического анализа кода, который проверяет его на соответствие правилам стиля и находит потенциальные ошибки **до** запуска — просто анализируя текст кода, без его выполнения.

Что линтер обычно ловит:

- Неиспользуемые переменные (`let x = 5` — и нигде не используется)
- Использование необъявленных переменных
- Несогласованный стиль кода (отступы, кавычки, точки с запятой)
- Потенциально опасные конструкции (`==` вместо `===`, забытый `break` в `switch`)
- Нарушения best practices (например, мутация параметров функции)

**ESLint** — самый популярный линтер для JavaScript/TypeScript. Работает через систему **плагинов** и **правил (rules)** — можно включать/выключать конкретные проверки, настраивать их строгость.

### Установка и базовая настройка

```bash
npm install -D eslint
npx eslint --init   # интерактивный мастер настройки
```

Создаётся конфиг (современный формат — `eslint.config.js`, «flat config»):

```javascript
// eslint.config.js
import js from '@eslint/js';
import tseslint from 'typescript-eslint';

export default [
  js.configs.recommended,
  ...tseslint.configs.recommended,
  {
    rules: {
      'no-unused-vars': 'warn',      // предупреждение, не ошибка
      'eqeqeq': 'error',              // требовать === вместо ==
      'no-console': 'warn', // предупреждать про оставленные console.log
      '@typescript-eslint/no-explicit-any': 'error',
    },
  },
];
```

Каждое правило может быть: `'off'` (выключено), `'warn'` (предупреждение, не ломает сборку), `'error'` (ошибка, останавливает CI).

### Запуск

```bash
npx eslint .              # проверить весь проект
npx eslint . --fix         # автоматически исправить то, что можно исправить безопасно
```

### Популярные готовые конфиги-пресеты

- `eslint-config-airbnb` — очень строгий, детальный набор правил
- `eslint-plugin-react`, `eslint-plugin-react-hooks` — правила специально для React (например, проверка зависимостей в `useEffect`)
- `eslint-config-next` — для Next.js проектов

Именно эта настройка стоит за командой `npm run lint`, которая уже указана в `package.json` из Шага 1 и будет вызываться в CI-пайплайне на Шаге 2.

## Шаг 1.6 — Настройка Prettier

>**Prettier** — это **форматтер** кода, а не линтер: он не ищет логические ошибки, а автоматически приводит **стиль** кода к единому виду (отступы, кавычки, длина строк, точки с запятой и т.д.) — убирает споры в команде о том, "как правильно форматировать".

### Установка

```bash
npm install -D prettier
```

### Конфигурация

```json
// .prettierrc
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 80,
  "arrowParens": "always"
}
```

|Опция|Значение|
|---|---|
|`semi`|ставить ли точки с запятой в конце строк|
|`singleQuote`|одинарные кавычки (`'`) вместо двойных (`"`)|
|`tabWidth`|размер отступа в пробелах|
|`trailingComma`|запятая после последнего элемента в массивах/объектах (`"es5"` — там, где это валидно в старых стандартах)|
|`printWidth`|максимальная длина строки, после которой Prettier переносит код|
|`arrowParens`|скобки вокруг единственного параметра стрелочной функции: `(x) => x` vs `x => x`|

### Исключение файлов — `.prettierignore`

```
node_modules
dist
build
*.min.js
```

### Запуск

```bash
npx prettier --write .     # отформатировать ВСЕ файлы проекта
npx prettier --check .      # только проверить, не форматируя (для CI — падает, если что-то не отформатировано)
```

### Связка ESLint + Prettier (важный момент — они могут конфликтовать!)

ESLint и Prettier иногда имеют **пересекающиеся** правила про форматирование (например, оба могут ругаться на кавычки) — это создаёт конфликты. Решение — отключить в ESLint все правила, касающиеся именно форматирования, и отдать эту зону полностью Prettier:

```bash
npm install -D eslint-config-prettier
```

```javascript
// eslint.config.js
import prettierConfig from 'eslint-config-prettier';

export default [
  // ...остальные конфиги
  prettierConfig,  // ВСЕГДА последним в массиве — отключает конфликтующие правила форматирования
];
```

Теперь: **ESLint** отвечает за поиск логических ошибок и code quality, **Prettier** — исключительно за визуальное форматирование, без пересечения зон ответственности.

### Автоматизация — форматирование при каждом коммите (Husky + lint-staged)

```bash
npm install -D husky lint-staged
npx husky init
```

```json
// package.json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": ["eslint --fix", "prettier --write"]
  }
}
```

```bash
echo "npx lint-staged" > .husky/pre-commit
```

Теперь при каждом `git commit` автоматически запускаются ESLint (с автофиксом) и Prettier — только на изменённых файлах, без ручного запуска команд каждый раз. Этот же механизм пригодится позже, в Части 6 (командный workflow), как часть Шага 3 — "Разработка с локальными проверками".

### Интеграция с VS Code — форматирование при сохранении

```json
// .vscode/settings.json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  }
}
```

Требует установленных расширений **Prettier — Code formatter** и **ESLint** в VS Code — тогда форматирование и автофикс происходят автоматически при каждом сохранении файла, без ручного запуска команд.

## Шаг 2 — Создание базового workflow-файла

Создай файл `.github/workflows/ci.yml` (папка и имя файла важны — GitHub Actions ищет конфиги именно там):

```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout код
        uses: actions/checkout@v4

      - name: Установить Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'       # кэширует node_modules между запусками — ускоряет пайплайн

      - name: Установить зависимости
        run: npm ci              # НЕ npm install! ci быстрее и детерминированнее для CI-окружений

      - name: Линтер
        run: npm run lint

      - name: Проверка типов
        run: npm run type-check

      - name: Юнит-тесты
        run: npm run test

      - name: Сборка
        run: npm run build
```

**Важные детали:**

- `npm ci` вместо `npm install` — устанавливает зависимости **строго** по `package-lock.json`, без модификации lock-файла; быстрее и надёжнее для воспроизводимых сборок в CI.
- `on: push` + `on: pull_request` — пайплайн запускается и при прямом пуше в защищённые ветки, и при открытии/обновлении PR.
- Порядок шагов важен: сначала быстрые и дешёвые проверки (lint, type-check), потом более медленные (тесты, сборка) — если что-то ломается на раннем, быстром шаге, не тратим время на дорогие последующие.

## Шаг 3 — Защита ветки `main` (Branch Protection)

На GitHub: `Settings → Branches → Add branch protection rule` для `main`:

- ✅ Require a pull request before merging — запрещает прямой пуш, только через PR
- ✅ Require status checks to pass before merging — выбрать job `build-and-test` из твоего workflow как обязательный
- ✅ Require branches to be up to date before merging — заставляет подтягивать актуальный `main` перед слиянием
- ✅ Require approvals (минимум 1 ревьюер) — код должен быть одобрен коллегой

Это гарантирует: **ни один коммит не попадёт в `main`, если пайплайн не прошёл и код не был проверен человеком.**

## Шаг 4 — Добавление кэширования и параллелизации (оптимизация)

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: 'npm' }
      - run: npm ci
      - run: npm run lint

  type-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: 'npm' }
      - run: npm ci
      - run: npm run type-check

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: 'npm' }
      - run: npm ci
      - run: npm run test
```

Вынос `lint`, `type-check`, `test` в **отдельные jobs** заставляет их выполняться **параллельно** (а не последовательно, как шаги внутри одного job) — экономит реальное время ожидания пайплайна. Минус: каждый job заново устанавливает зависимости (если не настроено разделяемое кэширование/артефакты) — приходится искать баланс между скоростью параллелизации и затратами на повторную установку.

## Шаг 5 — Добавление E2E-тестов (более тяжёлый и медленный шаг)

```yaml
  e2e:
    runs-on: ubuntu-latest
    needs: [test]              # запускать E2E только ПОСЛЕ успешных юнит-тестов
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: 'npm' }
      - run: npm ci
      - name: Установить браузеры Playwright
        run: npx playwright install --with-deps
      - name: Запустить E2E-тесты
        run: npm run test:e2e
      - name: Сохранить отчёт при падении
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
```

`needs: [test]` — явная зависимость между jobs: `e2e` начнётся только после успешного завершения `test`, что экономит ресурсы (не гонять долгие E2E, если сломались даже базовые юнит-тесты).

## Шаг 6 — Настройка CD (автоматический деплой)

```yaml
  deploy-staging:
    runs-on: ubuntu-latest
    needs: [build-and-test, e2e]
    if: github.ref == 'refs/heads/develop'   # деплоим на staging только из ветки develop
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npm run build
      - name: Деплой на Staging
        run: |
          echo "Деплоим на staging сервер..."
          # реальная команда деплоя — зависит от хостинга (Vercel CLI, rsync, docker push и т.д.)
        env:
          DEPLOY_TOKEN: ${{ secrets.STAGING_DEPLOY_TOKEN }}

  deploy-production:
    runs-on: ubuntu-latest
    needs: [build-and-test, e2e]
    if: github.ref == 'refs/heads/main'
    environment: production      # можно настроить обязательное ручное подтверждение для этого environment
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npm run build
      - name: Деплой на Production
        run: echo "Деплоим на production..."
        env:
          DEPLOY_TOKEN: ${{ secrets.PRODUCTION_DEPLOY_TOKEN }}
```

**Секреты** (`secrets.DEPLOY_TOKEN` и т.д.) настраиваются в `Settings → Secrets and variables → Actions` — никогда не хранятся в самом коде/YAML-файле в открытом виде.

`environment: production` в GitHub Actions позволяет настроить **обязательное ручное подтверждение** перед выполнением job — это превращает Continuous Deployment обратно в Continuous Delivery для критичного окружения, если команда хочет сохранить контроль над моментом релиза.

---

# ЧАСТЬ 3. ПРИМЕРЫ ТЕСТОВ ДЛЯ REACT

## 3.1 Инструменты

- **Vitest** или **Jest** — test runner (запуск тестов, ассерты)
- **React Testing Library (RTL)** — рендеринг компонентов и взаимодействие с ними в тестах, с философией "тестируй так, как пользователь взаимодействует со страницей", а не детали реализации

```bash
npm install -D vitest @testing-library/react @testing-library/jest-dom @testing-library/user-event jsdom
```

## 3.2 Настройка (для Vitest + Vite)

```javascript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',       // симуляция DOM в Node.js окружении
    setupFiles: './src/test/setup.ts',
    globals: true,               // позволяет использовать describe/it/expect без импорта
  },
});
```

```typescript
// src/test/setup.ts
import '@testing-library/jest-dom';
```

## 3.3 Пример: простой презентационный компонент

```tsx
// Button.tsx
type ButtonProps = {
  label: string;
  onClick: () => void;
  disabled?: boolean;
};

export function Button({ label, onClick, disabled = false }: ButtonProps) {
  return (
    <button onClick={onClick} disabled={disabled}>
      {label}
    </button>
  );
}
```

```tsx
// Button.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi } from 'vitest';
import { Button } from './Button';

describe('Button', () => {
  it('отображает переданный текст', () => {
    render(<Button label="Отправить" onClick={() => {}} />);
    expect(screen.getByText('Отправить')).toBeInTheDocument();
  });

  it('вызывает onClick при клике', async () => {
    const handleClick = vi.fn();               // мок-функция, отслеживает вызовы
    const user = userEvent.setup();

    render(<Button label="Отправить" onClick={handleClick} />);
    await user.click(screen.getByText('Отправить'));

    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('не вызывает onClick, если кнопка disabled', async () => {
    const handleClick = vi.fn();
    const user = userEvent.setup();

    render(<Button label="Отправить" onClick={handleClick} disabled />);
    await user.click(screen.getByText('Отправить'));

    expect(handleClick).not.toHaveBeenCalled();
  });
});
```

**Ключевые принципы RTL:**

- `screen.getByText(...)`, `getByRole(...)`, `getByLabelText(...)` — поиск элементов так, как их видит/находит пользователь (по тексту, по роли, по лейблу), а не по `className`/`id` — тест меньше ломается при рефакторинге вёрстки.
- `userEvent` — предпочтительнее, чем `fireEvent`, потому что более реалистично симулирует настоящее взаимодействие пользователя (фокус, последовательность событий).

## 3.4 Пример: компонент с состоянием (useState)

```tsx
// Counter.tsx
import { useState } from 'react';

export function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Счётчик: {count}</p>
      <button onClick={() => setCount((c) => c + 1)}>Увеличить</button>
    </div>
  );
}
```

```tsx
// Counter.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect } from 'vitest';
import { Counter } from './Counter';

describe('Counter', () => {
  it('увеличивает счётчик при клике', async () => {
    const user = userEvent.setup();
    render(<Counter />);

    expect(screen.getByText('Счётчик: 0')).toBeInTheDocument();

    await user.click(screen.getByText('Увеличить'));
    expect(screen.getByText('Счётчик: 1')).toBeInTheDocument();

    await user.click(screen.getByText('Увеличить'));
    expect(screen.getByText('Счётчик: 2')).toBeInTheDocument();
  });
});
```

## 3.5 Пример: асинхронный компонент (загрузка данных)

```tsx
// UserProfile.tsx
import { useEffect, useState } from 'react';

type User = { id: number; name: string };

export function UserProfile({ userId }: { userId: number }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then((res) => res.json())
      .then((data) => {
        setUser(data);
        setLoading(false);
      });
  }, [userId]);

  if (loading) return <p>Загрузка...</p>;
  return <p>Имя: {user?.name}</p>;
}
```

```tsx
// UserProfile.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { UserProfile } from './UserProfile';

describe('UserProfile', () => {
  beforeEach(() => {
    global.fetch = vi.fn(() =>
      Promise.resolve({
        json: () => Promise.resolve({ id: 1, name: 'Аня' }),
      })
    ) as unknown as typeof fetch;
  });

  it('показывает загрузку, затем данные пользователя', async () => {
    render(<UserProfile userId={1} />);

    expect(screen.getByText('Загрузка...')).toBeInTheDocument();

    // waitFor ждёт, пока асинхронное обновление не отразится в DOM
    await waitFor(() => {
      expect(screen.getByText('Имя: Аня')).toBeInTheDocument();
    });
  });
});
```

`vi.fn()` (или `jest.fn()` в Jest) создаёт мок-функцию — здесь используется, чтобы **подменить** реальный `fetch` на тестовый, не делающий настоящих сетевых запросов, чтобы тест был быстрым, детерминированным и не зависел от внешнего сервера.

## 3.6 Пример: тестирование кастомного хука

```tsx
// useCounter.ts
import { useState, useCallback } from 'react';

export function useCounter(initial = 0) {
  const [count, setCount] = useState(initial);
  const increment = useCallback(() => setCount((c) => c + 1), []);
  const reset = useCallback(() => setCount(initial), [initial]);
  return { count, increment, reset };
}
```

```tsx
// useCounter.test.ts
import { renderHook, act } from '@testing-library/react';
import { describe, it, expect } from 'vitest';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  it('начинается с переданного значения', () => {
    const { result } = renderHook(() => useCounter(5));
    expect(result.current.count).toBe(5);
  });

  it('увеличивает счётчик', () => {
    const { result } = renderHook(() => useCounter(0));

    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(1);
  });
});
```

`renderHook` — специальная утилита для тестирования хуков отдельно от компонентов. `act(...)` оборачивает действия, вызывающие обновление состояния React, чтобы тест дождался применения всех обновлений перед проверкой.

## 3.7 Error Boundary — обработка ошибок в React

>**Error Boundary** — специальный React-компонент, который **перехватывает JavaScript-ошибки**, возникающие в дереве его дочерних компонентов во время рендеринга, в методах жизненного цикла и в конструкторах, и вместо падения всего приложения показывает запасной UI ("fallback").

**Важное ограничение:** Error Boundary — это **классовый** компонент (на момент написания в React нет хук-эквивалента для этого механизма) — он должен реализовывать `static getDerivedStateFromError()` и/или `componentDidCatch()`.

```tsx
// ErrorBoundary.tsx
import { Component, ReactNode } from 'react';

type Props = {
  children: ReactNode;
  fallback: ReactNode;
};

type State = {
  hasError: boolean;
};

export class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false };

  static getDerivedStateFromError(): State {
    // вызывается во время рендеринга после ошибки — обновляет state для показа fallback UI
    return { hasError: true };
  }

  componentDidCatch(error: Error, info: { componentStack: string }) {
    // вызывается ПОСЛЕ рендера — сюда удобно писать логирование (например, в Sentry)
    console.error('Поймана ошибка:', error, info.componentStack);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback;
    }
    return this.props.children;
  }
}
```

### Использование

```tsx
<ErrorBoundary fallback={<p>Что-то пошло не так. Попробуйте обновить страницу.</p>}>
  <UserProfile userId={1} />
</ErrorBoundary>
```

Если внутри `UserProfile` (или любого его потомка) произойдёт необработанное исключение во время рендера — вместо "белого экрана смерти" (краха всего приложения) пользователь увидит fallback, а остальная часть страницы (вне `ErrorBoundary`) продолжит нормально работать.

### Что Error Boundary НЕ ловит

- Ошибки в обработчиках событий (`onClick` и т.д.) — их нужно ловить обычным `try/catch` внутри самого обработчика
- Ошибки в асинхронном коде (`setTimeout`, промисы) — тоже нужен `try/catch`/`.catch()`
- Ошибки серверного рендеринга (SSR)
- Ошибки в самом Error Boundary

```jsx
// ❌ Error Boundary это НЕ поймает
function Component() {
  const handleClick = () => {
    throw new Error("Ошибка в обработчике");
  };
  return <button onClick={handleClick}>Клик</button>;
}
```

### Практический совет

Обычно ставят несколько Error Boundary на разных уровнях приложения — один "глобальный" вокруг всего `App` (последний рубеж защиты), и несколько локальных вокруг независимых виджетов/секций страницы — так падение одного виджета не обрушивает всю страницу целиком.

### Тест для Error Boundary

```tsx
// ErrorBoundary.test.tsx
import { render, screen } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { ErrorBoundary } from './ErrorBoundary';

// Компонент, который намеренно "падает" при рендере — нужен только для теста
function BuggyComponent(): never {
  throw new Error('Тестовая ошибка рендера');
}

describe('ErrorBoundary', () => {
  it('показывает children, если ошибок нет', () => {
    render(
      <ErrorBoundary fallback={<p>Ошибка</p>}>
        <p>Всё хорошо</p>
      </ErrorBoundary>
    );

    expect(screen.getByText('Всё хорошо')).toBeInTheDocument();
  });

  it('показывает fallback при ошибке в дочернем компоненте', () => {
    // React логирует ошибку в консоль по умолчанию — подавляем шум в выводе теста
    const spy = vi.spyOn(console, 'error').mockImplementation(() => {});

    render(
      <ErrorBoundary fallback={<p>Что-то пошло не так</p>}>
        <BuggyComponent />
      </ErrorBoundary>
    );

    expect(screen.getByText('Что-то пошло не так')).toBeInTheDocument();
    expect(screen.queryByText('Всё хорошо')).not.toBeInTheDocument();

    spy.mockRestore();
  });
});
```

`vi.spyOn(console, 'error')` здесь используется не для проверки самого вызова, а чтобы **подавить** ожидаемый вывод ошибки в консоль во время теста (React и `componentDidCatch` намеренно логируют пойманные ошибки) — иначе вывод тестов будет засорён "ожидаемым шумом", который на самом деле не является проблемой.

---

# ЧАСТЬ 4. ПРИМЕРЫ ТЕСТОВ ДЛЯ TYPESCRIPT (ЧИСТАЯ ЛОГИКА)

## 4.1 Настройка (Vitest, без React)

```bash
npm install -D vitest typescript
```

```json
// package.json
{
  "scripts": {
    "test": "vitest run"
  }
}
```

## 4.2 Пример: тестирование чистой функции

```typescript
// utils/calculateDiscount.ts
export function calculateDiscount(total: number): number {
  if (total >= 5000) return 20;
  if (total >= 2000) return 10;
  if (total >= 500) return 5;
  return 0;
}
```

```typescript
// utils/calculateDiscount.test.ts
import { describe, it, expect } from 'vitest';
import { calculateDiscount } from './calculateDiscount';

describe('calculateDiscount', () => {
  it('возвращает 20% для суммы от 5000', () => {
    expect(calculateDiscount(5000)).toBe(20);
    expect(calculateDiscount(10000)).toBe(20);
  });

  it('возвращает 10% для суммы от 2000 до 4999', () => {
    expect(calculateDiscount(2000)).toBe(10);
    expect(calculateDiscount(4999)).toBe(10);
  });

  it('возвращает 0% для суммы меньше 500', () => {
    expect(calculateDiscount(100)).toBe(0);
    expect(calculateDiscount(0)).toBe(0);
  });

  // Тестирование граничных значений (edge cases) — важная практика
  it('корректно обрабатывает граничные значения', () => {
    expect(calculateDiscount(499)).toBe(0);
    expect(calculateDiscount(500)).toBe(5);
    expect(calculateDiscount(1999)).toBe(5);
    expect(calculateDiscount(4999)).toBe(10);
  });
});
```

**Хорошая практика:** тестировать не только "счастливый путь" (typical case), но и **граничные значения** (boundary values) — именно там чаще всего прячутся баги off-by-one.

## 4.3 Пример: тестирование класса с типами

```typescript
// ShoppingCart.ts
type CartItem = {
  id: string;
  price: number;
  quantity: number;
};

export class ShoppingCart {
  private items: CartItem[] = [];

  addItem(item: CartItem): void {
    const existing = this.items.find((i) => i.id === item.id);
    if (existing) {
      existing.quantity += item.quantity;
    } else {
      this.items.push(item);
    }
  }

  removeItem(id: string): void {
    this.items = this.items.filter((i) => i.id !== id);
  }

  getTotal(): number {
    return this.items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  }

  getItemCount(): number {
    return this.items.reduce((sum, item) => sum + item.quantity, 0);
  }
}
```

```typescript
// ShoppingCart.test.ts
import { describe, it, expect, beforeEach } from 'vitest';
import { ShoppingCart } from './ShoppingCart';

describe('ShoppingCart', () => {
  let cart: ShoppingCart;

  beforeEach(() => {
    cart = new ShoppingCart(); // свежий экземпляр перед КАЖДЫМ тестом — тесты изолированы друг от друга
  });

  it('добавляет новый товар', () => {
    cart.addItem({ id: '1', price: 100, quantity: 2 });
    expect(cart.getTotal()).toBe(200);
    expect(cart.getItemCount()).toBe(2);
  });

  it('увеличивает количество при повторном добавлении того же товара', () => {
    cart.addItem({ id: '1', price: 100, quantity: 1 });
    cart.addItem({ id: '1', price: 100, quantity: 2 });
    expect(cart.getItemCount()).toBe(3);
  });

  it('удаляет товар из корзины', () => {
    cart.addItem({ id: '1', price: 100, quantity: 1 });
    cart.addItem({ id: '2', price: 50, quantity: 1 });
    cart.removeItem('1');
    expect(cart.getTotal()).toBe(50);
  });

  it('возвращает 0 для пустой корзины', () => {
    expect(cart.getTotal()).toBe(0);
    expect(cart.getItemCount()).toBe(0);
  });
});
```

`beforeEach` создаёт новый экземпляр `ShoppingCart` перед каждым тестом — важный принцип **изоляции тестов**: результат одного теста никогда не должен влиять на другой.

## 4.4 Проверка типов как часть CI (отдельно от тестов)

```bash
tsc --noEmit    # только проверка типов, без генерации файлов — идеально для CI-шага
```

Это не "тест" в привычном смысле, но такой же обязательный gate в пайплайне — TypeScript-ошибки должны блокировать мердж не менее строго, чем упавшие юнит-тесты.

---

# ЧАСТЬ 5. ПРИМЕРЫ ТЕСТОВ ДЛЯ NODE.JS (BACKEND)

## 5.1 Инструменты

- **Jest** или **Vitest** — тест-раннер
- **Supertest** — для тестирования HTTP-эндпоинтов Express/Fastify-приложений без реального поднятия сервера на порту

```bash
npm install -D jest supertest @types/jest @types/supertest ts-jest
```

## 5.2 Пример: тестирование Express-роута

```typescript
// app.ts
import express from 'express';

export const app = express();
app.use(express.json());

app.get('/api/health', (req, res) => {
  res.status(200).json({ status: 'ok' });
});

app.post('/api/users', (req, res) => {
  const { name, email } = req.body;

  if (!name || !email) {
    return res.status(400).json({ error: 'Имя и email обязательны' });
  }

  res.status(201).json({ id: 1, name, email });
});
```

```typescript
// app.test.ts
import request from 'supertest';
import { describe, it, expect } from 'vitest'; // или '@jest/globals' для Jest
import { app } from './app';

describe('GET /api/health', () => {
  it('возвращает статус ok', async () => {
    const response = await request(app).get('/api/health');

    expect(response.status).toBe(200);
    expect(response.body).toEqual({ status: 'ok' });
  });
});

describe('POST /api/users', () => {
  it('создаёт пользователя при корректных данных', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ name: 'Аня', email: 'anna@example.com' });

    expect(response.status).toBe(201);
    expect(response.body).toMatchObject({ name: 'Аня', email: 'anna@example.com' });
  });

  it('возвращает 400 при отсутствии обязательных полей', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ name: 'Аня' }); // email отсутствует

    expect(response.status).toBe(400);
    expect(response.body.error).toBeDefined();
  });
});
```

`supertest` позволяет тестировать HTTP-роуты **без реального запуска сервера на сетевом порту** — напрямую работает с Express-приложением "в памяти", что делает тесты быстрыми и не зависящими от занятости портов.

## 5.3 Пример: тестирование с моком базы данных (изоляция от реальной БД)

```typescript
// userService.ts
type Database = {
  findUser(id: number): Promise<{ id: number; name: string } | null>;
};

export class UserService {
  constructor(private db: Database) {} // Dependency Injection — БД передаётся снаружи

  async getUserName(id: number): Promise<string> {
    const user = await this.db.findUser(id);
    if (!user) throw new Error('Пользователь не найден');
    return user.name;
  }
}
```

```typescript
// userService.test.ts
import { describe, it, expect, vi } from 'vitest';
import { UserService } from './userService';

describe('UserService', () => {
  it('возвращает имя пользователя', async () => {
    const mockDb = {
      findUser: vi.fn().mockResolvedValue({ id: 1, name: 'Аня' }),
    };

    const service = new UserService(mockDb);
    const name = await service.getUserName(1);

    expect(name).toBe('Аня');
    expect(mockDb.findUser).toHaveBeenCalledWith(1);
  });

  it('выбрасывает ошибку, если пользователь не найден', async () => {
    const mockDb = {
      findUser: vi.fn().mockResolvedValue(null),
    };

    const service = new UserService(mockDb);

    await expect(service.getUserName(999)).rejects.toThrow('Пользователь не найден');
  });
});
```

Благодаря **Dependency Injection** (принцип DIP из SOLID, который разбирали ранее) тест вообще не обращается к настоящей базе данных — используется мок-объект, реализующий тот же контракт (`findUser`). Тест быстрый, детерминированный, не требует поднятой БД в CI-окружении.

## 5.4 Виды тестов и их место в пайплайне — пирамида тестирования

```
        /\
       /  \    E2E-тесты (мало, медленные, дорогие, но проверяют всё                                                                 целиком)
      /----\
     /      \  Интеграционные тесты (среднее количество — проверяют                                                  взаимодействие модулей)
    /--------\
   /          \   Юнит-тесты (много, быстрые, дешёвые — проверяют                                   отдельные функции/модули изолированно)
  /------------\
```

- **Юнит-тесты** — тестируют одну функцию/класс/компонент в изоляции (примеры выше с `calculateDiscount`, `ShoppingCart`, `Button`)
- **Интеграционные тесты** — проверяют взаимодействие нескольких модулей вместе (пример с Express-роутом через `supertest` — уже ближе к интеграционному, так как задействует весь стек обработки запроса)
- **E2E (end-to-end)** — имитируют реального пользователя, кликающего по настоящему запущенному приложению в браузере (Playwright, Cypress)

```typescript
// Пример E2E-теста на Playwright
import { test, expect } from '@playwright/test';

test('пользователь может залогиниться и увидеть дашборд', async ({ page }) => {
  await page.goto('https://staging.example.com/login');

  await page.fill('input[name="email"]', 'test@example.com');
  await page.fill('input[name="password"]', 'password123');
  await page.click('button[type="submit"]');

  await expect(page).toHaveURL(/dashboard/);
  await expect(page.getByText('Добро пожаловать')).toBeVisible();
});
```

**Правило пирамиды:** юнит-тестов должно быть **много** (быстро выполняются, легко локализуют проблему), интеграционных — **умеренно**, E2E — **немного** (медленные, хрупкие, но покрывают критичные пользовательские сценарии целиком). Неправильный перекос (например, всё через E2E) сильно замедляет CI-пайплайн и усложняет поддержку тестов.

---

# ЧАСТЬ 6. ПОСЛЕДОВАТЕЛЬНОСТЬ РАБОТЫ В КОМАНДЕ (END-TO-END WORKFLOW)

Полный цикл — от получения задачи до попадания кода в продакшен.

## Шаг 1 — Получение задачи

Задача берётся из трекера (Jira, Linear, GitHub Issues), обычно уже с описанием, критериями приёмки (acceptance criteria). Разработчик берёт задачу в работу, переводит статус в "In Progress".

## Шаг 2 — Создание ветки

```bash
git checkout main
git pull                              # убедиться, что main актуален
git checkout -b feature/JIRA-123-add-user-avatar
```

Ветка называется по соглашению команды, часто включает номер задачи из трекера — упрощает отслеживание, что и зачем менялось.

## Шаг 3 — Разработка с локальными проверками

В процессе работы — регулярные коммиты, локальный запуск тестов и линтера **до** пуша:

```bash
npm run lint
npm run type-check
npm run test
```

**Pre-commit hooks (через Husky + lint-staged)** — автоматизация этого шага, чтобы нельзя было закоммитить код, не прошедший базовые проверки:

```json
// package.json
{
  "lint-staged": {
    "*.{ts,tsx}": ["eslint --fix", "prettier --write"]
  }
}
```

```bash
npx husky init
echo "npx lint-staged" > .husky/pre-commit
```

Теперь при каждом `git commit` автоматически запускается линтер/форматтер **только на изменённых файлах** — быстро, и не даёт закоммитить явно некорректный код.

## Шаг 4 — Push и открытие Pull Request

```bash
git add .
git commit -m "feat: добавлена загрузка аватара пользователя"
git push -u origin feature/JIRA-123-add-user-avatar
```

На GitHub/GitLab — открытие PR с:

- Понятным описанием: что сделано, зачем, как проверить
- Ссылкой на задачу в трекере
- Скриншотами/видео, если менялся UI

## Шаг 5 — Автоматический запуск CI

PR автоматически запускает пайплайн (`lint`, `type-check`, `test`, `build`, возможно `e2e`). Разработчик следит за статусом — если что-то упало, дорабатывает и пушит исправление в ту же ветку (пайплайн перезапускается автоматически).

## Шаг 6 — Code Review

Коллеги оставляют комментарии. Возможные исходы:

- **Approve** — можно сливать
- **Request Changes** — нужны правки перед слиянием
- **Comment** — вопросы/предложения без блокировки слияния

Автор вносит правки по замечаниям, пушит новые коммиты — ревьюеры проверяют повторно.

## Шаг 7 — Слияние (Merge)

После одобрения и зелёного CI — слияние в `main`/`develop` (в зависимости от принятой стратегии ветвления). Часто используется **Squash and Merge** — все коммиты фичи объединяются в один чистый коммит в основной ветке.

```bash
# После слияния — удаление отработавшей ветки
git branch -d feature/JIRA-123-add-user-avatar
git push origin --delete feature/JIRA-123-add-user-avatar
```

## Шаг 8 — Автоматический деплой на Staging

Слияние в `develop` (или `main`, в зависимости от модели) триггерит CD-часть пайплайна — автоматический деплой на staging-окружение.

## Шаг 9 — Тестирование на Staging (QA)

QA-инженер (или сам разработчик/продакт) проверяет функциональность на staging — окружении, максимально похожем на продакшен, но недоступном реальным пользователям.

## Шаг 10 — Релиз в Production

В зависимости от модели:

- **Continuous Deployment** — автоматически, сразу после успешного прохождения всех проверок
- **Continuous Delivery** — по кнопке, обычно с созданием тега релиза:

```bash
git checkout main
git tag -a v1.4.0 -m "Релиз 1.4.0: добавлена загрузка аватара"
git push origin v1.4.0
```

Тег триггерит отдельный deploy-workflow, специфичный для production-окружения.

## Шаг 11 — Мониторинг после релиза

После деплоя — отслеживание метрик и логов (Sentry для ошибок, Grafana/DataDog для метрик производительности) в течение некоторого времени, чтобы вовремя заметить регрессии.

## Шаг 12 — При проблеме — Hotfix или Rollback

```bash
git checkout main
git checkout -b hotfix/JIRA-456-critical-bug
# исправление
# ускоренный процесс ревью и мерджа для критичных багов
```

Или откат к предыдущей стабильной версии (в зависимости от инфраструктуры — повторный деплой предыдущего тега, откат Docker-образа и т.д.).

---

## 6.1 Итоговая схема всего цикла

```
Задача из трекера
      ↓
git checkout -b feature/...
      ↓
Разработка + локальные тесты + pre-commit hooks
      ↓
Push → Pull Request
      ↓
CI: lint → type-check → test → build → (e2e)
      ↓
Code Review (approve / request changes)
      ↓
Merge (squash) в main/develop
      ↓
CD: автодеплой на Staging
      ↓
QA-проверка на Staging
      ↓
Релиз в Production (авто или по кнопке/тегу)
      ↓
Мониторинг → (при проблеме) Hotfix/Rollback
```

---

# ИТОГОВАЯ СВОДКА

|Раздел|Ключевое|
|---|---|
|Подключение пайплайна|`package.json` скрипты → `.github/workflows/ci.yml` → branch protection → параллельные jobs → E2E → деплой|
|React-тесты|Vitest + React Testing Library, тестировать поведение (клики, рендер), не детали реализации|
|TypeScript-тесты|чистые функции/классы, граничные значения, изоляция через `beforeEach`|
|Node.js-тесты|Supertest для роутов, Dependency Injection + моки для изоляции от БД|
|Пирамида тестов|много юнит → умеренно интеграционных → мало E2E|
|Командный workflow|ветка → PR → CI → ревью → merge → CD staging → QA → production → мониторинг|