# JavaScript — Middle

## Вопросы

- [Что такое область видимости (scope)?](#что-такое-область-видимости-scope)
- [Что такое цепочка областей видимости (scope chain)?](#что-такое-цепочка-областей-видимости-scope-chain)
- [Что такое замыкание (closure)?](#что-такое-замыкание-closure)
- [Как работает this в JavaScript?](#как-работает-this-в-javascript)
- [Стрелочные функции vs обычные: полное сравнение?](#стрелочные-функции-vs-обычные-полное-сравнение)
- [Ключевые нововведения ES6+?](#ключевые-нововведения-es6)
- [Что такое деструктуризация, spread и rest?](#что-такое-деструктуризация-spread-и-rest)

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
