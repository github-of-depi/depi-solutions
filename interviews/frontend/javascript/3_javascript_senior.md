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
