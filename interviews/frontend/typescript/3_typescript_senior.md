# TypeScript — Senior

## Вопросы

- [Что такое template literal types?](#что-такое-template-literal-types)
- [Как типизировать API-ответы безопасно?](#как-типизировать-api-ответы-безопасно)
- [Что такое declaration merging?](#что-такое-declaration-merging)
- [Как работает вывод типов (type inference) в сложных случаях?](#как-работает-вывод-типов-в-сложных-случаях)
- [Что такое variance в TypeScript?](#что-такое-variance-в-typescript)
- [Как строить type-safe event emitter?](#как-строить-type-safe-event-emitter)
- [Что такое Mixins в TypeScript?](#что-такое-mixins-в-typescript)

---

## Что такое template literal types?

Template literal types позволяют строить строковые типы через шаблон. Комбинируются с `Capitalize`, `Lowercase`, `Uppercase`, `Uncapitalize`. Используются для: типизации CSS-свойств, имён событий, ключей API.

```typescript
type EventName = "click" | "focus" | "blur";
type Handler = `on${Capitalize<EventName>}`; // "onClick" | "onFocus" | "onBlur"

type CSSValue = `${number}px` | `${number}%` | `${number}rem`;

// Getters из объекта
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K]
};
```

---

## Как типизировать API-ответы безопасно?

Runtime валидация + статические типы. Используй `zod` (или `valibot`, `arktype`): схема одновременно валидирует данные в рантайме и предоставляет тип через `z.infer`. Никогда не приводи тип API-ответа через `as` — это скрывает баги.

```typescript
import { z } from "zod";

const UserSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
});

type User = z.infer<typeof UserSchema>;

async function fetchUser(id: number): Promise<User> {
  const data = await api.get(`/users/${id}`);
  return UserSchema.parse(data); // throws если данные невалидны
}
```

---

## Что такое declaration merging?

Declaration merging — возможность TypeScript объединять несколько деклараций одного имени. Работает для `interface`, `namespace` и `module`. Используется для: расширения типов сторонних библиотек (module augmentation), добавления свойств к глобальным объектам.

```typescript
// Расширение Express Request
declare global {
  namespace Express {
    interface Request {
      user?: User;
    }
  }
}

// Расширение Window
interface Window {
  analytics: Analytics;
}
```

---

## Как работает вывод типов в сложных случаях?

TypeScript использует bidirectional type inference: тип может вычисляться из контекста (contextual typing) или из присваиваемого значения. В дженериках TypeScript пытается унифицировать типы параметров. Проблема: вывод иногда слишком широкий (`string` вместо literal `"active"`) — решается через `as const` или явные параметры типа.

```typescript
// Без as const — тип string[]
const statuses = ["active", "inactive"];

// С as const — тип readonly ["active", "inactive"]
const statuses = ["active", "inactive"] as const;

// satisfies проверяет соответствие типу, сохраняя literal типы
const config = { port: 3000 } satisfies Partial<Config>;
config.port; // тип 3000, а не number
```

---

## Что такое variance в TypeScript?

Variance описывает, как подтипирование одного типа влияет на подтипирование составного. TypeScript использует структурную типизацию и structural subtyping.

- **Covariant** (ковариантный): `Array<Dog>` — подтип `Array<Animal>`. Безопасно для producer.
- **Contravariant** (контравариантный): функция `(x: Animal) => void` — подтип `(x: Dog) => void`. Безопасно для consumer.
- **Invariant**: `ref.current` — нельзя присвоить `Ref<Dog>` туда, где ожидается `Ref<Animal>`.

TypeScript 4.7+ поддерживает явные аннотации `in`/`out` для дженериков.

---

## Как строить type-safe event emitter?

Используй mapped types с дженериком событий для строгой типизации payload каждого события.

```typescript
type EventMap = {
  "user:created": { id: number; name: string };
  "user:deleted": { id: number };
};

class TypedEmitter<Events extends Record<string, unknown>> {
  on<K extends keyof Events>(event: K, handler: (data: Events[K]) => void): void {
    // ...
  }
  emit<K extends keyof Events>(event: K, data: Events[K]): void {
    // ...
  }
}

const emitter = new TypedEmitter<EventMap>();
emitter.on("user:created", ({ id, name }) => console.log(id, name)); // типобезопасно
emitter.emit("user:deleted", { id: 1 }); // TS проверяет payload
```

---

## Что такое Mixins в TypeScript?

Mixins — паттерн композиции поведения без множественного наследования. Функция принимает класс и возвращает расширенный класс (Higher-Order Class).

```typescript
// Constructor type — тип любого класса
type Constructor<T = {}> = new (...args: any[]) => T;

// Mixin-функции
function Serializable<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    serialize(): string { return JSON.stringify(this); }
    static deserialize(json: string) { return JSON.parse(json); }
  };
}

function Timestamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    createdAt = new Date();
    updatedAt = new Date();
    touch() { this.updatedAt = new Date(); }
  };
}

// Применение миксинов — композиция:
class BaseModel { constructor(public id: string) {} }
const TimestampedModel  = Timestamped(BaseModel);
const FullModel         = Serializable(TimestampedModel);

class User extends FullModel {
  constructor(id: string, public name: string) { super(id); }
}

const user = new User("1", "Alice");
user.serialize();  // из Serializable
user.touch();      // из Timestamped
```

Отличие от наследования: миксины можно **комбинировать произвольно**, нет жёсткой иерархии.
