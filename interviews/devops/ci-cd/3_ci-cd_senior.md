# CI/CD — Senior

## Вопросы

- [Как строить multi-stage pipeline?](#как-строить-multi-stage-pipeline)
- [Как строить GitOps workflow?](#как-строить-gitops-workflow)
- [Что такое DORA метрики?](#что-такое-dora-метрики)
- [Как обеспечить rollback в production?](#как-обеспечить-rollback-в-production)

---

## Как строить multi-stage pipeline?

```
Trigger: PR открыт
  → lint + type-check (parallel, fast)
  → unit tests (parallel)
  → build
  → preview deploy
  → E2E smoke tests against preview
  → [Human approve to merge]
Trigger: merge в main
  → full test suite
  → build production bundle
  → deploy to staging
  → regression E2E
  → [Human approve production]
  → deploy to production
  → smoke tests production
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как строить GitOps workflow?

GitOps: Git — единственный источник истины для инфраструктуры. ArgoCD / Flux — синхронизируют состояние Kubernetes с Git репозиторием. При push в infra repo → CD инструмент применяет изменения автоматически. Rollback = git revert.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое DORA метрики?

DORA — метрики производительности DevOps команды:
- **Deployment Frequency**: как часто деплоите (elite: несколько раз в день)
- **Lead Time for Changes**: от коммита до production (elite: <1 часа)
- **Time to Restore Service**: время восстановления после инцидента (elite: <1 часа)
- **Change Failure Rate**: процент деплоев с инцидентом (elite: <5%)

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как обеспечить rollback в production?

1. **Версионированные деплои**: каждый деплой имеет уникальный ID/SHA
2. **One-click rollback**: Vercel/Kubernetes — восстановить предыдущую версию
3. **Blue-Green**: переключить трафик обратно за секунды
4. **Feature flags**: отключить проблемную фичу без деплоя
5. **Database migrations**: backward compatible или с возможностью откатить

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
