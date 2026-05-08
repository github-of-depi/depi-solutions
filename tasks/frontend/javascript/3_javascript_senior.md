# JavaScript — Senior Tasks

## Задачи

- [Функция delay (Promise-обёртка над setTimeout)](#функция-delay-promise-обёртка-над-settimeout)
- [Реализация Promise.all и Promise.allSettled](#реализация-promiseall-и-promiseallsettled)
- [Параллельный fetch с обработкой ошибок](#параллельный-fetch-с-обработкой-ошибок)
- [Запрос с повторными попытками (retry с exponential backoff)](#запрос-с-повторными-попытками-retry-с-exponential-backoff)
- [Агрегация данных из нескольких API](#агрегация-данных-из-нескольких-api)
- [Класс EventEmitter](#класс-eventemitter)
- [Банкомат: выдача купюр (getMoney)](#банкомат-выдача-купюр-getmoney)

---

## Функция delay (Promise-обёртка над setTimeout)

Реализуйте функцию `delay(ms)`, которая возвращает Promise, резолвящийся через `ms` миллисекунд. Используйте её для имитации паузы в async-функции.

```javascript
async function example() {
  console.log("start");
  await delay(1000);
  console.log("after 1 second");
}
```

**Связанные вопросы:**

- [Как работают Promise?](../../../interviews/frontend/javascript/3_javascript_senior.md#как-работают-promise)

<details>
<summary>Решение</summary>

```javascript
function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// Использование
async function example() {
  console.log("start");
  await delay(1000);
  console.log("after 1 second");
}

// С передачей значения
function delayWithValue(ms, value) {
  return new Promise(resolve => setTimeout(() => resolve(value), ms));
}

delayWithValue(500, 42).then(v => console.log(v)); // 42 через 500ms
```

</details>

---

## Реализация Promise.all и Promise.allSettled

Реализуйте функции `myPromiseAll(promises)` и `myPromiseAllSettled(promises)`, воспроизводящие поведение нативных `Promise.all` и `Promise.allSettled`.

- `myPromiseAll` резолвится массивом результатов, если все выполнились, и реджектится при первой ошибке.
- `myPromiseAllSettled` всегда резолвится массивом объектов `{ status, value/reason }`.

**Связанные вопросы:**

- [Чем отличаются Promise.all, allSettled, race, any?](../../../interviews/frontend/javascript/3_javascript_senior.md#чем-отличаются-promiseall-allsettled-race-any)

<details>
<summary>Решение</summary>

```javascript
function myPromiseAll(promises) {
  return new Promise((resolve, reject) => {
    if (promises.length === 0) return resolve([]);

    const results = new Array(promises.length);
    let resolved = 0;

    promises.forEach((p, i) => {
      Promise.resolve(p).then(value => {
        results[i] = value;
        resolved++;
        if (resolved === promises.length) resolve(results);
      }).catch(reject);
    });
  });
}

function myPromiseAllSettled(promises) {
  return new Promise(resolve => {
    if (promises.length === 0) return resolve([]);

    const results = new Array(promises.length);
    let settled = 0;

    promises.forEach((p, i) => {
      Promise.resolve(p)
        .then(value => {
          results[i] = { status: 'fulfilled', value };
        })
        .catch(reason => {
          results[i] = { status: 'rejected', reason };
        })
        .finally(() => {
          settled++;
          if (settled === promises.length) resolve(results);
        });
    });
  });
}
```

</details>

---

## Параллельный fetch с обработкой ошибок

Напишите функцию `fetchAll(urls)`, которая запрашивает все переданные URL параллельно и возвращает массив результатов. Если какой-то запрос упал — в результирующем массиве на его месте должно быть `null`, остальные результаты должны присутствовать.

```javascript
const results = await fetchAll([
  "https://api.example.com/users",
  "https://api.example.com/invalid",
  "https://api.example.com/posts",
]);
// [ [...users], null, [...posts] ]
```

**Связанные вопросы:**

- [Async/Await: как работает под капотом?](../../../interviews/frontend/javascript/3_javascript_senior.md#asyncawait-как-работает-под-капотом)
- [Чем отличаются Promise.all, allSettled, race, any?](../../../interviews/frontend/javascript/3_javascript_senior.md#чем-отличаются-promiseall-allsettled-race-any)

<details>
<summary>Решение</summary>

```javascript
async function fetchAll(urls) {
  const results = await Promise.allSettled(
    urls.map(url => fetch(url).then(res => {
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      return res.json();
    }))
  );

  return results.map(r =>
    r.status === 'fulfilled' ? r.value : null
  );
}

// Альтернатива: через Promise.all + индивидуальный catch
async function fetchAllAlt(urls) {
  return Promise.all(
    urls.map(url =>
      fetch(url)
        .then(res => res.ok ? res.json() : Promise.reject(res.status))
        .catch(() => null)
    )
  );
}
```

</details>

---

## Запрос с повторными попытками (retry с exponential backoff)

Реализуйте функцию `fetchWithRetry(url, retries, baseDelay)`, которая выполняет HTTP-запрос и при неудаче повторяет его до `retries` раз с экспоненциальной задержкой: 1-я попытка — `baseDelay` мс, 2-я — `baseDelay * 2`, 3-я — `baseDelay * 4` и т.д. Если все попытки исчерпаны — бросить ошибку.

```javascript
// Пример: 3 попытки, начальная задержка 200ms
const data = await fetchWithRetry("/api/data", 3, 200);
```

**Связанные вопросы:**

- [Async/Await: как работает под капотом?](../../../interviews/frontend/javascript/3_javascript_senior.md#asyncawait-как-работает-под-капотом)

<details>
<summary>Решение</summary>

```javascript
function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

async function fetchWithRetry(url, retries = 3, baseDelay = 300) {
  for (let attempt = 0; attempt <= retries; attempt++) {
    try {
      const response = await fetch(url);
      if (!response.ok) throw new Error(`HTTP ${response.status}`);
      return await response.json();
    } catch (err) {
      if (attempt === retries) throw err;
      const waitTime = baseDelay * Math.pow(2, attempt);
      console.warn(`Attempt ${attempt + 1} failed. Retrying in ${waitTime}ms...`);
      await delay(waitTime);
    }
  }
}

// Вызов
fetchWithRetry("/api/data", 3, 200)
  .then(data => console.log(data))
  .catch(err => console.error("All retries failed:", err.message));
```

</details>

---

## Агрегация данных из нескольких API

Напишите асинхронную функцию `getPosts()`, которая параллельно запрашивает данные из трёх эндпоинтов и возвращает агрегированный список постов:

- `GET /posts` — список постов (`id`, `userId`, `title`)
- `GET /users` — пользователи (`id`, `name`)
- `GET /comments` — комментарии (`postId`, ...)

Результат — массив объектов вида:
```javascript
// [{ id, title, userName, commentsCount }, ...]
```

**Связанные вопросы:**

- [Чем отличаются Promise.all, allSettled, race, any?](../../../interviews/frontend/javascript/3_javascript_senior.md#чем-отличаются-promiseall-allsettled-race-any)

<details>
<summary>Решение</summary>

```javascript
const BASE = 'https://jsonplaceholder.typicode.com';

const getPosts = async () => {
  const [posts, users, comments] = await Promise.all([
    fetch(`${BASE}/posts`).then(r => r.json()),
    fetch(`${BASE}/users`).then(r => r.json()),
    fetch(`${BASE}/comments`).then(r => r.json()),
  ]);

  // Индексы для быстрого поиска O(1)
  const usersById = Object.fromEntries(users.map(u => [u.id, u.name]));
  const commentCountByPost = comments.reduce((acc, c) => {
    acc[c.postId] = (acc[c.postId] || 0) + 1;
    return acc;
  }, {});

  return posts.map(post => ({
    id: post.id,
    title: post.title,
    userName: usersById[post.userId] ?? 'Unknown',
    commentsCount: commentCountByPost[post.id] ?? 0,
  }));
};

getPosts().then(data => console.log(data));
```

</details>

---

## Класс EventEmitter

Реализуйте класс `EventEmitter` с тремя методами:
- `on(event, listener)` — подписаться на событие
- `off(event, listener)` — отписаться
- `emit(event, ...args)` — вызвать все подписчики события

```javascript
const emitter = new EventEmitter();

const greetListener = (name) => console.log(`Hello, ${name}!`);

emitter.on('greet', greetListener);
emitter.emit('greet', 'Alice'); // Hello, Alice!

emitter.off('greet', greetListener);
emitter.emit('greet', 'Bob');   // Без вывода
```

**Связанные вопросы:**

<!-- Связанных вопросов нет -->

<details>
<summary>Решение</summary>

```javascript
class EventEmitter {
  constructor() {
    this.events = {}; // Record<string, Function[]>
  }

  on(event, listener) {
    if (!this.events[event]) this.events[event] = [];
    this.events[event].push(listener);
    return this; // позволяет цепочку .on().on()
  }

  off(event, listener) {
    if (!this.events[event]) return this;
    this.events[event] = this.events[event].filter(l => l !== listener);
    return this;
  }

  emit(event, ...args) {
    if (!this.events[event]) return;
    // копия массива чтобы off внутри листенера не сломал итерацию
    [...this.events[event]].forEach(listener => listener(...args));
  }
}
```

</details>

---

## Банкомат: выдача купюр (getMoney)

**Задача 1:** Реализуйте функцию `getMoney(amount)`, которая возвращает объект с количеством купюр каждого номинала (минимальное количество купюр). Доступные номиналы: 5000, 2000, 1000, 500, 100, 50.

**Задача 2:** Добавить ограничения по каждому номиналу через объект `limits`.

```javascript
getMoney(6200);
// { 5000: 1, 2000: 0, 1000: 1, 500: 0, 100: 2, 50: 0 }

getMoney(6200, { 5000: 0, 2000: 2, 1000: 7, 100: 5 });
// { 5000: 0, 2000: 2, 1000: 2, 100: 2 } (остаток выдаётся в пределах доступного)
```

**Связанные вопросы:**

<!-- Связанных вопросов нет -->

<details>
<summary>Решение</summary>

```javascript
// Задача 1: без ограничений
function getMoney(amount) {
  const nominals = [5000, 2000, 1000, 500, 100, 50];
  const result = {};

  for (const nominal of nominals) {
    const count = Math.floor(amount / nominal);
    result[nominal] = count;
    amount -= count * nominal;
  }

  return result;
}

// Задача 2: с ограничениями
function getMoney(amount, limits = {}) {
  const nominals = [5000, 2000, 1000, 500, 100, 50];
  const result = {};

  for (const nominal of nominals) {
    if (limits[nominal] === 0) continue; // номинал закончился
    const maxAvailable = limits[nominal] !== undefined
      ? limits[nominal]
      : Infinity;
    const count = Math.min(Math.floor(amount / nominal), maxAvailable);
    if (count > 0) result[nominal] = count;
    amount -= count * nominal;
  }

  return result;
}

console.log(getMoney(6200));                                   // { 5000: 1, 1000: 1, 100: 2 }
console.log(getMoney(6200, { 5000: 0, 2000: 2, 1000: 7, 100: 5 })); // { 2000: 2, 1000: 2, 100: 2 }
```

</details>
