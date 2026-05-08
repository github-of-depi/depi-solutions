# Architecture Decision Records — Expert

## Вопросы

- [Как масштабировать ADR процесс в большой организации?](#как-масштабировать-adr-процесс-в-большой-организации)
- [Как автоматизировать ADR workflow?](#как-автоматизировать-adr-workflow)

---

## Как масштабировать ADR процесс в большой организации?

**Уровни ADR**:
1. **Team ADR** — локальные решения команды (в team репозитории)
2. **Product ADR** — решения на уровне продукта (в product repo)
3. **Org ADR** — стандарты организации (в внутреннем wiki/handbook)

Governance: Architecture Review Board для cross-team решений. Inner-source: команды могут предлагать изменения org ADR через PR.

---

## Как автоматизировать ADR workflow?

```yaml
# GitHub Actions: проверка наличия ADR при изменениях архитектуры
- name: Check for ADR
  run: |
    if git diff --name-only HEAD~1 | grep -q "src/app\|vite.config"; then
      if ! git diff --name-only HEAD~1 | grep -q "docs/adr"; then
        echo "⚠️ Архитектурные изменения без ADR! Добавьте ADR в docs/adr/"
        exit 1
      fi
    fi
```

Инструменты: `adr-tools` (CLI), Log4brains (web UI для ADR). ADR как docs-as-code: автоматически публиковать в Confluence/Notion.
