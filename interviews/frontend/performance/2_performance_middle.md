# Performance — Middle

## Вопросы

- [Как работает code splitting?](#как-работает-code-splitting)
- [Как анализировать JavaScript бандл?](#как-анализировать-javascript-бандл)
- [Что такое виртуализация списков?](#что-такое-виртуализация-списков)
- [Как оптимизировать изображения?](#как-оптимизировать-изображения)
- [Что такое мемоизация и когда её применять?](#что-такое-мемоизация-и-когда-её-применять)
- [Что такое tree shaking?](#что-такое-tree-shaking)
- [Как использовать prefetch и preload?](#как-использовать-prefetch-и-preload)

---

## Как работает code splitting?

Code splitting — разбиение бандла на части, загружаемые по необходимости. `import()` (dynamic import) — сигнал сборщику создать отдельный chunk. `React.lazy` + `Suspense` для компонентов. Vite/Webpack автоматически создают chunks для `import()` и route-based splitting.

```typescript
const Dashboard = lazy(() => import("./Dashboard"));
<Suspense fallback={<Loader />}><Dashboard /></Suspense>
```

---

## Как анализировать JavaScript бандл?

`rollup-plugin-visualizer` (Vite) или `webpack-bundle-analyzer` (Webpack) — визуальное дерево зависимостей. Ищем: дублирование (lodash в нескольких местах), тяжёлые зависимости (moment.js вместо date-fns), неиспользуемый код.

```bash
npx vite-bundle-visualizer
# или в vite.config.ts: plugins: [visualizer({ open: true })]
```

---

## Что такое виртуализация списков?

Виртуализация рендерит только видимые элементы + буфер. Список из 10000 элементов в DOM — тысячи узлов, медленный scroll. С виртуализацией — всегда ~20-30 DOM узлов. Библиотеки: `@tanstack/react-virtual`, `react-window`, `react-virtuoso`.

---

## Как оптимизировать изображения?

1. Современные форматы: WebP, AVIF (30-50% меньше JPEG)
2. Правильные размеры: `srcset` для разных плотностей экрана
3. Lazy loading: `loading="lazy"` или Intersection Observer
4. `aspect-ratio` CSS для предотвращения CLS
5. `next/image` или аналоги для автоматической оптимизации

---

## Что такое мемоизация и когда её применять?

Мемоизация — кэширование результата вычисления. React: `useMemo` для дорогих вычислений, `useCallback` для стабильных функций, `React.memo` для компонентов. Применять только при измеренной проблеме производительности — overhead мемоизации тоже стоит.

---

## Что такое tree shaking?

Tree shaking — исключение неиспользуемого кода при сборке. Работает с ES modules (`import`/`export`). Библиотека должна быть marked как side-effect-free (`"sideEffects": false` в package.json). Именованные импорты дружественны к tree shaking: `import { format } from "date-fns"`.

---

## Как использовать prefetch и preload?

`<link rel="preload">` — загрузить ресурс с высоким приоритетом до того, как браузер его найдёт (шрифты, критические изображения). `<link rel="prefetch">` — загрузить ресурс с низким приоритетом для будущих переходов. В Next.js `Link` делает prefetch автоматически для видимых ссылок.

```html
<link rel="preload" as="font" href="/fonts/inter.woff2" crossorigin />
<link rel="prefetch" href="/about" />
```
