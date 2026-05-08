# ООП — Senior

## Вопросы

- [Как применять DDD с ООП?](#как-применять-ddd-с-ооп)
- [Что такое Liskov Substitution Principle на практике?](#что-такое-liskov-substitution-principle-на-практике)
- [Когда ООП лучше функционального программирования?](#когда-ооп-лучше-функционального-программирования)

---

## Как применять DDD с ООП?

DDD Entity — объект с идентичностью (id). Value Object — объект без идентичности (Email, Money), иммутабельный. Aggregate — группа объектов с единым корнем транзакций. Domain Service — логика, не принадлежащая конкретной сущности.

```typescript
class Money {
  constructor(readonly amount: number, readonly currency: string) {}
  add(other: Money): Money {
    if (this.currency !== other.currency) throw new Error("Currency mismatch");
    return new Money(this.amount + other.amount, this.currency);
  }
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое Liskov Substitution Principle на практике?

LSP: подкласс можно использовать вместо базового класса без изменения поведения программы.

```typescript
// Нарушение: Square extends Rectangle меняет поведение setWidth/setHeight
// Решение: отдельная иерархия или использовать интерфейс Shape
interface Shape { area(): number; }
class Rectangle implements Shape { area() { return this.w * this.h; } }
class Square implements Shape { area() { return this.side ** 2; } }
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Когда ООП лучше функционального программирования?

**ООП лучше**: сложные stateful системы (GUI, game objects), моделирование бизнес-доменов (DDD), когда поведение и состояние тесно связаны. **FP лучше**: трансформации данных, параллельные вычисления, предсказуемость (нет side effects). В JavaScript/TypeScript — гибрид: классы для структуры, pure functions для логики.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
