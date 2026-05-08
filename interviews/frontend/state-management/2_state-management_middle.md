# State Management — Middle

## Вопросы

- [Как устроен Redux Toolkit slice?](#как-устроен-redux-toolkit-slice)
- [Что такое RTK Query?](#что-такое-rtk-query)
- [Как работает Zustand — подписки и селекторы?](#как-работает-zustand--подписки-и-селекторы)
- [Redux vs Zustand — когда что выбирать?](#redux-vs-zustand--когда-что-выбирать)
- [Что такое иммутабельность и зачем она в Redux?](#что-такое-иммутабельность-и-зачем-она-в-redux)
- [Что такое мемоизированные селекторы (Reselect)?](#что-такое-мемоизированные-селекторы-reselect)
- [Как работает middleware в Redux?](#как-работает-middleware-в-redux)

---

## Как устроен Redux Toolkit slice?

Slice объединяет reducer и action creators для одного домена состояния. `createSlice` автоматически генерирует action creators по именам редьюсеров. Внутри slice можно писать «мутирующий» код — Immer под капотом обеспечивает иммутабельность.

```typescript
const counterSlice = createSlice({
  name: "counter",
  initialState: { value: 0 },
  reducers: {
    increment: state => { state.value += 1; }, // Immer!
    addAmount: (state, action: PayloadAction<number>) => {
      state.value += action.payload;
    },
  },
});
export const { increment, addAmount } = counterSlice.actions;
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое RTK Query?

RTK Query — встроенный в Redux Toolkit инструмент для управления серверным состоянием: кэширование, дедупликация запросов, автоматические загрузочные состояния, инвалидация. Генерирует хуки из описания эндпоинтов.

```typescript
const api = createApi({
  reducerPath: "api",
  baseQuery: fetchBaseQuery({ baseUrl: "/api" }),
  endpoints: builder => ({
    getUser: builder.query<User, number>({
      query: id => `/users/${id}`,
      providesTags: ["User"],
    }),
    updateUser: builder.mutation<User, Partial<User>>({
      query: body => ({ url: `/users/${body.id}`, method: "PATCH", body }),
      invalidatesTags: ["User"],
    }),
  }),
});
export const { useGetUserQuery, useUpdateUserMutation } = api;
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работает Zustand — подписки и селекторы?

Zustand компонент подписывается только на часть стора через селектор. Если выбранная часть не изменилась (по ссылке) — ре-рендера нет. Для сложных вычислений используй `useShallow` или внешние мемоизированные селекторы.

```typescript
// Подписка только на count — ре-рендер только при изменении count
const count = useStore(state => state.count);

// useShallow — сравнение объектов/массивов
import { useShallow } from "zustand/react/shallow";
const { count, name } = useStore(useShallow(s => ({ count: s.count, name: s.name })));
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Redux vs Zustand — когда что выбирать?

| Критерий | Redux Toolkit | Zustand |
|----------|--------------|---------|
| Boilerplate | Умеренный | Минимальный |
| DevTools | Отличные | Базовые |
| Middleware | Мощные | Через middleware option |
| Серверное состояние | RTK Query | TanStack Query |
| Размер команды | Большая | Любая |

Redux — для крупных проектов с историей действий, time-travel debugging, сложными side effects. Zustand — для большинства остальных случаев.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое иммутабельность и зачем она в Redux?

Иммутабельность — данные не изменяются напрямую, создаётся новая копия. Redux требует иммутабельных обновлений для: корректной работы React (сравнение ссылок при ре-рендере), time-travel debugging (сохранение истории), предсказуемости. RTK использует Immer, позволяя писать "мутирующий" синтаксис безопасно.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое мемоизированные селекторы (Reselect)?

`createSelector` из Reselect создаёт мемоизированный селектор: пересчитывается только если изменились входные данные. Предотвращает лишние ре-рендеры при сложных вычислениях из стора. RTK экспортирует `createSelector`.

```typescript
const selectCompletedTodos = createSelector(
  (state: RootState) => state.todos,
  todos => todos.filter(t => t.completed) // пересчёт только при изменении todos
);
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работает middleware в Redux?

Middleware — функции между dispatch и reducer: `store => next => action => next(action)`. Используются для: логирования, async операций (redux-thunk встроен в RTK), аналитики. `redux-thunk` позволяет dispatch-ить функции вместо plain objects.

```typescript
// Кастомный logger middleware
const logger = store => next => action => {
  console.log("dispatching:", action);
  const result = next(action);
  console.log("next state:", store.getState());
  return result;
};
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
