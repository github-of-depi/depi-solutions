# E2E Testing — Junior

## Вопросы

- [Что такое E2E тестирование?](#что-такое-e2e-тестирование)
- [Что такое Playwright?](#что-такое-playwright)
- [Как написать первый E2E тест?](#как-написать-первый-e2e-тест)
- [Чем E2E отличается от unit тестов?](#чем-e2e-отличается-от-unit-тестов)

---

## Что такое E2E тестирование?

E2E (End-to-End) — тестирование полного пользовательского сценария через реальный браузер. Проверяет, что все части системы работают вместе. Медленнее unit тестов, но ближе к реальному поведению.

---

## Что такое Playwright?

Playwright — modern E2E фреймворк от Microsoft. Поддержка: Chrome, Firefox, Safari. Auto-wait: не нужны `sleep`. Parallel execution. Трассировка и видео при падениях. Codegen для записи тестов.

---

## Как написать первый E2E тест?

```typescript
import { test, expect } from "@playwright/test";

test("пользователь может войти", async ({ page }) => {
  await page.goto("/login");
  await page.fill('[name="email"]', "user@example.com");
  await page.fill('[name="password"]', "password123");
  await page.click('button[type="submit"]');
  await expect(page).toHaveURL("/dashboard");
  await expect(page.getByText("Добро пожаловать")).toBeVisible();
});
```

---

## Чем E2E отличается от unit тестов?

| | Unit | E2E |
|-|------|-----|
| Скорость | Мс | Секунды |
| Изоляция | Полная | Нет (реальный стек) |
| Надёжность | Высокая | Нижe (flaky) |
| Сложность настройки | Низкая | Высокая |
| Что проверяет | Функцию | Пользователя |
