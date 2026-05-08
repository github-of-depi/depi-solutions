# GraphQL — Middle (нет Junior уровня)

## Вопросы

- [Что такое GraphQL и чем он отличается от REST?](#что-такое-graphql-и-чем-он-отличается-от-rest)
- [Что такое query, mutation и subscription?](#что-такое-query-mutation-и-subscription)
- [Как использовать Apollo Client?](#как-использовать-apollo-client)
- [Что такое fragments в GraphQL?](#что-такое-fragments-в-graphql)
- [Как работают variables в GraphQL?](#как-работают-variables-в-graphql)
- [Что такое Apollo cache?](#что-такое-apollo-cache)
- [Как генерировать TypeScript типы из схемы?](#как-генерировать-typescript-типы-из-схемы)

---

## Что такое GraphQL и чем он отличается от REST?

GraphQL — язык запросов для API. Клиент запрашивает точно нужные поля, нет over/under-fetching. Один endpoint вместо множества. Строго типизированная схема. Минусы: сложнее кэширование (HTTP GET), N+1 проблема на сервере, сложнее настройка.

---

## Что такое query, mutation и subscription?

- **query** — чтение данных (idempotent)
- **mutation** — изменение данных (создание, обновление, удаление)
- **subscription** — real-time подписка через WebSocket

```graphql
query GetUser($id: ID!) {
  user(id: $id) { id name email }
}

mutation CreateUser($input: CreateUserInput!) {
  createUser(input: $input) { id name }
}

subscription OnMessage($roomId: ID!) {
  messageAdded(roomId: $roomId) { id text author { name } }
}
```

---

## Как использовать Apollo Client?

```typescript
const client = new ApolloClient({ uri: "/graphql", cache: new InMemoryCache() });

// useQuery
const { data, loading, error } = useQuery(GET_USER, { variables: { id: "1" } });

// useMutation
const [createUser, { loading }] = useMutation(CREATE_USER, {
  update(cache, { data: { createUser } }) {
    cache.modify({ fields: { users: (existing) => [...existing, createUser] } });
  },
});
```

---

## Что такое fragments в GraphQL?

Fragments — переиспользуемые части запросов. Избегают дублирования полей между запросами. Collocated fragments (рядом с компонентом) — паттерн для data co-location.

```graphql
fragment UserFields on User { id name email avatar }

query GetUsers { users { ...UserFields } }
query GetUser($id: ID!) { user(id: $id) { ...UserFields orders { id } } }
```

---

## Как работают variables в GraphQL?

Variables — параметры запроса, переданные отдельно от query string. Типобезопасны, кэшируемы, не требуют строковой интерполяции (защита от инъекций).

---

## Что такое Apollo cache?

Apollo InMemoryCache нормализует данные по `__typename + id`. При обновлении одного объекта — все запросы, содержащие его, автоматически обновляются. `cache.modify` и `cache.writeFragment` для ручного обновления кэша после мутации.

---

## Как генерировать TypeScript типы из схемы?

`graphql-codegen` генерирует TypeScript типы + типизированные хуки из GraphQL схемы и операций.

```yaml
# codegen.yml
generates:
  src/generated/graphql.ts:
    plugins: ["typescript", "typescript-operations", "typescript-react-apollo"]
```
