# REST — Expert

## Вопросы

- [Что такое HTTP/3 (QUIC) и когда его использовать?](#что-такое-http3-quic-и-когда-его-использовать)
- [Как проектировать API Gateway для frontend?](#как-проектировать-api-gateway-для-frontend)
- [Как реализовать offline-first с REST?](#как-реализовать-offline-first-с-rest)
- [Что такое gRPC-Web и когда он лучше REST?](#что-такое-grpc-web-и-когда-он-лучше-rest)

---

## Что такое HTTP/3 (QUIC) и когда его использовать?

HTTP/3 работает поверх QUIC (UDP-based). Преимущества: нет head-of-line blocking на транспортном уровне (в HTTP/2 потеря одного TCP пакета блокирует все streams), быстрое восстановление соединения (0-RTT reconnect), лучше на потере пакетов (мобильные сети). Поддерживается Cloudflare, Vercel, nginx 1.25+.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как проектировать API Gateway для frontend?

API Gateway (Kong, AWS API GW, Nginx) = единая точка входа для всех API. Функции: аутентификация (JWT validation), rate limiting, routing, трансформация запросов/ответов, логирование, кэширование. BFF поверх API Gateway — агрегация для UI.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как реализовать offline-first с REST?

1. **Service Worker** — перехват запросов, отдача из кэша при offline
2. **Background Sync API** — отложенная отправка при восстановлении сети
3. **Optimistic updates** + **queue** — накапливать мутации, синхронизировать при online
4. **Conflict resolution** — last-write-wins или CRDT при конфликтах

```typescript
// Service Worker — cache-first стратегия
self.addEventListener("fetch", event => {
  event.respondWith(
    caches.match(event.request).then(cached =>
      cached ?? fetch(event.request).then(response => {
        const clone = response.clone();
        caches.open("v1").then(cache => cache.put(event.request, clone));
        return response;
      })
    )
  );
});
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое gRPC-Web и когда он лучше REST?

gRPC — бинарный протокол на основе Protobuf, строгая типизация, code generation для клиентов. gRPC-Web — адаптер для браузеров. Лучше REST: бинарный формат (меньше трафика), строгий контракт (Protobuf), streaming (server/bi-directional), автогенерация клиентов. Хуже: сложнее отладка, не поддерживается через `fetch` напрямую.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
