# Browser — Senior

## Вопросы

- [Как работает критический путь рендеринга?](#как-работает-критический-путь-рендеринга)
- [Что такое Content Security Policy (CSP)?](#что-такое-content-security-policy-csp)
- [Как работает HTTP кэширование?](#как-работает-http-кэширование)
- [Что такое Web Workers и Shared Workers?](#что-такое-web-workers-и-shared-workers)
- [Как работает WebAssembly?](#как-работает-webassembly)
- [Что такое Performance API?](#что-такое-performance-api)
- [Как интегрировать Sentry во frontend?](#как-интегрировать-sentry-во-frontend)
- [Как найти утечку памяти в React-приложении?](#как-найти-утечку-памяти-в-react-приложении)
- [Как отлаживать production-сборку без sourcemaps?](#как-отлаживать-production-сборку-без-sourcemaps)

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

---

## Как интегрировать Sentry во frontend-приложение?

Sentry — платформа мониторинга ошибок: автоматически отлавливает исключения, отправляет на сервер с контекстом (стек-трейс, breadcrumbs, user info).

**Базовая настройка (React):**
```typescript
// main.tsx / index.tsx
import * as Sentry from '@sentry/react';

Sentry.init({
  dsn: import.meta.env.VITE_SENTRY_DSN,
  environment: import.meta.env.MODE, // 'production' | 'staging'
  release: import.meta.env.VITE_APP_VERSION, // для связки с sourcemaps
  tracesSampleRate: 0.1, // 10% транзакций для performance мониторинга
  integrations: [Sentry.browserTracingIntegration()],
});
```

**Source maps — видеть оригинальный код в трейсах:**
```bash
# vite.config.ts — загрузка sourcemaps при сборке:
import { sentryVitePlugin } from '@sentry/vite-plugin';

export default defineConfig({
  build: { sourcemap: true },
  plugins: [
    sentryVitePlugin({
      org: 'my-org',
      project: 'my-project',
      authToken: process.env.SENTRY_AUTH_TOKEN,
    }),
  ],
});
```

**Обогащение данных — кто получил ошибку:**
```typescript
// После логина пользователя:
Sentry.setUser({ id: user.id, email: user.email });

// Добавить контекст к ошибке:
Sentry.setContext('cart', { itemCount: cart.length, total: cart.total });

// Breadcrumbs (breadcrumbs = хлебные крошки / лог действий):
Sentry.addBreadcrumb({ message: 'User clicked checkout', level: 'info' });
```

**Error Boundary для React:**
```typescript
// Sentry предоставляет готовый Error Boundary:
import { ErrorBoundary } from '@sentry/react';

function App() {
  return (
    <ErrorBoundary fallback={<ErrorPage />} showDialog>
      <Router />
    </ErrorBoundary>
  );
}

// Ручная отправка (для обработанных ошибок):
try {
  await api.submitOrder(order);
} catch (error) {
  Sentry.captureException(error, {
    tags: { component: 'Checkout' },
    extra: { orderId: order.id },
  });
  showToast('Ошибка оформления заказа');
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как найти утечку памяти в React-приложении?

Утечка памяти — объекты не освобождаются Garbage Collector'ом, потому что на них остаются ссылки (таймеры, подписки, замыкания).

**Симптомы:**
- Память в DevTools постоянно растёт
- Страница замедляется со временем
- Предупреждение: "Can't perform state update on unmounted component"

**Инструменты Chrome DevTools:**

1. **Heap Snapshot (Memory → Take Snapshot):**
   - Сделать снапшот → выполнить действие → сделать снапшот → сравнить
   - `Comparison` view покажет объекты которые не были освобождены
   - Искать строки с `Detached` (DOM-узлы оторваны от дерева, но не GC)

2. **Allocation Timeline:**
   - `Memory → Allocation instrumentation on timeline`
   - Выполнить действие несколько раз
   - Синие столбики = аллокации, серые = освобождены, синие остающиеся = утечка

**Типичные источники утечек в React:**

```typescript
// ❌ УТЕЧКА: таймер не очищен
useEffect(() => {
  const id = setInterval(() => fetchData(), 5000);
  // нет return с clearInterval!
}, []);

// ✅ ИСПРАВЛЕНИЕ:
useEffect(() => {
  const id = setInterval(() => fetchData(), 5000);
  return () => clearInterval(id);
}, []);

// ❌ УТЕЧКА: подписка на событие
useEffect(() => {
  window.addEventListener('resize', handleResize);
}, []);

// ✅ ИСПРАВЛЕНИЕ:
useEffect(() => {
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, []);

// ❌ УТЕЧКА: fetch запрос обновляет размонтированный компонент
useEffect(() => {
  fetch('/api/data').then(r => r.json()).then(data => setState(data));
}, []);

// ✅ ИСПРАВЛЕНИЕ через AbortController:
useEffect(() => {
  const controller = new AbortController();
  fetch('/api/data', { signal: controller.signal })
    .then(r => r.json())
    .then(data => setState(data))
    .catch(e => { if (e.name !== 'AbortError') throw e; });
  return () => controller.abort();
}, []);
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как отлаживать production-сборку без sourcemaps?

Идеально — всегда иметь sourcemaps. Но если их нет, есть альтернативные стратегии.

**Стратегии:**

1. **Воспроизвести локально в production-режиме:**
   ```bash
   npm run build && npx serve dist
   # Тест с теми же данными что у пользователя
   ```

2. **Логи и мониторинг:**
   - Sentry / Datadog — ошибки с breadcrumbs
   - Структурированные логи с достаточным контекстом
   - Флаги ошибок с `errorId` для поиска по логам

3. **Feature flags + постепенный rollout:**
   - Откат фичи для affected пользователей
   - A/B testing для изоляции проблемы

4. **Source maps в приватном хранилище:**
   - Загружать sourcemaps в Sentry (не в public CDN)
   - Sentry покажет оригинальный стек, пользователи не увидят

5. **Добавить диагностику в код:**
   ```typescript
   // Версия сборки для идентификации:
   console.info('App version:', import.meta.env.VITE_APP_VERSION);

   // Custom error reporter:
   window.onerror = (msg, src, line, col, error) => {
     reportError({ msg, src, line, col, stack: error?.stack });
   };
   ```

**Превентивные меры:**
- Всегда загружать sourcemaps в Sentry при деплое
- Хранить sourcemaps отдельно от публичных assets
- Настроить `release` в Sentry для соответствия commit-у

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
