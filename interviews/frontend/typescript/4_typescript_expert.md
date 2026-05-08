# TypeScript — Expert

## Вопросы

- [Что такое HKT (Higher-Kinded Types) и как их эмулировать в TypeScript?](#что-такое-hkt-и-как-их-эмулировать-в-typescript)
- [Как работает TypeScript компилятор — основные фазы?](#как-работает-typescript-компилятор--основные-фазы)
- [Что такое module augmentation и ambient declarations?](#что-такое-module-augmentation-и-ambient-declarations)
- [Как писать типобезопасные трансформации данных на уровне типов?](#как-писать-типобезопасные-трансформации-данных-на-уровне-типов)
- [Как оптимизировать производительность TypeScript в большом проекте?](#как-оптимизировать-производительность-typescript-в-большом-проекте)
- [Что такое .map файлы (Source Maps) и как они работают?](#что-такое-map-файлы-source-maps-и-как-они-работают)

---

## Что такое HKT и как их эмулировать в TypeScript?

Higher-Kinded Types — типы, принимающие другие типы-конструкторы в качестве параметров (`Functor<F>`, где `F` — это `Array`, `Promise` и т.д.). TypeScript не поддерживает HKT нативно, но их можно эмулировать через interface merging и lookup типы (паттерн «HKT encoding»).

```typescript
// Реестр типов-конструкторов
interface URItoKind<A> {}
type URIS = keyof URItoKind<unknown>;

// Регистрация Array
declare module "./hkt" {
  interface URItoKind<A> { Array: Array<A> }
}

// Тип Functor
interface Functor<F extends URIS> {
  map<A, B>(fa: URItoKind<A>[F], f: (a: A) => B): URItoKind<B>[F];
}
```

---

## Как работает TypeScript компилятор — основные фазы?

1. **Scanner** — токенизация исходного кода в поток токенов
2. **Parser** — построение AST (Abstract Syntax Tree) из токенов
3. **Binder** — построение Symbol Table: связывает имена с их объявлениями, определяет scopes
4. **Checker** — type checking: анализ AST с Symbol Table, вывод и проверка типов (самая медленная фаза)
5. **Emitter** — генерация JavaScript и `.d.ts` файлов из типизированного AST

TypeScript Compiler API (`ts.createProgram`) даёт программный доступ ко всем фазам для написания кастомных трансформеров и линтеров.

---

## Что такое module augmentation и ambient declarations?

Module augmentation — расширение существующих модулей через `declare module "name" { }`. Ambient declarations (`declare`) описывают типы для кода без TypeScript (глобальные переменные, CDN-скрипты). `.d.ts` файлы содержат только ambient declarations.

```typescript
// Расширение сторонней библиотеки
declare module "express-serve-static-core" {
  interface Request { userId?: string; }
}

// Ambient declaration для глобальной переменной из CDN
declare const __VERSION__: string;
declare function gtag(command: string, ...args: unknown[]): void;
```

---

## Как писать типобезопасные трансформации данных на уровне типов?

Через рекурсивные conditional types и `infer` можно строить сложные трансформации: DeepPartial, DeepReadonly, Paths (все пути объекта), PathValue (тип значения по пути).

```typescript
// Все вложенные пути объекта (упрощённо)
type Paths<T, K extends keyof T = keyof T> =
  K extends string
    ? T[K] extends Record<string, unknown>
      ? `${K}.${Paths<T[K]>}` | K
      : K
    : never;

type UserPaths = Paths<{ user: { name: string; age: number } }>;
// "user" | "user.name" | "user.age"

// Значение по пути
type PathValue<T, P extends string> =
  P extends `${infer K}.${infer Rest}`
    ? K extends keyof T ? PathValue<T[K], Rest> : never
    : P extends keyof T ? T[P] : never;
```

---

## Как оптимизировать производительность TypeScript в большом проекте?

1. **Project references** (`composite: true`) — разбивка монолита на подпроекты с инкрементальной компиляцией
2. **skipLibCheck: true** — пропуск проверки `.d.ts` файлов из `node_modules`
3. **Избегать сложных рекурсивных типов** — они экспоненциально замедляют checker
4. **isolatedModules: true** — каждый файл компилируется независимо (необходимо для esbuild/SWC)
5. **`tsc --noEmit` + Turborepo/nx caching** — кэширование результатов type-check
6. **Профилирование**: `tsc --generateTrace trace` → анализ в `chrome://tracing`

---

## Что такое .map файлы (Source Maps) и как они работают?

Source Maps — файлы формата JSON (`.js.map`, `.ts.map`), создающие маппинг между скомпилированным кодом и исходным TypeScript. Браузер/Node использует их для показа реального источника ошибок.

```json
// tsconfig.json
{
  "compilerOptions": {
    "sourceMap": true,       // генерирует file.js.map
    "inlineSourceMap": false, // встроить в JS как data URL (не рекомендуется)
    "inlineSources": true,   // включить исходный .ts в .map файл
    "declarationMap": true   // .d.ts.map для навигации по типам
  }
}
```

Структура `.map` файла:
```json
{
  "version": 3,
  "file": "app.js",
  "sources": ["../src/app.ts"],
  "sourcesContent": ["...исходный TypeScript..."],
  "names": ["MyClass", "method"],
  "mappings": "AAAA,IAAM,OAAO..."  // VLQ-encoded позиции
}
```

Ключевые моменты:
- **`mappings`** — Base64 VLQ-кодированные координаты: (столбец JS → файл → строка TS → столбец TS → имя)
- **DevTools** автоматически подхватывает `.map` если файл содержит комментарий `//# sourceMappingURL=app.js.map`
- **В production**: source maps лучше загружать отдельно (только для Sentry/мониторинга), не отдавать публично
- **Sentry integration**: upload `.map` файлов позволяет видеть оригинальный TypeScript в стектрейсах ошибок
- **esbuild/Vite** генерируют source maps значительно быстрее `tsc`
