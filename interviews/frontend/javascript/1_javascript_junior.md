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

---

## JavaScript — компилируемый или интерпретируемый язык?

JS — **интерпретируемый** язык с элементами компиляции (JIT). Современные движки (V8, SpiderMonkey) компилируют JS в машинный код во время выполнения (Just-In-Time compilation), что делает его значительно быстрее чистой интерпретации. Формально: исходный код выполняется без предварительной компиляции в отдельный артефакт.

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
