# Performance — Expert

## Вопросы

- [Что такое React Compiler и как он оптимизирует производительность?](#что-такое-react-compiler-и-как-он-оптимизирует-производительность)
- [Как работает streaming SSR и как он влияет на производительность?](#как-работает-streaming-ssr-и-как-он-влияет-на-производительность)
- [Как строить performance monitoring систему?](#как-строить-performance-monitoring-систему)
- [Как оптимизировать JavaScript парсинг и выполнение?](#как-оптимизировать-javascript-парсинг-и-выполнение)

---

## Что такое React Compiler и как он оптимизирует производительность?

React Compiler (React 19+, ранее React Forget) — статический компилятор, автоматически добавляющий мемоизацию. Анализирует код и оборачивает компоненты/хуки в эквивалент `useMemo`/`useCallback`/`React.memo` там, где это безопасно. Устраняет необходимость ручной мемоизации. Работает как Babel плагин / Vite transform.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работает streaming SSR и как он влияет на производительность?

Streaming SSR (React 18 + Next.js App Router) — сервер отправляет HTML по частям (HTTP chunked transfer encoding). Браузер получает и отрисовывает части страницы до завершения сервером. Suspense boundaries определяют границы стриминга. Преимущества: TTFB улучшается (первые байты раньше), FCP улучшается (контент видим быстрее), медленные данные не блокируют быстрые части.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как строить performance monitoring систему?

1. **Real User Monitoring (RUM)**: Web Vitals API → отправка в аналитику на каждой странице
2. **Sentry Performance** или **Datadog RUM**: автоматический сбор + трейсинг
3. **PerformanceObserver**: программный сбор метрик
4. **Alerting**: дашборды с p75/p95 метриками, алерты при деградации

```typescript
import { onCLS, onINP, onLCP } from "web-vitals";

onLCP(metric => sendToAnalytics({ name: metric.name, value: metric.value }));
onINP(metric => sendToAnalytics({ name: metric.name, value: metric.value }));
onCLS(metric => sendToAnalytics({ name: metric.name, value: metric.value }));
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как оптимизировать JavaScript парсинг и выполнение?

Парсинг и компиляция JS — синхронные операции, блокируют main thread. Стратегии:
1. **Меньше JS** — лучший JS-бандл тот, которого нет (Server Components, HTML-first)
2. **Code splitting** — загружать только нужный код
3. **Defer/async** скрипты — не блокировать HTML парсинг
4. **PRPL паттерн**: Push, Render, Pre-cache, Lazy-load
5. **Компрессия**: Brotli > Gzip — лучший compression ratio
6. **Module preload**: `<link rel="modulepreload">` для критических ES модулей

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
