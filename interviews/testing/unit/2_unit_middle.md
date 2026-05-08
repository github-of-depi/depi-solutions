# Unit Testing — Middle

## Вопросы

- [Как тестировать хуки?](#как-тестировать-хуки)
- [Как тестировать async операции?](#как-тестировать-async-операции)
- [Как мокировать модули в Jest?](#как-мокировать-модули-в-jest)
- [Что такое test coverage и как его интерпретировать?](#что-такое-test-coverage-и-как-его-интерпретировать)
- [Как тестировать с MSW?](#как-тестировать-с-msw)

---

## Как тестировать хуки?

`renderHook` из `@testing-library/react` для тестирования хуков изолированно.

```typescript
import { renderHook, act } from "@testing-library/react";
import { useCounter } from "./useCounter";

test("инкремент увеличивает счётчик", () => {
  const { result } = renderHook(() => useCounter(0));
  act(() => result.current.increment());
  expect(result.current.count).toBe(1);
});
```

---

## Как тестировать async операции?

`waitFor` для ожидания async обновлений. `findBy*` queries — async версии `getBy*`.

```typescript
test("загружает пользователей", async () => {
  render(<UserList />);
  expect(screen.getByText(/загрузка.../i)).toBeInTheDocument();
  const users = await screen.findAllByRole("listitem");
  expect(users).toHaveLength(3);
});
```

---

## Как мокировать модули в Jest?

```typescript
// Мок всего модуля
jest.mock("./api");
// Мок конкретной функции
jest.mock("./api", () => ({ fetchUser: jest.fn() }));

// В тесте
import { fetchUser } from "./api";
(fetchUser as jest.Mock).mockResolvedValue({ id: 1 });

// Автоматический reset
afterEach(() => jest.clearAllMocks());
```

---

## Что такое test coverage и как его интерпретировать?

Coverage показывает процент кода, выполненного тестами. Метрики: statements, branches, functions, lines. 80% — распространённый target. Высокий coverage ≠ хорошие тесты. Фокус на ветках (branches) важнее строк.

---

## Как тестировать с MSW?

MSW (Mock Service Worker) перехватывает реальные HTTP запросы. Одни handlers работают в браузере и в тестах.

```typescript
// handlers.ts
export const handlers = [
  http.get("/api/users", () => HttpResponse.json([{ id: 1, name: "Alice" }])),
];

// setup.ts
beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```
