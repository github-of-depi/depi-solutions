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
