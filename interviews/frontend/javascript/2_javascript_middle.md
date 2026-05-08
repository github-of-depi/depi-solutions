# JavaScript — Middle

## Вопросы

- [Что такое область видимости (scope)?](#что-такое-область-видимости-scope)
- [Что такое цепочка областей видимости (scope chain)?](#что-такое-цепочка-областей-видимости-scope-chain)
- [Что такое замыкание (closure)?](#что-такое-замыкание-closure)
- [Как работает this в JavaScript?](#как-работает-this-в-javascript)
- [Стрелочные функции vs обычные: полное сравнение?](#стрелочные-функции-vs-обычные-полное-сравнение)
- [Ключевые нововведения ES6+?](#ключевые-нововведения-es6)
- [Что такое деструктуризация, spread и rest?](#что-такое-деструктуризация-spread-и-rest)
- [Как работают call, apply и bind?](#как-работают-call-apply-и-bind)
- [Глубокое vs поверхностное копирование объектов?](#глубокое-vs-поверхностное-копирование-объектов)
- [Как работает цикл событий (event loop)?](#как-работает-цикл-событий-event-loop)
- [Что такое debounce и throttle?](#что-такое-debounce-и-throttle)

---

## Что такое область видимости (scope)?

Scope — контекст, в котором переменные доступны. Типы:

- **Глобальная** — переменные вне функций (`window` в браузере)
- **Функциональная** — переменные внутри функции
- **Блочная** — переменные `let`/`const` внутри `{}`

```javascript
let global = "global";

function outer() {
  let funcScope = "function";

  if (true) {
    let blockScope = "block"; // только здесь
    console.log(global, funcScope, blockScope); // все доступны
  }

  console.log(blockScope); // ReferenceError
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое цепочка областей видимости (scope chain)?

При поиске переменной JS смотрит сначала в текущем scope, затем во внешнем, и так до глобального. Это и есть scope chain (лексическое окружение). Определяется в момент **определения** функции, не вызова.

```javascript
let x = "global";

function outer() {
  let x = "outer";
  function inner() {
    // x не найден в inner → ищем в outer → находим "outer"
    console.log(x); // "outer"
  }
  inner();
}
outer();
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое замыкание (closure)?

Замыкание — функция, которая **запоминает** лексическое окружение места своего создания, даже после выхода из него.

```javascript
function makeCounter(start = 0) {
  let count = start; // переменная в замыкании

  return {
    increment() { return ++count; },
    decrement() { return --count; },
    value()     { return count; },
  };
}

const counter = makeCounter(10);
counter.increment(); // 11
counter.increment(); // 12
counter.value();     // 12
// count недоступен снаружи — инкапсуляция через замыкание
```

**Практические применения**: модульный паттерн, мемоизация, фабричные функции, частичное применение.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работает this в JavaScript?

`this` — контекст вызова. Значение определяется **в момент вызова**, а не определения (кроме стрелочных функций).

```javascript
const obj = {
  name: "Alice",
  greet() { console.log(this.name); }, // this = obj
  greetArrow: () => console.log(this.name), // this = внешний (window)
};

obj.greet();       // "Alice"
obj.greetArrow();  // undefined

// Явное задание через call/apply/bind:
function greet(greeting) { return `${greeting}, ${this.name}`; }
greet.call({ name: "Bob" }, "Hi");   // "Hi, Bob"
greet.apply({ name: "Bob" }, ["Hi"]); // "Hi, Bob"
const boundGreet = greet.bind({ name: "Bob" });
boundGreet("Hey"); // "Hey, Bob"
```

**Правила this** (по приоритету): `new` > `bind/call/apply` > метод объекта > по умолчанию (window/undefined в strict).

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Стрелочные функции vs обычные: полное сравнение?

| Характеристика | Обычная | Стрелочная |
|---|---|---|
| `this` | Динамический | Лексический (от внешнего) |
| `arguments` | Есть | Нет (используй `...args`) |
| Конструктор (`new`) | Да | Нет |
| `prototype` | Есть | Нет |
| Синтаксис | `function(){}` | `() => {}` |

```javascript
// Правило: стрелочные там, где нужен внешний this
class Timer {
  start() {
    setInterval(() => {
      this.tick(); // this = экземпляр Timer (стрелочная)
    }, 1000);
  }
}

// Обычные там, где нужен динамический this или new
function Person(name) {
  this.name = name;
}
const alice = new Person("Alice"); // работает
const arrow = (name) => { this.name = name; };
new arrow("Bob"); // TypeError: arrow is not a constructor
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Ключевые нововведения ES6+?

**ES2015 (ES6):**
- `let`/`const` — блочная область видимости
- Стрелочные функции
- Классы
- Деструктуризация
- Spread/Rest (`...`)
- Шаблонные строки `` `${expr}` ``
- Промисы (`Promise`)
- Модули (`import`/`export`)
- `Map`, `Set`, `WeakMap`, `WeakSet`
- `Symbol`
- Генераторы (`function*`)
- `for...of`
- Default/named exports

**ES2017+:**
- `async`/`await`
- `Object.entries()`, `Object.values()`
- Optional chaining `?.`
- Nullish coalescing `??`
- `structuredClone()`
- `Array.at()`, `Object.hasOwn()`

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое деструктуризация, spread и rest?

**Деструктуризация** — извлечение значений из объекта/массива:

```javascript
const { name, age = 0, address: { city } = {} } = user;
const [first, , third, ...rest] = [1, 2, 3, 4, 5];
```

**Spread** — разворачивание итерируемых:

```javascript
const merged = { ...defaults, ...overrides }; // merge объектов
const copy = [...array, newItem];             // клон + добавление
Math.max(...[1, 2, 3]);                       // spread в аргументы
```

**Rest** — сбор оставшихся аргументов:

```javascript
function log(level, ...messages) {
  console[level](messages.join(" "));
}
log("info", "hello", "world"); // messages = ["hello", "world"]
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работают call, apply и bind?

Все три метода позволяют явно задать `this` для функции, но применяются по-разному.

- **`call(thisArg, arg1, arg2, ...)`** — вызывает функцию немедленно, аргументы перечисляются через запятую.
- **`apply(thisArg, [args])`** — вызывает функцию немедленно, аргументы передаются массивом.
- **`bind(thisArg, arg1, ...)`** — возвращает новую функцию с привязанным `this` (и опционально первыми аргументами), вызов отложен.

```javascript
function greet(greeting, punctuation) {
  return `${greeting}, ${this.name}${punctuation}`;
}

const user = { name: "Alice" };

greet.call(user, "Привет", "!");    // "Привет, Alice!"
greet.apply(user, ["Привет", "!"]); // "Привет, Alice!"

const boundGreet = greet.bind(user, "Привет");
boundGreet("?"); // "Привет, Alice?"

// Практический кейс: заимствование метода
const arrayLike = { 0: "a", 1: "b", length: 2 };
Array.prototype.slice.call(arrayLike); // ["a", "b"]
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: Function.prototype.bind](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)

---

## Глубокое vs поверхностное копирование объектов?

**Поверхностная копия (shallow copy)** — копируются только свойства первого уровня. Вложенные объекты копируются по ссылке, поэтому изменение вложенного объекта в копии влияет на оригинал.

**Глубокая копия (deep copy)** — рекурсивно копируются все уровни вложенности.

```javascript
const original = { a: 1, nested: { b: 2 } };

// Поверхностная — через Object.assign или spread
const shallow = { ...original };
shallow.nested.b = 99; // меняет и original.nested.b!

// Глубокая — через structuredClone (современный способ)
const deep = structuredClone(original);
deep.nested.b = 99; // original.nested.b не изменится

// Альтернатива: JSON.parse(JSON.stringify(...))
// Не работает с: undefined, функциями, Date, Map, Set, RegExp, circular refs
const jsonCopy = JSON.parse(JSON.stringify(original));
```

**Когда что использовать:** shallow copy дешевле, подходит для плоских объектов и иммутабельных обновлений состояния (Redux-паттерн). `structuredClone` — для глубокого копирования с поддержкой большинства типов данных.

**Связанные задачи:**

- [Глубокое копирование объекта](../../../tasks/frontend/javascript/2_javascript_middle.md#глубокое-копирование-объекта)

**Материалы для изучения:**

- [MDN: structuredClone](https://developer.mozilla.org/ru/docs/Web/API/structuredClone)

---

## Как работает цикл событий (event loop)?

JavaScript — однопоточный язык. Event loop — механизм, позволяющий выполнять асинхронный код без блокировки потока.

**Очереди задач:**
- **Call stack** — синхронный код, выполняется немедленно.
- **Microtask queue** — Promise-колбэки (`.then`, `.catch`), `queueMicrotask`, `MutationObserver`. Опустошается **полностью** после каждого шага event loop, перед следующей макрозадачей.
- **Macrotask queue (task queue)** — `setTimeout`, `setInterval`, события ввода/вывода. Выбирается по одной задаче за итерацию.

```javascript
console.log("1"); // sync

setTimeout(() => console.log("2"), 0); // macrotask

Promise.resolve().then(() => console.log("3")); // microtask

console.log("4"); // sync

// Вывод: 1 → 4 → 3 → 2
```

**Порядок:** сначала весь синхронный код → все микрозадачи → одна макрозадача → снова все микрозадачи → ...

**Связанные задачи:**

- [Порядок вывода в event loop](../../../tasks/frontend/javascript/2_javascript_middle.md#порядок-вывода-в-event-loop)

**Материалы для изучения:**

- [MDN: Event loop](https://developer.mozilla.org/ru/docs/Web/JavaScript/Event_loop)

---

## Что такое debounce и throttle?

Оба паттерна ограничивают частоту вызова функции, но по-разному.

**Debounce** — откладывает вызов до тех пор, пока между событиями не пройдёт заданная пауза. Подходит для поиска по вводу, авторезмера окна.

**Throttle** — гарантирует вызов не чаще, чем раз в N мс. Подходит для обработки скролла, mousemove.

```javascript
// Debounce
function debounce(fn, delay) {
  let timer;
  return function(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

const onSearch = debounce((query) => fetchResults(query), 300);

// Throttle
function throttle(fn, limit) {
  let lastCall = 0;
  return function(...args) {
    const now = Date.now();
    if (now - lastCall >= limit) {
      lastCall = now;
      return fn.apply(this, args);
    }
  };
}

const onScroll = throttle(() => updateHeader(), 100);
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: setTimeout](https://developer.mozilla.org/ru/docs/Web/API/setTimeout)
