# ЧАСТЬ 5. TAILWIND CSS

## 5.1 Общая идея — utility-first подход

>**Tailwind** — CSS-фреймворк, придерживающийся философии **utility-first**: вместо написания собственных CSS-классов с семантическими именами (`.card`, `.button-primary`) стили применяются напрямую в HTML/JSX через большое количество **готовых, атомарных утилитарных классов**, каждый из которых отвечает за одно конкретное CSS-свойство.

### Традиционный подход vs Tailwind

```html
<!-- Традиционный CSS -->
<button class="btn-primary">Отправить</button>
```

```css
.btn-primary {
  background-color: #3498db;
  padding: 8px 16px;
  border-radius: 4px;
  color: white;
  font-weight: bold;
}
```

```html
<!-- Tailwind — стили прямо в разметке, без отдельного CSS-файла для этого компонента -->
<button class="bg-blue-500 px-4 py-2 rounded text-white font-bold">
  Отправить
</button>
```

## 5.2 Установка (для проекта на Vite)

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

```javascript
// tailwind.config.js
export default {
  content: ["./index.html", "./src/**/*.{js,ts,jsx,tsx}"], // ГДЕ искать используемые классы
  theme: {
    extend: {},
  },
  plugins: [],
};
```

```css
/* main.css */
@tailwind base;       /* сброс стилей браузера + базовые стили */
@tailwind components;  /* слой для кастомных переиспользуемых компонентов */
@tailwind utilities;    /* сами utility-классы */
```

**Важная деталь:** Tailwind анализирует поле `content` и включает в итоговый CSS-файл **только те классы**, которые реально встречаются в коде проекта (Just-In-Time компиляция/**Purge**) — это даёт очень маленький итоговый размер CSS в продакшене, несмотря на огромное количество доступных утилит "на бумаге".

## 5.3 Основные категории утилит

```html
<!-- Отступы: p=padding, m=margin; t/r/b/l/x/y — сторона; число = шаг шкалы (обычно ×4px) -->
<div class="p-4 mt-2 mx-auto px-6 py-3"></div>

<!-- Размеры -->
<div class="w-full h-screen max-w-md min-h-0"></div>

<!-- Цвета (текст, фон, граница) — по палитре с оттенками 50-950 -->
<div class="text-gray-700 bg-blue-500 border-red-300"></div>

<!-- Flexbox / Grid -->
<div class="flex items-center justify-between gap-4"></div>
<div class="grid grid-cols-3 gap-2"></div>

<!-- Типографика -->
<p class="text-lg font-bold leading-tight tracking-wide"></p>

<!-- Границы и тени -->
<div class="border rounded-lg shadow-md"></div>

<!-- Позиционирование -->
<div class="absolute top-0 right-0 z-10"></div>
```

## 5.4 Псевдоклассы и состояния через префиксы

```html
<button class="bg-blue-500 hover:bg-blue-700 focus:ring-2 disabled:opacity-50">
  Кнопка
</button>
```

Префиксы `hover:`, `focus:`, `active:`, `disabled:`, `checked:` и т.д. — прямое соответствие обычным CSS-псевдоклассам, которые разбирали в Части 2 этого конспекта, но применённые как модификатор перед именем утилитарного класса.

## 5.5 Адаптивность через префиксы брейкпоинтов

```html
<div class="w-full md:w-1/2 lg:w-1/3">
  <!-- по умолчанию (mobile) — 100% ширины; от md (768px) — 50%; от lg (1024px) — 33% -->
</div>
```

Это прямое воплощение **mobile-first** подхода (разбирали в Части 1): базовый класс без префикса — стиль по умолчанию (для самых маленьких экранов), префиксы `sm:`/`md:`/`lg:`/`xl:`/`2xl:` — переопределения от соответствующей ширины экрана и выше — та же логика, что у `@media (min-width: ...)`.

## 5.6 Тёмная тема

```html
<div class="bg-white dark:bg-gray-900 text-black dark:text-white">
  Контент
</div>
```

Требует настройки в конфиге:

```javascript
export default {
  darkMode: 'class', // переключение через класс "dark" на <html>, а не системную настройку автоматически
};
```

## 5.7 Кастомизация темы

```javascript
// tailwind.config.js
export default {
  theme: {
    extend: {
      colors: {
        brand: '#FF6B35',      // теперь доступен класс bg-brand, text-brand и т.д.
      },
      spacing: {
        '18': '4.5rem',          // теперь доступен p-18, m-18 и т.д.
      },
    },
  },
};
```

## 5.8 Вынесение повторяющихся комбинаций — `@apply`

Когда одна и та же длинная комбинация utility-классов повторяется много раз, можно вынести её в собственный CSS-класс:

```css
.btn-primary {
  @apply bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded;
}
```

```html
<button class="btn-primary">Отправить</button>
```

Это способ соблюсти DRY, не теряя удобства Tailwind — используется умеренно, чтобы не "откатиться" обратно к традиционному подходу написания всего CSS вручную.

## 5.9 Плюсы и минусы подхода utility-first (частый вопрос на собеседовании)

**Плюсы:**

- Не нужно придумывать имена классов (частая проблема традиционного CSS — "как назвать этот блок")
- Стили и разметка в одном месте — не нужно переключаться между HTML/JSX и CSS-файлом
- Маленький итоговый bundle (Purge/JIT убирает неиспользуемые классы)
- Согласованность дизайна "из коробки" через готовую шкалу отступов/цветов/размеров

**Минусы:**

- Разметка становится многословной, длинные списки классов в HTML/JSX могут снижать читаемость
- Требует привыкания к именам утилит (хотя они достаточно интуитивны после освоения)
- Меньше "семантики" в самой разметке на первый взгляд (хотя решается через компонентный подход в React/Vue — сама логика переиспользования переносится на уровень компонентов, а не CSS-классов)