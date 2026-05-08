# Browser — Senior

## Вопросы

- [Как работает критический путь рендеринга?](#как-работает-критический-путь-рендеринга)
- [Что такое Content Security Policy (CSP)?](#что-такое-content-security-policy-csp)
- [Как работает HTTP кэширование?](#как-работает-http-кэширование)
- [Что такое Web Workers и Shared Workers?](#что-такое-web-workers-и-shared-workers)
- [Как работает WebAssembly?](#как-работает-webassembly)
- [Что такое Performance API?](#что-такое-performance-api)

---

## Как работает критический путь рендеринга?

CRP (Critical Rendering Path): HTML → DOM, CSS → CSSOM (параллельно), DOM + CSSOM → Render Tree → Layout → Paint → Composite. Оптимизации: inline critical CSS, defer/async для некритических скриптов, preload для критических ресурсов, минимизировать render-blocking.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое Content Security Policy (CSP)?

CSP — HTTP заголовок, ограничивающий источники загружаемых ресурсов. Предотвращает XSS атаки: даже если злоумышленник внедрил скрипт — браузер его не выполнит, если origin не в белом списке. `script-src 'self'` — только скрипты с того же домена. Nonce-based CSP — для inline скриптов.

```
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-abc123'; img-src *
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работает HTTP кэширование?

`Cache-Control: max-age=N` — кэшировать на N секунд (no-request). `ETag` — fingerprint контента, `If-None-Match` — условный запрос (304 Not Modified). `Last-Modified` / `If-Modified-Since` — аналог через дату. Стратегия: статика с hash в имени — `max-age=31536000, immutable`; HTML — `no-cache` (проверять каждый раз).

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое Web Workers и Shared Workers?

**Web Worker** — отдельный поток JS, изолированный от main thread и других воркеров. **Shared Worker** — разделяется между несколькими вкладками одного origin. **Service Worker** — специальный воркер с сетевыми перехватами. Общение через `postMessage` / `MessageChannel`.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работает WebAssembly?

WebAssembly (Wasm) — бинарный формат, выполняемый браузером близко к native скорости. Компилируется из C/C++/Rust/Go. Используется для: CPU-heavy задач (image/video processing, crypto, physics), перенос существующих библиотек в браузер. Не заменяет JS, дополняет его.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое Performance API?

`performance.now()` — высокоточный timestamp. `PerformanceObserver` — подписка на метрики (LCP, CLS, FID, resource timing, long tasks). `performance.mark`/`measure` — кастомные маркеры для профилирования.

```typescript
const observer = new PerformanceObserver(list => {
  for (const entry of list.getEntries()) {
    console.log(entry.name, entry.startTime, entry.duration);
  }
});
observer.observe({ entryTypes: ["largest-contentful-paint", "long-task"] });
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
