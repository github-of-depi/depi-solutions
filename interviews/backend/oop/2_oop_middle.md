# ООП — Middle

## Вопросы

- [SOLID принципы с примерами на TypeScript?](#solid-принципы-с-примерами-на-typescript)
- [Что такое abstract class vs interface?](#что-такое-abstract-class-vs-interface)
- [Что такое composition over inheritance?](#что-такое-composition-over-inheritance)
- [Что такое паттерны GoF?](#что-такое-паттерны-gof)

---

## SOLID принципы с примерами на TypeScript?

```typescript
// S — Single Responsibility
class UserRepository { findById(id: string) { /* только работа с БД */ } }
class UserEmailService { sendWelcome(user: User) { /* только отправка email */ } }

// O — Open/Closed
interface Discount { calculate(price: number): number; }
class StudentDiscount implements Discount { calculate(p: number) { return p * 0.8; } }
// Новая скидка — новый класс, не изменяем существующий

// D — Dependency Inversion
class OrderService {
  constructor(private repo: UserRepository) {} // зависит от абстракции
}
```

---

## Что такое abstract class vs interface?

**Interface** — только контракт, нет реализации, множественное "наследование". **Abstract class** — может иметь реализацию методов, единственное наследование, может иметь состояние.

```typescript
abstract class Animal {
  abstract speak(): string;     // обязательно переопределить
  breathe(): void { /* базовая реализация */ }
}
```

---

## Что такое composition over inheritance?

Предпочитать составление объектов наследованию. Избегает "хрупкого базового класса", лучше гибкость.

```typescript
// Вместо: class FlyingSwimmingDuck extends Duck extends FlyingBird extends SwimmingBird
interface Flyable { fly(): void; }
interface Swimmable { swim(): void; }
class Duck implements Flyable, Swimmable { fly() {...}  swim() {...} }
```

---

## Что такое паттерны GoF?

GoF (Gang of Four) — 23 классических паттерна из книги "Design Patterns". Категории: Creational (создание), Structural (структура), Behavioral (поведение). Важнейшие: Factory, Singleton, Observer, Strategy, Decorator, Facade.
