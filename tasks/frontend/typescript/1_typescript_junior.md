# TypeScript — Junior — Задачи

## Задачи

- [Типизация функции с union-параметром](#типизация-функции-с-union-параметром)
- [Описание типов объектов](#описание-типов-объектов)

---

## Типизация функции с union-параметром

**Сложность:** Easy

**Связанные вопросы:**

- [Что такое any, unknown и never?](../../../interviews/frontend/typescript/1_typescript_junior.md#что-такое-any-unknown-и-never)
- [Что такое type assertion?](../../../interviews/frontend/typescript/1_typescript_junior.md#что-такое-type-assertion)

**Материалы для изучения:**

- [TypeScript: Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)

**Задание:**

Напишите функцию `formatValue(value)`, которая принимает `string | number | boolean` и возвращает отформатированную строку:

- `string` → возвращает строку в верхнем регистре
- `number` → возвращает число с 2 знаками после запятой
- `boolean` → возвращает `"Да"` или `"Нет"`

Функция должна быть полностью типизирована без `any`.

**Пример:**

```typescript
formatValue("hello")  // "HELLO"
formatValue(3.14159)  // "3.14"
formatValue(true)     // "Да"
formatValue(false)    // "Нет"
```

<details>
<summary>Решение</summary>

```typescript
function formatValue(value: string | number | boolean): string {
  if (typeof value === "string") {
    return value.toUpperCase();
  }
  if (typeof value === "number") {
    return value.toFixed(2);
  }
  return value ? "Да" : "Нет";
}
```

</details>

---

## Описание типов объектов

**Сложность:** Easy

**Связанные вопросы:**

- [В чём разница между type и interface?](../../../interviews/frontend/typescript/1_typescript_junior.md#в-чём-разница-между-type-и-interface)
- [Что такое опциональные поля и оператор ??](../../../interviews/frontend/typescript/1_typescript_junior.md#что-такое-опциональные-поля-и-оператор-)

**Материалы для изучения:**

- [TypeScript: Object Types](https://www.typescriptlang.org/docs/handbook/2/objects.html)

**Задание:**

Опишите типы для следующего сценария. Используйте `interface` для `User` и `type` для `Role`. Все ограничения должны быть на уровне типов, без значений по умолчанию в рантайме.

Требования:
- `User` имеет обязательные поля `id` (число), `name` (строка), `email` (строка)
- `User` имеет опциональное поле `avatarUrl` (строка)
- `User` имеет поле `role` — одно из значений: `"admin"`, `"editor"`, `"viewer"`
- `User` имеет поле `createdAt` — только для чтения
- `UserPreview` — тип, содержащий только `id` и `name` из `User`

**Пример:**

```typescript
// Должно работать без ошибок:
const user: User = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
  role: "admin",
  createdAt: new Date(),
};

// Должно работать:
const preview: UserPreview = { id: 1, name: "Alice" };

// Должна быть ошибка:
user.createdAt = new Date(); // readonly
```

<details>
<summary>Решение</summary>

```typescript
type Role = "admin" | "editor" | "viewer";

interface User {
  id: number;
  name: string;
  email: string;
  role: Role;
  readonly createdAt: Date;
  avatarUrl?: string;
}

// Pick берёт только нужные поля — не нужно дублировать вручную
type UserPreview = Pick<User, "id" | "name">;
```

</details>
