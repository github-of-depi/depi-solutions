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
- [В чём разница между hasOwnProperty и оператором in?](#в-чём-разница-между-hasownproperty-и-оператором-in)
- [Что возвращает ['1','7','11'].map(parseInt) и почему?](#что-возвращает-1711mapparseint-и-почему)
- [Что такое IIFE и для чего он нужен?](#что-такое-iife-и-для-чего-он-нужен)
- [Разница между undefined и undeclared?](#разница-между-undefined-и-undeclared)
- [Отличие forEach от map?](#отличие-foreach-от-map)
- [Явное и неявное преобразование типов?](#явное-и-неявное-преобразование-типов)
- [ООП: принципы и реализация в JavaScript?](#ооп-принципы-и-реализация-в-javascript)
- [Классы в JS: синтаксис, наследование, static и приватные поля?](#классы-в-js-синтаксис-наследование-static-и-приватные-поля)
- [Геттеры и сеттеры?](#геттеры-и-сеттеры)
- [Что такое дескриптор свойства объекта?](#что-такое-дескриптор-свойства-объекта)
- [Как запретить изменение объекта или его свойства?](#как-запретить-изменение-объекта-или-его-свойства)
- [Пользовательские ошибки и расширение Error?](#пользовательские-ошибки-и-расширение-error)
- [Object.getOwnPropertyNames vs Object.keys?](#objectgetownpropertynames-vs-objectkeys)
- [Чейнинг методов (method chaining)?](#чейнинг-методов-method-chaining)
- [callback vs Promise vs async/await — сравнение?](#callback-vs-promise-vs-asyncawait--сравнение)
- [Дата и время (Date API)?](#дата-и-время-date-api)
- [Глобальные объекты и разница между host и нативными объектами?](#глобальные-объекты-и-разница-между-host-и-нативными-объектами)

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

---

## В чём разница между hasOwnProperty и оператором in?

Оба способа проверяют наличие свойства в объекте, но с разным охватом:

- **`in`** — проверяет и собственные свойства, и **унаследованные** через цепочку прототипов.
- **`hasOwnProperty`** — проверяет **только собственные** свойства объекта, унаследованные игнорирует.

```javascript
class Animal {
  constructor(name) { this.name = name; }
  sound() { console.log("..."); }
}

class Dog extends Animal {
  constructor(name) { super(name); this.breed = "Lab"; }
  bark() { console.log("Woof!"); }
}

const dog = new Dog("Buddy");

dog.hasOwnProperty('name');  // true  — name задано в конструкторе
dog.hasOwnProperty('breed'); // true  — breed задано в конструкторе Dog
dog.hasOwnProperty('sound'); // false — sound на прототипе Animal
dog.hasOwnProperty('bark');  // false — bark на прототипе Dog

'name'  in dog; // true
'sound' in dog; // true  — унаследовано, но in его видит!
'bark'  in dog; // true
```

**Современная альтернатива:** `Object.hasOwn(obj, key)` — стандартный способ без необходимости вызывать метод через прототип.

```javascript
// Безопаснее, чем hasOwnProperty (работает даже если метод переопределён)
Object.hasOwn(dog, 'name'); // true
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: Object.hasOwn](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwn)

---

## Что возвращает ['1','7','11'].map(parseInt) и почему?

Результат: `[1, NaN, 3]` — и это классическая ловушка JavaScript.

Метод `Array.prototype.map` передаёт в колбэк **три аргумента**: `(элемент, индекс, массив)`. Функция `parseInt(string, radix)` принимает два аргумента: строку и **основание системы счисления**.

```javascript
['1', '7', '11'].map(parseInt)
// Раскрывается как:
// parseInt('1',  0)  → 1   (radix=0 → используется 10)
// parseInt('7',  1)  → NaN (единичная система счисления не имеет цифры 7)
// parseInt('11', 2)  → 3   (двоичное 11 = десятичное 3)
```

Чтобы корректно преобразовать строки в числа:

```javascript
['1', '7', '11'].map(Number);        // [1, 7, 11]
['1', '7', '11'].map(n => parseInt(n, 10)); // [1, 7, 11]
```

**Вывод:** никогда не передавай функции с необязательными параметрами напрямую в `map/filter/forEach` без обёртки — лишние аргументы могут изменить поведение.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: parseInt](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/parseInt)

---

## Что такое IIFE и для чего он нужен?

**IIFE** (Immediately Invoked Function Expression) — функция, которая определяется и немедленно вызывается. Создаёт изолированную область видимости.

**Для чего:**
- Изоляция переменных — не загрязнять глобальную область
- Module pattern — создание приватного состояния через замыкание
- Инициализация без именованной функции

```javascript
// Классический синтаксис
(function() {
  const privateVar = 'secret';
  console.log('IIFE выполнилась');
})();

// Со стрелкой
(() => {
  const local = 42;
})();

// Module pattern: публичный API через замыкание
const counter = (() => {
  let count = 0; // приватное
  return {
    inc: () => ++count,
    get: () => count,
  };
})();

counter.inc(); // 1
counter.get(); // 1
counter.count; // undefined
```

С ES6-модулями актуальность IIFE снизилась, но паттерн встречается в старых кодебазах и сборщиках.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Разница между undefined и undeclared?

**`undefined`** — переменная объявлена, но ей не присвоено значение. Юридически существует.

**`undeclared`** — переменная вообще не объявлялась в доступной области. Обращение к ней бросает `ReferenceError`.

```javascript
let x;           // undefined — объявлена, но без значения
console.log(x);  // undefined

console.log(y);  // ReferenceError: y is not defined

// typeof безопасен даже для undeclared
typeof y;        // 'undefined' — не выбрасывает ошибку

// Пример: проверка опционального цепочного полифила
if (typeof somePolyfill === 'undefined') {
  // безопасно даже если переменная не объявлена
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Отличие forEach от map?

| | `forEach` | `map` |
|---|---|---|
| Возвращает | `undefined` | новый массив |
| Оригинал | не меняет | не меняет |
| Цепочка | невозможна | возможна |
| `break`/`return` | не прерывает цикл | не прерывает цикл |

```javascript
const arr = [1, 2, 3];

// forEach — побочный эффект, без результата
arr.forEach(x => console.log(x)); // undefined

// map — трансформация, возвращает новый массив
const doubled = arr.map(x => x * 2); // [2, 4, 6]

// Частая ошибка:
arr.forEach(x => x * 2); // undefined, результат теряется!
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Явное и неявное преобразование типов?

**Явное** (принужденное) — разработчик явно указывает тип: `Number(x)`, `String(x)`, `Boolean(x)`, `parseInt(x)`, `parseFloat(x)`, оператор `+` перед значением (`+str`).

**Неявное** — происходит автоматически в контексте, например при операции `+`, сравнении `==`, или в булевом контексте.

```javascript
// Явное
Number('42')   // 42
String(42)     // '42'
Boolean(0)     // false
parseInt('3px') // 3

// Неявное
'5' + 3        // '53'  — 3 стал строкой (строковая конкатенация)
'5' - 3        // 2    — '5' стала числом (-)
null + 1       // 1    — null → 0
true + true    // 2    — true → 1
[] + []        // ''   — два пустых массива в строку

// == вызывает неявное преобразование, === — нет
'1' == 1   // true
'1' === 1  // false
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## ООП: принципы и реализация в JavaScript?

Объектно-ориентированное программирование базируется на четырёх принципах:

- **Инкапсуляция** — скрытие внутреннего состояния, доступ только через методы
- **Наследование** — дочерний класс получает свойства родительского
- **Полиморфизм** — один интерфейс, разное поведение
- **Абстракция** — работа с интерфейсом, а не деталями

Особенности ООП в JS: вместо классического классового наследования — **прототипное**. `class` — это синтаксический сахар над функцией-конструктором + прототипом.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Классы в JS: синтаксис, наследование, static и приватные поля?

```javascript
class Animal {
  #name; // приватное поле (ES2022)
  static count = 0; // статическое свойство

  constructor(name) {
    this.#name = name;
    Animal.count++;
  }

  get name() { return this.#name; }          // геттер
  set name(val) { this.#name = val; }        // сеттер

  speak() { return `${this.#name} says...`; } // метод экземпляра

  static create(name) { return new Animal(name); } // статик метод
}

class Dog extends Animal {
  #breed;

  constructor(name, breed) {
    super(name);   // обязательно до обращения к this
    this.#breed = breed;
  }

  speak() { return `${this.name} barks!`; }  // переопределение
}

const d = new Dog('Rex', 'Lab');
d.speak();         // 'Rex barks!'
d.name;            // 'Rex' (через геттер)
d instanceof Dog;  // true
d instanceof Animal; // true
Animal.count;      // 1
```

`class` — синтаксический сахар над прототипным наследованием, однако вносит ряд удобств: `#private`, `static`, `get`/`set`, необходимость `super()` в конструкторе.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: Classes](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Classes)

---

## Геттеры и сеттеры?

`get` и `set` — специальные методы, позволяющие работать с свойством как с обычным, но вызывающие функцию при обращении.

```javascript
const user = {
  _name: '',
  get name() { return this._name.trim(); },
  set name(val) {
    if (typeof val !== 'string') throw new TypeError('Expected string');
    this._name = val;
  }
};

user.name = '  Alice  ';
console.log(user.name); // 'Alice' (через trim)

// В классе... — см.выше пример Animal

// Через Object.defineProperty
Object.defineProperty(obj, 'fullName', {
  get() { return `${this.first} ${this.last}`; },
  enumerable: true,
  configurable: true,
});
```

Применяются для: валидации, вычисляемых свойств, логгирования, ленивого вычисления (lazy getter).

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое дескриптор свойства объекта?

Kaждое свойство объекта в JS содержит скрытый **дескриптор** с четырьмя флагами:

- `value` — значение
- `writable` — можно ли перезаписать
- `enumerable` — появляется ли в `for..in`, `Object.keys()`
- `configurable` — можно ли удалить или изменить дескриптор

```javascript
const obj = {};

Object.defineProperty(obj, 'PI', {
  value: 3.14159,
  writable: false,     // нельзя перезаписать
  enumerable: false,   // не видно в Object.keys()
  configurable: false, // нельзя удалить/изменить дескриптор
});

obj.PI = 3; // тихо проваливается (strict mode: TypeError)
console.log(obj.PI);      // 3.14159
Object.keys(obj);         // [] — PI не перечисляемое

// Прочитать дескриптор
Object.getOwnPropertyDescriptor(obj, 'PI');
// { value: 3.14159, writable: false, enumerable: false, configurable: false }
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: Object.defineProperty](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)

---

## Как запретить изменение объекта или его свойства?

Три метода с нарастающей жёсткостью ограничений:

| Метод | Новые св-ва | Изменение | Удаление |
|---|---|---|---|
| `preventExtensions` | ✘ | ✔ | ✔ |
| `seal` | ✘ | ✔ | ✘ |
| `freeze` | ✘ | ✘ | ✘ |

```javascript
const obj = { x: 1, y: { z: 2 } };

// Object.freeze — нельзя добавлять/менять/удалять
Object.freeze(obj);
obj.x = 99;   // тихо проваливается (strict: TypeError)
obj.x;        // 1
obj.y.z = 99; // работает! freeze поверхностный

// Дееп-фриз — рекурсивный Object.freeze
function deepFreeze(obj) {
  Object.getOwnPropertyNames(obj).forEach(key => {
    if (obj[key] !== null && typeof obj[key] === 'object') deepFreeze(obj[key]);
  });
  return Object.freeze(obj);
}

// Object.seal — можно менять существующие, но не добавлять/удалять
const sealed = Object.seal({ a: 1 });
sealed.a = 2;  // ок
sealed.b = 3;  // тихо проваливается
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Пользовательские ошибки и расширение Error?

```javascript
// Встроенные типы ошибок
Error, TypeError, RangeError, ReferenceError,
SyntaxError, URIError, EvalError

// Создание пользовательской ошибки через расширение Error
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = 'ValidationError'; // обязательно!
    this.field = field;
  }
}

class NetworkError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.name = 'NetworkError';
    this.statusCode = statusCode;
  }
}

// Использование
try {
  throw new ValidationError('Поле обязательно', 'email');
} catch (err) {
  if (err instanceof ValidationError) {
    console.log(err.field); // 'email'
  } else {
    throw err; // пробросить неизвестные ошибки
  }
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Object.getOwnPropertyNames vs Object.keys?

| | `Object.keys` | `Object.getOwnPropertyNames` |
|---|---|---|
| Неперечисляемые св-ва | ✘ | ✔ |
| Символьные ключи | ✘ | ✘ |
| Прототипные св-ва | ✘ | ✘ |

```javascript
const obj = { a: 1 };

Object.defineProperty(obj, 'hidden', {
  value: 2,
  enumerable: false, // неперечисляемое
});

Object.keys(obj)                  // ['a']          — только перечисляемые
Object.getOwnPropertyNames(obj)   // ['a', 'hidden'] — все собственные

// Для символьных ключей:
Object.getOwnPropertySymbols(obj) // []

// Все ключи включая символьные:
Reflect.ownKeys(obj)              // ['a', 'hidden']
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Чейнинг методов (method chaining)?

Чейнинг — паттерн, в котором каждый метод возвращает `this`, позволяя вызывать следующий метод на том же объекте.

```javascript
class QueryBuilder {
  #query = '';

  select(table) { this.#query += `SELECT * FROM ${table}`; return this; }
  where(cond)   { this.#query += ` WHERE ${cond}`;          return this; }
  limit(n)      { this.#query += ` LIMIT ${n}`;             return this; }
  build()       { return this.#query; }
}

const sql = new QueryBuilder()
  .select('users')
  .where('age > 18')
  .limit(10)
  .build();
// 'SELECT * FROM users WHERE age > 18 LIMIT 10'

// Нативные примеры: методы массива, Promise API
[1,2,3].filter(x => x > 1).map(x => x * 2).join(',');
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## callback vs Promise vs async/await — сравнение?

| | Callback | Promise | async/await |
|---|---|---|---|
| Читаемость | низкая (callback hell) | средняя | высокая |
| Ошибки | вручную | `.catch()` | `try/catch` |
| Последовательность | вложенность | `.then()` цепочка | синхронный стиль |
| Параллельность | сложно | `Promise.all` | `Promise.all` + `await` |

```javascript
// Callback hell
getUser(id, (user) =>
  getPosts(user.id, (posts) =>
    getComments(posts[0].id, (comments) => {
      // треугольник смерти
    })
  )
);

// Promises
getUser(id)
  .then(user => getPosts(user.id))
  .then(posts => getComments(posts[0].id))
  .catch(err => console.error(err));

// async/await
async function load(id) {
  try {
    const user = await getUser(id);
    const posts = await getPosts(user.id);
    const comments = await getComments(posts[0].id);
  } catch (err) {
    console.error(err);
  }
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Дата и время (Date API)?

```javascript
const now = new Date();

now.getFullYear() // 2026
now.getMonth()    // 0-11 (ноябрь = 10!)
now.getDate()     // 1-31
now.getDay()      // 0=Вс, 1=Пн, ..., 6=Сб
now.getTime()     // Unix timestamp в мс
now.toISOString() // '2026-05-11T10:00:00.000Z'
now.toLocaleDateString('ru-RU') // 'дд.мм.гггг'

// Создание
const d = new Date(2026, 4, 11); // месяц = 4 = май!
const d2 = new Date('2026-05-11');
new Date(timestamp);

// Разница между датами
const diff = Math.abs(d2 - d) / (1000 * 60 * 60 * 24); // дней

// Текущий timestamp
Date.now() // без создания объекта, быстрее
+new Date() // то же самое

// Современная альтернатива: Intl.DateTimeFormat, date-fns, dayjs
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Глобальные объекты и разница между host и нативными объектами?

**Нативные (native) объекты** — часть стандарта ECMAScript, доступны в любом окружении:
`Object`, `Array`, `Function`, `String`, `Number`, `Boolean`, `Date`, `Math`, `JSON`, `Promise`, `Map`, `Set`, `RegExp`.

**Host-объекты** — предоставляются средой выполнения. В браузере:
`window`, `document`, `navigator`, `location`, `XMLHttpRequest`, `fetch`, `localStorage`, `setTimeout`, `HTMLElement`, `Event`, `Worker`.
В Node.js: `process`, `require`, `Buffer`, `__dirname`.

**Глобальные объекты:**
- `globalThis` — универсальная ссылка на глобальный объект (браузер = `window`, Node = `global`)
- `Math`, `JSON`, `Intl` — статические, нельзя создать `new Math()`

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
