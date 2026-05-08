# ООП — Expert

## Вопросы

- [Как строить type-safe ООП иерархии в TypeScript?](#как-строить-type-safe-ооп-иерархии-в-typescript)
- [Метапрограммирование и декораторы в TypeScript?](#метапрограммирование-и-декораторы-в-typescript)

---

## Как строить type-safe ООП иерархии в TypeScript?

```typescript
// Дискриминированные union + visitor паттерн
interface Circle { kind: "circle"; radius: number; }
interface Rectangle { kind: "rect"; width: number; height: number; }
type Shape = Circle | Rectangle;

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle": return Math.PI * shape.radius ** 2;
    case "rect": return shape.width * shape.height;
  }
}
// TypeScript проверяет exhaustiveness — нельзя забыть новый тип
```

---

## Метапрограммирование и декораторы в TypeScript?

TypeScript декораторы (Stage 3) — метапрограммирование для классов, методов, полей. NestJS использует декораторы для: DI, роутинг, валидация, guard.

```typescript
// Method decorator — логирование
function log(target: any, key: string, descriptor: PropertyDescriptor) {
  const original = descriptor.value;
  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${key}`, args);
    const result = original.apply(this, args);
    console.log(`${key} returned`, result);
    return result;
  };
  return descriptor;
}

class UserService {
  @log
  findById(id: string) { return this.repo.find(id); }
}
```

Reflect.metadata — хранить метаданные типов. Используется NestJS DI container.
