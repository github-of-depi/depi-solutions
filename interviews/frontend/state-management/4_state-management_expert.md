# State Management — Expert

## Вопросы

- [Как реализовать state machine для управления сложным UI (XState)?](#как-реализовать-state-machine-для-управления-сложным-ui-xstate)
- [Как построить offline-first архитектуру состояния?](#как-построить-offline-first-архитектуру-состояния)
- [Как реализовать collaborative state (real-time синхронизация)?](#как-реализовать-collaborative-state-real-time-синхронизация)
- [Как оптимизировать память при большом объёме данных в сторе?](#как-оптимизировать-память-при-большом-объёме-данных-в-сторе)
- [Как тестировать сложную логику состояния?](#как-тестировать-сложную-логику-состояния)

---

## Как реализовать state machine для управления сложным UI (XState)?

State machines явно описывают все допустимые состояния и переходы, устраняя класс багов «невозможное состояние» (например, `isLoading && isError && data`). XState — полноценная реализация с иерархическими машинами, guards, actions, invoke (Promise/Observable).

```typescript
import { createMachine, assign } from "xstate";
const fetchMachine = createMachine({
  initial: "idle",
  states: {
    idle: { on: { FETCH: "loading" } },
    loading: {
      invoke: {
        src: (ctx) => fetchUser(ctx.id),
        onDone: { target: "success", actions: assign({ user: (_, e) => e.data }) },
        onError: { target: "failure", actions: assign({ error: (_, e) => e.data }) },
      },
    },
    success: { on: { REFETCH: "loading" } },
    failure: { on: { RETRY: "loading" } },
  },
});
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как построить offline-first архитектуру состояния?

Offline-first: операции сохраняются в локальную очередь (IndexedDB), синхронизируются с сервером при восстановлении сети. Ключевые паттерны: оптимистичные обновления, conflict resolution, idempotent mutations. Библиотеки: TanStack Query (persistQueryClient), RxDB, WatermelonDB, Electric SQL.

Архитектура: локальный SQLite/IndexedDB как source of truth → фоновая синхронизация → CRDT для conflict resolution в коллаборативных приложениях.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как реализовать collaborative state (real-time синхронизация)?

CRDT (Conflict-free Replicated Data Types) — структуры данных, допускающие независимые изменения с детерминированным слиянием. Используй Yjs или Automerge — они управляют состоянием документа с поддержкой real-time collaborative editing. Zustand + Yjs через `zustand-yjs` provider.

```typescript
import * as Y from "yjs";
import { WebsocketProvider } from "y-websocket";

const doc = new Y.Doc();
const provider = new WebsocketProvider("wss://sync.example.com", "room-id", doc);
const yMap = doc.getMap("state");

// Все изменения автоматически синхронизируются между клиентами
yMap.observe(e => store.setState(yMap.toJSON()));
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как оптимизировать память при большом объёме данных в сторе?

1. **Pagination/windowing** в сторе — не хранить все 100k записей, только видимую страницу
2. **LRU cache** для нормализованных сущностей — TanStack Query делает это автоматически (gcTime)
3. **WeakRef** для кэширования крупных объектов — GC может их собрать
4. **Структурное шаринг** (structural sharing) — Immer и TanStack Query не клонируют неизменённые части
5. **Избегать circular references** в сторе — усложняют GC и сериализацию

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как тестировать сложную логику состояния?

1. **Reducers** — чистые функции, тестируются просто: `expect(reducer(state, action)).toEqual(expected)`
2. **Thunks** — мокировать API, проверять dispatched actions через `redux-mock-store`
3. **Selectors** — unit тесты с набором state fixtures
4. **XState machines** — `@xstate/test` генерирует тест-планы из графа состояний
5. **Integration тесты** — renderWithStore + реальный стор, взаимодействие через userEvent

```typescript
// Тест reducer
it("increment", () => {
  expect(counterReducer({ value: 0 }, increment())).toEqual({ value: 1 });
});
// Тест selector
it("selectCompleted", () => {
  const state = { todos: [{ id: 1, done: true }, { id: 2, done: false }] };
  expect(selectCompletedTodos(state)).toHaveLength(1);
});
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
