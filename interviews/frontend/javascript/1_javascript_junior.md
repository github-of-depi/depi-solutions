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
- [Какой тип у функции в JS?](#какой-тип-у-функции-в-js)
- [Какие виды функций существуют в JavaScript?](#какие-виды-функций-существуют-в-javascript)
- [Чем похожи объект, массив и функция?](#чем-похожи-объект-массив-и-функция)
- [Какие операторы есть в JavaScript?](#какие-операторы-есть-в-javascript)
- [Инкремент и декремент: ++/--](#инкремент-и-декремент-)
- [Управляющие конструкции: if и switch..case](#управляющие-конструкции-if-и-switchcase)
- [Какие циклы есть в JS?](#какие-циклы-есть-в-js)
- [Разница между for..of и for..in](#разница-между-forof-и-forin)
- [Как создать объект в JavaScript?](#как-создать-объект-в-javascript)
- [Что делает ключевое слово new?](#что-делает-ключевое-слово-new)
- [Что такое ES / ECMAScript?](#что-такое-es--ecmascript)
- [Диалоговые окна: alert, prompt, confirm](#диалоговые-окна-alert-prompt-confirm)
- [Как проверить, является ли объект массивом?](#как-проверить-является-ли-объект-массивом)
- [Как проверить, является ли число конечным?](#как-проверить-является-ли-число-конечным)
- [setTimeout и setInterval — отличия и применение](#settimeout-и-setinterval--отличия-и-применение)
- [Object.keys, Object.values, Object.entries](#objectkeys-objectvalues-objectentries)
- [Оператор нулевого слияния (??) и опциональная цепочка (?.)](#оператор-нулевого-слияния--и-опциональная-цепочка-)
- [Методы массивов — обзор](#методы-массивов--обзор)
- [Обработка ошибок: try..catch..finally](#обработка-ошибок-trycatchfinally)
- [use strict — зачем нужен?](#use-strict--зачем-нужен)
- [Что такое функция обратного вызова (callback)?](#что-такое-функция-обратного-вызова-callback)
- [Что такое модули (import/export)?](#что-такое-модули-importexport)
- [Как очистить массив?](#как-очистить-массив)

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

---

## Какой тип у функции в JS?

`typeof function() {} === 'function'` — у функций есть собственный тип. Однако функция — это **объект** (подтип объекта): у неё есть свойства (`name`, `length`), её можно присвоить переменной, передать аргументом, вернуть из другой функции. `instanceof Function === true` и `instanceof Object === true`.

```javascript
function foo() {}

typeof foo          // "function"
foo instanceof Function  // true
foo instanceof Object    // true

foo.name   // "foo"
foo.length // количество объявленных параметров
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Какие виды функций существуют в JavaScript?

| Вид | Синтаксис | Особенности |
|---|---|---|
| Function declaration | `function f() {}` | Hoisting, собственный `this` |
| Function expression | `const f = function() {}` | Нет hoisting |
| Arrow function | `const f = () => {}` | Нет `this`, `arguments` |
| Анонимная | `function() {}` | Нет имени, как выражение |
| Named function expression | `const f = function foo() {}` | Имя видно только внутри |
| IIFE | `(function() {})()` | Немедленный вызов |
| Генератор | `function* g() {}` | `yield`, возвращает итератор |
| Async | `async function f() {}` | `await`, возвращает Promise |
| Метод | `{ f() {} }` | Сокращённый синтаксис метода объекта |

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Чем похожи объект, массив и функция?

Все три — **объекты** (reference types). Хранятся в куче, переменная содержит ссылку, а не значение. У всех есть свойства и методы через прототипную цепочку.

```javascript
typeof {}  // "object"
typeof []  // "object"  — массив тоже объект!
typeof function(){} // "function" — но тоже объект

[] instanceof Object    // true
function(){} instanceof Object // true

// Массив — объект с числовыми ключами и length
const arr = [1, 2];
arr.foo = 'bar';   // можно добавить произвольное свойство (не рекомендуется)

// Функция — объект с вызываемостью
const fn = () => {};
fn.meta = 42;      // тоже можно
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Какие операторы есть в JavaScript?

- **Арифметические:** `+`, `-`, `*`, `/`, `%`, `**` (возведение в степень)
- **Сравнения:** `==`, `!=`, `===`, `!==`, `<`, `>`, `<=`, `>=`
- **Логические:** `&&`, `||`, `!`, `??` (нулевое слияние)
- **Присваивания:** `=`, `+=`, `-=`, `*=`, `/=`, `&&=`, `||=`, `??=`
- **Унарные:** `typeof`, `void`, `delete`, `!`, `-`, `+`, `~`
- **Тернарный:** `condition ? a : b`
- **Побитовые:** `&`, `|`, `^`, `~`, `<<`, `>>`, `>>>`
- **Spread/Rest:** `...`
- **Опциональная цепочка:** `?.`

**Унарный** — один операнд (`typeof x`). **Бинарный** — два операнда (`a + b`). **Тернарный** — три операнда (`cond ? a : b`).

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Инкремент и декремент: ++/--

`++` увеличивает значение на 1, `--` уменьшает. Разница между **постфиксной** и **префиксной** формами:

- **Постфикс** (`x++`) — возвращает старое значение, затем увеличивает.
- **Префикс** (`++x`) — сначала увеличивает, затем возвращает новое значение.

```javascript
let a = 5;
console.log(a++); // 5  — вернул старое, затем стал 6
console.log(a);   // 6

let b = 5;
console.log(++b); // 6  — сначала увеличил, вернул 6
console.log(b);   // 6

// Типичная ловушка в выражениях
let i = 1;
let j = i++ + i; // j = 1 + 2 = 3 (i++ вернул 1, потом i стало 2)
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Управляющие конструкции: if и switch..case

**if/else if/else** — проверяет условие, выполняет блок при `true`.

**switch..case** — сравнивает значение с константами через `===`. Нужен `break` чтобы не провалиться в следующий case (fall-through). `default` — блок по умолчанию.

```javascript
// if
const grade = 85;
if (grade >= 90) console.log('A');
else if (grade >= 80) console.log('B'); // выведет 'B'
else console.log('C');

// switch
const day = 'Monday';
switch (day) {
  case 'Saturday':
  case 'Sunday':
    console.log('Weekend');
    break;
  case 'Monday':
    console.log('Work'); // выведет 'Work'
    break;
  default:
    console.log('Other');
}
```

Когда использовать `switch`: много вариантов сравнения с одним значением. `if` — для сложных условий с диапазонами и логическими выражениями.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Какие циклы есть в JS?

| Цикл | Применение |
|---|---|
| `for (init; cond; step)` | Когда известно число итераций |
| `while (cond)` | Пока условие истинно |
| `do..while` | Выполнить хотя бы раз, затем проверять |
| `for..of` | Итерируемые объекты (массив, строка, Set, Map) |
| `for..in` | Ключи объекта (включая прототипные!) |
| `forEach` | Метод массива, нет `break`/`return` для выхода |

`break` — прервать цикл. `continue` — пропустить итерацию.

```javascript
// for..of — значения
for (const item of [1, 2, 3]) console.log(item);

// for..in — ключи (осторожно с прототипами!)
for (const key in { a: 1, b: 2 }) console.log(key); // 'a', 'b'

// while vs do..while
let n = 0;
while (n > 0) { } // тело не выполнится
do { console.log(n); } while (n > 0); // выполнится один раз
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Разница между for..of и for..in

**`for..of`** — итерирует **значения** итерируемых объектов (массив, строка, Set, Map, NodeList). Работает через протокол `[Symbol.iterator]`.

**`for..in`** — итерирует **ключи** объекта, включая ключи из цепочки прототипов. Порядок не гарантирован. Для массивов лучше не использовать — возвращает строковые индексы и может включить лишние ключи.

```javascript
const arr = [10, 20, 30];
arr.foo = 'bar';

for (const val of arr)  console.log(val); // 10, 20, 30
for (const key in arr)  console.log(key); // '0', '1', '2', 'foo' — лишнее!

const obj = { a: 1, b: 2 };
for (const key in obj)  console.log(key);   // 'a', 'b'
for (const val of obj)  console.log(val);   // TypeError: obj is not iterable
```

**Правило:** `for..of` для массивов/итерируемых; `for..in` с `hasOwnProperty` для объектов (или лучше `Object.keys()`).

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как создать объект в JavaScript?

Несколько способов:

```javascript
// 1. Литерал объекта (самый частый)
const obj = { name: 'Alice', age: 30 };

// 2. new Object()
const obj2 = new Object();
obj2.name = 'Bob';

// 3. Функция-конструктор
function Person(name) { this.name = name; }
const p = new Person('Carol');

// 4. Object.create(proto) — задаёт прототип явно
const animal = { speak() { console.log('...'); } };
const dog = Object.create(animal);
dog.bark = () => console.log('Woof');

// 5. Класс (ES6)
class Car { constructor(model) { this.model = model; } }
const car = new Car('Tesla');

// 6. Фабричная функция
const createUser = (name) => ({ name, greet() { return `Hi, ${this.name}`; } });
const user = createUser('Dave');

// 7. Object.assign / spread
const copy = Object.assign({}, obj);
const spread = { ...obj, extra: true };
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что делает ключевое слово new?

`new Constructor()` выполняет 4 шага:
1. Создаёт новый пустой объект `{}`
2. Устанавливает `__proto__` этого объекта в `Constructor.prototype`
3. Выполняет тело конструктора, привязывая `this` к новому объекту
4. Возвращает объект (или явно возвращённый объект, если конструктор его возвращает)

```javascript
function User(name) {
  this.name = name;
  // неявный return this
}

const u = new User('Alice');
u instanceof User // true
u.__proto__ === User.prototype // true

// Эмуляция new вручную
function myNew(Constructor, ...args) {
  const obj = Object.create(Constructor.prototype);
  const result = Constructor.apply(obj, args);
  return result instanceof Object ? result : obj;
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое ES / ECMAScript?

**ECMAScript** — стандарт языка, который определяет синтаксис и поведение JavaScript. Разрабатывается комитетом **TC39**. JavaScript — самая известная реализация этого стандарта.

Ключевые версии:
- **ES5** (2009) — `"use strict"`, JSON, Array методы
- **ES6 / ES2015** — `let/const`, стрелки, классы, Promise, деструктуризация, модули, `...rest/spread`, `Map/Set`, шаблонные строки
- **ES7 (2016)** — `Array.prototype.includes`, `**` (возведение в степень)
- **ES8 (2017)** — `async/await`, `Object.entries/values`
- **ES2020+** — `??`, `?.`, `BigInt`, `Promise.allSettled`, `globalThis`, `structuredClone`, `Array.at()`

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Диалоговые окна: alert, prompt, confirm

Встроенные браузерные функции для взаимодействия с пользователем. Блокируют выполнение скрипта до ответа пользователя (синхронные). В современных приложениях практически не используются — вместо них UI-компоненты.

```javascript
alert('Сообщение');          // показывает сообщение, возвращает undefined
const name = prompt('Имя?', 'default'); // возвращает строку или null (Cancel)
const ok = confirm('Уверены?');         // возвращает true/false
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как проверить, является ли объект массивом?

```javascript
Array.isArray([]);        // true  — правильный способ
Array.isArray({});        // false
Array.isArray('string');  // false

// Почему не typeof?
typeof [] // "object" — не отличает массив от объекта

// Почему не instanceof?
// Ломается при передаче массива из другого iframe (другой Array)
[] instanceof Array // true (работает в одном контексте)

// Универсальный вариант:
Object.prototype.toString.call([]) // "[object Array]"
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как проверить, является ли число конечным?

```javascript
Number.isFinite(42);        // true
Number.isFinite(Infinity);  // false
Number.isFinite(-Infinity); // false
Number.isFinite(NaN);       // false
Number.isFinite('42');      // false — не приводит тип!

// Старый isFinite() — приводит аргумент к числу
isFinite('42');  // true  — '42' → 42 → конечное
isFinite('foo'); // false — 'foo' → NaN

// Проверка на целое число
Number.isInteger(3.0)  // true
Number.isInteger(3.5)  // false
Number.isSafeInteger(Number.MAX_SAFE_INTEGER) // true
```

Предпочитай `Number.isFinite()` — она строже и не приводит тип.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## setTimeout и setInterval — отличия и применение

**`setTimeout(fn, ms)`** — выполняет функцию **один раз** через `ms` миллисекунд. Возвращает id таймера.

**`setInterval(fn, ms)`** — вызывает функцию **повторно** каждые `ms` мс. Возвращает id интервала.

Обе функции асинхронны — колбэк попадает в очередь макрозадач, не блокирует выполнение.

```javascript
const timerId = setTimeout(() => console.log('once'), 1000);
clearTimeout(timerId); // отменить, пока не сработал

const intervalId = setInterval(() => console.log('repeat'), 500);
clearInterval(intervalId); // остановить интервал

// Рекурсивный setTimeout = более точный интервал
// (следующий вызов планируется ПОСЛЕ завершения предыдущего)
function tick() {
  doWork();
  setTimeout(tick, 1000);
}
tick();

// setTimeout(fn, 0) — выполнить после текущего синхронного кода
setTimeout(() => console.log('after sync'), 0);
console.log('sync'); // выведется первым
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Object.keys, Object.values, Object.entries

Три статических метода для перебора **собственных перечислимых** свойств объекта:

```javascript
const obj = { a: 1, b: 2, c: 3 };

Object.keys(obj)    // ['a', 'b', 'c']    — массив ключей
Object.values(obj)  // [1, 2, 3]          — массив значений
Object.entries(obj) // [['a',1],['b',2],['c',3]] — массив пар [key, value]

// Практичные паттерны
const doubled = Object.fromEntries(
  Object.entries(obj).map(([k, v]) => [k, v * 2])
); // { a: 2, b: 4, c: 6 }

// Итерация по объекту
for (const [key, value] of Object.entries(obj)) {
  console.log(`${key}: ${value}`);
}

// Не включают прототипные свойства (в отличие от for..in)
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Оператор нулевого слияния (??) и опциональная цепочка (?.)

**`??` (Nullish Coalescing)** — возвращает правый операнд только если левый `null` или `undefined`. В отличие от `||`, не срабатывает на `0`, `false`, `''`.

**`?.` (Optional Chaining)** — безопасный доступ к свойству/методу: возвращает `undefined` вместо ошибки, если цепочка прерывается на `null`/`undefined`.

```javascript
// ?? vs ||
const value = 0;
console.log(value || 'default');  // 'default' — 0 falsy!
console.log(value ?? 'default');  // 0  — 0 не null/undefined

// ?.
const user = null;
console.log(user?.address?.city); // undefined (не выбросит ошибку)
console.log(user?.getName?.());   // undefined (безопасный вызов метода)
console.log(user?.friends?.[0]);  // undefined (безопасное обращение по индексу)

// Комбинация
const city = user?.address?.city ?? 'Не указан';
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Методы массивов — обзор

**Мутирующие** (изменяют оригинал): `push`, `pop`, `shift`, `unshift`, `splice`, `sort`, `reverse`, `fill`, `copyWithin`

**Немутирующие** (возвращают новый): `map`, `filter`, `reduce`, `slice`, `concat`, `flat`, `flatMap`, `find`, `findIndex`, `some`, `every`, `includes`, `indexOf`, `join`, `at`

```javascript
const arr = [1, 2, 3, 4, 5];

arr.map(x => x * 2)          // [2, 4, 6, 8, 10]  — трансформация
arr.filter(x => x > 2)       // [3, 4, 5]         — фильтрация
arr.reduce((s, x) => s + x, 0) // 15              — свёртка
arr.find(x => x > 3)         // 4                 — первый подходящий
arr.some(x => x > 4)         // true
arr.every(x => x > 0)        // true
arr.flat(Infinity)            // рекурсивное разворачивание вложенности

// Разница forEach vs map
arr.forEach(x => console.log(x)); // undefined — нет return value
arr.map(x => x + 1);              // [2,3,4,5,6] — возвращает новый массив
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Обработка ошибок: try..catch..finally

`try` — код, который может выбросить ошибку. `catch(err)` — перехватывает любую синхронную ошибку. `finally` — выполняется **всегда**, даже после `return` или `throw`.

```javascript
try {
  JSON.parse('invalid'); // выбросит SyntaxError
} catch (err) {
  console.log(err instanceof SyntaxError); // true
  console.log(err.message);  // "Unexpected token..."
  console.log(err.name);     // "SyntaxError"
  // throw err; // можно пробросить дальше
} finally {
  console.log('Выполнится в любом случае');
}

// Создание собственной ошибки
throw new Error('Что-то пошло не так');
throw new TypeError('Ожидалась строка');

// try..catch не ловит асинхронные ошибки (setTimeout)
// Для async/await — ловит:
async function load() {
  try {
    const data = await fetch('/api');
  } catch (err) {
    console.error('Fetch failed:', err);
  }
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## use strict — зачем нужен?

`"use strict"` — директива, включающая строгий режим выполнения JS. Появилась в ES5. В ES6-модулях и классах включён автоматически.

**Что меняет строгий режим:**
- Запрещает необъявленные переменные (`x = 5` → ReferenceError)
- `this` в обычной функции — `undefined` (не `window`)
- Запрещает дублирование параметров: `function f(a, a)` → SyntaxError
- Запрещает `with`
- Запрещает `delete` на неудаляемых свойствах

```javascript
'use strict';

x = 10; // ReferenceError: x is not defined

function showThis() { console.log(this); }
showThis(); // undefined (не window)

// В ES6 модулях, классах — strict всегда включён автоматически
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое функция обратного вызова (callback)?

Callback — это функция, переданная как аргумент другой функции и вызываемая ею внутри по завершению какой-либо операции. Основной механизм асинхронности до появления Promise.

```javascript
function fetchData(url, onSuccess, onError) {
  // имитация асинхронного запроса
  setTimeout(() => {
    const data = { user: 'Alice' };
    onSuccess(data);   // вызываем callback по завершении
  }, 1000);
}

fetchData('/api/user',
  (data) => console.log(data),       // onSuccess callback
  (err)  => console.error(err)        // onError callback
);

// Встроенные callback: события, методы массива
[1, 2, 3].map(x => x * 2);       // (x => x * 2) — callback
[1, 2, 3].forEach(console.log);  // console.log — callback
```

**Проблема callback hell**: глубокое вложение callback-ов образует пирамидальное нечитаемое дерево. Решение — Promise / async/await.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
- [MDN: Callback function](https://developer.mozilla.org/ru/docs/Glossary/Callback_function)

---

## Что такое модули (import/export)?

Модули (ES Modules) — стандартный способ разбить код на отдельные файлы с публичным API. Каждый модуль имеет свою область видимости — переменные не попадают в глобальную область.

```javascript
// math.js — named exports
export const PI = 3.14159;
export function add(a, b) { return a + b; }

// app.js — named imports
import { PI, add } from './math.js';
import { add as sum } from './math.js'; // переименование

// config.js — default export (один на модуль)
export default { apiUrl: '/api' };

// app.js — default import (имя произвольное)
import config from './config.js';
import * as math from './math.js'; // импорт всего
```

**Особенности:**
- Динамический import: `const module = await import('./heavy.js')` (ленивая загрузка)
- ES Modules работают в strict mode автоматически
- В старых Node.js проектах используется CommonJS: `module.exports = {}` / `const x = require('./x')`

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
- [MDN: JavaScript modules](https://developer.mozilla.org/ru/docs/Web/JavaScript/Guide/Modules)

---

## Как очистить массив?

Несколько способов с разными последствиями:

```javascript
const arr = [1, 2, 3];

// 1. length = 0 — быстро, мутирует оригинальный массив
arr.length = 0;

// 2. splice — мутирует оригинальный массив
arr.splice(0);

// 3. Перезапись переменной — новый массив, оригинал (если есть другие ссылки) остаётся
let a = [1, 2, 3];
a = [];

// 4. pop в цикле — работает, но медленно
while (arr.length > 0) arr.pop();
```

**Рекомендация**: если не нужна мутация — `arr = []`. Если нужно очистить и сохранить ссылку для всех потребителей — `arr.length = 0` или `arr.splice(0)`.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
