# Browser — Expert

## Вопросы

- [Как работает движок V8 — компиляция и оптимизация JS?](#как-работает-движок-v8--компиляция-и-оптимизация-js)
- [Что такое Back/Forward Cache (bfcache)?](#что-такое-backforward-cache-bfcache)
- [Как работает rendering pipeline в Chrome?](#как-работает-rendering-pipeline-в-chrome)
- [Как интегрировать WebAssembly в React приложение?](#как-интегрировать-webassembly-в-react-приложение)
- [Что такое Speculation Rules API?](#что-такое-speculation-rules-api)

---

## Как работает движок V8 — компиляция и оптимизация JS?

V8 (Chrome, Node.js): JS → Ignition (bytecode interpreter) → TurboFan (JIT compiler). Ignition быстро компилирует в bytecode. Горячий код (часто исполняемый) — TurboFan оптимизирует в машинный код. «Деоптимизация» — если предположение о типах нарушается (hidden class change). Советы: монотипные функции, не менять форму объектов после создания.

---

## Что такое Back/Forward Cache (bfcache)?

bfcache — браузер сохраняет полный снимок страницы (включая JS heap) при навигации назад/вперёд. Страница восстанавливается мгновенно без повторного запроса. Страницы исключаются из bfcache при: `unload` обработчиках, открытых IndexedDB транзакциях, `Cache-Control: no-store`. Обработчики `pageshow`/`pagehide` для реагирования на bfcache restore.

---

## Как работает rendering pipeline в Chrome?

1. **Parse** — HTML → DOM tree
2. **Style** — применение CSS → computed styles
3. **Layout** — геометрия элементов (позиции, размеры)
4. **Layer tree** — определение compositor layers (transform, opacity, will-change)
5. **Paint** — инструкции рисования для каждого layer
6. **Rasterize** — пиксели (на GPU через Skia/ANGLE)
7. **Composite** — объединение layers на GPU

Compositor thread работает независимо от main thread — анимации transform/opacity не блокируются JS.

---

## Как интегрировать WebAssembly в React приложение?

Wasm модуль компилируется заранее. В браузере: `WebAssembly.instantiateStreaming()` для загрузки. Vite поддерживает `?init` суффикс. Rust → Wasm через `wasm-pack`, генерирует JS bindings автоматически.

```typescript
import init, { heavy_computation } from "./pkg/my_wasm";

async function setupWasm() {
  await init(); // инициализация Wasm модуля
  const result = heavy_computation(largeData); // вызов Rust функции
}
```

---

## Что такое Speculation Rules API?

Speculation Rules — новый API для prefetch и prerender страниц с декларативными правилами. Позволяет браузеру полностью пререндерить следующую страницу в фоне — навигация происходит мгновенно. Безопаснее чем `<link rel="prefetch">`.

```html
<script type="speculationrules">
{
  "prerender": [{ "urls": ["/checkout", "/dashboard"] }],
  "prefetch": [{ "where": { "href_matches": "/products/*" } }]
}
</script>
```
