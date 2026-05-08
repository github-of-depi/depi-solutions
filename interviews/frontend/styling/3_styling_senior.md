# Styling — Senior

## Вопросы

- [Как строить design system на основе CSS переменных и design tokens?](#как-строить-design-system-на-основе-css-переменных-и-design-tokens)
- [Как обеспечить server-side rendering для CSS-in-JS?](#как-обеспечить-server-side-rendering-для-css-in-js)
- [Tailwind vs CSS-in-JS — что выбрать для большого проекта?](#tailwind-vs-css-in-js--что-выбрать-для-большого-проекта)
- [Что такое Style Dictionary и как его использовать?](#что-такое-style-dictionary-и-как-его-использовать)
- [Как тестировать стили компонентов?](#как-тестировать-стили-компонентов)

---

## Как строить design system на основе CSS переменных и design tokens?

Трёхуровневая модель: **Primitive tokens** (raw values: `--color-blue-500: #3b82f6`), **Semantic tokens** (значение через контекст: `--color-interactive: var(--color-blue-500)`), **Component tokens** (`--button-bg: var(--color-interactive)`). Semantic слой делает тему возможной без изменения компонентов.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как обеспечить server-side rendering для CSS-in-JS?

Styled-components: `ServerStyleSheet` — собирает стили в строку, вставляется в HTML. Emotion: `extractCritical`. Next.js App Router + styled-components: нужен `serverComponentsExternalPackages` и custom `_document` (Pages Router) или registry (App Router). Альтернатива: перейти на zero-runtime.

```typescript
// Next.js App Router + styled-components registry
"use client";
function StyledComponentsRegistry({ children }: { children: React.ReactNode }) {
  const [styledComponentsStyleSheet] = useState(() => new ServerStyleSheet());
  useServerInsertedHTML(() => {
    const styles = styledComponentsStyleSheet.getStyleElement();
    styledComponentsStyleSheet.instance.clearTag();
    return <>{styles}</>;
  });
  return <StyleSheetManager sheet={styledComponentsStyleSheet.instance}>{children}</StyleSheetManager>;
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Tailwind vs CSS-in-JS — что выбрать для большого проекта?

**Tailwind**: быстрая разработка, нет рантайм, отличная поддержка, легко онбордировать. Минус: длинные className строки, не очевидна бизнес-семантика. **CSS-in-JS** (zero-runtime): TypeScript-first, semantic names, co-location. Минус: сложнее настройка.

Тренд 2025: Tailwind + shadcn/ui как стандарт для новых проектов. Для design systems с строгими требованиями — vanilla-extract или StyleX.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое Style Dictionary и как его использовать?

Style Dictionary (Amazon) — transform pipeline для design tokens: JSON/JSON5 с токенами → CSS variables, SCSS, JS objects, Swift, Kotlin. Позволяет иметь единый источник токенов для всех платформ.

```json
{
  "color": { "brand": { "500": { "value": "#3b82f6", "type": "color" } } }
}
```

Выходной CSS: `--color-brand-500: #3b82f6;` — автоматически.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как тестировать стили компонентов?

1. **Visual regression testing**: Playwright screenshots или Chromatic (Storybook) — сравнение снимков
2. **`toHaveStyle`** в RTL — проверка конкретных CSS свойств
3. **Storybook** — изоляция компонента во всех состояниях
4. **CSS custom properties**: проверять через `getComputedStyle` в тестах или в браузере

```typescript
expect(element).toHaveStyle("display: flex");
expect(element).toHaveClass("bg-blue-500");
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
