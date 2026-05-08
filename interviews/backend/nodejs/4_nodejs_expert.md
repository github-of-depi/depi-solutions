# Node.js — Expert

## Вопросы

- [Как строить high-throughput Node.js сервис?](#как-строить-high-throughput-nodejs-сервис)
- [Что такое libuv?](#что-такое-libuv)

---

## Как строить high-throughput Node.js сервис?

1. **Fastify вместо Express** — в 2-3x быстрее (schema validation, logging)
2. **uWebSockets.js** для WebSocket — C++ native performance
3. **Async/await** везде — не блокировать event loop
4. **Database**: prepared statements, connection pooling, read replicas
5. **Caching layers**: memory cache (node-lru-cache) → Redis → DB
6. **Benchmarking**: autocannon, k6 для нагрузочного тестирования

---

## Что такое libuv?

libuv — C библиотека, реализующая event loop для Node.js. Управляет: async I/O через epoll/kqueue/IOCP, thread pool (по умолчанию 4 потока для crypto, fs, DNS). `UV_THREADPOOL_SIZE` — увеличить при CPU-heavy async operations. Node.js → V8 (JS execution) + libuv (async I/O) + C++ bindings.
