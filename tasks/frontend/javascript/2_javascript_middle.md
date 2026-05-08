# JavaScript — Middle Tasks

## Задачи

- [Глубокое копирование объекта](#глубокое-копирование-объекта)
- [Порядок вывода в event loop](#порядок-вывода-в-event-loop)
- [Плоский объект из вложенного (flatten)](#плоский-объект-из-вложенного-flatten)
- [Получить значение по dot-path строке](#получить-значение-по-dot-path-строке)
- [Контекст this — что выведет консоль?](#контекст-this--что-выведет-консоль)
- [Полифил для Array.prototype.map](#полифил-для-arrayprototypemap)
- [Стоимость бронирования отеля](#стоимость-бронирования-отеля)
- [Event loop — quizzes: что будет выведено?](#event-loop--quizzes-что-будет-выведено)
- [Сгруппировать массив по полю (groupBy)](#сгруппировать-массив-по-полю-groupby)

---

## Глубокое копирование объекта

Реализуйте функцию `deepClone(obj)`, которая создаёт глубокую копию объекта **без** использования `structuredClone` или `JSON.parse/JSON.stringify`. Функция должна корректно обрабатывать вложенные объекты, массивы и примитивы.

```javascript
const original = { a: 1, nested: { b: [2, 3], c: { d: 4 } } };
const clone = deepClone(original);
clone.nested.b.push(99);
console.log(original.nested.b); // [2, 3] — оригинал не изменился
```

**Связанные вопросы:**

- [Глубокое vs поверхностное копирование объектов?](../../../interviews/frontend/javascript/2_javascript_middle.md#глубокое-vs-поверхностное-копирование-объектов)

<details>
<summary>Решение</summary>

```javascript
function deepClone(obj) {
  if (obj === null || typeof obj !== 'object') return obj;

  if (Array.isArray(obj)) {
    return obj.map(item => deepClone(item));
  }

  const clone = {};
  for (const key in obj) {
    if (Object.prototype.hasOwnProperty.call(obj, key)) {
      clone[key] = deepClone(obj[key]);
    }
  }
  return clone;
}

const original = { a: 1, nested: { b: [2, 3], c: { d: 4 } } };
const clone = deepClone(original);
clone.nested.b.push(99);
console.log(original.nested.b); // [2, 3]
```

</details>

---

## Порядок вывода в event loop

Определите порядок вывода для каждого из двух фрагментов кода без их запуска. Объясните почему.

**Фрагмент 1:**

```javascript
console.log("A");

setTimeout(() => console.log("B"), 0);

Promise.resolve()
  .then(() => console.log("C"))
  .then(() => console.log("D"));

console.log("E");
```

**Фрагмент 2:**

```javascript
Promise.resolve()
  .then(() => {
    console.log("1");
    setTimeout(() => console.log("2"), 0);
  });

setTimeout(() => {
  console.log("3");
  Promise.resolve().then(() => console.log("4"));
}, 0);
```

**Связанные вопросы:**

- [Как работает цикл событий (event loop)?](../../../interviews/frontend/javascript/2_javascript_middle.md#как-работает-цикл-событий-event-loop)

<details>
<summary>Решение</summary>

**Фрагмент 1:** `A → E → C → D → B`

- `A`, `E` — синхронный код, выполняется сразу
- `C`, `D` — микрозадачи (Promise), выполняются после всего синхронного кода, перед макрозадачами
- `B` — макрозадача (setTimeout), выполняется последней

**Фрагмент 2:** `1 → 3 → 4 → 2`

- `1` — микрозадача Promise, выполняется первой
- Внутри неё ставим `setTimeout("2")` в очередь макрозадач
- `3` — следующая макрозадача (setTimeout 0)
- `4` — микрозадача, поставленная внутри `3`, выполняется сразу после `3` (перед следующей макрозадачей)
- `2` — следующая макрозадача

</details>

---

## Плоский объект из вложенного (flatten)

Напишите функцию `flattenObject(obj, prefix)`, которая рекурсивно «сплющивает» вложенный объект в плоский, используя точку как разделитель для вложенных ключей.

```javascript
flattenObject({ a: 1, b: { c: 2, d: { e: 3 } } })
// { "a": 1, "b.c": 2, "b.d.e": 3 }

flattenObject({ x: { y: { z: 42 } }, n: 1 })
// { "x.y.z": 42, "n": 1 }
```

**Связанные вопросы:**

<!-- Связанных вопросов нет -->

<details>
<summary>Решение</summary>

```javascript
function flattenObject(obj, prefix = '') {
  const result = {};

  for (const key in obj) {
    if (!Object.prototype.hasOwnProperty.call(obj, key)) continue;

    const fullKey = prefix ? `${prefix}.${key}` : key;
    const value = obj[key];

    if (value !== null && typeof value === 'object' && !Array.isArray(value)) {
      Object.assign(result, flattenObject(value, fullKey));
    } else {
      result[fullKey] = value;
    }
  }

  return result;
}

console.log(flattenObject({ a: 1, b: { c: 2, d: { e: 3 } } }));
// { "a": 1, "b.c": 2, "b.d.e": 3 }
```

</details>

---

## Получить значение по dot-path строке

Напишите функцию `getByPath(obj, path)`, которая принимает объект и строку вида `"a.b.c"` и возвращает значение по этому пути. Если путь не существует — вернуть `undefined`.

```javascript
const obj = { user: { address: { city: "Berlin" } } };

getByPath(obj, "user.address.city");   // "Berlin"
getByPath(obj, "user.address.zip");    // undefined
getByPath(obj, "user.phone.number");   // undefined
```

**Связанные вопросы:**

<!-- Связанных вопросов нет -->

<details>
<summary>Решение</summary>

```javascript
function getByPath(obj, path) {
  return path.split('.').reduce((current, key) => {
    return current !== null && current !== undefined
      ? current[key]
      : undefined;
  }, obj);
}

// Альтернатива: рекурсивная
function getByPathRecursive(obj, path) {
  const [first, ...rest] = path.split('.');
  if (obj === null || obj === undefined) return undefined;
  if (rest.length === 0) return obj[first];
  return getByPathRecursive(obj[first], rest.join('.'));
}

const obj = { user: { address: { city: "Berlin" } } };
console.log(getByPath(obj, "user.address.city"));  // "Berlin"
console.log(getByPath(obj, "user.phone.number"));  // undefined
```

</details>

---

## Контекст this — что выведет консоль?

Определите, что будет выведено в каждом случае, не запуская код. Объясните почему.

**Кейс 1: потеря контекста**
```javascript
const obj = {
  name: 'David',
  getName() { console.log(`name is: ${this.name}`); },
};

const fn = obj.getName;
fn(); // ?
```

**Кейс 2: стрелочная vs обычная функция внутри метода**
```javascript
function foo() {
  return {
    x: 20,
    bar: function () { console.log(this.x); },
    baz: () => { console.log(this.x); },
  };
}

const obj = foo();
obj.bar(); // ?
obj.baz(); // ?

const obj2 = foo.call({ x: 30 });
obj2.bar();       // ?
obj2.baz();       // ?
obj2.bar.call({}); // ?
```

**Кейс 3: bind применяется только один раз**
```javascript
const fn = obj.bar.bind({ x: 100 }).bind({ x: 999 });
fn(); // ?
```

**Связанные вопросы:**

- [Как работает this в JavaScript?](../../../interviews/frontend/javascript/2_javascript_middle.md#как-работает-this-в-javascript)
- [Как работают call, apply и bind?](../../../interviews/frontend/javascript/2_javascript_middle.md#как-работают-call-apply-и-bind)

<details>
<summary>Решение</summary>

**Кейс 1:**
```javascript
fn(); // "name is: undefined"
// fn вызвана без контекста — this === window (или undefined в strict mode)
// Исправление: const fn = obj.getName.bind(obj);
```

**Кейс 2:**
```javascript
obj.bar(); // 20   — this === obj, this.x === 20
obj.baz(); // undefined — стрелка захватила this из foo(), не из объекта

const obj2 = foo.call({ x: 30 });
obj2.bar();       // 20  — this === obj2 при вызове как метод, obj2.x === 20
obj2.baz();       // 30  — стрелка захватила { x: 30 } из foo.call({ x: 30 })
obj2.bar.call({}); // undefined — call изменил this у обычной функции
```

**Кейс 3:**
```javascript
fn(); // 100 — первый bind «замораживает» this, повторный bind игнорируется
```

</details>

---

## Полифил для Array.prototype.map

**Задача 1:** Напишите функцию `customMap(array, callback)`, воспроизводящую поведение `Array.prototype.map`.

**Задача 2:** Добавьте метод `customMap` на `Array.prototype` так, чтобы его можно было вызывать как `array.customMap(fn)`.

```javascript
const array = [{ id: 1 }, { id: 2 }];
const format = (el, index) => `${el.id}|${index}`;

customMap(array, format);    // ["1|0", "2|1"]
array.customMap(format);     // ["1|0", "2|1"]
```

**Связанные вопросы:**

- [Что такое высшие функции (higher-order functions)?](../../../interviews/frontend/javascript/1_javascript_junior.md#что-такое-высшие-функции-higher-order-functions)

<details>
<summary>Решение</summary>

```javascript
// Задача 1: standalone функция
function customMap(array, callback) {
  const result = [];
  for (let i = 0; i < array.length; i++) {
    result.push(callback(array[i], i, array));
  }
  return result;
}

// Задача 2: полифил на прототипе
Array.prototype.customMap = function(callback) {
  const result = [];
  for (let i = 0; i < this.length; i++) {
    result.push(callback(this[i], i, this));
  }
  return result;
};

const array = [{ id: 1 }, { id: 2 }];
const format = (el, index) => `${el.id}|${index}`;
console.log(customMap(array, format));   // ["1|0", "2|1"]
console.log(array.customMap(format));    // ["1|0", "2|1"]
```

</details>

---

## Стоимость бронирования отеля

Реализуйте функцию `bookingCalculate(nights, startDate?)` для расчёта стоимости проживания в отеле. Если дата не передана, отсчёт ведётся от текущего дня.

- Будние дни (Пн–Пт): **1500 руб/ночь**
- Выходные дни (Сб–Вс): **2200 руб/ночь**

```javascript
bookingCalculate(7)                         // зависит от текущей даты
bookingCalculate(3, new Date('2023-11-10')) // 5900 (пт 1500 + сб 2200 + вс 2200)
```

**Связанные вопросы:**

<!-- Связанных вопросов нет -->

<details>
<summary>Решение</summary>

```javascript
const prices = { weekday: 1500, holiday: 2200 };

const isWeekday = (date) => {
  const day = new Date(date).getDay();
  return day > 0 && day < 6; // 1=Пн ... 5=Пт
};

const addDays = (date, days) => {
  const result = new Date(date);
  result.setDate(result.getDate() + days);
  return result;
};

function bookingCalculate(nights, startDate = new Date()) {
  let weekdays = 0;
  let weekends = 0;

  for (let i = 0; i < nights; i++) {
    const currentDay = addDays(startDate, i);
    if (isWeekday(currentDay)) weekdays++;
    else weekends++;
  }

  return weekdays * prices.weekday + weekends * prices.holiday;
}

console.log(bookingCalculate(3, new Date('2023-11-10'))); // 5900
```

</details>

---

## Event loop — quizzes: что будет выведено?

Серия задач на понимание очерёдности выполнения синхронного кода, микрозадач (Promise) и макрозадач (setTimeout).

**Квиз 1:**
```javascript
setTimeout(() => console.log('setTimeout 1'), 0);

new Promise((resolve) => {
  console.log('Promise 1');
  resolve();
  console.log('Promise 2');
}).then(() => console.log('Promise 3'));

Promise.resolve().then(() => setTimeout(() => console.log('setTimeout 2'), 0));
Promise.resolve().then(() => console.log('Promise 4'));
setTimeout(() => console.log('setTimeout 3'), 0);
console.log('final');
```

**Квиз 2:**
```javascript
Promise.reject("a")
  .catch(p => p + "b")
  .catch(p => p + "c")
  .then(p => p + "d")
  .then(p => console.log(p));
```

**Квиз 3:**
```javascript
const p = new Promise((resolve) => { resolve(''); console.log('B') });

console.log('A');
p.then(() => { p.then(() => console.log('C')); console.log('D'); });
setTimeout(() => console.log('E'), 0);
p.then(() => console.log('F'));
```

**Связанные вопросы:**

- [Как работает цикл событий (event loop)?](../../../interviews/frontend/javascript/2_javascript_middle.md#как-работает-цикл-событий-event-loop)

<details>
<summary>Решение</summary>

**Квиз 1:** `Promise 1 → Promise 2 → final → Promise 3 → Promise 4 → setTimeout 1 → setTimeout 3 → setTimeout 2`

- Конструктор Promise выполняется синхронно
- Микрозадачи (Promise 3, Promise 4) — перед макрозадачами
- setTimeout 2 добавляется в очередь макрозадач позже setTimeout 1 и 3

**Квиз 2:** `"abd"`

- `reject("a")` — ошибка, первый `.catch` поглощает её → возвращает `"ab"`
- Второй `.catch` пропускается (ошибки уже нет)
- `.then` → `"abd"`

**Квиз 3:** `B → A → D → F → C → E`

- `B` — синхронно в конструкторе Promise
- `A` — синхронно
- `D`, `F` — микрозадачи первого уровня
- `C` — микрозадача, добавленная внутри `D`, выполняется после `F`
- `E` — макрозадача

</details>

---

## Сгруппировать массив по полю (groupBy)

**Задача 1:** Напишите функцию `groupByType(arr)`, которая принимает массив объектов с полем `type` и возвращает объект, где ключи — значения `type`, значения — массивы элементов с таким `type`.

**Задача 2:** Обобщите до `groupBy(arr, key)` — группировка по любому полю.

```javascript
const arr = [
  { type: "banana", weight: 32 },
  { type: "orange", weight: 12 },
  { type: "banana", weight: 10 },
  { type: "orange", weight: 5  },
];

groupByType(arr);
// { banana: [{type:"banana",weight:32},{type:"banana",weight:10}], orange: [...] }

groupBy(arr, 'type');
// тот же результат
```

**Связанные вопросы:**

<!-- Связанных вопросов нет -->

<details>
<summary>Решение</summary>

```javascript
// Задача 1: groupByType
const groupByType = (arr) => {
  return arr.reduce((acc, item) => {
    const key = item.type;
    if (!acc[key]) acc[key] = [];
    acc[key].push(item);
    return acc;
  }, {});
};

// Задача 2: обобщённая groupBy
const groupBy = (arr, key) => {
  return arr.reduce((acc, item) => {
    const groupKey = item[key];
    if (!acc[groupKey]) acc[groupKey] = [];
    acc[groupKey].push(item);
    return acc;
  }, {});
};

console.log(groupBy(arr, 'type'));
```

</details>
