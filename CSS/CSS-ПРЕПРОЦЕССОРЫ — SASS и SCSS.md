ЧАСТЬ 4. CSS-ПРЕПРОЦЕССОРЫ — SASS/SCSS

4.1 Зачем нужны препроцессоры

>CSS-препроцессор — язык, расширяющий возможности обычного CSS (переменные, вложенность, функции, миксины), который затем компилируется в обычный, валидный CSS, понятный браузеру. Браузеры никогда не выполняют SCSS напрямую — только результат компиляции.

4.2 Sass vs SCSS — в чём разница

>Sass — сам препроцессор/язык в целом. У него есть два синтаксиса:

.sass — старый, отступы вместо фигурных скобок, без точек с запятой (менее популярен сегодня)
.scss (Sassy CSS) — синтаксис, очень похожий на обычный CSS, с фигурными скобками и точками с запятой — практически весь существующий CSS уже является валидным SCSS. Именно этот синтаксис используют в подавляющем большинстве проектов сегодня.

```css
sass// .sass синтаксис (без скобок, через отступы)
.button
  color: blue
  font-size: 16px
```

```css
scss// .scss синтаксис (похож на обычный CSS)
.button {
  color: blue;
  font-size: 16px;
}
```

4.3 Установка и компиляция

```bash
npm install -D sass
```

```bash 
sass styles.scss styles.css          # разовая компиляция
```

```bash
sass --watch styles.scss:styles.css # автоматическая перекомпиляция при изменениях
```

В современных сборщиках (Vite, Webpack) обычно достаточно просто установить пакет sass — импорт .scss-файлов в JS/компоненты обрабатывается автоматически.

4.4 Переменные

```css
scss
$primary-color: #3498db;
$spacing-unit: 8px;
$font-stack: 'Helvetica Neue', sans-serif;

.button {
  background: $primary-color;
  padding: $spacing-unit * 2;
  font-family: $font-stack;
}
```

Компилируется в:

```css
css.button {
  background: #3498db;
  padding: 16px;
  font-family: 'Helvetica Neue', sans-serif;
}
```

Отличие от нативных CSS-переменных (--var): SCSS-переменные существуют только на этапе компиляции и не видны в итоговом CSS вообще (полностью подставляются как значения). Нативные CSS custom properties (--main-color: blue; color: var(--main-color);), наоборот, остаются в итоговом CSS и могут динамически меняться через JavaScript или медиа-запросы в рантайме браузера — это разные механизмы с разными возможностями.

4.5 Вложенность (nesting)

```css
scss.card {
  padding: 16px;
  border: 1px solid gray;

  .title {              // компилируется в .card .title
    font-size: 20px;
  }

  &:hover {                // & означает "родительский селектор целиком" → .card:hover
    box-shadow: 0 0 10px rgba(0,0,0,0.2);
  }

  &.active {                 // → .card.active (без пробела — сам элемент с доп. классом)
    border-color: blue;
  }

  &__icon {                    // популярный паттерн для BEM-методологии → .card__icon
    width: 24px;
  }
}
```

Компилируется в:

```css
css.card { padding: 16px; border: 1px solid gray; }
.card .title { font-size: 20px; }
.card:hover { box-shadow: 0 0 10px rgba(0,0,0,0.2); }
.card.active { border-color: blue; }
.card__icon { width: 24px; }
```

⚠️ Предостережение: избыточная вложенность (более 3 уровней) увеличивает специфичность и делает итоговый CSS трудноуправляемым — общая рекомендация "не углубляйся больше чем на 3 уровня".

4.6 Миксины (mixins) — переиспользуемые блоки стилей

```css
scss@mixin flex-center($direction: row) {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: $direction;
}

.container {
  @include flex-center;
}

.column-container {
  @include flex-center(column); // передаём аргумент, переопределяя значение по умолчанию
}
```

Компилируется в:

```css
css.container {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: row;
}
.column-container {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}
```

Миксины — способ избежать дублирования одного и того же набора свойств (принцип DRY, который разбирали ранее) на уровне стилей.

4.7 Функции

```css
scss@function rem($px, $base: 16px) {
  @return ($px / $base) * 1rem;
}

.title {
  font-size: rem(24px); // → 1.5rem
}
```

В отличие от миксина (который вставляет набор свойств), функция возвращает одно значение, используемое в качестве части свойства.

4.8 Наследование через @extend

```css
scss.btn-base {
  padding: 10px 20px;
  border-radius: 4px;
  border: none;
}

.btn-primary {
  @extend .btn-base;
  background: blue;
  color: white;
}
```

Компилируется так, что .btn-base и .btn-primary объединяются в общий селектор в CSS (а не дублируют свойства) — более "экономно" по итоговому размеру файла, чем миксин, но менее гибко (нельзя передать параметры, как в миксине).

4.9 Партиалы и @use/@import — модульность

```css
scss// _variables.scss (подчёркивание в начале имени = "партиал", не компилируется в отдельный CSS-файл сам по себе)
$primary-color: #3498db;
$spacing: 8px;

scss// main.scss
@use 'variables' as vars;

.button {
  color: vars.$primary-color;
  padding: vars.$spacing;
}
```

@use — современный способ подключения (модульный, с явными пространствами имён через as), пришёл на замену устаревающему @import (который был глобальным и мог приводить к конфликтам имён/повторной компиляции одного файла много раз).

4.10 Циклы и условия (для сложных случаев генерации стилей)

```css
scss@for $i from 1 through 5 {
  .col-#{$i} {                  // #{} — интерполяция, вставка значения переменной в имя селектора/свойства
    width: percentage($i / 5);
  }
}

scss@mixin theme($mode) {
  @if $mode == dark {
    background: black;
    color: white;
  } @else {
    background: white;
    color: black;
  }
}
```

