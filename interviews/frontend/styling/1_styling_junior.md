# Styling — Junior

## Вопросы

- [Что такое CSS-in-JS?](#что-такое-css-in-js)
- [Что такое CSS Modules?](#что-такое-css-modules)
- [Что такое Tailwind CSS?](#что-такое-tailwind-css)
- [Чем styled-components отличается от CSS Modules?](#чем-styled-components-отличается-от-css-modules)
- [Что такое utility-first CSS?](#что-такое-utility-first-css)

---

## Что такое CSS-in-JS?

CSS-in-JS — подход, при котором стили пишутся в JavaScript/TypeScript файлах. Преимущества: scoping, dynamic styles, colocation с компонентом, TypeScript-поддержка. Недостатки: рантайм overhead (классические реализации), сложность SSR. Библиотеки: styled-components, Emotion, vanilla-extract (zero-runtime).

---

## Что такое CSS Modules?

CSS Modules — файлы `.module.css`, классы которых автоматически становятся уникальными (локальный scope) при импорте в JS. Нет конфликтов имён, стандартный CSS синтаксис, нет рантайм overhead.

```typescript
import styles from "./Button.module.css";
<button className={styles.primary}>Click</button>
// HTML: <button class="Button_primary__abc123">
```

---

## Что такое Tailwind CSS?

Tailwind — utility-first CSS фреймворк: атомарные классы для каждого CSS-свойства. Не нужно писать кастомный CSS, стили задаются прямо в className. Неиспользуемые классы удаляются при сборке (purge/JIT). Конфигурируется через `tailwind.config.js`.

```tsx
<button className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 transition">
  Кнопка
</button>
```

---

## Чем styled-components отличается от CSS Modules?

CSS Modules — статические стили, нет рантайм, обычный CSS синтаксис. styled-components — динамические стили через JS-интерполяции, рантайм генерация классов, TypeScript для props. Оба обеспечивают scoping.

---

## Что такое utility-first CSS?

Utility-first — подход с маленькими одноцелевыми классами (`text-lg`, `font-bold`, `mt-4`) вместо семантических (`.card-title`). Разработка быстрее, нет необходимости придумывать имена, стили консистентны через design tokens. Tailwind — эталонная реализация.
