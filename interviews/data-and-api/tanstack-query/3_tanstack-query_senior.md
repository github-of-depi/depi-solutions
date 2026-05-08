# TanStack Query — Senior

## Вопросы

- [Как работает кэширование и нормализация в TanStack Query?](#как-работает-кэширование-и-нормализация-в-tanstack-query)
- [Как использовать prefetchQuery и dehydrate для SSR?](#как-использовать-prefetchquery-и-dehydrate-для-ssr)
- [Как работает Suspense mode?](#как-работает-suspense-mode)
- [Как реализовать real-time обновления с TanStack Query?](#как-реализовать-real-time-обновления-с-tanstack-query)
- [Как строить кастомные хуки поверх TanStack Query?](#как-строить-кастомные-хуки-поверх-tanstack-query)

---

## Как работает кэширование и нормализация в TanStack Query?

TQ использует queryKey как ключ кэша. Нет встроенной нормализации (в отличие от Apollo/RTK Query). Паттерн: использовать `setQueryData` для обновления всех затронутых queries после мутации, или использовать `queryClient.setQueriesData` для массового обновления.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как использовать prefetchQuery и dehydrate для SSR?

На сервере: `queryClient.prefetchQuery` → `dehydrate(queryClient)`. На клиенте: `HydrationBoundary` восстанавливает кэш. В Next.js App Router — через server component.

```typescript
// Server Component (Next.js)
const queryClient = new QueryClient();
await queryClient.prefetchQuery({ queryKey: ["users"], queryFn: getUsers });
return (
  <HydrationBoundary state={dehydrate(queryClient)}>
    <UserList />
  </HydrationBoundary>
);
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работает Suspense mode?

`useSuspenseQuery` — query в режиме Suspense: компонент "приостанавливается" пока данные загружаются. Нет нужды в `isLoading` проверках. `data` всегда defined. Оборачивать в `<Suspense>`.

```typescript
function UserList() {
  const { data } = useSuspenseQuery({ queryKey: ["users"], queryFn: getUsers });
  return <ul>{data.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}
// <Suspense fallback={<Spinner />}><UserList /></Suspense>
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как реализовать real-time обновления с TanStack Query?

WebSocket + `queryClient.setQueryData` для обновления кэша из WS сообщений. Или TanStack Query Observer + polling (для простых случаев).

```typescript
useEffect(() => {
  const ws = new WebSocket(wsUrl);
  ws.onmessage = (e) => {
    const update = JSON.parse(e.data);
    queryClient.setQueryData(["price", update.symbol], update);
  };
  return () => ws.close();
}, []);
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как строить кастомные хуки поверх TanStack Query?

Инкапсулировать queryKey, queryFn и опции в кастомный хук. Типизировать через generics.

```typescript
function useUser(id: number) {
  return useQuery({
    queryKey: ["users", id],
    queryFn: () => fetchUser(id),
    enabled: id > 0,
    select: (data) => ({
      ...data,
      fullName: `${data.firstName} ${data.lastName}`,
    }),
  });
}

function useCreateUser() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: createUser,
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ["users"] }),
  });
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
