# Styling — Expert

## Вопросы

- [Как устроена архитектура атомарного CSS (StyleX, Tailwind JIT)?](#как-устроена-архитектура-атомарного-css-stylex-tailwind-jit)
- [Как реализовать multi-brand theming в design system?](#как-реализовать-multi-brand-theming-в-design-system)
- [Что такое CSS Scope и как он изменит изоляцию стилей?](#что-такое-css-scope-и-как-он-изменит-изоляцию-стилей)
- [Как обеспечить производительность анимаций в сложных UI?](#как-обеспечить-производительность-анимаций-в-сложных-ui)

---

## Как устроена архитектура атомарного CSS (StyleX, Tailwind JIT)?

Атомарный CSS: одно правило — один класс. `text-blue-500` = `.text-blue-500 { color: #3b82f6 }`. Компонент получает набор атомарных классов, CSS файл растёт логарифмически (не линейно) при добавлении компонентов. StyleX (Meta) делает это через статический анализ: во время сборки генерирует минимальный набор атомарных классов и разрешает конфликты через specificity-free order.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как реализовать multi-brand theming в design system?

Трёхуровневые токены + CSS layers + per-brand override файл. Каждый бренд определяет semantic токены, primitive токены могут отличаться. Переключение через `data-brand` атрибут на root.

```css
[data-brand="acme"] {
  --color-interactive: var(--acme-color-primary);
  --font-family-base: "Roboto", sans-serif;
}
[data-brand="globex"] {
  --color-interactive: var(--globex-color-primary);
  --font-family-base: "Inter", sans-serif;
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое CSS Scope и как он изменит изоляцию стилей?

`@scope` (CSS Scoping Module) — нативная изоляция стилей без классов или CSS Modules. Позволяет ограничить область применения правил конкретным DOM-поддеревом, включая «donut scope» (от .component до .excluded).

```css
@scope (.card) to (.card__footer) {
  h2 { font-size: 1.5rem; } /* только внутри .card, не затрагивает .card__footer */
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как обеспечить производительность анимаций в сложных UI?

1. Только `transform` и `opacity` — compositor-only, 60fps без layout/paint
2. `will-change: transform` — заранее, только перед анимацией
3. `requestAnimationFrame` вместо setTimeout для JS-анимаций
4. Web Animations API (`element.animate()`) — нативный, оптимизирован браузером
5. CSS `animation-fill-mode: both` + `animation-play-state` для контролируемых анимаций
6. Избегать `filter` и `backdrop-filter` на часто анимируемых элементах — дорого

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
