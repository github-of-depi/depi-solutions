# CSS — Middle

## Вопросы

- [Как работает CSS Grid?](#как-работает-css-grid)
- [Как работают CSS-анимации и переходы?](#как-работают-css-анимации-и-переходы)
- [Что такое CSS custom properties (переменные)?](#что-такое-css-custom-properties-переменные)
- [Что такое BEM методология?](#что-такое-bem-методология)
- [Что такое stacking context и как он создаётся?](#что-такое-stacking-context-и-как-он-создаётся)
- [Как работает адаптивная вёрстка?](#как-работает-адаптивная-вёрстка)
- [Что такое CSS container queries?](#что-такое-css-container-queries)
- [Что такое схлопывание отступов (margin collapsing)?](#что-такое-схлопывание-отступов-margin-collapsing)
- [Что такое псевдокласс :has()?](#что-такое-псевдокласс-has)
- [Что такое медиафункция prefers-reduced-motion?](#что-такое-медиафункция-prefers-reduced-motion)
- [Desktop-first vs Mobile-first?](#desktop-first-vs-mobile-first)
- [Perfect pixel — что это?](#perfect-pixel--что-это)
- [CSS-спрайты?](#css-спрайты)
- [Ретина-дисплей и ретинизация?](#ретина-дисплей-и-ретинизация)
- [vmax, vmin, vh, vw — когда использовать?](#vmax-vmin-vh-vw--когда-использовать)
- [object-fit и object-position?](#object-fit-и-object-position)
- [text-shadow и box-shadow?](#text-shadow-и-box-shadow)
- [Градиенты (linear-gradient, radial-gradient)?](#градиенты-linear-gradient-radial-gradient)
- [word-wrap, word-break, word-spacing?](#word-wrap-word-break-word-spacing)
- [counter-increment и counter-reset?](#counter-increment-и-counter-reset)
- [background-origin, background-clip, background-size?](#background-origin-background-clip-background-size)
- [@font-face?](#font-face)
- [@charset?](#charset)
- [CSS-методологии: SMACSS, OOCSS, Atomic CSS?](#css-методологии-smacss-oocss-atomic-css)
- [Производительность CSS-селекторов?](#производительность-css-селекторов)
- [CSS Modules и изоляция стилей?](#css-modules-и-изоляция-стилей)
- [Асинхронная загрузка CSS?](#асинхронная-загрузка-css)

---

## Как работает CSS Grid?

Grid — двумерная система раскладки. `grid-template-columns`/`rows` определяют структуру. Дети размещаются автоматически или явно через `grid-column`/`grid-row`. `fr` — дробная единица распределения свободного пространства. `gap` между ячейками. `auto-fill`/`auto-fit` для адаптивных сеток.

```css
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 24px;
}
.hero { grid-column: 1 / -1; } /* на всю ширину */
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работают CSS-анимации и переходы?

`transition` — плавный переход между двумя состояниями при изменении свойства. `animation` + `@keyframes` — многошаговая анимация. Для производительности анимируй только `transform` и `opacity` — они не вызывают layout/paint, обрабатываются на compositor thread.

```css
.btn { transition: transform 0.2s ease, opacity 0.2s; }
.btn:hover { transform: scale(1.05); }

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-8px); }
  to { opacity: 1; transform: translateY(0); }
}
.modal { animation: fadeIn 0.3s ease-out; }
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое CSS custom properties (переменные)?

CSS-переменные — именованные значения, объявленные через `--name: value` и используемые через `var(--name)`. Каскадируются как обычные свойства, могут быть переопределены в любом скоупе. Основа для тем и design tokens.

```css
:root { --color-primary: #3b82f6; --spacing-4: 16px; }
.dark { --color-primary: #60a5fa; }
.btn {
  background: var(--color-primary);
  padding: var(--spacing-4);
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое BEM методология?

BEM (Block__Element--Modifier) — соглашение об именовании CSS-классов для избежания конфликтов и улучшения читаемости. Block — независимый компонент, Element — часть Block, Modifier — вариант.

```css
.card {}                  /* Block */
.card__title {}           /* Element */
.card__button {}          /* Element */
.card__button--primary {} /* Modifier */
.card--featured {}        /* Block Modifier */
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое stacking context и как он создаётся?

Stacking context — независимое 3D-пространство, внутри которого дочерние элементы упорядочиваются по `z-index`. Элементы из разных stacking contexts не конкурируют по `z-index` с друг другом. Создаётся: `position: relative/absolute` + `z-index != auto`, `opacity < 1`, `transform`, `filter`, `will-change`, `isolation: isolate`.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работает адаптивная вёрстка?

Mobile-first подход: базовые стили для мобильных, `min-width` breakpoints для больших экранов. `viewport` meta тег обязателен. CSS Grid/Flexbox с `auto-fill`/`fr` для fluid layouts. Избегать фиксированных ширин.

```css
/* Mobile first */
.container { padding: 16px; }
@media (min-width: 768px) { .container { padding: 32px; max-width: 1200px; } }
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое CSS container queries?

Container queries позволяют применять стили в зависимости от размера контейнера, а не viewport. Решают проблему компонентов, используемых в разных контекстах (сайдбар, основной контент).

```css
.card-wrapper { container-type: inline-size; }
@container (min-width: 400px) {
  .card { display: flex; }
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое схлопывание отступов (margin collapsing)?

Когда вертикальные `margin` двух соседних блочных элементов или родителя и первого/последнего ребёнка соприкасаются, они объединяются в один — равный наибольшему из двух. Это называется схлопыванием.

Схлопывание **не происходит** в: flex-контейнерах, grid-контейнерах, элементах с `overflow` ≠ `visible`, элементах с `display: flow-root` (создающих BFC).

```css
/* Два соседних абзаца: margin 24px + 16px → итого 24px (не 40px) */
p { margin-bottom: 24px; }
p + p { margin-top: 16px; }

/* Отключить схлопывание родитель–ребёнок */
.parent { overflow: hidden; } /* или padding-top: 1px; или display: flow-root */
```

**Связанные задачи:**

- [Схлопывание отступов](../../../tasks/frontend/css/2_css_middle.md#схлопывание-отступов)

**Материалы для изучения:**

- [MDN: Схлопывание внешних отступов](https://developer.mozilla.org/ru/docs/Web/CSS/CSS_box_model/Mastering_margin_collapsing)

---

## Что такое псевдокласс :has()?

`:has()` — «родительский» псевдокласс: применяет стили к элементу, если внутри него содержится совпадающий потомок. Впервые позволяет стилизовать родителя, опираясь на состояние дочернего элемента.

```css
/* Карточка с изображением получает другой padding */
.card:has(img) { padding: 0; }

/* Форма без валидных полей */
form:has(input:invalid) .submit-btn { opacity: 0.5; pointer-events: none; }

/* label подсвечивается когда связанный checkbox отмечен */
label:has(+ input:checked) { font-weight: bold; color: blue; }
```

`:has()` поддерживается во всех современных браузерах с 2023 года.

**Связанные задачи:**

<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: :has()](https://developer.mozilla.org/ru/docs/Web/CSS/:has)

---

## Что такое медиафункция prefers-reduced-motion?

`prefers-reduced-motion` — медиазапрос, который определяет, что пользователь в настройках системы запросил минимум анимаций (важно для людей с вестибулярными расстройствами). При значении `reduce` нужно отключать или сильно упрощать анимации.

```css
@keyframes slideIn {
  from { transform: translateX(-100%); }
  to { transform: translateX(0); }
}

.modal { animation: slideIn 0.4s ease; }

/* Отключаем анимацию по запросу пользователя */
@media (prefers-reduced-motion: reduce) {
  .modal { animation: none; }
  * { transition-duration: 0.01ms !important; }
}
```

**Связанные задачи:**

<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: prefers-reduced-motion](https://developer.mozilla.org/ru/docs/Web/CSS/@media/prefers-reduced-motion)
- [web.dev: prefers-reduced-motion](https://web.dev/articles/prefers-reduced-motion)

---

## Desktop-first vs Mobile-first?

**Desktop-first** — базовые стили для широкого экрана, адаптив через `max-width`.

**Mobile-first** — базовые стили для мобильных, адаптив через `min-width`. Рекомендует Google и общее best practice.

```css
/* Mobile-first (рекомендуется) */
.card { flex-direction: column; }

@media (min-width: 768px) {
  .card { flex-direction: row; }
}

/* Desktop-first (legacy) */
.card { flex-direction: row; }
@media (max-width: 767px) {
  .card { flex-direction: column; }
}
```

**Преимущества mobile-first:** меньше CSS для слабых устройств, лучше SEO, вынуждает обдумывать приоритет контента.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Perfect pixel — что это?

**Pixel perfect** — вёрстка, максимально точно совпадающая с макетом. Обычно проверяется путём наложения макета полупрозрачно на результат.

**Инструменты:** Perfect Pixel (Chrome extension), Figma Inspect, DevTools Rulers.

**Современный подход:** вместо точности до пикселя — система дизайна с токенами (цвета, отступы, шрифты). Адаптивность важнее попиксельной точности.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## CSS-спрайты?

**CSS-спрайт** — одно изображение, содержащее несколько иконок. CSS позиционирует нужную область через `background-position`. Цель: сокращение HTTP-запросов.

```css
.icon { width: 24px; height: 24px; background-image: url('sprites.png'); }
.icon-home   { background-position: 0 0; }
.icon-search { background-position: -24px 0; }
```

**Современные альтернативы:** SVG-спрайт (`<symbol>` + `<use>`), шрифтовые иконки, HTTP/2 (снизил актуальность спрайтов).

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Ретина-дисплей и ретинизация?

**Retina** — дисплеи с `devicePixelRatio ≥ 2`. Если подать изображение в 1×, оно будет размытым.

```html
<img src="logo.png" srcset="logo.png 1x, logo@2x.png 2x" alt="Logo">
```

```css
.logo { background-image: url('logo.png'); }

@media (-webkit-min-device-pixel-ratio: 2), (min-resolution: 192dpi) {
  .logo { background-image: url('logo@2x.png'); background-size: 50% auto; }
}
```

SVG масштабируется бесконечно — идеально для иконок и логотипов. `background-size: 50%` нужен, чтобы изображение 2× отображалось в нужном CSS-размере.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## vmax, vmin, vh, vw — когда использовать?

- `vw` — 1% ширины viewport. Когда: адаптивная ширина/шрифт.
- `vh` — 1% высоты viewport. Когда: full-screen секции `height: 100vh`.
- `vmin` — меньшая из vw/vh. Когда: шрифт должен расти в обеих ориентациях.
- `vmax` — большая. Когда: размер по доминирующей стороне.

```css
h1 { font-size: clamp(1.5rem, 4vw, 3rem); }
.hero { height: 100vh; }
.square { width: 50vmin; height: 50vmin; }
```

**Проблема 100vh на iOS:** мобильные браузеры учитывают адресную строку. Решение: `height: 100svh` (Small Viewport Height, 2023).

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## object-fit и object-position?

`object-fit` управляет как `<img>` или `<video>` подгоняется под фиксированный блок размеров.

```css
img {
  width: 300px; height: 200px;

  object-fit: fill;       /* растянет, исказит */
  object-fit: contain;    /* впишет, сохраняет пропорции, чёрные поля */
  object-fit: cover;      /* заполняет блок, обрезает, пропорции сохраняются */
  object-fit: none;       /* оригинальный размер */
  object-fit: scale-down; /* меньшее из none и contain */

  object-position: center top; /* фокус изображения (default: center) */
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## text-shadow и box-shadow?

```css
/* text-shadow: x y blur color */
.heading { text-shadow: 2px 2px 4px rgba(0,0,0,0.3); }

/* box-shadow: x y blur spread color */
.card {
  box-shadow: 0 4px 16px rgba(0,0,0,0.1);
  box-shadow: inset 0 2px 4px rgba(0,0,0,0.15); /* внутренняя тень */
  /* несколько теней */
  box-shadow: 0 2px 8px rgba(0,0,0,0.1), 0 0 0 3px rgba(37,99,235,0.3);
}

/* Горизонтальный border через box-shadow (spread без blur) */
.outline { box-shadow: 0 0 0 2px blue; }
```

`box-shadow` не влияет на layout. В отличие от `outline`, `box-shadow` учитывает `border-radius`.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Градиенты (linear-gradient, radial-gradient)?

```css
.el {
  background: linear-gradient(to right, #ff6b6b, #4ecdc4);
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.circle-bg {
  background: radial-gradient(circle at center, #fff 0%, #e0e0e0 100%);
}

/* conic-gradient: конический */
.pie { background: conic-gradient(red 0 30%, blue 30% 70%, green 70% 100%); }

/* Повторяющийся */
.stripes {
  background: repeating-linear-gradient(
    45deg, #f0f0f0, #f0f0f0 10px, #e0e0e0 10px, #e0e0e0 20px
  );
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## word-wrap, word-break, word-spacing?

```css
/* overflow-wrap (word-wrap) — перенос длинных слов */
.el {
  overflow-wrap: normal;     /* не переносить (default) */
  overflow-wrap: break-word; /* перенести длинное слово */
  overflow-wrap: anywhere;   /* более агрессивное разбиение */
}

/* word-break — алгоритм разрыва */
.el {
  word-break: normal;    /* default */
  word-break: break-all; /* разрывать в любом месте */
  word-break: keep-all;  /* не разрывать корейский/китайский */
}

/* word-spacing — расстояние между словами */
.el { word-spacing: 4px; }
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## counter-increment и counter-reset?

Счётчики CSS автоматически нумеруют элементы без `<ol>`.

```css
article { counter-reset: section; }
h2 { counter-increment: section; }
h2::before { content: counter(section) '. '; }

/* Вложенные счётчики */
article { counter-reset: chapter; }
section { counter-reset: subsection; }
h2 { counter-increment: chapter; }
h3 { counter-increment: subsection; }
h3::before { content: counter(chapter) '.' counter(subsection) ' '; }
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## background-origin, background-clip, background-size?

```css
.el {
  /* background-origin: откуда отсчитывается позиция фона */
  background-origin: padding-box; /* default */
  background-origin: border-box;
  background-origin: content-box;

  /* background-clip: до какого предела отображается фон */
  background-clip: border-box;  /* default */
  background-clip: padding-box;
  background-clip: content-box;
  background-clip: text;        /* фон под текстом! */

  /* background-size */
  background-size: cover;       /* заполнить блок, обрезать */
  background-size: contain;     /* вписать целиком */
  background-size: 200px 100px;
}

/* Градиентный текст */
.gradient-text {
  background: linear-gradient(135deg, #ff6b6b, #4ecdc4);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## @font-face?

```css
@font-face {
  font-family: 'MyFont';
  src: url('myfont.woff2') format('woff2'),
       url('myfont.woff')  format('woff');
  font-weight: 400;
  font-style: normal;
  font-display: swap; /* показывать fallback до загрузки */
}

body { font-family: 'MyFont', Arial, sans-serif; }
```

**`font-display` значения:** `swap` — фоллбэк, затем замена (для SEO); `block` — прятать (FOIT); `fallback` — короткая блокировка; `optional` — не заменять если медленно.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## @charset?

```css
/* Должен быть первой строкой в CSS-файле */
@charset "UTF-8";
```

Сообщает браузеру кодировку CSS-файла. Сейчас практически не нужен, если сервер отдаёт `Content-Type: text/css; charset=utf-8`.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## CSS-методологии: SMACSS, OOCSS, Atomic CSS?

| Методология | Идея |
|---|---|
| **BEM** | Блок\_\_Элемент--Модификатор |
| **SMACSS** | Категории: Base, Layout, Module, State, Theme |
| **OOCSS** | Отделять структуру от оформления; повторно используемые объекты |
| **Atomic CSS** | Один класс = одно свойство (Tailwind) |

```css
/* SMACSS */
.l-container { max-width: 1200px; } /* Layout */
.btn { padding: 8px 16px; }         /* Module */
.is-active { display: block; }      /* State */
```

```html
<!-- Atomic CSS (Tailwind) -->
<div class="flex items-center gap-4 p-4 bg-white rounded-lg shadow">...</div>
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Производительность CSS-селекторов?

Браузер читает CSS-селекторы **справа налево** — сначала определяет правое (ключевое), затем по цепочке.

**От быстрого к медленному:**
1. `#id` — быстрый
2. `.class` — быстрый
3. `div` — средний
4. `[attr]`, `:not()`, `:first-child` — медленнее
5. `*` — медленный
6. Длинные цепочки (`div > * > p > span`) — медленные

```css
/* Плохо */
div > * > p > span { color: red; }

/* Хорошо */
.card__title { color: red; }
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## CSS Modules и изоляция стилей?

**Проблема:** конфликты имён классов в глобальном CSS.

**Подходы:**
- **CSS Modules** — `Button.module.css`, классы хэшируются: `.button` → `.Button_button__xK2p9`
- **CSS-in-JS** (styled-components, Emotion) — стили в JS, автоматическая изоляция
- **Shadow DOM** — изоляция веб-компонентов
- **Scoped CSS** (Vue) — атрибут `data-v-XXXX` на каждом элементе

```jsx
import styles from './Button.module.css';
<button className={styles.button}>Нажми</button>
// рендерится: class="Button_button__xK2p9"
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Асинхронная загрузка CSS?

Обычный `<link rel="stylesheet">` блокирует рендеринг. Для некритических стилей:

```html
<!-- rel="preload" + onload (правильный способ) -->
<link rel="preload" href="non-critical.css" as="style"
      onload="this.onload=null;this.rel='stylesheet'">
<noscript><link rel="stylesheet" href="non-critical.css"></noscript>

<!-- media trick -->
<link rel="stylesheet" href="print.css" media="print"
      onload="this.media='all'">
```

**Когда использовать:** некритические стили (print, слайдеры, компоненты ниже фолда). Критические стили встраивай inline.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
