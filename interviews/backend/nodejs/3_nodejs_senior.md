# Node.js — Senior

## Вопросы

- [Как строить production-ready Node.js сервер?](#как-строить-production-ready-nodejs-сервер)
- [Что такое worker_threads?](#что-такое-worker_threads)
- [Как оптимизировать производительность Node.js?](#как-оптимизировать-производительность-nodejs)

---

## Как строить production-ready Node.js сервер?

1. **Graceful shutdown**: SIGTERM → прекратить принимать запросы → дождаться завершения активных → закрыть соединения
2. **Health checks**: `/health` endpoint для load balancer
3. **Clustering**: `cluster` модуль или PM2 — использовать все CPU ядра
4. **Rate limiting**: `express-rate-limit` — защита от DoS
5. **Security headers**: `helmet` middleware
6. **Logging**: pino (самый быстрый JSON logger) + correlation ID

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое worker_threads?

Worker Threads — реальные потоки для CPU-интенсивных задач (не I/O). Не прерывают event loop. Общая память через `SharedArrayBuffer`. Кейсы: image processing, crypto, heavy computation.

```typescript
import { Worker } from "worker_threads";
const worker = new Worker("./worker.js", { workerData: { imageBuffer } });
worker.on("message", (result) => sendResponse(result));
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как оптимизировать производительность Node.js?

1. **Не блокировать event loop**: тяжёлые вычисления → worker_threads
2. **Connection pooling**: pg-pool, Mongoose poolSize
3. **Caching**: Redis для частых запросов
4. **Streaming**: streaming ответов для больших данных
5. **Profiling**: Node.js built-in profiler, clinic.js, 0x

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
