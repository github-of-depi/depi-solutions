# Архитектурные паттерны — Junior

## Вопросы

- [Что такое SOLID?](#что-такое-solid)
- [Что такое DRY, KISS, YAGNI?](#что-такое-dry-kiss-yagni)
- [Что такое паттерн Observer?](#что-такое-паттерн-observer)
- [Что такое паттерн Factory?](#что-такое-паттерн-factory)
- [Что такое паттерн Singleton?](#что-такое-паттерн-singleton)

---

## Что такое SOLID?

SOLID — 5 принципов объектно-ориентированного дизайна:

- **S** (Single Responsibility) — класс/модуль отвечает за одно
- **O** (Open/Closed) — открыт для расширения, закрыт для изменения
- **L** (Liskov Substitution) — подтипы заменимы базовым типом
- **I** (Interface Segregation) — узкие интерфейсы лучше широких
- **D** (Dependency Inversion) — зависеть от абстракций, не от реализаций

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое DRY, KISS, YAGNI?

- **DRY** (Don't Repeat Yourself) — не дублировать логику, выносить в переиспользуемые абстракции
- **KISS** (Keep It Simple, Stupid) — простые решения лучше сложных
- **YAGNI** (You Aren't Gonna Need It) — не добавлять функциональность "на будущее"

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое паттерн Observer?

Observer (наблюдатель) — объект (subject) оповещает подписчиков (observers) об изменениях. DOM события, EventEmitter, RxJS Observable — реализации паттерна.

```typescript
class EventEmitter {
  private listeners = new Map<string, Set<Function>>();
  on(event: string, fn: Function) { 
    if (!this.listeners.has(event)) this.listeners.set(event, new Set());
    this.listeners.get(event)!.add(fn);
  }
  emit(event: string, data?: unknown) { this.listeners.get(event)?.forEach(fn => fn(data)); }
  off(event: string, fn: Function) { this.listeners.get(event)?.delete(fn); }
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое паттерн Factory?

Factory создаёт объекты без указания конкретного класса. Инкапсулирует логику создания, легко расширять новыми типами.

```typescript
type ButtonVariant = "primary" | "danger" | "ghost";
function createButton(variant: ButtonVariant): ButtonConfig {
  const configs: Record<ButtonVariant, ButtonConfig> = {
    primary: { color: "blue", outline: false },
    danger:  { color: "red",  outline: false },
    ghost:   { color: "gray", outline: true  },
  };
  return configs[variant];
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое паттерн Singleton?

Singleton — один экземпляр класса в приложении. В TS: статический instance. Примеры: logger, database connection, config. В React: Context + Provider на уровне App часто заменяет Singleton.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
