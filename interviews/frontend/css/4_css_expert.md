# CSS — Expert

## Вопросы

- [Что такое CSS Houdini?](#что-такое-css-houdini)
- [Как работает алгоритм layout браузера?](#как-работает-алгоритм-layout-браузера)
- [Как строить scalable CSS архитектуру для больших проектов?](#как-строить-scalable-css-архитектуру-для-больших-проектов)
- [Что такое логические свойства CSS?](#что-такое-логические-свойства-css)
- [Как работает subgrid?](#как-работает-subgrid)

---

## Что такое CSS Houdini?

CSS Houdini — набор низкоуровневых браузерных API для расширения CSS движка. Позволяет: писать кастомные CSS properties с типами и анимацией (`CSS.registerProperty`), создавать paint worklets (рисовать в background через Canvas API), layout worklets. Даёт доступ к ранее невозможным эффектам без JS перерисовки.

```javascript
CSS.paintWorklet.addModule("ripple.js");
CSS.registerProperty({
  name: "--ripple-color",
  syntax: "<color>",
  inherits: false,
  initialValue: "transparent",
});
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работает алгоритм layout браузера?

Layout (Flow Layout) — браузер вычисляет позиции и размеры элементов. BFC (Block Formatting Context) изолирует внутренний layout: `overflow != visible`, `display: flow-root`, `float`, `absolute`/`fixed`. IFC (Inline Formatting Context) — для строчных элементов. Layout алгоритм: прямоугольники боксов, constraint-based sizing, margin collapsing (только вертикальный, только в потоке).

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как строить scalable CSS архитектуру для больших проектов?

1. **Design tokens** (CSS variables) как единственный источник правды для значений
2. **@layer** для управления каскадом без битв специфичности
3. **CSS Modules** или **scoped CSS** для изоляции компонентов
4. **Utility classes** (Tailwind) для композиции без custom CSS
5. **Style Dictionary** для синхронизации токенов между CSS, JS, iOS, Android

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое логические свойства CSS?

Логические свойства заменяют физические (`left`, `right`, `top`, `bottom`) на логические относительно writing mode. Для поддержки RTL и вертикальных языков без медиазапросов.

```css
/* Физические — сломается в RTL */
.card { margin-left: 16px; padding-top: 8px; }

/* Логические — работают в любом writing mode */
.card { margin-inline-start: 16px; padding-block-start: 8px; }
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работает subgrid?

`subgrid` позволяет дочернему grid-контейнеру наследовать треки родительского grid — элементы глубоко вложенные выравниваются по родительской сетке. Решает проблему выравнивания элементов карточек в сетке.

```css
.grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 24px; }
.card {
  grid-column: span 1;
  display: grid;
  grid-row: span 3;
  grid-template-rows: subgrid; /* наследует треки родителя */
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
