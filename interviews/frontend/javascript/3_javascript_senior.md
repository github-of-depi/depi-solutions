# JavaScript — Senior

## Вопросы

- [Как работают Promise?](#как-работают-promise)
- [Async/Await: как работает под капотом?](#asyncawait-как-работает-под-капотом)
- [Чистые функции и функциональное программирование?](#чистые-функции-и-функциональное-программирование)
- [Каррирование и частичное применение?](#каррирование-и-частичное-применение)
- [Set, Map, WeakMap, WeakSet — когда использовать?](#set-map-weakmap-weakset--когда-использовать)
- [Прототипная цепочка в JavaScript?](#прототипная-цепочка-в-javascript)
- [Что такое Big O нотация?](#что-такое-big-o-нотация)
- [Генераторы и итераторы?](#генераторы-и-итераторы)
- [Чем отличаются Promise.all, allSettled, race, any?](#чем-отличаются-promiseall-allsettled-race-any)
- [Что такое мемоизация и для чего она нужна?](#что-такое-мемоизация-и-для-чего-она-нужна)
- [F.prototype и прототипное наследование через функции-конструкторы?](#fprototype-и-прототипное-наследование-через-функции-конструкторы)
- [Встроенные прототипы и их расширение?](#встроенные-прототипы-и-их-расширение)
- [Примеси (mixins) в JavaScript?](#примеси-mixins-в-javascript)
- [Промиссификация (promisify)?](#промиссификация-promisify)
- [Observable vs Promise: реактивное программирование?](#observable-vs-promise-реактивное-программирование)
- [Композиция вс наследование (composition vs inheritance)?](#композиция-вс-наследование-composition-vs-inheritance)

---

## Как работают Promise?

Promise — объект, представляющий результат асинхронной операции. Состояния: **pending** → **fulfilled** или **rejected**.

```javascript
const promise = new Promise((resolve, reject) => {
  setTimeout(() => resolve("данные"), 1000);
});

promise
  .then(data => console.log(data))   // "данные"
  .catch(err => console.error(err))
  .finally(() => console.log("done")); // всегда

// Параллельное выполнение:
const [users, posts] = await Promise.all([fetchUsers(), fetchPosts()]);

// Первый успешный:
const result = await Promise.any([slow(), fast()]);

// Все результаты (включая ошибки):
const results = await Promise.allSettled([p1, p2, p3]);
```

Promise помещаются в **microtask queue** — выполняются до следующего макротаска, после текущего call stack.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Async/Await: как работает под капотом?

`async/await` — синтаксический сахар над Promise. Движок преобразует `async` функцию в генератор с автоматическим управлением.

```javascript
async function fetchUser(id) {
  try {
    const response = await fetch(`/api/users/${id}`); // await = .then()
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return await response.json();
  } catch (err) {
    console.error("Ошибка:", err);
    throw err; // пробросить выше
  }
}

// Параллельно (правильно):
const [user, posts] = await Promise.all([fetchUser(1), fetchPosts(1)]);

// Последовательно (когда зависят друг от друга):
const user = await fetchUser(1);
const posts = await fetchPosts(user.id);
```

`await` без `async` — синтаксическая ошибка. `await` приостанавливает **только текущую функцию**, не event loop.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Чистые функции и функциональное программирование?

**Чистая функция**: одинаковые аргументы → одинаковый результат, нет побочных эффектов.

```javascript
// Нечистая — зависит от внешнего состояния и мутирует данные
let total = 0;
function addToTotal(n) { total += n; return total; }

// Чистая — предсказуема, тестируема
function add(a, b) { return a + b; }

// Функциональный стиль:
const prices = [10, 20, 30];
const total = prices
  .filter(p => p > 15)           // нет мутации
  .map(p => p * 1.2)             // возвращает новый массив
  .reduce((sum, p) => sum + p, 0); // 60
```

**Принципы ФП**: иммутабельность, чистые функции, функции высшего порядка, composition, declarative стиль.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Каррирование и частичное применение?

**Каррирование** — преобразование функции с N аргументами в цепочку функций с 1 аргументом:

```javascript
// Каррирование
const add = (a) => (b) => a + b;
const add5 = add(5);     // частично применённая функция
add5(3);  // 8
add5(10); // 15

// Автоматическое каррирование (lodash _.curry):
const multiply = _.curry((a, b, c) => a * b * c);
multiply(2)(3)(4);   // 24
multiply(2, 3)(4);   // 24
multiply(2)(3, 4);   // 24
```

**Частичное применение** — фиксация части аргументов:

```javascript
function partial(fn, ...presetArgs) {
  return (...laterArgs) => fn(...presetArgs, ...laterArgs);
}

const multiply = (a, b) => a * b;
const double = partial(multiply, 2);
double(5); // 10
```

Практика: переиспользуемые трансформеры данных, точечный стиль (point-free).

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Set, Map, WeakMap, WeakSet — когда использовать?

**Set** — коллекция уникальных значений:

```javascript
const set = new Set([1, 2, 2, 3]); // {1, 2, 3}
set.has(2); // true
set.add(4); set.delete(1);

// Дедупликация массива:
const unique = [...new Set(array)];
```

**Map** — ключ-значение с любыми типами ключей:

```javascript
const map = new Map();
map.set(objKey, "value"); // объект как ключ!
map.get(objKey);          // "value"
map.size;                 // количество записей
// Итерация: map.keys(), map.values(), map.entries()
```

**WeakMap** / **WeakSet** — слабые ссылки (ключ/значение могут быть GC). Используются для: metadata хранения без утечек памяти, приватных данных класса, кэширования DOM элементов.

```javascript
const cache = new WeakMap();
function process(element) {
  if (cache.has(element)) return cache.get(element);
  const result = expensiveOperation(element);
  cache.set(element, result); // удалится когда element GC'd
  return result;
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Прототипная цепочка в JavaScript?

Каждый объект имеет скрытое свойство `[[Prototype]]` (доступно через `Object.getPrototypeOf()`). При поиске свойства JS идёт по цепочке прототипов до `null`.

```javascript
const animal = { breathe() { return "breathing"; } };
const dog = Object.create(animal); // dog.__proto__ = animal
dog.bark = function() { return "woof"; };

dog.bark();    // "woof" — собственный метод
dog.breathe(); // "breathing" — из прототипа animal

// Классы ES6 — синтаксический сахар над прототипами:
class Animal { breathe() { return "breathing"; } }
class Dog extends Animal { bark() { return "woof"; } }
// Dog.prototype.__proto__ === Animal.prototype
```

`hasOwnProperty` / `Object.hasOwn` — проверить что свойство собственное, не унаследованное.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое Big O нотация?

Big O — описание роста сложности алгоритма в зависимости от размера входных данных:

| O() | Название | Пример |
|-----|----------|--------|
| O(1) | Константная | Доступ по индексу массива, Map.get() |
| O(log n) | Логарифмическая | Бинарный поиск |
| O(n) | Линейная | Перебор массива |
| O(n log n) | Линеарифмическая | Array.sort() |
| O(n²) | Квадратичная | Вложенные циклы |

```javascript
// O(1) — Map лучше объекта для частых lookups
const map = new Map();
map.get(key); // O(1)

// O(n) — Set для проверки вхождения
const set = new Set(largeArray);
set.has(value); // O(1) вместо array.includes() = O(n)
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Генераторы и итераторы?

**Итератор** — объект с методом `next()`, возвращающим `{ value, done }`. **Генератор** — функция, возвращающая итератор через `yield`.

```javascript
function* range(start, end, step = 1) {
  for (let i = start; i < end; i += step) {
    yield i; // приостановить и вернуть значение
  }
}

for (const n of range(0, 10, 2)) {
  console.log(n); // 0, 2, 4, 6, 8
}

// Бесконечный генератор:
function* idGenerator() {
  let id = 1;
  while (true) yield id++;
}
const gen = idGenerator();
gen.next().value; // 1
gen.next().value; // 2
```

Генераторы используются в: `async/await` (под капотом), lazy evaluation, пагинация данных, конечные автоматы.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Чем отличаются Promise.all, allSettled, race, any?

Все четыре метода принимают массив промисов, но по-разному реагируют на ошибки и успехи:

| Метод | Резолвится | Реджектится |
|---|---|---|
| `Promise.all` | когда все выполнились | при первой ошибке |
| `Promise.allSettled` | когда все завершились (любой исход) | никогда |
| `Promise.race` | первый завершившийся | первый завершившийся |
| `Promise.any` | первый выполнившийся | если все отклонены (AggregateError) |

```javascript
// all: нужен результат всех, откажется, если хоть один упал
const [user, posts] = await Promise.all([fetchUser(), fetchPosts()]);

// allSettled: даже при ошибках получим все результаты
const results = await Promise.allSettled([fetchUser(), fetchPosts()]);
results.forEach(r => {
  if (r.status === 'fulfilled') console.log(r.value);
  else console.error(r.reason);
});

// race: получить результат или ошибку первого завершившегося (таймаут и т.д.)
const result = await Promise.race([fetchData(), timeout(5000)]);
```

**Связанные задачи:**

- [Реализация Promise.all и Promise.allSettled](../../../tasks/frontend/javascript/3_javascript_senior.md#реализация-promiseall-и-promiseallsettled)

**Материалы для изучения:**

- [MDN: Promise.allSettled](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled)

---

## Что такое мемоизация и для чего она нужна?

Мемоизация — техника оптимизации: функция запоминает (кэширует) результат для заданных входных данных и возвращает сохранённый результат при повторном вызове с теми же аргументами.

```javascript
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

const expensiveCalc = memoize((n) => {
  console.log('Computing...');
  return n * n;
});

expensiveCalc(5); // Computing... 25
expensiveCalc(5); // 25 (без Computing!)
expensiveCalc(6); // Computing... 36
```

**Когда применять:** чистые функции с дорогим вычислением (рекурсивный Fibonacci, парсинг, сложные трансформации). Не подходит для функций с побочными эффектами, недетерминированным вводом или бесконечным множеством уникальных аргументов.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: Map](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Map)

---

## F.prototype и прототипное наследование через функции-конструкторы?

Каждая функция в JS имеет свойство `prototype` — объект, который становится `__proto__` нового экземпляра при вызове `new F()`.

```javascript
function Animal(name) {
  this.name = name;
}

// Методы на прототипе — разделяются между всеми экземплярами
Animal.prototype.speak = function() {
  return `${this.name} says...`;
};

const cat = new Animal('Cat');
cat.speak();              // 'Cat says...'
cat.__proto__ === Animal.prototype; // true

// Наследование
function Dog(name, breed) {
  Animal.call(this, name); // заимствовать свойства
  this.breed = breed;
}

Dog.prototype = Object.create(Animal.prototype); // установить цепочку
Dog.prototype.constructor = Dog;                 // восстановить constructor

Dog.prototype.bark = function() { return 'Woof'; };

const dog = new Dog('Rex', 'Lab');
dog.speak();  // 'Rex says...' — из Animal.prototype
dog.bark();   // 'Woof'
dog instanceof Dog;    // true
dog instanceof Animal; // true
```

`class` ES6 — сахар над этим механизмом, но принцип тот же.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Встроенные прототипы и их расширение?

Все встроенные типы (`Array`, `String`, `Object`, ...) имеют `prototype`. Можно добавлять методы, но это почти всегда плохая практика.

```javascript
// Polyfill: безопасно если проверять вначале
if (!Array.prototype.last) {
  Array.prototype.last = function() {
    return this[this.length - 1];
  };
}
[1, 2, 3].last(); // 3

// Почему НЕЛЬЗЯ без проверки:
// 1. Конфликт с будущими версиями языка
// 2. Поломка сторонних библиотек
// 3. for..in на массивах начнёт возвращать новый метод

// Структура прототипной цепочки:
// instance -> Array.prototype -> Object.prototype -> null
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Примеси (mixins) в JavaScript?

**Mixin** — паттерн добавления методов в класс/объект без наследования. Решает проблему отсутствия множественного наследования в JS.

```javascript
// Простой mixin: копирование методов
const Serializable = {
  serialize() { return JSON.stringify(this); },
  deserialize(str) { return Object.assign(this, JSON.parse(str)); },
};

const Validatable = {
  validate() { return Object.keys(this).every(k => this[k] !== null); },
};

class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }
}

// Нанести mixinы на прототип
Object.assign(User.prototype, Serializable, Validatable);

const u = new User('Alice', 'alice@mail.com');
u.validate();   // true
u.serialize();  // '{"name":"Alice","email":"alice@mail.com"}'

// Альтернатива: mixin-фабрика (Higher-Order Component pattern)
const withTimestamps = (Base) => class extends Base {
  createdAt = new Date();
};
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Промиссификация (promisify)?

**Promisify** — преобразование callback-API в Promise-базированное.

```javascript
// callback-стиль Node.js: (err, result) =>
function readFile(path, callback) { /* ... */ }

// promisify вручную
function promisify(fn) {
  return function(...args) {
    return new Promise((resolve, reject) => {
      fn(...args, (err, result) => {
        if (err) reject(err);
        else resolve(result);
      });
    });
  };
}

const readFileAsync = promisify(readFile);

// Использование
const content = await readFileAsync('./file.txt');

// Node.js: util.promisify(require('fs').readFile)
import { promisify } from 'util';
const readFileAsync2 = promisify(require('fs').readFile);
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Observable vs Promise: реактивное программирование?

| | Promise | Observable (RxJS) |
|---|---|---|
| Значений | 1 | 0..N |
| Отмена | невозможна | `.unsubscribe()` |
| Ленивое | да (eager) | да (по умолчанию lazy) |
| Операторы | `.then` | `map`, `filter`, `switchMap`, `debounceTime`... |
| Сокеты | нет | есть |

```javascript
// Promise — одно значение
const p = fetch('/api/user').then(r => r.json());

// Observable (RxJS) — поток событий
import { fromEvent, interval } from 'rxjs';
import { debounceTime, map } from 'rxjs/operators';

const clicks$ = fromEvent(button, 'click').pipe(
  debounceTime(300),
  map(e => e.target.value)
);

const sub = clicks$.subscribe(val => console.log(val));
sub.unsubscribe(); // отписаться

// Реактивное программирование (RP) — программирование с асинхронными
// потоками данных. Популяризовано RxJS, используется в Angular.
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Композиция вс наследование (composition vs inheritance)?

**Наследование** — "есть" (собака есть животное). Жёсткая связь.
**Композиция** — "имеет" (собака имеет поведение плаваца). Слабая связь.

Правило: **предпочитай композицию наследованию** (GoF, "Prefer composition over inheritance").

```javascript
// Проблема наследования: хрупкая иерархия
class Animal {}
class FlyingAnimal extends Animal {} // не все животные летают!
class SwimmingAnimal extends Animal {}
// Дутонос летает И плавает — что без множественного наследования?

// Композиция: изолированные поведения
const canFly = { fly: () => 'Flying' };
const canSwim = { swim: () => 'Swimming' };
const canWalk = { walk: () => 'Walking' };

function createDuck(name) {
  return Object.assign({ name }, canFly, canSwim, canWalk);
}

const duck = createDuck('Donald');
duck.fly();  // 'Flying'
duck.swim(); // 'Swimming'
```

**Когда наследование оправдано:** когда отношение действительно типа "есть" (например, `Square extends Rectangle` спорно, но `Dog extends Animal` оправдано).

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
