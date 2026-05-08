# REST — Senior

## Вопросы

- [Как работает HTTP кэширование — ETag и Cache-Control?](#как-работает-http-кэширование--etag-и-cache-control)
- [Как проектировать REST API с точки зрения фронтенда?](#как-проектировать-rest-api-с-точки-зрения-фронтенда)
- [Что такое Rate Limiting и как его обрабатывать?](#что-такое-rate-limiting-и-как-его-обрабатывать)
- [Как работает HTTP/2 и что он даёт?](#как-работает-http2-и-что-он-даёт)
- [Что такое BFF (Backend for Frontend)?](#что-такое-bff-backend-for-frontend)
- [Как реализовать optimistic updates с REST API?](#как-реализовать-optimistic-updates-с-rest-api)

---

## Как работает HTTP кэширование — ETag и Cache-Control?

`Cache-Control: max-age=3600` — кэшировать 1 час, без запроса. `ETag: "abc123"` — fingerprint ресурса. При повторном запросе: `If-None-Match: "abc123"` → сервер возвращает `304 Not Modified` без тела. `stale-while-revalidate`: отдать кэш, обновить в фоне — лучший UX.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как проектировать REST API с точки зрения фронтенда?

1. **Версионирование**: `/api/v1/` — для обратной совместимости
2. **Консистентные ошибки**: `{ error: { code, message, details } }`
3. **Pagination**: cursor-based для бесконечной прокрутки
4. **Sparse fieldsets**: `?fields=id,name` — только нужные поля
5. **Nesting**: максимум 2 уровня (`/users/1/orders`) — глубже через query params

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое Rate Limiting и как его обрабатывать?

Rate Limiting — ограничение количества запросов. Сервер возвращает `429 Too Many Requests` с `Retry-After: 30` заголовком. Обработка: exponential backoff + jitter, показ пользователю сообщения, очередь запросов.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работает HTTP/2 и что он даёт?

HTTP/2: мультиплексирование (несколько запросов в одном TCP соединении), бинарный протокол, сжатие заголовков (HPACK), server push. Устраняет head-of-line blocking HTTP/1. HTTP/3 (QUIC) — поверх UDP, ещё быстрее на ненадёжных соединениях.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое BFF (Backend for Frontend)?

BFF — отдельный API-слой для конкретного UI клиента (web, mobile). Агрегирует вызовы к микросервисам, адаптирует данные под нужды клиента, хранит сессию. Next.js API routes / Route Handlers — легковесный BFF для веб-клиента.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как реализовать optimistic updates с REST API?

Немедленно обновить UI, параллельно отправить запрос. При ошибке — откатить. TanStack Query `onMutate`/`onError`/`onSettled` — стандартный паттерн.

```typescript
const mutation = useMutation({
  mutationFn: toggleTodo,
  onMutate: async (id) => {
    await queryClient.cancelQueries({ queryKey: ["todos"] });
    const prev = queryClient.getQueryData<Todo[]>(["todos"]);
    queryClient.setQueryData(["todos"], (old: Todo[]) =>
      old.map(t => t.id === id ? { ...t, done: !t.done } : t)
    );
    return { prev };
  },
  onError: (_, __, ctx) => queryClient.setQueryData(["todos"], ctx?.prev),
  onSettled: () => queryClient.invalidateQueries({ queryKey: ["todos"] }),
});
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
