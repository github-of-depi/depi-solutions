# CI/CD — Junior

## Вопросы

- [Что такое CI/CD?](#что-такое-cicd)
- [Что такое GitHub Actions?](#что-такое-github-actions)
- [Как настроить базовый CI pipeline?](#как-настроить-базовый-ci-pipeline)
- [Что такое environment variables в CI?](#что-такое-environment-variables-в-ci)

---

## Что такое CI/CD?

**CI** (Continuous Integration) — автоматическая сборка и тестирование при каждом push. Раннее обнаружение ошибок. **CD** (Continuous Delivery/Deployment) — автоматический деплой после прохождения CI. Delivery — в staging с ручным approve. Deployment — в production автоматически.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое GitHub Actions?

GitHub Actions — CI/CD платформа встроенная в GitHub. Конфигурация через YAML в `.github/workflows/`. Триггеры: push, PR, schedule, manual. Runners: ubuntu, macos, windows.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как настроить базовый CI pipeline?

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm }
      - run: npm ci
      - run: npm run type-check
      - run: npm run lint
      - run: npm test -- --coverage
      - run: npm run build
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое environment variables в CI?

Secrets в GitHub: Settings → Secrets and Variables. В workflow через `${{ secrets.API_KEY }}`. Никогда не логировать secrets (`echo $API_KEY` — опасно). Environment-specific secrets через GitHub Environments (staging, production).

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
