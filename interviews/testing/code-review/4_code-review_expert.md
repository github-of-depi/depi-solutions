# Code Review — Expert

## Вопросы

- [Как автоматизировать code review?](#как-автоматизировать-code-review)
- [Как строить культуру quality code в команде?](#как-строить-культуру-quality-code-в-команде)

---

## Как автоматизировать code review?

1. **Static analysis**: ESLint, TypeScript, SonarQube — в CI как gate
2. **AI review**: GitHub Copilot PR review, CodeRabbit — находит паттерны
3. **Danger.js** — кастомные rules для PR (размер PR, CHANGELOG, coverage)
4. **Bundle size checks** — автокомментарий с размером bundle изменений
5. **Architecture tests** — dependency-cruiser проверяет правила импортов

```javascript
// Danger.js: предупреждение о большом PR
if (danger.github.pr.additions > 500) {
  warn("Большой PR! Рассмотрите разбивку на несколько частей.");
}
```

---

## Как строить культуру quality code в команде?

1. **Definition of Done** включает code review standards
2. **Engineering principles** документированы (как мы пишем код)
3. **Learning from reviews**: periodic retrospectives на типичные паттерны
4. **Pair programming** для сложных задач — предотвращает проблемы до review
5. **Blameless culture**: ошибки в коде — системная проблема, не personal failure
