# JavaScript — Middle Tasks

## Задачи

- [Глубокое копирование объекта](#глубокое-копирование-объекта)
- [Порядок вывода в event loop](#порядок-вывода-в-event-loop)
- [Плоский объект из вложенного (flatten)](#плоский-объект-из-вложенного-flatten)
- [Получить значение по dot-path строке](#получить-значение-по-dot-path-строке)

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
