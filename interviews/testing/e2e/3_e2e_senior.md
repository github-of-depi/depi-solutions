# E2E Testing — Senior

## Вопросы

- [Как строить E2E стратегию для большого проекта?](#как-строить-e2e-стратегию-для-большого-проекта)
- [Как параллелизировать E2E тесты?](#как-параллелизировать-e2e-тесты)
- [Как тестировать mobile в Playwright?](#как-тестировать-mobile-в-playwright)
- [Что такое visual regression testing?](#что-такое-visual-regression-testing)

---

## Как строить E2E стратегию для большого проекта?

1. **Smoke suite** (5-10 мин): критические пути — login, checkout, core features
2. **Regression suite** (30-60 мин): полное покрытие, запуск ночью
3. **Critical path**: каждый критический путь — хотя бы 1 E2E тест
4. **Not testing**: детали UI — для unit/component тестов
5. **Ownership**: команды владеют E2E тестами своих фич

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как параллелизировать E2E тесты?

```typescript
// playwright.config.ts
export default defineConfig({
  workers: process.env.CI ? 4 : 2, // параллельные воркеры
  fullyParallel: true,              // тесты внутри файла тоже параллельно
  retries: process.env.CI ? 2 : 0,
  use: { baseURL: process.env.BASE_URL || "http://localhost:3000" },
});
```

Sharding в CI: `--shard=1/4 --shard=2/4` — запуск на нескольких машинах.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как тестировать mobile в Playwright?

```typescript
// Emulate mobile
test("мобильная навигация", async ({ browser }) => {
  const page = await browser.newPage({ ...devices["iPhone 14"] });
  await page.goto("/");
  await page.getByLabel("Меню").click();
  await expect(page.getByRole("navigation")).toBeVisible();
});
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое visual regression testing?

Сравнение скриншотов между запусками. Playwright: `expect(page).toHaveScreenshot()`. Chromatic: visual diffs на уровне компонентов в Storybook. Полезно для: дизайн-системы, предотвращения CSS регрессий.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
