# Архитектурные паттерны — Expert

## Вопросы

- [Как строить event-driven архитектуру на фронтенде?](#как-строить-event-driven-архитектуру-на-фронтенде)
- [Что такое Hexagonal Architecture (Ports and Adapters)?](#что-такое-hexagonal-architecture-ports-and-adapters)
- [Как применять паттерны для масштабируемого фронтенда?](#как-применять-паттерны-для-масштабируемого-фронтенда)

---

## Как строить event-driven архитектуру на фронтенде?

Event-driven: компоненты/модули общаются через события, не прямые вызовы. Инструменты: EventEmitter, BroadcastChannel, Redux actions, RxJS Subject. Преимущества: слабая связанность, легко тестировать. Риски: "событийный спагетти" при злоупотреблении.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое Hexagonal Architecture (Ports and Adapters)?

Ядро приложения (доменная логика) определяет интерфейсы (ports). Адаптеры (adapters) реализуют эти интерфейсы для конкретных технологий (HTTP API, localStorage, WebSocket). Ядро не зависит от инфраструктуры — легко тестировать и менять адаптеры.

```
Domain Logic (Core)
  ├── UserRepository (port/interface)
  └── EventBus (port/interface)
  
Adapters:
  ├── ApiUserRepository implements UserRepository
  ├── LocalStorageUserRepository implements UserRepository
  └── ReduxEventBus implements EventBus
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как применять паттерны для масштабируемого фронтенда?

Принципы масштабируемости:
1. **Dependency Inversion** — зависеть от контрактов (interfaces/types)
2. **Feature isolation** — фичи не импортируют друг друга напрямую
3. **Shared kernel** — минимальный общий код между фичами
4. **Anti-corruption layer** — адаптеры при интеграции внешних API (защита от vendor lock-in)
5. **Composition over inheritance** — React-способ расширения компонентов

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
