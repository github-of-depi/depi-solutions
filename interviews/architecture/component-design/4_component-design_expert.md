# Дизайн компонентов — Expert

## Вопросы

- [Как строить компоненты для работы в Server Components окружении?](#как-строить-компоненты-для-работы-в-server-components-окружении)
- [Как проектировать компоненты для максимальной производительности?](#как-проектировать-компоненты-для-максимальной-производительности)
- [Как строить компонентную библиотеку с нуля?](#как-строить-компонентную-библиотеку-с-нуля)

---

## Как строить компоненты для работы в Server Components окружении?

RSC-compatible компоненты: нет `useState`/`useEffect`/event handlers → Server Component. Граница: `"use client"` директива минимально возможно низко в дереве. Паттерн: shell — Server Component, interactive parts — Client Components через `children`.

```typescript
// Server Component — данные, layout
async function ProductPage({ id }: { id: string }) {
  const product = await db.products.findById(id); // прямой доступ к БД
  return (
    <div>
      <ProductInfo product={product} />
      <AddToCartButton productId={id} />  {/* Client Component */}
    </div>
  );
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как проектировать компоненты для максимальной производительности?

1. **Lazy loading** — `React.lazy` + `Suspense` для компонентов не в initial viewport
2. **Virtualization** — `@tanstack/react-virtual` для длинных списков
3. **Мемоизация** — `memo` + стабильные пропсы (useCallback, useMemo)
4. **Code splitting** — компонент на свой chunk через dynamic import
5. **Избегать context overhead** — useContextSelector или atomic state (Jotai)

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как строить компонентную библиотеку с нуля?

1. **Инфраструктура**: Turborepo monorepo, tsup/rollup для сборки, Storybook для разработки
2. **Accessibility**: Radix UI primitives или React Aria как основа
3. **Styling**: CSS Variables + Tailwind или zero-runtime CSS-in-JS (Linaria)
4. **Testing**: Vitest + RTL + Chromatic
5. **Publishing**: npm, changesets для versioning, автодокументация через TypeDoc

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
