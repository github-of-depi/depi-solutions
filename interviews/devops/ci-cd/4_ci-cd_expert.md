# CI/CD — Expert

## Вопросы

- [Как оптимизировать время CI pipeline?](#как-оптимизировать-время-ci-pipeline)
- [Как строить Platform Engineering для фронтенда?](#как-строить-platform-engineering-для-фронтенда)

---

## Как оптимизировать время CI pipeline?

1. **Параллелизация**: независимые jobs параллельно (lint || tests || build)
2. **Test sharding**: `--shard=1/4` — разделить тесты между воркерами
3. **Турборепо affected**: пересобирать только изменённые пакеты
4. **Docker layer caching** и npm cache — не переустанавливать зависимости
5. **Merge queues**: GitHub Merge Queue — избежать concurrent build failures
6. **Self-hosted runners**: мощные машины для E2E

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как строить Platform Engineering для фронтенда?

Platform Engineering — внутренняя платформа для продуктовых команд:
1. **Golden path**: шаблон нового проекта с CI/CD из коробки
2. **Shared workflows**: переиспользуемые GitHub Actions workflows
3. **Developer portal** (Backstage): каталог сервисов, документация
4. **Feature flags platform** (LaunchDarkly/Unleash): самообслуживание для команд
5. **Observability**: unified logging, metrics, tracing dashboard

```yaml
# Переиспользуемый workflow
on:
  workflow_call:
    inputs:
      node-version: { type: string, default: "20" }

jobs:
  test:
    uses: ./.github/workflows/reusable-test.yml
    with:
      node-version: ${{ inputs.node-version }}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
