# TypeScript — Junior

## Вопросы

- [В чём разница между type и interface?](#в-чём-разница-между-type-и-interface)
- [Что такое any, unknown и never?](#что-такое-any-unknown-и-never)
- [Что такое type assertion?](#что-такое-type-assertion)
- [Что такое опциональные поля и оператор ??](#что-такое-опциональные-поля-и-оператор-)
- [Чем enum отличается от union типа?](#чем-enum-отличается-от-union-типа)
- [Что такое readonly и const?](#что-такое-readonly-и-const)
- [Что такое ReadonlyArray?](#что-такое-readonlyarray)
- [Что такое модификаторы доступа?](#что-такое-модификаторы-доступа)
- [Что такое геттеры и сеттеры?](#что-такое-геттеры-и-сеттеры)
- [Что такое абстрактные классы?](#что-такое-абстрактные-классы)

---

## В чём разница между type и interface?

`interface` — только для описания форм объектов и классов, поддерживает декларативное слияние (declaration merging) и `extends`. `type` — универсальнее: union, intersection, mapped types, tuple, primitive aliases. Для объектов оба работают одинаково, но `interface` предпочтителен в публичных API библиотек из-за merging.

```typescript
interface User { id: number; name: string; }
interface User { email: string; } // merging — OK

type ID = string | number; // только через type
type Point = { x: number } & { y: number }; // intersection
```

---

## Что такое any, unknown и never?

- `any` — отключает проверку типов, небезопасен. Запрет на `any` — стандарт в проектах.
- `unknown` — безопасная альтернатива `any`: нельзя использовать без сужения типа (type guard или assertion).
- `never` — тип без значений: функция, которая всегда бросает исключение или бесконечный цикл.

```typescript
function assertNever(x: never): never { throw new Error("Unexpected: " + x); }

let val: unknown = getData();
if (typeof val === "string") val.toUpperCase(); // OK после сужения
```

---

## Что такое type assertion?

Type assertion — явное указание компилятору типа значения, когда разработчик знает больше, чем TypeScript. Не производит runtime-преобразования. Используй аккуратно — неверный assertion скрывает баги.

```typescript
const input = document.getElementById("search") as HTMLInputElement;
input.value; // теперь TS знает, что это HTMLInputElement

// Non-null assertion — говорим что значение точно не null/undefined
const name = user.name!.toUpperCase();
```

---

## Что такое опциональные поля и оператор ??

Опциональный параметр `?` означает, что поле может быть `undefined`. Оператор `??` (nullish coalescing) возвращает правый операнд, если левый `null` или `undefined` (в отличие от `||`, который срабатывает на любое falsy значение).

```typescript
interface Config { timeout?: number; }

const timeout = config.timeout ?? 3000; // только null/undefined -> 3000
const label = value || "default"; // 0, "" тоже -> "default"
```

---

## Чем enum отличается от union типа?

`enum` компилируется в реальный объект в рантайме (числовой или строковый). String union — только тип, нет рантайм-объекта, легче рефакторинг, работает с `satisfies`. Обычно предпочитают string union или `as const` объекты.

```typescript
// enum — есть в рантайме
enum Status { Active = "active", Inactive = "inactive" }

// Предпочтительно — только тип
type Status = "active" | "inactive";

// as const — объект + тип
const STATUS = { Active: "active", Inactive: "inactive" } as const;
type Status = typeof STATUS[keyof typeof STATUS];
```

---

## Что такое readonly и const?

`const` — блокирует переприсваивание переменной (рантайм). `readonly` — блокирует изменение свойства объекта на уровне типов (только TypeScript). `Readonly<T>` делает все поля readonly.

```typescript
const arr = [1, 2, 3]; // нельзя arr = [], но можно arr.push(4)
const arr2: readonly number[] = [1, 2, 3]; // нельзя arr2.push()

interface Config { readonly host: string; }
```

---

## Что такое ReadonlyArray?

`ReadonlyArray<T>` (или `readonly T[]`) — массив, запрещающий мутирующие методы (`push`, `pop`, `splice`, `sort`) на уровне типов. Runtime не защищает.

```typescript
const nums: ReadonlyArray<number> = [1, 2, 3];
// эквивалентно: readonly number[]

nums.push(4); // TS Error: Property 'push' does not exist on ReadonlyArray
nums[0] = 99; // TS Error: Index signature read-only

// Функция, которая не изменяет массив:
function sum(arr: readonly number[]): number {
  return arr.reduce((a, b) => a + b, 0);
}
```

---

## Что такое модификаторы доступа?

TypeScript поддерживает 4 модификатора доступа для членов класса:

```typescript
class BankAccount {
  public owner: string;          // доступно везде (default)
  private balance: number;       // только внутри класса
  protected accountType: string; // внутри класса и подклассов
  readonly id: string;           // только для чтения

  static bank = "MyBank";        // принадлежит классу, не экземпляру

  constructor(owner: string, balance: number) {
    this.owner = owner;
    this.balance = balance;
    this.id = crypto.randomUUID();
  }

  getBalance(): number { return this.balance; } // public метод

  // Сокращённый синтаксис в конструкторе:
  // constructor(public name: string, private age: number) {}
}
```

---

## Что такое геттеры и сеттеры?

Геттеры и сеттеры — специальные методы для контролируемого доступа к свойствам:

```typescript
class Temperature {
  private _celsius: number;

  constructor(celsius: number) { this._celsius = celsius; }

  get fahrenheit(): number {
    return this._celsius * 9/5 + 32;
  }

  get celsius(): number { return this._celsius; }

  set celsius(value: number) {
    if (value < -273.15) throw new RangeError("Ниже абсолютного нуля!");
    this._celsius = value;
  }
}

const t = new Temperature(0);
t.fahrenheit; // 32 — вызов без ()
t.celsius = 100; // сеттер с валидацией
t.celsius = -300; // RangeError
```

---

## Что такое абстрактные классы?

Абстрактный класс — класс, который нельзя инстанциировать напрямую. Служит базовым классом с общей реализацией. `abstract` методы обязательны к реализации в подклассах.

```typescript
abstract class Shape {
  constructor(public color: string) {}

  abstract area(): number;       // нет реализации — ОБЯЗАТЕЛЬНО переопределить
  abstract perimeter(): number;

  describe(): string {           // обычный метод — наследуется
    return `${this.color} shape, area: ${this.area().toFixed(2)}`;
  }
}

class Circle extends Shape {
  constructor(color: string, public radius: number) { super(color); }
  area() { return Math.PI * this.radius ** 2; }
  perimeter() { return 2 * Math.PI * this.radius; }
}

new Shape("red");  // TS Error: Cannot create instance of abstract class
new Circle("red", 5); // OK
```
