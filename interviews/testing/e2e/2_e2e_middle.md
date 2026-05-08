# E2E Testing — Middle

## Вопросы

- [Что такое Page Object Model?](#что-такое-page-object-model)
- [Как интегрировать E2E тесты в CI?](#как-интегрировать-e2e-тесты-в-ci)
- [Как справляться с flaky тестами?](#как-справляться-с-flaky-тестами)
- [Как тестировать аутентификацию в Playwright?](#как-тестировать-аутентификацию-в-playwright)
- [Как использовать API для подготовки данных?](#как-использовать-api-для-подготовки-данных)

---

## Что такое Page Object Model?

POM — паттерн организации E2E тестов. Страница инкапсулирует свои элементы и действия. Тест использует объект страницы, не селекторы напрямую.

```typescript
class LoginPage {
  constructor(private page: Page) {}
  
  async login(email: string, password: string) {
    await this.page.fill('[name="email"]', email);
    await this.page.fill('[name="password"]', password);
    await this.page.click('button[type="submit"]');
  }
  
  async getError() { return this.page.getByRole("alert").textContent(); }
}

// В тесте
const loginPage = new LoginPage(page);
await loginPage.login("user@test.com", "wrong");
expect(await loginPage.getError()).toContain("Неверный пароль");
```

---

## Как интегрировать E2E тесты в CI?

```yaml
# GitHub Actions
- name: Install Playwright
  run: npx playwright install --with-deps chromium
- name: Run E2E tests
  run: npx playwright test --project=chromium
- name: Upload test report
  uses: actions/upload-artifact@v3
  if: failure()
  with:
    name: playwright-report
    path: playwright-report/
```

---

## Как справляться с flaky тестами?

1. **Локализовать**: запустить тест 10 раз, определить частоту
2. **Причины**: таймауты, состояние гонки, внешние зависимости
3. **Auto-wait**: использовать `expect(locator).toBeVisible()` не `sleep`
4. **Retry**: `retries: 2` в playwright.config.ts (не маскирует, а даёт время)
5. **Изоляция данных**: каждый тест создаёт свои данные, не зависит от других
6. **Network intercept**: `page.route()` для нестабильных внешних API

---

## Как тестировать аутентификацию в Playwright?

```typescript
// Один раз логинимся, сохраняем storage state
setup("authenticate", async ({ page }) => {
  await page.goto("/login");
  await page.fill('[name="email"]', process.env.TEST_EMAIL!);
  await page.fill('[name="password"]', process.env.TEST_PASSWORD!);
  await page.click('button[type="submit"]');
  await page.context().storageState({ path: "playwright/.auth/user.json" });
});

// В тестах переиспользуем
test.use({ storageState: "playwright/.auth/user.json" });
```

---

## Как использовать API для подготовки данных?

Через Playwright `request` fixture — прямые API вызовы для создания/очистки данных без UI.

```typescript
test.beforeEach(async ({ request }) => {
  await request.post("/api/test/seed", { data: { scenario: "cart-checkout" } });
});
test.afterEach(async ({ request }) => {
  await request.post("/api/test/cleanup");
});
```
