# JavaScript — Junior

## Вопросы

- [JavaScript — компилируемый или интерпретируемый язык?](#javascript--компилируемый-или-интерпретируемый-язык)
- [Какие типы данных есть в JavaScript?](#какие-типы-данных-есть-в-javascript)
- [В чём разница между null и undefined?](#в-чём-разница-между-null-и-undefined)
- [В чём разница между == и ===?](#в-чём-разница-между--и-)
- [Отличия var, let и const?](#отличия-var-let-и-const)
- [Что такое hoisting?](#что-такое-hoisting)
- [Что такое Temporal Dead Zone?](#что-такое-temporal-dead-zone)
- [Что делают typeof и instanceof?](#что-делают-typeof-и-instanceof)
- [Разница между function declaration и function expression?](#разница-между-function-declaration-и-function-expression)
- [Что такое falsy-значения и приведение к булеву типу?](#что-такое-falsy-значения-и-приведение-к-булеву-типу)
- [Что такое высшие функции (higher-order functions)?](#что-такое-высшие-функции-higher-order-functions)
- [Что такое NaN и почему NaN !== NaN?](#что-такое-nan-и-почему-nan--nan)
- [Неочевидные [приведения типов]: typeof null, 0.1 + 0.2, [] + {}](#неочевидные-приведения-типов-typeof-null-01--02---)

---

## JavaScript — компилируемый или интерпретируемый язык?

JS — **интерпретируемый** язык с элементами компиляции (JIT). Современные движки (V8, SpiderMonkey) компилируют JS в машинный код во время выполнения (Just-In-Time compilation), что делает его значительно быстрее чистой интерпретации. Формально: исходный код выполняется без предварительной компиляции в отдельный артефакт.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Какие типы данных есть в JavaScript?

**Примитивные (7 типов):**
- `string` — строка
- `number` — числа (включая `Infinity`, `NaN`)
- `bigint` — большие целые числа
- `boolean` — `true` / `false`
- `undefined` — переменная не инициализирована
- `null` — намеренное отсутствие значения
- `symbol` — уникальный идентификатор

**Ссылочные:**
- `object` (включает массивы, функции, Date, Map, Set…)

Примитивы хранятся по значению, объекты — по ссылке.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## В чём разница между null и undefined?

```javascript
let a;           // undefined — значение не присвоено
let b = null;    // null — намеренно пустое значение

typeof undefined // "undefined"
typeof null      // "object" (историческая ошибка JS)

null == undefined  // true  (нестрогое)
null === undefined // false (строгое)
```

`undefined` — системное отсутствие. `null` — программист явно говорит "значения нет".

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## В чём разница между == и ===?

`===` (строгое равенство) — сравнивает значение **и тип**. `==` — выполняет **приведение типов** (type coercion) перед сравнением.

```javascript
0 == "0"    // true  (строка "0" приводится к числу)
0 === "0"   // false (разные типы)

null == undefined  // true
null === undefined // false

[] == false  // true ([] → "" → 0, false → 0)
[] === false // false
```

**Правило**: всегда использовать `===`. `==` допустим только для проверки `null/undefined` одновременно.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Отличия var, let и const?

| | `var` | `let` | `const` |
|--|-------|-------|---------|
| Область видимости | Функция | Блок `{}` | Блок `{}` |
| Hoisting | Да (значение `undefined`) | Да (TDZ) | Да (TDZ) |
| Повторное объявление | Да | Нет | Нет |
| Переприсваивание | Да | Да | Нет |

```javascript
function example() {
  if (true) {
    var x = 1;   // видна во всей функции
    let y = 2;   // видна только в блоке if
    const z = 3; // видна только в блоке if, нельзя переприсвоить
  }
  console.log(x); // 1
  console.log(y); // ReferenceError
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое hoisting?

Hoisting (подъём) — механизм JS, при котором **объявления** переменных и функций перемещаются в начало своей области видимости во время компиляции.

```javascript
console.log(x); // undefined (не ReferenceError — var поднят)
var x = 5;

// Функциональные объявления поднимаются полностью:
greet(); // "Hello!" — работает до объявления
function greet() { console.log("Hello!"); }

// let/const — поднимаются, но не инициализируются (TDZ):
console.log(y); // ReferenceError
let y = 10;
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое Temporal Dead Zone?

TDZ — промежуток от начала блока до строки объявления `let`/`const`, в котором переменная существует, но обращение к ней вызывает `ReferenceError`.

```javascript
{
  // TDZ для name начинается здесь
  console.log(name); // ReferenceError: Cannot access 'name' before initialization
  let name = "Alice"; // TDZ заканчивается здесь
  console.log(name);  // "Alice"
}
```

TDZ защищает от использования переменных до их инициализации — ошибка раннее чем баг.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что делают typeof и instanceof?

`typeof` — возвращает строку с типом операнда:

```javascript
typeof "hello"    // "string"
typeof 42         // "number"
typeof true       // "boolean"
typeof undefined  // "undefined"
typeof null       // "object" (баг!)
typeof {}         // "object"
typeof []         // "object"
typeof function(){} // "function"
typeof Symbol()   // "symbol"
```

`instanceof` — проверяет, является ли объект экземпляром класса (через цепочку прототипов):

```javascript
[] instanceof Array   // true
[] instanceof Object  // true
"str" instanceof String // false (примитив, не объект)
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Разница между function declaration и function expression?

**Function declaration** — объявление через ключевое слово `function` в позиции инструкции. Полностью поднимается (hoisting): вызвать можно до объявления в коде.

**Function expression** — присвоение функции переменной или передача как аргумента. Не поднимается (поднимается только переменная, но без значения).

```javascript
// Function declaration — можно вызвать ДО объявления
greet(); // "Hello"
function greet() { console.log("Hello"); }

// Function expression — нельзя вызвать до присвоения
sayHi(); // TypeError: sayHi is not a function
const sayHi = function() { console.log("Hi"); };

// Named function expression (имя видно только внутри)
const factorial = function fact(n) {
  return n <= 1 ? 1 : n * fact(n - 1);
};
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: Function declaration](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Statements/function)

---

## Что такое falsy-значения и приведение к булеву типу?

Falsy-значения — значения, которые приводятся к `false` в булевом контексте. В JavaScript их ровно **8**: `false`, `0`, `-0`, `0n` (BigInt нуль), `""` (пустая строка), `null`, `undefined`, `NaN`. Всё остальное — truthy, включая `[]`, `{}`, `"0"`, `"false"`.

Оператор `!!` — двойное отрицание, явное приведение к `boolean`:

```javascript
!!0        // false
!!""       // false
!!null     // false
!!undefined // false

!![]       // true — пустой массив truthy!
!!{}       // true — пустой объект truthy!
!!"0"      // true — непустая строка truthy!

// Практичный паттерн
const isLoggedIn = Boolean(user?.id);
const hasItems  = !!cart.items.length;
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: Falsy](https://developer.mozilla.org/ru/docs/Glossary/Falsy)

---

## Что такое высшие функции (higher-order functions)?

Высшая функция (Higher-Order Function, HOF) — функция, которая принимает другую функцию как аргумент или возвращает функцию. Это основа функционального программирования в JS.

```javascript
// Принимает функцию как аргумент
[1, 2, 3].map(x => x * 2);    // [2, 4, 6]
[1, 2, 3].filter(x => x > 1); // [2, 3]
[1, 2, 3].reduce((acc, x) => acc + x, 0); // 6

// Возвращает функцию (фабрика функций)
function multiplier(factor) {
  return (number) => number * factor;
}

const double = multiplier(2);
const triple = multiplier(3);

double(5); // 10
triple(5); // 15
```

**Связанные задачи:**

- [Функция capitalize](../../../tasks/frontend/javascript/1_javascript_junior.md#функция-capitalize)

**Материалы для изучения:**

- [MDN: Функции высшего порядка](https://developer.mozilla.org/ru/docs/Glossary/First-class_Function)

---

## Что такое NaN и почему NaN !== NaN?

`NaN` (Not a Number) — специальное числовое значение JavaScript, означающее результат невалидной числовой операции. Имеет три особенности:

1. `typeof NaN === 'number'` — это тип `number`, смотря на название
2. `NaN !== NaN` — **единственное** значение JS, не равное самому себе
3. Проверять наличие NaN нужно через `Number.isNaN()` или `isNaN()`

```javascript
typeof NaN        // "number"
NaN === NaN       // false
NaN !== NaN       // true
Math.sqrt(-1)     // NaN

// Неправильный способ проверки
if (value === NaN) { ... } // никогда не выполнится!

// Правильные способы
Number.isNaN(NaN) // true  (не приводит аргумент)
isNaN('hello')    // true  (сначала приводит к number!)
Number.isNaN('hello') // false

// Через Object.is (ES6)
Object.is(NaN, NaN) // true
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: NaN](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/NaN)

---

## Неочевидные приведения типов: typeof null, 0.1 + 0.2, [] + {}

Несколько классических подвохов JavaScript, с которыми часто спрашивают на интервью:

```javascript
// 1. typeof null — историческая ошибка в JS
typeof null === 'object' // true (но null не объект!)
typeof undefined === 'undefined' // true

// 2. Плавающая точка (IEEE 754)
0.1 + 0.2 === 0.3 // false !
0.1 + 0.2         // 0.30000000000000004
// Решение: сравнивать через погрешность или Number.EPSILON
Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON // true

// 3. [] + {} вс. {} + []
[] + {}  // "[object Object]"  — массив в "", объект в "[object Object]"
{} + []  // 0  — считается блоком кода, +[] == 0

// 4. Невероятные сравнения с приведением
1 < 2 < 3  // true  (1<2=true, true<3 = 1<3 = true)
3 > 2 > 1  // false (3>2=true, true>1 = 1>1 = false!)
```

Итог: в коде производства серьёзных приложений всегда используй `===` вместо `==`, для чисел с плавающей точкой используй `Number.EPSILON`, проверку на NaN делай через `Number.isNaN()`.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: Number.EPSILON](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Number/EPSILON)
