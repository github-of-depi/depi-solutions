# Styling — Middle

## Вопросы

- [Как кастомизировать Tailwind CSS?](#как-кастомизировать-tailwind-css)
- [Что такое tailwind-merge и clsx?](#что-такое-tailwind-merge-и-clsx)
- [Как реализовать тему (light/dark) через CSS переменные?](#как-реализовать-тему-lightdark-через-css-переменные)
- [Что такое design tokens?](#что-такое-design-tokens)
- [CSS-in-JS производительность — в чём проблема?](#css-in-js-производительность--в-чём-проблема)
- [Что такое zero-runtime CSS-in-JS?](#что-такое-zero-runtime-css-in-js)

---

## Как кастомизировать Tailwind CSS?

В `tailwind.config.ts`: расширение `theme.extend` — добавить токены поверх дефолтных. `theme` — полное переопределение. Кастомные цвета, шрифты, отступы, breakpoints. Плагины для кастомных утилит и компонентов.

```typescript
export default {
  theme: {
    extend: {
      colors: { brand: { 500: "#3b82f6", 600: "#2563eb" } },
      fontFamily: { sans: ["Inter", ...defaultTheme.fontFamily.sans] },
    },
  },
  plugins: [require("@tailwindcss/forms")],
};
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое tailwind-merge и clsx?

`clsx` — утилита для условного объединения className строк. `tailwind-merge` — умный merge: разрешает конфликты Tailwind классов (последний побеждает). Стандартная комбинация в shadcn/ui и других библиотеках.

```typescript
import { clsx } from "clsx";
import { twMerge } from "tailwind-merge";

function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

// twMerge разрешает конфликт: px-2 проигрывает px-4
cn("px-2 py-1", isLarge && "px-4") // "py-1 px-4"
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как реализовать тему (light/dark) через CSS переменные?

CSS переменные меняются через смену класса или `data-theme` на `:root`. JavaScript переключает класс, CSS-переменные каскадируются автоматически.

```css
:root { --bg: #fff; --text: #111; }
.dark { --bg: #111; --text: #fff; }
/* или через prefers-color-scheme */
@media (prefers-color-scheme: dark) { :root { --bg: #111; --text: #fff; } }
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое design tokens?

Design tokens — атомарные дизайн-решения: цвета, шрифты, отступы, тени — хранятся как именованные значения. Источник правды для дизайн-системы. Sync между Figma, CSS, JS через Style Dictionary или Token Studio.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## CSS-in-JS производительность — в чём проблема?

Классический CSS-in-JS (styled-components, Emotion) генерирует стили в рантайме: парсинг template literals, вставка в DOM через `<style>` теги. Это блокирует рендер, усложняет SSR (нужна server-side extraction). При SSR без extraction возникает FOUC.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое zero-runtime CSS-in-JS?

Zero-runtime (vanilla-extract, Linaria, StyleX) генерирует CSS во время сборки — нет рантайм overhead. Vanilla-extract: TypeScript-first, полная типобезопасность, генерирует `.css` файлы. StyleX (Meta): атомарный CSS, используется в production на facebook.com.

```typescript
// vanilla-extract
import { style } from "@vanilla-extract/css";
export const button = style({
  padding: "8px 16px",
  background: vars.color.brand,
  selectors: { "&:hover": { background: vars.color.brandDark } },
});
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
