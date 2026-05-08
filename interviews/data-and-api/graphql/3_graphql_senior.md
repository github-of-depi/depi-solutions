# GraphQL — Senior

## Вопросы

- [Как оптимизировать Apollo кэш в большом приложении?](#как-оптимизировать-apollo-кэш-в-большом-приложении)
- [Как решить проблему N+1 на фронтенде?](#как-решить-проблему-n1-на-фронтенде)
- [Как строить type-safe GraphQL с кодогенерацией?](#как-строить-type-safe-graphql-с-кодогенерацией)
- [Как реализовать subscriptions в production?](#как-реализовать-subscriptions-в-production)
- [Как тестировать GraphQL компоненты?](#как-тестировать-graphql-компоненты)

---

## Как оптимизировать Apollo кэш в большом приложении?

1. **`keyFields`** — кастомные ключи нормализации (не только id)
2. **`merge`** функции — для pagination (merge новых страниц в кэш)
3. **`read`** функции — computed fields в кэше
4. **GC**: `cache.gc()` — удаление недостижимых объектов
5. Избегать `network-only` без необходимости — убивает смысл кэша

```typescript
const cache = new InMemoryCache({
  typePolicies: {
    Query: {
      fields: {
        users: { keyArgs: ["filter"], merge: (existing = [], incoming) => [...existing, ...incoming] },
      },
    },
    User: { keyFields: ["email"] }, // нормализация по email
  },
});
```

---

## Как решить проблему N+1 на фронтенде?

N+1 — нет на фронтенде (клиент делает один запрос). Но фрагменты помогают избежать N+1 на сервере: collocated fragments дают серверу всё что нужно одним запросом. Используй DataLoader на сервере.

---

## Как строить type-safe GraphQL с кодогенерацией?

graphql-codegen + `typed-document-node`: операции становятся TypedDocumentNode с встроенными типами. Полная типизация от схемы до компонента.

```typescript
// После codegen — операция со встроенными типами
const { data } = useQuery(GetUserDocument, { variables: { id: "1" } });
data?.user.name; // типизировано!
```

---

## Как реализовать subscriptions в production?

GraphQL subscriptions через WebSocket. `graphql-ws` (современный) или `subscriptions-transport-ws` (legacy). Apollo: `GraphQLWsLink` + `split` для разделения запросов и subscriptions.

```typescript
const wsLink = new GraphQLWsLink(createClient({ url: "wss://api.example.com/graphql" }));
const splitLink = split(({ query }) => {
  const def = getMainDefinition(query);
  return def.kind === "OperationDefinition" && def.operation === "subscription";
}, wsLink, httpLink);
```

---

## Как тестировать GraphQL компоненты?

MSW (`msw`) для мокирования GraphQL запросов — реальные запросы перехватываются на уровне сети.

```typescript
const handlers = [
  graphql.query("GetUser", (req, res, ctx) =>
    res(ctx.data({ user: { id: "1", name: "Alice" } }))
  ),
];
server.use(...handlers);
```
