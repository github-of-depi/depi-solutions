# Micro-frontends — Senior

## Вопросы

- [Как организовать CI/CD для micro-frontend монорепозитория?](#как-организовать-cicd-для-micro-frontend-монорепозитория)
- [Как обеспечить консистентный UX между MFE?](#как-обеспечить-консистентный-ux-между-mfe)
- [Как строить shell application?](#как-строить-shell-application)
- [Как тестировать micro-frontend интеграции?](#как-тестировать-micro-frontend-интеграции)

---

## Как организовать CI/CD для micro-frontend монорепозитория?

Turborepo affected builds — пересобирать только изменённые MFE. Независимый деплой каждого remote: версионированные remote URLs (не latest). Feature flags для koordinации rollout между shell и remotes.

---

## Как обеспечить консистентный UX между MFE?

1. **Design System** — shared npm пакет с компонентами и токенами
2. **CSS Custom Properties** — theming на уровне root
3. **Lint правила** — единые ESLint/Prettier конфиги
4. **Storybook federation** — объединённая документация компонентов
5. **Contract testing** — MFE тестируют против shared компонентов

---

## Как строить shell application?

Shell минимален: routing, global layouts, authentication, загрузка remotes. Не содержит бизнес-логику. Ошибки загрузки remote — graceful degradation (показывать fallback, не ломать всё). Performance: preload критичных remotes, lazy load остальных.

---

## Как тестировать micro-frontend интеграции?

1. **Unit**: каждый MFE тестируется независимо
2. **Integration**: contract tests (Pact) — MFE проверяет, что shell ожидает правильные props/events
3. **E2E**: Playwright против deployed preview — полная интеграция
4. **Visual regression**: Chromatic per MFE
