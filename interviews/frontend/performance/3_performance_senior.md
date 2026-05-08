# Performance — Senior

## Вопросы

- [Как улучшить LCP?](#как-улучшить-lcp)
- [Как устранить CLS?](#как-устранить-cls)
- [Как профилировать React приложение?](#как-профилировать-react-приложение)
- [Что такое Web Workers и когда их использовать?](#что-такое-web-workers-и-когда-их-использовать)
- [Как оптимизировать Time to Interactive (TTI)?](#как-оптимизировать-time-to-interactive-tti)
- [Как управлять performance budget?](#как-управлять-performance-budget)

---

## Как улучшить LCP?

LCP = время отрисовки наибольшего элемента (обычно hero image или H1). Оптимизации:
1. `<link rel="preload">` для LCP-изображения
2. `priority` атрибут в `next/image`
3. Оптимизация TTFB (Edge CDN, кэширование)
4. Избегать render-blocking CSS/JS
5. Inline critical CSS
6. Server-side rendering вместо CSR для первого экрана

---

## Как устранить CLS?

CLS — layout shifts при загрузке. Причины: изображения без размеров, динамически вставляемый контент, web fonts (FOUT/FOIT).

Решения: всегда задавать `width`/`height` или `aspect-ratio` для изображений, резервировать место для динамического контента, использовать `font-display: optional` или `size-adjust` для web fonts, избегать вставки контента над видимым текстом.

---

## Как профилировать React приложение?

1. **React DevTools Profiler** — flamegraph ре-рендеров: какой компонент рендерится и почему
2. **Chrome DevTools Performance** — timeline: layout, paint, scripting
3. **`why-did-you-render`** — логирует лишние ре-рендеры в dev mode
4. **`<Profiler>` API** — программный замер рендеров

```typescript
<Profiler id="List" onRender={(id, phase, actualDuration) => {
  console.log(`${id} ${phase}: ${actualDuration}ms`);
}}>
  <List />
</Profiler>
```

---

## Что такое Web Workers и когда их использовать?

Web Workers — JavaScript в отдельном потоке (не main thread). Используются для тяжёлых CPU задач: обработка данных, шифрование, парсинг CSV, ML inference. Не имеют доступа к DOM. Общение через `postMessage`.

```typescript
// worker.ts
self.onmessage = (e) => {
  const result = heavyComputation(e.data);
  self.postMessage(result);
};
// main thread
const worker = new Worker(new URL("./worker.ts", import.meta.url));
worker.postMessage(largeData);
worker.onmessage = (e) => setResult(e.data);
```

---

## Как оптимизировать Time to Interactive (TTI)?

TTI — время до полной интерактивности страницы. Стратегии: Server-side rendering + hydration (быстрый первый контент), Progressive Hydration (гидрировать компоненты по мере видимости), Islands Architecture (минимальный JS на странице), React Server Components (меньше JS на клиенте).

---

## Как управлять performance budget?

Performance budget — явные лимиты на размер ресурсов и метрики (LCP < 2.5s, JS < 200KB). Автоматическая проверка в CI: Lighthouse CI, Bundlesize (размер бандла), Web Vitals через Playwright. При превышении — сборка падает с ошибкой.
