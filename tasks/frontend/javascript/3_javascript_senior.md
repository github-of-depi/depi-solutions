# JavaScript — Senior Tasks

## Задачи

- [Функция delay (Promise-обёртка над setTimeout)](#функция-delay-promise-обёртка-над-settimeout)
- [Реализация Promise.all и Promise.allSettled](#реализация-promiseall-и-promiseallsettled)
- [Параллельный fetch с обработкой ошибок](#параллельный-fetch-с-обработкой-ошибок)
- [Запрос с повторными попытками (retry с exponential backoff)](#запрос-с-повторными-попытками-retry-с-exponential-backoff)

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
