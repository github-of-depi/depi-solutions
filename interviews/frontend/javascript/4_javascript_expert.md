# JavaScript — Expert

## Вопросы

- [Управление памятью и сборка мусора в JavaScript?](#управление-памятью-и-сборка-мусора-в-javascript)
- [Как работает V8 под капотом?](#как-работает-v8-под-капотом)
- [Что такое Proxy и Reflect?](#что-такое-proxy-и-reflect)
- [Что такое Symbol и зачем нужен?](#что-такое-symbol-и-зачем-нужен)
- [ESM vs CommonJS — как работают модули?](#esm-vs-commonjs--как-работают-модули)

---

## Управление памятью и сборка мусора в JavaScript?

JS управляет памятью автоматически через **Garbage Collector** (GC). Алгоритм: **Mark-and-Sweep** — помечает все достижимые объекты от корней (глобальный объект, стек вызовов), затем удаляет непомеченные.

**Утечки памяти в браузере:**

```javascript
// 1. Глобальные переменные
function leak() { globalVar = "утечка"; } // забыли let/const

// 2. Незакрытые таймеры со ссылками
const data = getLargeData();
setInterval(() => useData(data), 1000); // data не освободится

// 3. Неудалённые event listeners
const handler = () => {};
element.addEventListener("click", handler);
// Нужно: element.removeEventListener("click", handler);

// 4. Замыкания с большими данными
function createClosure() {
  const hugeArray = new Array(1_000_000).fill(0);
  return () => hugeArray[0]; // hugeArray не освобождается
}
```

**WeakRef** — слабая ссылка, не мешает GC:

```javascript
const weakRef = new WeakRef(heavyObject);
// Позже:
const obj = weakRef.deref(); // undefined если уже GC'd
if (obj) useObject(obj);
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работает V8 под капотом?

V8 (Google Chrome, Node.js) — JIT-компилятор JS:

1. **Parsing** → AST (Abstract Syntax Tree)
2. **Ignition** (интерпретатор) → байткод, выполняется сразу
3. **Turbofan** (оптимизирующий компилятор) → машинный код для горячих функций

**Hidden Classes** — V8 создаёт внутренние классы для объектов с одинаковой структурой. Если добавлять свойства в одинаковом порядке — объекты делят hidden class → быстрее:

```javascript
// Хорошо — одна hidden class
function Point(x, y) { this.x = x; this.y = y; }

// Плохо — разные hidden classes → deoptimization
const p1 = {}; p1.x = 1; p1.y = 2;
const p2 = {}; p2.y = 1; p2.x = 2; // другой порядок!
```

**Deoptimization** — если функция получает разные типы аргументов, Turbofan "откатывается" к интерпретатору.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое Proxy и Reflect?

**Proxy** — перехватчик операций над объектом (get, set, has, apply, construct…):

```javascript
const handler = {
  get(target, key) {
    console.log(`Читаем ${key}`);
    return Reflect.get(target, key);
  },
  set(target, key, value) {
    if (typeof value !== "number") throw new TypeError("Только числа!");
    return Reflect.set(target, key, value);
  },
};

const validated = new Proxy({}, handler);
validated.age = 25;   // OK
validated.age = "25"; // TypeError

// Реактивность в Vue 3 построена на Proxy
```

**Reflect** — зеркало стандартных объектных операций. Возвращает булево при set (не выбрасывает), унифицирует API:

```javascript
Reflect.get(obj, "key");           // obj["key"]
Reflect.set(obj, "key", value);    // obj["key"] = value
Reflect.has(obj, "key");           // "key" in obj
Reflect.deleteProperty(obj, "key"); // delete obj["key"]
Reflect.ownKeys(obj);              // все ключи включая Symbol
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое Symbol и зачем нужен?

Symbol — уникальный примитивный тип. Каждый Symbol уникален, даже с одним описанием:

```javascript
const s1 = Symbol("id");
const s2 = Symbol("id");
s1 === s2; // false — всегда уникальны

// Приватные ключи объекта:
const _private = Symbol("private");
class MyClass {
  constructor() { this[_private] = "secret"; }
}
// Недоступно через перебор (for...in, Object.keys), но через Reflect.ownKeys

// Well-known symbols — переопределяют поведение:
class Range {
  constructor(from, to) { this.from = from; this.to = to; }
  [Symbol.iterator]() {
    let current = this.from;
    return {
      next: () => current <= this.to
        ? { value: current++, done: false }
        : { done: true },
    };
  }
}
for (const n of new Range(1, 3)) console.log(n); // 1, 2, 3
```

Well-known symbols: `Symbol.iterator`, `Symbol.toPrimitive`, `Symbol.hasInstance`, `Symbol.toStringTag`.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## ESM vs CommonJS — как работают модули?

**CommonJS (CJS)** — Node.js исторический стандарт:

```javascript
// Синхронный, динамический
const fs = require("fs");          // загружается в runtime
module.exports = { myFunc };
const { myFunc } = require("./mod"); // может быть в условии
```

**ES Modules (ESM)** — стандарт браузера и современного Node:

```javascript
// Статический анализ, асинхронный
import { myFunc } from "./mod.js"; // только на верхнем уровне
export const myFunc = () => {};
export default class MyClass {}

// Динамический import:
const { myFunc } = await import("./mod.js"); // runtime, любое место
```

**Ключевые отличия:**

| | CJS | ESM |
|--|-----|-----|
| Загрузка | Синхронная | Асинхронная |
| Анализ | Runtime | Static (build time) |
| Tree shaking | Нет | Да |
| `this` в корне | `module.exports` | `undefined` |
| Live bindings | Нет (копия) | Да (ссылка) |

ESM live bindings — экспортируемое значение обновляется автоматически у всех импортеров при изменении.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
