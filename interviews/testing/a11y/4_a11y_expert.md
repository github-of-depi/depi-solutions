# Accessibility (A11y) — Expert

## Вопросы

- [Как строить compliance программу для a11y?](#как-строить-compliance-программу-для-a11y)
- [Как тестировать сложные a11y паттерны?](#как-тестировать-сложные-a11y-паттерны)

---

## Как строить compliance программу для a11y?

1. **Baseline**: полный WCAG AA аудит текущего состояния
2. **Roadmap**: приоритизация нарушений (severity × reach)
3. **VPAT** (Voluntary Product Accessibility Template) — документация для enterprise клиентов
4. **Ongoing**: a11y в definition of done, QA checklists
5. **Training**: команда знает a11y принципы
6. **Legal**: ARIA-AT conformance reports при необходимости

---

## Как тестировать сложные a11y паттерны?

**Data Grid**: aria-grid, row/col navigation, sort announcement. **Date Picker**: aria-dialog, keyboard calendar navigation, value announcement. **Tree View**: aria-tree, expand/collapse, multi-select.

Playwright + VoiceOver через AppleScript (macOS):
```typescript
// Playwright axe integration
await page.evaluate(async () => {
  const results = await axe.run();
  return results.violations;
});
```

Ручное тестирование с реальными AT пользователями — золотой стандарт.
