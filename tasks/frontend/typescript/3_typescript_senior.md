# TypeScript — Senior — Задачи

## Задачи

- [Пути к листьям вложенного объекта](#пути-к-листьям-вложенного-объекта)
- [ResolvableKeysOf](#resolvablekeysof)

---

## Пути к листьям вложенного объекта

**Сложность:** Hard

**Связанные вопросы:**

- [Что такое conditional types?](../../../interviews/frontend/typescript/2_typescript_middle.md#что-такое-conditional-types)
- [Что такое template literal types?](../../../interviews/frontend/typescript/3_typescript_senior.md#что-такое-template-literal-types)

**Материалы для изучения:**

- [TypeScript: Template Literal Types](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html)
- [TypeScript: Recursive Conditional Types](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-1.html#recursive-conditional-types)

**Задание:**

Есть объект бесконечной вложенности, где конечные (листовые) значения всегда `string`. Напишите тип `LeafPaths<T>`, который генерирует union из всех путей к листовым ключам через точечную нотацию.

```typescript
const i18n = {
  greeting: {
    hello: "Hello",
    goodbye: "Goodbye",
  },
  user: {
    profile: "Profile",
    settings: "Settings",
  },
} as const;

type Paths = LeafPaths<typeof i18n>;
// Ожидается: "greeting.hello" | "greeting.goodbye" | "user.profile" | "user.settings"

function t(path: LeafPaths<typeof i18n>): string { /* ... */ }

t("greeting.hello");   // OK
t("greeting.goodbye"); // OK
t("user.profile");     // OK
t("greeting");         // TS Error — не листовой путь
t("user.missing");     // TS Error — не существует
```

<details>
<summary>Решение</summary>

```typescript
type LeafPaths<T, Prefix extends string = ""> =
  T extends string
    ? Prefix extends "" ? never : Prefix  // листовое значение — возвращаем накопленный путь
    : {
        [K in keyof T & string]:
          LeafPaths<
            T[K],
            Prefix extends "" ? K : `${Prefix}.${K}`
          >
      }[keyof T & string];

// Проверка:
const i18n = {
  greeting: { hello: "Hello", goodbye: "Goodbye" },
  user: { profile: "Profile", settings: "Settings" },
} as const;

type Paths = LeafPaths<typeof i18n>;
// "greeting.hello" | "greeting.goodbye" | "user.profile" | "user.settings"

function t(path: Paths): string {
  return path.split(".").reduce((obj: Record<string, unknown>, key) => {
    return (obj as Record<string, unknown>)[key] as Record<string, unknown>;
  }, i18n as unknown as Record<string, unknown>) as unknown as string;
}

t("greeting.hello");   // OK → "Hello"
t("user.profile");     // OK → "Profile"
t("greeting");         // TS Error
```

**Как это работает:**
- Рекурсия: если `T extends string` — дошли до листа, возвращаем накопленный `Prefix`
- Иначе итерируем по ключам через mapped type и рекурсивно углубляемся
- `Prefix extends "" ? K : \`${Prefix}.${K}\`` — строим путь через точку
- `[keyof T & string]` — дистрибуция: собирает все результаты в union

</details>

---

## ResolvableKeysOf

**Сложность:** Hard

**Связанные вопросы:**

- [Что такое conditional types?](../../../interviews/frontend/typescript/2_typescript_middle.md#что-такое-conditional-types)
- [Какие utility types есть в TypeScript?](../../../interviews/frontend/typescript/2_typescript_middle.md#какие-utility-types-есть-в-typescript)

**Материалы для изучения:**

- [TypeScript: Conditional Types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html)

**Задание:**

Реализуйте тип `ResolvableKeysOf<T, R>`, который возвращает union из ключей класса/объекта `T`, чьи значения «разрешаются» в тип `R`.

Под «разрешением» понимается: значение является `R`, либо `Promise<R>`, либо `() => R`, либо `() => Promise<R>`.

```typescript
class Test {
  public a = 1;
  public b = 2;
  public c = Promise.resolve(3);
  public d = () => 5;
  public e = () => Promise.resolve(6);

  public x = "not";
  public y = Promise.resolve(null);
  public z = () => true;
}

type NumberKeys = ResolvableKeysOf<Test, number>;       // "a" | "b" | "c" | "d" | "e"
type StringKeys = ResolvableKeysOf<Test, string>;       // "x"
type NullKeys   = ResolvableKeysOf<Test, null>;         // "y"
type BoolKeys   = ResolvableKeysOf<Test, boolean>;      // "z"
type StrOrNull  = ResolvableKeysOf<Test, string | null>; // "x" | "y"
```

<details>
<summary>Решение</summary>

```typescript
// Вспомогательный тип: разворачивает значение до его «корневого» типа
type Resolve<V> =
  V extends () => Promise<infer R> ? R :
  V extends () => infer R          ? R :
  V extends Promise<infer R>       ? R :
  V;

// ResolvableKeysOf: ключи, чей Resolve<V> расширяет R
type ResolvableKeysOf<T, R> = {
  [K in keyof T]: Resolve<T[K]> extends R ? K : never;
}[keyof T];

// Проверка:
type NumberKeys = ResolvableKeysOf<Test, number>;        // "a" | "b" | "c" | "d" | "e"
type StringKeys = ResolvableKeysOf<Test, string>;        // "x"
type NullKeys   = ResolvableKeysOf<Test, null>;          // "y"
type BoolKeys   = ResolvableKeysOf<Test, boolean>;       // "z"
type StrOrNull  = ResolvableKeysOf<Test, string | null>; // "x" | "y"
```

**Как это работает:**

1. `Resolve<V>` — conditional type с `infer`:
   - Сначала проверяем `() => Promise<R>` (самый специфичный случай)
   - Затем `() => R` (функция без промиса)
   - Затем `Promise<R>` (не функция, но промис)
   - Иначе возвращаем `V` как есть

2. `ResolvableKeysOf` — mapped type с дистрибуцией:
   - `[K in keyof T]: ... ? K : never` — возвращаем ключ если условие выполнено, иначе `never`
   - `[keyof T]` — собираем всё в union (never отфильтровываются автоматически)

</details>
