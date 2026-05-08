# CI/CD — Middle

## Вопросы

- [Что такое deployment стратегии?](#что-такое-deployment-стратегии)
- [Как настроить CD для фронтенд приложения?](#как-настроить-cd-для-фронтенд-приложения)
- [Что такое preview deployments?](#что-такое-preview-deployments)
- [Как настроить matrix builds?](#как-настроить-matrix-builds)
- [Что такое caching в GitHub Actions?](#что-такое-caching-в-github-actions)

---

## Что такое deployment стратегии?

- **Blue-Green**: две production среды, переключить трафик на новую → instant rollback
- **Canary**: постепенное увеличение трафика (1% → 10% → 100%)
- **Rolling**: поэтапная замена инстансов без downtime
- **Feature flags**: деплой кода без включения фичи (включить позже)

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как настроить CD для фронтенд приложения?

```yaml
deploy:
  needs: test
  runs-on: ubuntu-latest
  if: github.ref == 'refs/heads/main'
  environment: production
  steps:
    - uses: actions/checkout@v4
    - run: npm ci ; npm run build
    - name: Deploy to Vercel
      run: npx vercel --prod
      env:
        VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое preview deployments?

Preview deployment — уникальный URL для каждого PR. Vercel/Netlify создают автоматически. Позволяет: QA тестирование до merge, демонстрация заказчику, E2E тесты против реального деплоя. GitHub Action: комментировать PR с preview URL.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как настроить matrix builds?

```yaml
strategy:
  matrix:
    node: [18, 20, 22]
    os: [ubuntu-latest, windows-latest]
runs-on: ${{ matrix.os }}
steps:
  - uses: actions/setup-node@v4
    with: { node-version: ${{ matrix.node }} }
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое caching в GitHub Actions?

```yaml
- uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: ${{ runner.os }}-node-
```

`actions/setup-node` с `cache: npm` делает это автоматически.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
