# TanStack Query — Middle

## Вопросы

- [Как работает useMutation?](#как-работает-usemutation)
- [Как инвалидировать кэш после мутации?](#как-инвалидировать-кэш-после-мутации)
- [Как реализовать optimistic updates?](#как-реализовать-optimistic-updates)
- [Как работает useInfiniteQuery?](#как-работает-useinfinitequery)
- [Как использовать enabled и dependent queries?](#как-использовать-enabled-и-dependent-queries)
- [Как настроить retry?](#как-настроить-retry)

---

## Как работает useMutation?

`useMutation` для POST/PATCH/DELETE запросов. Возвращает `mutate`/`mutateAsync`, `isPending`, `isError`. Колбэки: `onSuccess`, `onError`, `onSettled`.

```typescript
const { mutate, isPending } = useMutation({
  mutationFn: (newUser: NewUser) => createUser(newUser),
  onSuccess: (data) => {
    queryClient.invalidateQueries({ queryKey: ["users"] });
    toast.success("Пользователь создан");
  },
  onError: (error) => toast.error(error.message),
});
```

---

## Как инвалидировать кэш после мутации?

`queryClient.invalidateQueries` помечает matching queries как stale — следующий рендер или фокус вызовет ре-фетч. `queryClient.setQueryData` — обновить кэш напрямую без запроса.

```typescript
// Инвалидировать все queries с ключом начинающимся на "users"
queryClient.invalidateQueries({ queryKey: ["users"] });

// Обновить конкретную запись в кэше
queryClient.setQueryData(["users", userId], updatedUser);
```

---

## Как реализовать optimistic updates?

```typescript
const mutation = useMutation({
  mutationFn: updateTodo,
  onMutate: async (updated) => {
    await queryClient.cancelQueries({ queryKey: ["todos"] });
    const previous = queryClient.getQueryData<Todo[]>(["todos"]);
    queryClient.setQueryData(["todos"], (old: Todo[]) =>
      old.map(t => t.id === updated.id ? { ...t, ...updated } : t)
    );
    return { previous }; // context для rollback
  },
  onError: (_, __, ctx) => queryClient.setQueryData(["todos"], ctx?.previous),
  onSettled: () => queryClient.invalidateQueries({ queryKey: ["todos"] }),
});
```

---

## Как работает useInfiniteQuery?

`useInfiniteQuery` для pagination/infinite scroll. `getNextPageParam` определяет параметр следующей страницы. `fetchNextPage` загружает следующую порцию.

```typescript
const { data, fetchNextPage, hasNextPage } = useInfiniteQuery({
  queryKey: ["posts"],
  queryFn: ({ pageParam = 1 }) => fetchPosts(pageParam),
  getNextPageParam: (lastPage, pages) => lastPage.nextCursor ?? undefined,
  initialPageParam: 1,
});
// data.pages — массив страниц
```

---

## Как использовать enabled и dependent queries?

`enabled: false` — не запускать query автоматически. Для dependent queries — включать на основе наличия данных от предыдущего запроса.

```typescript
const { data: user } = useQuery({ queryKey: ["user", id], queryFn: getUser });

const { data: orders } = useQuery({
  queryKey: ["orders", user?.id],
  queryFn: () => getOrders(user!.id),
  enabled: !!user, // запустить только после получения user
});
```

---

## Как настроить retry?

```typescript
useQuery({
  queryKey: ["data"],
  queryFn: fetchData,
  retry: 3,                // 3 попытки
  retryDelay: attempt => Math.min(1000 * 2 ** attempt, 30000), // exponential backoff
  retry: (count, error) => error.status !== 404 && count < 3,  // кастомная логика
});
```
