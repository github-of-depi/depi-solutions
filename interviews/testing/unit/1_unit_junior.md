# Unit Testing — Junior

## Вопросы

- [Что такое unit тест?](#что-такое-unit-тест)
- [Как писать тесты с Jest?](#как-писать-тесты-с-jest)
- [Что такое React Testing Library?](#что-такое-react-testing-library)
- [Что такое mock и stub?](#что-такое-mock-и-stub)
- [Что тестировать в первую очередь?](#что-тестировать-в-первую-очередь)

---

## Что такое unit тест?

Unit тест — проверка наименьшей изолированной единицы кода (функция, компонент). Быстрый, изолированный (нет HTTP, БД), детерминированный. Пирамида тестирования: много unit → меньше integration → мало E2E.

---

## Как писать тесты с Jest?

```typescript
import { sum } from "./math";

describe("sum", () => {
  it("складывает два числа", () => {
    expect(sum(1, 2)).toBe(3);
  });
  
  it("обрабатывает отрицательные числа", () => {
    expect(sum(-1, 1)).toBe(0);
  });
});
```

---

## Что такое React Testing Library?

RTL — тестирование компонентов с точки зрения пользователя: что видит и делает, не детали реализации. Запросы: `getByRole`, `getByText`, `getByLabelText`. Предпочтительнее `getByTestId`.

```typescript
import { render, screen, fireEvent } from "@testing-library/react";
import { Counter } from "./Counter";

test("увеличивает счётчик при нажатии", async () => {
  render(<Counter />);
  const button = screen.getByRole("button", { name: /увеличить/i });
  await userEvent.click(button);
  expect(screen.getByText("1")).toBeInTheDocument();
});
```

---

## Что такое mock и stub?

- **Stub** — замена зависимости с предопределённым ответом
- **Mock** — заглушка с проверкой вызовов
- **Spy** — обёртка реальной функции с записью вызовов

```typescript
const fetchUser = jest.fn().mockResolvedValue({ id: 1, name: "Alice" });
// ... тест ...
expect(fetchUser).toHaveBeenCalledWith(1);
```

---

## Что тестировать в первую очередь?

1. Критическая бизнес-логика (расчёты, валидация)
2. Компоненты с нетривиальной логикой
3. Утилиты и хелперы
4. Баги — каждый исправленный баг должен иметь тест
