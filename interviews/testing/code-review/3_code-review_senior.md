# Code Review — Senior

## Вопросы

- [Как проводить архитектурный review?](#как-проводить-архитектурный-review)
- [Как mentor через code review?](#как-mentor-через-code-review)
- [Как balance review quality vs velocity?](#как-balance-review-quality-vs-velocity)

---

## Как проводить архитектурный review?

Смотреть не на строки, а на:
- Соответствие архитектурным паттернам проекта (FSD, слои)
- Нарушение SOLID принципов (особенно SRP, OCP)
- Tight coupling который усложнит изменения позже
- Масштабируемость: как это поведёт себя при 10x нагрузке
- ADR: документировано ли архитектурное решение

---

## Как mentor через code review?

1. **Объяснять принципы**, не только конкретные исправления
2. **Ссылаться** на документацию, статьи, ADR
3. **Pair programming** вместо долгих текстовых объяснений
4. **Положительное подкрепление** хороших решений
5. **Follow-up**: отслеживать прогресс разработчика

---

## Как balance review quality vs velocity?

Классификация PR по риску: **High risk** (auth, payments, data migration) → thorough review. **Medium** (new feature) → standard review. **Low** (fix, refactor с тестами) → quick review. Автоматизировать: CI, lint, type check, тесты снимают часть ручной работы. DORA метрики: Lead Time — если review bottleneck, оптимизировать процесс.
