# E2E Testing — Expert

## Вопросы

- [Как строить E2E инфраструктуру для нескольких команд?](#как-строить-e2e-инфраструктуру-для-нескольких-команд)
- [Как интегрировать E2E в deployment pipeline?](#как-интегрировать-e2e-в-deployment-pipeline)

---

## Как строить E2E инфраструктуру для нескольких команд?

1. **Shared test utilities** — common Page Objects, fixtures, helpers в пакете
2. **Test environments** — staging per PR (preview deployments)
3. **Test ownership** — CODEOWNERS для E2E файлов
4. **Flaky test tracking** — дашборд с историей нестабильных тестов
5. **Quarantine** — изолировать flaky тесты, не ломать CI

---

## Как интегрировать E2E в deployment pipeline?

```
PR открыт
  → Unit + Integration tests (быстрые, блокируют PR)
  → Preview deployment создан
  → Smoke E2E на preview (блокирует merge)
Merge в main
  → Full regression suite на staging
  → При прохождении — deploy в production
  → Post-deploy smoke на production
```

Canary testing: E2E против production с синтетическим трафиком (Datadog Synthetics, Checkly).
