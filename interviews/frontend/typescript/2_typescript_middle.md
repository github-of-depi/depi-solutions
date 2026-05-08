# TypeScript — Middle

## Вопросы

- [Что такое дженерики и зачем они нужны?](#что-такое-дженерики-и-зачем-они-нужны)
- [Какие utility types есть в TypeScript?](#какие-utility-types-есть-в-typescript)
- [Что такое type guards?](#что-такое-type-guards)
- [Что такое discriminated unions?](#что-такое-discriminated-unions)
- [Что такое keyof и typeof?](#что-такое-keyof-и-typeof)
- [Что такое mapped types?](#что-такое-mapped-types)
- [Что такое conditional types?](#что-такое-conditional-types)
- [Перегрузка функций в TypeScript?](#перегрузка-функций-в-typescript)
- [extends vs implements — в чём разница?](#extends-vs-implements--в-чём-разница)
- [Union vs intersection типы?](#union-vs-intersection-типы)
- [Что такое модули в TypeScript?](#что-такое-модули-в-typescript)
- [Что такое декораторы в TypeScript?](#что-такое-декораторы-в-typescript)

---

## Что такое дженерики и зачем они нужны?

Дженерики — параметры типа, позволяющие писать переиспользуемый типобезопасный код без дублирования. Функция, класс или интерфейс принимает тип как аргумент. Ограничения (`extends`) позволяют сузить допустимые типы.

```typescript
function identity<T>(value: T): T { return value; }
identity<string>("hello"); // T = string
identity(42); // T = number (вывод)

function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Какие utility types есть в TypeScript?

TypeScript предоставляет встроенные utility types для преобразования типов:

```typescript
Partial<User>          // все поля опциональны
Required<User>         // все поля обязательны
Readonly<User>         // все поля readonly
Pick<User, "id"|"name"> // только указанные поля
Omit<User, "password"> // все кроме указанных
Record<string, number> // словарь
ReturnType<typeof fn>  // тип возвращаемого значения функции
Parameters<typeof fn>  // tuple параметров функции
NonNullable<T>         // исключает null и undefined
Awaited<Promise<T>>    // тип разрешённого промиса
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое type guards?

Type guards — выражения, сужающие тип в определённой ветке кода. TypeScript отслеживает сужение автоматически (`typeof`, `instanceof`, `in`). Кастомные type guards используют синтаксис `value is Type`.

```typescript
// Встроенные guards
if (typeof val === "string") { val.toUpperCase(); }
if (val instanceof Date) { val.getFullYear(); }
if ("name" in obj) { obj.name; }

// Кастомный type guard
function isUser(value: unknown): value is User {
  return typeof value === "object" && value !== null && "id" in value;
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое discriminated unions?

Discriminated union — объединение типов с общим полем-дискриминатором (обычно `type` или `kind`). TypeScript использует дискриминатор для сужения типа в switch/if. Паттерн exhaustiveness check с `never` гарантирует обработку всех вариантов.

```typescript
type Action =
  | { type: "increment"; amount: number }
  | { type: "reset" };

function reduce(state: number, action: Action): number {
  switch (action.type) {
    case "increment": return state + action.amount;
    case "reset": return 0;
    default: return assertNever(action); // exhaustiveness check
  }
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое keyof и typeof?

`keyof T` — union из строковых ключей типа T. `typeof` в позиции типа — получает тип переменной или выражения (не путать с рантайм typeof). Вместе позволяют строить безопасный доступ к свойствам.

```typescript
type UserKeys = keyof User; // "id" | "name" | "email"

const config = { host: "localhost", port: 3000 } as const;
type Config = typeof config; // { readonly host: "localhost"; readonly port: 3000 }
type ConfigKeys = keyof typeof config; // "host" | "port"
```

**Связанные задачи:**

- [Тип из ключей объекта](../../../tasks/frontend/typescript/2_typescript_middle.md#тип-из-ключей-объекта)
- [Типизация функции getProperty](../../../tasks/frontend/typescript/2_typescript_middle.md#типизация-функции-getproperty)

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое mapped types?

Mapped types создают новый тип, итерируя по ключам существующего. Модификаторы `+/-?` и `+/-readonly` управляют опциональностью и мутабельностью. Используются для реализации utility types.

```typescript
// Реализация Partial
type MyPartial<T> = { [K in keyof T]?: T[K] };

// Реализация Readonly
type MyReadonly<T> = { readonly [K in keyof T]: T[K] };

// Remap keys через as
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K]
};
```

**Связанные задачи:**

- [Реализация Partial](../../../tasks/frontend/typescript/2_typescript_middle.md#реализация-partial)
- [Реализация Pick](../../../tasks/frontend/typescript/2_typescript_middle.md#реализация-pick)

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое conditional types?

Conditional types — тернарный оператор на уровне типов: `T extends U ? X : Y`. Позволяют строить типы, зависящие от условий. `infer` внутри `extends` извлекает часть типа.

```typescript
type IsArray<T> = T extends unknown[] ? true : false;
type IsArray<string[]> // true
type IsArray<string>   // false

// infer — извлечение типа элемента массива
type ElementType<T> = T extends (infer E)[] ? E : never;
type ElementType<string[]> // string

// Awaited использует infer для разворачивания Promise
type Awaited<T> = T extends Promise<infer R> ? Awaited<R> : T;
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Перегрузка функций в TypeScript?

Function overloading — несколько сигнатур одной функции для разных наборов аргументов. Компилятор выбирает подходящую сигнатуру.

```typescript
// Объявление сигнатур (overload signatures):
function process(input: string): string;
function process(input: number): number;
function process(input: string[]): string[];
// Реализация (implementation signature — не видна снаружи):
function process(input: string | number | string[]): string | number | string[] {
  if (typeof input === "string") return input.toUpperCase();
  if (typeof input === "number") return input * 2;
  return input.map(s => s.toUpperCase());
}

process("hello"); // string
process(5);       // number
process("hello", 5); // TS Error — нет подходящей сигнатуры
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## extends vs implements — в чём разница?

`extends` — наследование (класс от класса или интерфейс от интерфейса). Подкласс **получает** реализацию.  
`implements` — реализация контракта (класс обязуется реализовать интерфейс). Класс может реализовывать несколько интерфейсов одновременно.

```typescript
interface Serializable { serialize(): string; }
interface Loggable    { log(): void; }

class Base { protected id = crypto.randomUUID(); }

// extends — ONE parent class + многие интерфейсы:
class Entity extends Base implements Serializable, Loggable {
  serialize() { return JSON.stringify({ id: this.id }); }
  log()       { console.log(this.id); }
}

// Интерфейс extends интерфейс:
interface AdminUser extends Serializable, Loggable {
  role: "admin";
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Union vs intersection типы?

**Union (`|`)** — значение принадлежит **одному** из перечисленных типов. **Intersection (`&`)** — значение удовлетворяет **всем** типам одновременно.

```typescript
// Union — одно из
type StringOrNumber = string | number;
let val: StringOrNumber = "hello"; // или число
// Операции только общие для string И number (toString, valueOf)

// Intersection — все сразу
type Named = { name: string };
type Aged  = { age: number };
type Person = Named & Aged; // { name: string; age: number }

// Практика — merge объектов:
type WithTimestamps<T> = T & { createdAt: Date; updatedAt: Date };
type UserRecord = WithTimestamps<{ id: string; email: string }>;

// Discriminated union (tagged union) — union с литеральным дискриминатором:
type Result<T> =
  | { ok: true;  value: T }
  | { ok: false; error: string };
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое модули в TypeScript?

Модуль — файл с хотя бы одним `import` или `export`. Без них файл — **глобальный скрипт** (переменные доступны везде).

```typescript
// math.ts — named exports
export function add(a: number, b: number): number { return a + b; }
export const PI = 3.14159;

// main.ts — импорт
import { add, PI } from "./math";

// default export — один на файл
export default class Calculator { /* ... */ }
import Calculator from "./Calculator";

// Re-export (barrel export — index.ts):
export { add, PI } from "./math";
export { default as Calculator } from "./Calculator";

// Namespace import:
import * as MathUtils from "./math";
MathUtils.add(1, 2);

// Dynamic import — lazy loading:
const module = await import("./heavy-module");
```

TypeScript модули компилируются в CJS или ESM в зависимости от `tsconfig.json` (`"module": "esnext"` / `"commonjs"`). `moduleResolution: "bundler"` — для Vite/webpack.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое декораторы в TypeScript?

Декоратор — это функция, применяемая через `@` к классам, методам, свойствам или параметрам для изменения их поведения. Декораторы являются частью спецификации ECMAScript (Stage 3) и находят широкое применение в NestJS, Angular, TypeORM, inversify.

Виды декораторов:

- **Class decorator** — принимает конструктор класса
- **Method decorator** — принимает `target`, `propertyKey`, `descriptor`
- **Property decorator** — принимает `target`, `propertyKey`
- **Parameter decorator** — принимает `target`, `propertyKey`, `parameterIndex`

```typescript
// Method decorator — логирование вызовов
function Log(target: object, key: string, descriptor: PropertyDescriptor) {
  const original = descriptor.value;
  descriptor.value = function (...args: unknown[]) {
    console.log(`Calling ${key} with`, args);
    return original.apply(this, args);
  };
  return descriptor;
}

class UserService {
  @Log
  findById(id: number) {
    return { id, name: "Alice" };
  }
}

// Class decorator — добавляет метаданные
function Injectable(target: Function) {
  Reflect.defineMetadata("injectable", true, target);
}

@Injectable
class AuthService {}
```

> **Prerequisite**: для использования декораторов с `reflect-metadata` необходимо: `"experimentalDecorators": true, "emitDecoratorMetadata": true` в `tsconfig.json`.

**Связанные задачи:**

<!-- Связанных задач нет -->

**Материалы для изучения:**

- [TypeScript: Decorators](https://www.typescriptlang.org/docs/handbook/decorators.html)
