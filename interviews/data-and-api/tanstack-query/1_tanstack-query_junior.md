# TanStack Query — Junior

## Вопросы

- [Что такое TanStack Query и зачем он нужен?](#что-такое-tanstack-query-и-зачем-он-нужен)
- [Как использовать useQuery?](#как-использовать-usequery)
- [Что такое staleTime и gcTime?](#что-такое-staletime-и-gctime)
- [Что такое QueryClient?](#что-такое-queryclient)
- [Как обрабатывать loading и error состояния?](#как-обрабатывать-loading-и-error-состояния)

---

## Что такое TanStack Query и зачем он нужен?

TanStack Query (React Query) — библиотека для управления серверным состоянием: кэширование, синхронизация, загрузка, обработка ошибок. Устраняет boilerplate useEffect + useState для fetch запросов. Автоматически ре-фетчит при фокусе окна, повторном подключении к сети.

---

## Как использовать useQuery?

```typescript
const { data, isLoading, isError, error } = useQuery({
  queryKey: ["users"],          // уникальный ключ для кэша
  queryFn: () => fetchUsers(),  // функция загрузки данных
  staleTime: 5 * 60 * 1000,    // данные свежие 5 минут
});
```

---

## Что такое staleTime и gcTime?

**staleTime** — как долго данные считаются свежими (нет повторного запроса). **gcTime** (было cacheTime) — как долго неиспользуемые данные хранятся в кэше перед удалением. По умолчанию: `staleTime: 0` (мгновально устаревают), `gcTime: 5 минут`.

---

## Что такое QueryClient?

`QueryClient` — центральное хранилище кэша. Создаётся один раз, передаётся через `QueryClientProvider`. Методы: `invalidateQueries`, `setQueryData`, `prefetchQuery`, `resetQueries`.

---

## Как обрабатывать loading и error состояния?

```typescript
const { data, status, isLoading, isFetching, isError, error } = useQuery({...});

if (isLoading) return <Spinner />;
if (isError) return <Error message={error.message} />;
return <UserList users={data} />;
// isFetching — true при background refetch (data уже есть)
```
