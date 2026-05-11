# JavaScript — Expert

## Вопросы

- [Управление памятью и сборка мусора в JavaScript?](#управление-памятью-и-сборка-мусора-в-javascript)
- [Как работает V8 под капотом?](#как-работает-v8-под-капотом)
- [Что такое Proxy и Reflect?](#что-такое-proxy-и-reflect)
- [Что такое Symbol и зачем нужен?](#что-такое-symbol-и-зачем-нужен)
- [ESM vs CommonJS — как работают модули?](#esm-vs-commonjs--как-работают-модули)
- [ArrayBuffer и TypedArray?](#arraybuffer-и-typedarray)
- [eval() и конструктор Function: зачем и почему это опасно?](#eval-и-конструктор-function-зачем-и-почему-это-опасно)
- [BigInt — когда использовать?](#bigint--когда-использовать)

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

---

## ArrayBuffer и TypedArray?

**`ArrayBuffer`** — фиксированный блок сырых бинарных данных в памяти. Напрямую недоступен — нужно создать вид (представление).

**`TypedArray`** — вид на ArrayBuffer определённого типа: `Int8Array`, `Uint8Array`, `Uint8ClampedArray`, `Int16Array`, `Uint32Array`, `Float32Array`, `Float64Array`.

**`DataView`** — гибкое представление, позволяющее читать разные типы из одного буфера.

```javascript
// Создать буфер 16 байт
const buffer = new ArrayBuffer(16);

// TypedArray-вид на этот буфер
const int32View = new Int32Array(buffer); // 4 элемента по 4 байта
int32View[0] = 42;
int32View[1] = 100;

console.log(int32View[0]); // 42

// Использование:
// - WebGL (графика), звук (Web Audio API)
// - WebSockets / fetch binary data
// - FileReader.readAsArrayBuffer
// - canvas.getImageData() — возвращает Uint8ClampedArray

// Связь между ArrayBuffer и обычным Array:
// TypedArray: фиксированный размер, тип элемента, контигуально, быстрее
// Array: динамический, любые типы
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: ArrayBuffer](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer)

---

## eval() и конструктор Function: зачем и почему это опасно?

**`eval(str)`** — выполняет переданную строку как JS-код в текущем окружении. При этом имеет доступ к локальным переменным.

**`new Function(args, body)`** — создаёт новую функцию из строк. Выполняется в глобальной области — нет доступа к локальным переменным.

```javascript
// eval
const x = 10;
eval('console.log(x)'); // 10 — видит локальные переменные

// new Function
const add = new Function('a', 'b', 'return a + b');
add(2, 3); // 5

const y = 42;
const fn = new Function('console.log(y)'); // ReferenceError: y is not defined
```

**Почему опасно:**
1. **XSS**: выполнение пользовательских данных как кода
2. **Производительность**: мешает оптимизации V8
3. **Отладка**: сложная для трессировки
4. **CSP**: политика Content Security Policy может заблокировать `eval`

**Практически никогда не использовать.** Единственный оправданный кейс `new Function` — шаблонизаторы (template compilers, JSON-базированный код), если вход полностью доверенный.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: eval](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/eval)

---

## BigInt — когда использовать?

**`BigInt`** — встроенный тип ES2020 для работы с целыми числами произвольной точности — выходящими за `Number.MAX_SAFE_INTEGER` (2^53 - 1).

```javascript
Number.MAX_SAFE_INTEGER // 9007199254740991

const big = 9007199254740991n;   // суффикс n
const big2 = BigInt('12345678901234567890');

big + 1n;  // 9007199254740992n
big * 2n;  // 18014398509481982n

// Нельзя смешивать с Number!
42n + 1;   // TypeError
42n + 1n;  // 43n — ок

// Сравнение
1n == 1;   // true  (с приведением)
1n === 1;  // false (разные типы)

// Случаи применения:
// - Криптография (большие числа для ключей)
// - Точные ID (твиттер-ID выходят за Number.MAX_SAFE_INTEGER)
// - Операции с timestamp в микросекундах

// Важно: JSON.stringify не поддерживает BigInt!
JSON.stringify(1n); // TypeError
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: BigInt](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/BigInt)
