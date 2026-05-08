# GraphQL — Middle

## Вопросы

> Это продолжение от файла 1_graphql_junior.md который содержит базовый уровень.
> Данный файл — паттерны и работа с Apollo.

- [Как работает Apollo Link?](#как-работает-apollo-link)
- [Как обрабатывать ошибки в GraphQL?](#как-обрабатывать-ошибки-в-graphql)
- [Что такое optimistic response?](#что-такое-optimistic-response)
- [Как работает polling и refetching?](#как-работает-polling-и-refetching)
- [Что такое Apollo Local State?](#что-такое-apollo-local-state)
- [Как работает fetchPolicy?](#как-работает-fetchpolicy)
- [Что такое Urql и когда он лучше Apollo?](#что-такое-urql-и-когда-он-лучше-apollo)

---

## Как работает Apollo Link?

Apollo Link — middleware pipeline для запросов. Каждый link обрабатывает операцию и передаёт дальше. Типичная цепочка: authLink → errorLink → httpLink.

```typescript
const authLink = new ApolloLink((operation, forward) => {
  operation.setContext({ headers: { authorization: `Bearer ${getToken()}` } });
  return forward(operation);
});
const client = new ApolloClient({ link: from([authLink, errorLink, httpLink]) });
```

---

## Как обрабатывать ошибки в GraphQL?

GraphQL возвращает `200 OK` даже при ошибках — ошибки в поле `errors`. Apollo: `onError` link для глобальной обработки, `error` в `useQuery`/`useMutation` для локальной.

```typescript
const errorLink = onError(({ graphQLErrors, networkError }) => {
  if (graphQLErrors) graphQLErrors.forEach(e => console.error(e.message));
  if (networkError) handleNetworkError(networkError);
});
```

---

## Что такое optimistic response?

`optimisticResponse` — мгновенное обновление UI до ответа сервера. Apollo временно записывает в кэш.

```typescript
addTodo({ variables: { text },
  optimisticResponse: {
    addTodo: { __typename: "Todo", id: "temp", text, done: false }
  }
});
```

---

## Как работает polling и refetching?

`pollInterval` — автоматический ре-фетч каждые N мс. `refetch()` — ручной ре-фетч. `networkOnly` fetchPolicy — всегда с сервера.

---

## Что такое Apollo Local State?

Локальное состояние в Apollo cache через `@client` директиву или reactive variables. Позволяет использовать одинаковый API для серверных и клиентских данных.

---

## Как работает fetchPolicy?

- `cache-first` (default) — кэш → сеть если нет
- `cache-and-network` — кэш сразу + обновление из сети
- `network-only` — только сеть, пишет в кэш
- `no-cache` — только сеть, не кэширует
- `cache-only` — только кэш

---

## Что такое Urql и когда он лучше Apollo?

Urql — легковесный GraphQL клиент. Меньше boilerplate, лучше tree shaking, exchanges (аналог Apollo links). Лучше Apollo для: небольших проектов, когда не нужна сложная нормализация кэша. Apollo лучше для: complex caching, большой экосистемы.
