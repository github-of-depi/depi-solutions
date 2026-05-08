# GraphQL — Expert

## Вопросы

- [Что такое Schema Federation?](#что-такое-schema-federation)
- [Как реализовать кастомные scalar типы?](#как-реализовать-кастомные-scalar-типы)
- [Что такое persisted queries?](#что-такое-persisted-queries)
- [Как строить GraphQL micro-frontends?](#как-строить-graphql-micro-frontends)

---

## Что такое Schema Federation?

Apollo Federation позволяет разделить GraphQL схему между несколькими сервисами (subgraphs). Gateway объединяет их в единую схему для клиента. Каждый subgraph определяет свою часть типов, может расширять типы других.

---

## Как реализовать кастомные scalar типы?

Кастомные scalars для нестандартных типов: Date, JSON, URL, EmailAddress. На клиенте: Apollo `possibleTypes` + кастомный scalar handler.

```graphql
scalar DateTime
scalar JSON
type Event { id: ID! createdAt: DateTime! metadata: JSON }
```

---

## Что такое persisted queries?

Persisted queries: клиент отправляет hash операции вместо полного текста. Сервер кэширует операции по hash. Преимущества: меньше payload, защита от произвольных запросов (только заранее одобренные). Автоматически через Apollo Client + persisted queries link.

---

## Как строить GraphQL micro-frontends?

Каждый micro-frontend имеет свой GraphQL клиент. Federation gateway — единая точка для всех. Collocated fragments в каждом MFE — данные загружаются от нужного subgraph прозрачно.
