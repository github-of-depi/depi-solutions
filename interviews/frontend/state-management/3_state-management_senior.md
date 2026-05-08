# State Management — Senior

## Вопросы

- [Как нормализовать состояние в Redux?](#как-нормализовать-состояние-в-redux)
- [Как оптимизировать ре-рендеры при использовании глобального состояния?](#как-оптимизировать-ре-рендеры-при-использовании-глобального-состояния)
- [Как организовать архитектуру состояния в большом приложении?](#как-организовать-архитектуру-состояния-в-большом-приложении)
- [TanStack Query vs RTK Query — в чём разница?](#tanstack-query-vs-rtk-query--в-чём-разница)
- [Что такое атомарное состояние (Jotai, Recoil)?](#что-такое-атомарное-состояние-jotai-recoil)
- [Как обеспечить совместимость состояния с Server Components?](#как-обеспечить-совместимость-состояния-с-server-components)

---

## Как нормализовать состояние в Redux?

Нормализация — хранение сущностей в flat-структуре (словарь по id) вместо вложенных массивов. Устраняет дублирование, упрощает обновление. `createEntityAdapter` из RTK автоматизирует CRUD операции над нормализованными данными.

```typescript
const usersAdapter = createEntityAdapter<User>();
const usersSlice = createSlice({
  name: "users",
  initialState: usersAdapter.getInitialState(),
  reducers: {
    addUser: usersAdapter.addOne,
    updateUser: usersAdapter.updateOne,
    removeUser: usersAdapter.removeOne,
  },
});
// State: { ids: [1,2,3], entities: { 1: {...}, 2: {...} } }
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как оптимизировать ре-рендеры при использовании глобального состояния?

1. **Гранулярные селекторы** — подписываться только на нужные поля, не на весь стор
2. **Мемоизация** — `createSelector` для производных данных
3. **Split stores** (Zustand) — несколько маленьких сторов вместо одного большого
4. **Context splitting** — разделить часто и редко обновляемые данные в разные Context
5. **React.memo** + `useCallback` для компонентов, получающих callbacks из стора

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как организовать архитектуру состояния в большом приложении?

Разделяй состояние по доменам: серверное (TanStack Query/RTK Query) — отдельно, UI state — локально или Zustand slice, глобальный UI (тема, текущий пользователь) — Context или маленький Zustand стор. Не помещай всё в Redux — это антипаттерн.

```
store/
  auth.slice.ts        → текущий пользователь
  ui.slice.ts          → sidebar open, theme
api/
  users.query.ts       → TanStack Query — данные с сервера
components/
  List.tsx             → useQuery — локальный loading/error state
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## TanStack Query vs RTK Query — в чём разница?

| | TanStack Query | RTK Query |
|---|---|---|
| Зависимость | Независим | Требует Redux |
| Фреймворки | React, Vue, Svelte, Solid | Только Redux |
| DX | Отличный, богатые возможности | Хороший, интегрирован с Redux |
| Optimistic UI | Ручная настройка | Через `onQueryStarted` |
| Infinite queries | Встроено | Ограничено |

Выбирай TanStack Query если нет Redux в проекте. RTK Query если уже используешь Redux.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое атомарное состояние (Jotai, Recoil)?

Атомарный подход: состояние разбивается на маленькие независимые единицы (атомы). Компонент подписывается только на нужные атомы — нет проблемы over-rendering всего дерева. `Jotai` — минималистичная реализация: `atom()` + `useAtom()`. Подходит для сложного взаимозависимого UI-состояния.

```typescript
import { atom, useAtom } from "jotai";
const countAtom = atom(0);
const doubledAtom = atom(get => get(countAtom) * 2); // derived atom

function Counter() {
  const [count, setCount] = useAtom(countAtom);
  const doubled = useAtomValue(doubledAtom);
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как обеспечить совместимость состояния с Server Components?

Server Components не имеют доступа к браузерным API и React state. Глобальный стор (Zustand, Redux) инициализируется только в Client Components. Правило: серверные данные — через RSC props или TanStack Query. Клиентский стор инициализируется в корневом Client Component через hydration из серверных props.

```typescript
// Паттерн: StoreInitializer Client Component
"use client";
function StoreInitializer({ initialUser }: { initialUser: User }) {
  useEffect(() => {
    useAuthStore.setState({ user: initialUser });
  }, []);
  return null;
}
// В Server Layout: <StoreInitializer initialUser={await getUser()} />
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
