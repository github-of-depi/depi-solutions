# Архитектурные паттерны — Senior

## Вопросы

- [Что такое Domain-Driven Design (DDD)?](#что-такое-domain-driven-design-ddd)
- [Как применять DDD на фронтенде?](#как-применять-ddd-на-фронтенде)
- [Что такое паттерн Mediator?](#что-такое-паттерн-mediator)
- [Что такое CQRS и Event Sourcing?](#что-такое-cqrs-и-event-sourcing)
- [Как выбрать паттерн для конкретной задачи?](#как-выбрать-паттерн-для-конкретной-задачи)

---

## Что такое Domain-Driven Design (DDD)?

DDD — подход к разработке ПО с акцентом на доменную модель (бизнес-логику). Ключевые концепции: Ubiquitous Language (единый язык команды и бизнеса), Bounded Context (ограниченный контекст), Entities/Value Objects/Aggregates, Domain Services, Repository, Domain Events.

---

## Как применять DDD на фронтенде?

DDD на фронтенде в рамках Feature-Sliced Design:
- **Entities** — бизнес-объекты (User, Order) с логикой
- **Bounded Contexts** — отдельные фичи как изолированные контексты
- **Domain Events** — общение через события (не прямые зависимости)
- **Value Objects** — типы с бизнес-валидацией (Email, Money)

---

## Что такое паттерн Mediator?

Mediator — объект, координирующий взаимодействие других объектов. Уменьшает coupling: объекты не знают друг о друге, только о медиаторе. В UI: event bus, Redux store как медиатор между компонентами.

---

## Что такое CQRS и Event Sourcing?

**CQRS** (Command Query Responsibility Segregation) — разделение операций чтения и записи. На фронтенде: RTK Query (queries) + useMutation (commands). **Event Sourcing** — состояние как последовательность событий, а не текущее значение. Redux reducers — вариация Event Sourcing.

---

## Как выбрать паттерн для конкретной задачи?

- Взаимозаменяемые алгоритмы → Strategy
- Подписка на события → Observer/EventEmitter
- Упрощение интерфейса → Facade
- Абстракция данных → Repository
- История действий → Command + Memento
- Координация → Mediator
- Создание объектов → Factory/Builder

Принцип: решать конкретную проблему, не навязывать паттерн ради паттерна.
