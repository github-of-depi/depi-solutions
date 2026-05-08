# Unit Testing — Senior

## Вопросы

- [Что такое TDD и когда его применять?](#что-такое-tdd-и-когда-его-применять)
- [Как тестировать Redux/Zustand store?](#как-тестировать-reduxzustand-store)
- [Как строить тестовую пирамиду на проекте?](#как-строить-тестовую-пирамиду-на-проекте)
- [Как избегать brittle tests?](#как-избегать-brittle-tests)

---

## Что такое TDD и когда его применять?

TDD (Test-Driven Development): Red → Green → Refactor. Сначала пишешь тест (он падает), затем минимальный код (тест проходит), затем рефакторинг. Хорошо работает для: алгоритмов, бизнес-логики, utility функций. Сложнее для: UI компонентов, интеграций.

---

## Как тестировать Redux/Zustand store?

**Redux**: тестировать reducers как чистые функции. RTK: тестировать thunks через `createAsyncThunk`.

```typescript
import { configureStore } from "@reduxjs/toolkit";
import { cartReducer, addItem } from "./cartSlice";

test("добавляет товар в корзину", () => {
  const store = configureStore({ reducer: { cart: cartReducer } });
  store.dispatch(addItem({ id: "1", name: "Product", price: 100 }));
  expect(store.getState().cart.items).toHaveLength(1);
});
```

**Zustand**:
```typescript
import { act, renderHook } from "@testing-library/react";
import { useCartStore } from "./cartStore";
beforeEach(() => useCartStore.getState().reset());
```

---

## Как строить тестовую пирамиду на проекте?

Пирамида: много unit (быстро, дёшево) → integration (API + компонент) → E2E (дорого, медленно). Для фронтенда: 70% unit/component → 20% integration → 10% E2E. Не trophy shape (много integration) — медленно. Не ice-cream (много E2E) — дорого и нестабильно.

---

## Как избегать brittle tests?

1. Тестировать поведение, не детали реализации
2. Не использовать CSS классы как селекторы
3. Предпочитать `getByRole`/`getByText` вместо `getByTestId`
4. Не снимать snapshot с логикой — только UI snapshot
5. Избегать implementation-coupled assertions (`expect(component.state.count)`)
