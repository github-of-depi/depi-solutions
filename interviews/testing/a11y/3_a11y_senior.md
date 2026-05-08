# Accessibility (A11y) — Senior

## Вопросы

- [Как встроить a11y в процесс разработки?](#как-встроить-a11y-в-процесс-разработки)
- [Как тестировать со screen reader?](#как-тестировать-со-screen-reader)
- [Как строить accessible design system?](#как-строить-accessible-design-system)
- [Что такое WCAG 2.2 новые критерии?](#что-такое-wcag-22-новые-критерии)

---

## Как встроить a11y в процесс разработки?

1. **axe DevTools** в браузере разработчика
2. **ESLint jsx-a11y** — статический анализ JSX
3. **jest-axe** в unit тестах (автоматически)
4. **Storybook axe addon** — a11y в Storybook stories
5. **Manual keyboard testing** в PR процессе
6. **Axe CI** — блокировать сборку при нарушениях

---

## Как тестировать со screen reader?

Комбинации: VoiceOver + Safari (Mac/iOS), NVDA/JAWS + Chrome (Windows), TalkBack + Chrome (Android). Проверять: порядок чтения, наличие контекста, объявление динамических изменений, работу с формами. Регулярно тестировать ключевые user flows.

---

## Как строить accessible design system?

1. Использовать Radix UI / React Aria как a11y-first primitives
2. Каждый компонент проходит keyboard + screen reader тест
3. WCAG AA как минимальный стандарт всех компонентов
4. Документировать a11y использование в Storybook
5. Contrast ratios — автоматическая проверка color tokens

---

## Что такое WCAG 2.2 новые критерии?

Новые в WCAG 2.2 (2023): Focus Appearance (2.4.11/12) — размер и контраст фокус-индикатора, Dragging Movements (2.5.7) — альтернатива для drag&drop, Target Size (2.5.8) — минимальный размер 24x24px, Redundant Entry (3.3.7) — не запрашивать введённые данные повторно.
