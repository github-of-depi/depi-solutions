# TanStack Query — Expert

## Вопросы

- [Как реализовать кастомный QueryClient с middleware?](#как-реализовать-кастомный-queryclient-с-middleware)
- [Как оптимизировать производительность при большом количестве queries?](#как-оптимизировать-производительность-при-большом-количестве-queries)
- [Как строить offline-first приложение с TanStack Query?](#как-строить-offline-first-приложение-с-tanstack-query)
- [Как тестировать компоненты с TanStack Query?](#как-тестировать-компоненты-с-tanstack-query)

---

## Как реализовать кастомный QueryClient с middleware?

```typescript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000,
      gcTime: 10 * 60 * 1000,
      retry: (count, error) => !(error instanceof NotFoundError) && count < 3,
      queryFn: async ({ queryKey }) => {
        // глобальный fetcher
        const res = await fetch(`/api/${queryKey.join("/")}`);
        if (!res.ok) throw new ApiError(res.status);
        return res.json();
      },
    },
    mutations: {
      onError: (error) => globalErrorHandler(error),
    },
  },
});
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как оптимизировать производительность при большом количестве queries?

1. **`select`** — трансформировать данные в hook, компонент ре-рендерится только при изменении выбранных данных
2. **Batch invalidation** — `invalidateQueries` с широким фильтром вместо множества вызовов
3. **`notifyOnChangeProps`** — ограничить, на какие изменения реагировать
4. **Structural sharing** — TQ встроенно, не клонирует неизменённые части
5. Разделять read-heavy и write-heavy queries на разные QueryClient инстансы (advanced)

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как строить offline-first приложение с TanStack Query?

`persistQueryClient` + `createSyncStoragePersister` (localStorage) или `createAsyncStoragePersister` (IndexedDB). Кэш сохраняется между сессиями. Мутации в offline — через `experimental_createPersister` или кастомную очередь.

```typescript
import { persistQueryClient } from "@tanstack/react-query-persist-client";
import { createSyncStoragePersister } from "@tanstack/query-sync-storage-persister";

persistQueryClient({
  queryClient,
  persister: createSyncStoragePersister({ storage: window.localStorage }),
  maxAge: 24 * 60 * 60 * 1000, // 24 часа
});
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как тестировать компоненты с TanStack Query?

Создавать свежий QueryClient для каждого теста. Мокировать `queryFn` или использовать MSW (Mock Service Worker) для перехвата запросов.

```typescript
const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } },
  });
  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  );
};

it("показывает пользователей", async () => {
  server.use(rest.get("/api/users", (_, res, ctx) => res(ctx.json(mockUsers))));
  render(<UserList />, { wrapper: createWrapper() });
  expect(await screen.findByText("Alice")).toBeInTheDocument();
});
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
