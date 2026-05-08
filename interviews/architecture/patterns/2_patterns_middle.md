# Архитектурные паттерны — Middle

## Вопросы

- [Как применять SOLID в фронтенде?](#как-применять-solid-в-фронтенде)
- [Что такое паттерн Strategy?](#что-такое-паттерн-strategy)
- [Что такое паттерн Decorator?](#что-такое-паттерн-decorator)
- [Что такое паттерн Facade?](#что-такое-паттерн-facade)
- [Что такое паттерн Repository?](#что-такое-паттерн-repository)
- [Что такое паттерн Command?](#что-такое-паттерн-command)

---

## Как применять SOLID в фронтенде?

- **SRP**: компонент — UI, логика — хук, запросы — API слой
- **OCP**: кастомизация через props/children, не через условия внутри компонента
- **LSP**: кастомные компоненты полностью заменяют HTML элементы (Button с теми же атрибутами)
- **ISP**: не передавать весь объект если нужны 1-2 поля
- **DIP**: компонент зависит от интерфейса данных, не от конкретного API источника

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое паттерн Strategy?

Strategy — семейство алгоритмов, инкапсулированных в отдельные объекты, взаимозаменяемых. В JS: функции как стратегии.

```typescript
type SortStrategy<T> = (a: T, b: T) => number;

function sortUsers(users: User[], strategy: SortStrategy<User>): User[] {
  return [...users].sort(strategy);
}

const byName: SortStrategy<User> = (a, b) => a.name.localeCompare(b.name);
const byDate: SortStrategy<User> = (a, b) => a.createdAt.getTime() - b.createdAt.getTime();
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое паттерн Decorator?

Decorator оборачивает объект, добавляя поведение без изменения оригинала. В TypeScript: декораторы классов. В React: HOC — декоратор компонента.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое паттерн Facade?

Facade — упрощённый интерфейс для сложной системы. API слой в приложении — Facade над HTTP.

```typescript
// Facade скрывает детали fetch, auth headers, error handling
export const api = {
  getUsers: () => client.get<User[]>("/users"),
  createUser: (data: NewUser) => client.post<User>("/users", data),
};
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое паттерн Repository?

Repository абстрагирует доступ к данным. Компонент работает с репозиторием, не с конкретным API/storage. Легко мокировать в тестах, легко сменить источник данных.

```typescript
interface UserRepository { findById(id: string): Promise<User>; save(user: User): Promise<void>; }
class ApiUserRepository implements UserRepository {
  findById(id: string) { return fetch(`/api/users/${id}`).then(r => r.json()); }
  save(user: User) { return fetch(`/api/users/${user.id}`, { method: "PUT", body: JSON.stringify(user) }).then(() => {}); }
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое паттерн Command?

Command инкапсулирует запрос как объект. Поддержка undo/redo — каждая команда знает как отменить себя. Использование: редакторы, история действий.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
