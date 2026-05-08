# TypeScript — Middle — Задачи

## Задачи

- [Реализация Partial](#реализация-partial)
- [Реализация Pick](#реализация-pick)
- [Тип из ключей объекта](#тип-из-ключей-объекта)
- [Типизация функции getProperty](#типизация-функции-getproperty)

---

## Реализация Partial

**Сложность:** Medium

**Связанные вопросы:**

- [Что такое mapped types?](../../../interviews/frontend/typescript/2_typescript_middle.md#что-такое-mapped-types)
- [Какие utility types есть в TypeScript?](../../../interviews/frontend/typescript/2_typescript_middle.md#какие-utility-types-есть-в-typescript)

**Материалы для изучения:**

- [TypeScript: Mapped Types](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html)

**Задание:**

Реализуйте свою версию встроенного `Partial<T>` — тип, делающий все поля объекта опциональными. Нельзя использовать встроенный `Partial`.

```typescript
type User = {
  id: number;
  name: string;
  email: string;
};

type MyPartial<T> = /* ваша реализация */

// Должно работать без ошибок:
const update: MyPartial<User> = { name: "Bob" }; // id и email опциональны
const empty: MyPartial<User> = {};               // все поля опциональны
```

<details>
<summary>Решение</summary>

```typescript
type MyPartial<T> = {
  [K in keyof T]?: T[K];
};

// Проверка:
type User = { id: number; name: string; email: string };
type PartialUser = MyPartial<User>;
// Эквивалентно: { id?: number; name?: string; email?: string }

const update: PartialUser = { name: "Bob" }; // OK
```

**Как это работает:**
- `keyof T` — union из ключей типа `T` (`"id" | "name" | "email"`)
- `[K in keyof T]` — итерация по каждому ключу
- `?:` — делает каждое поле опциональным
- `T[K]` — сохраняет оригинальный тип значения

</details>

---

## Реализация Pick

**Сложность:** Medium

**Связанные вопросы:**

- [Что такое mapped types?](../../../interviews/frontend/typescript/2_typescript_middle.md#что-такое-mapped-types)
- [Что такое дженерики и зачем они нужны?](../../../interviews/frontend/typescript/2_typescript_middle.md#что-такое-дженерики-и-зачем-они-нужны)

**Материалы для изучения:**

- [TypeScript: Mapped Types](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html)

**Задание:**

Реализуйте свою версию `Pick<T, K>` — тип, оставляющий только указанные ключи из объекта `T`. Нельзя использовать встроенный `Pick`.

```typescript
type User = {
  id: number;
  name: string;
  surname: string;
  email: string;
};

type MyPick<T, K> = /* ваша реализация */

type UserPreview = MyPick<User, "id" | "name">;
// Ожидается: { id: number; name: string }

// Должна быть ошибка при несуществующем ключе:
type Wrong = MyPick<User, "id" | "phone">; // TS Error
```

<details>
<summary>Решение</summary>

```typescript
type MyPick<T, K extends keyof T> = {
  [P in K]: T[P];
};

// Проверка:
type User = { id: number; name: string; surname: string; email: string };
type UserPreview = MyPick<User, "id" | "name">;
// { id: number; name: string }

type Wrong = MyPick<User, "id" | "phone">; // TS Error: Type '"phone"' does not satisfy the constraint 'keyof User'
```

**Ключевой момент:**
- `K extends keyof T` — ограничивает `K` только существующими ключами `T`. Это и обеспечивает ошибку при несуществующем ключе.

</details>

---

## Тип из ключей объекта

**Сложность:** Easy

**Связанные вопросы:**

- [Что такое keyof и typeof?](../../../interviews/frontend/typescript/2_typescript_middle.md#что-такое-keyof-и-typeof)

**Материалы для изучения:**

- [TypeScript: typeof Type Operator](https://www.typescriptlang.org/docs/handbook/2/typeof-types.html)

**Задание:**

Есть объект `obj`. Нужно создать тип `ObjKey`, чтобы переменной этого типа можно было присвоить только существующий ключ объекта. При добавлении нового ключа в объект тип должен обновляться автоматически.

```typescript
const obj = {
  name: "Nik",
  age: 25,
};

type ObjKey = /* ваша реализация */

// Должно работать без ошибок:
const var1: ObjKey = "name";
const var2: ObjKey = "age";

// Должны быть ошибки типов:
const var3: ObjKey = "test";  // TS Error
const var4: ObjKey = 25;      // TS Error
```

<details>
<summary>Решение</summary>

```typescript
const obj = {
  name: "Nik",
  age: 25,
} as const; // as const — делает значения literal-типами (необязательно для ключей, но хорошая практика)

type ObjKey = keyof typeof obj; // "name" | "age"

const var1: ObjKey = "name"; // OK
const var2: ObjKey = "age";  // OK
const var3: ObjKey = "test"; // TS Error
const var4: ObjKey = 25;     // TS Error
```

**Как это работает:**
- `typeof obj` — получает тип переменной `obj` в позиции типа (не рантайм `typeof`)
- `keyof typeof obj` — извлекает union из ключей → `"name" | "age"`

</details>

---

## Типизация функции getProperty

**Сложность:** Medium

**Связанные вопросы:**

- [Что такое дженерики и зачем они нужны?](../../../interviews/frontend/typescript/2_typescript_middle.md#что-такое-дженерики-и-зачем-они-нужны)
- [Что такое keyof и typeof?](../../../interviews/frontend/typescript/2_typescript_middle.md#что-такое-keyof-и-typeof)

**Материалы для изучения:**

- [TypeScript: Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html)

**Задание:**

Добавьте правильную типизацию функции `getProperty` так, чтобы:
- она принимала объект и ключ этого объекта
- ключ был ограничен только существующими ключами переданного объекта (нет доступа к ключам другого объекта)
- возвращаемый тип соответствовал типу значения по этому ключу

```typescript
const someObject  = { a: 1, b: 2, c: 3, d: 4 };
const someObject2 = { m: 1, b: 2, c: 3, d: 4 };

const getProperty = (obj, key) => obj[key]; // нужно типизировать

getProperty(someObject,  "a"); // OK
getProperty(someObject,  "m"); // TS Error — "m" нет в someObject
getProperty(someObject2, "m"); // OK
```

<details>
<summary>Решение</summary>

```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// Или стрелочная функция:
const getProperty = <T, K extends keyof T>(obj: T, key: K): T[K] => obj[key];

// Проверка:
const someObject  = { a: 1, b: 2, c: 3, d: 4 };
const someObject2 = { m: 1, b: 2, c: 3, d: 4 };

getProperty(someObject,  "a"); // OK, возвращает number
getProperty(someObject,  "m"); // TS Error: Argument of type '"m"' is not assignable to parameter of type '"a" | "b" | "c" | "d"'
getProperty(someObject2, "m"); // OK, возвращает number
```

**Как это работает:**
- `T` — тип объекта, выводится из первого аргумента
- `K extends keyof T` — ключ ограничен ключами конкретного объекта `T`
- `T[K]` — indexed access type: тип значения по ключу `K`

</details>
