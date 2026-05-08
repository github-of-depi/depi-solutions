# REST — Middle

## Вопросы

- [Как работают interceptors в axios?](#как-работают-interceptors-в-axios)
- [Как отменить HTTP запрос?](#как-отменить-http-запрос)
- [Как правильно обрабатывать ошибки API?](#как-правильно-обрабатывать-ошибки-api)
- [Как реализовать retry логику?](#как-реализовать-retry-логику)
- [Что такое HTTP заголовки авторизации?](#что-такое-http-заголовки-авторизации)
- [Как загружать файлы через REST API?](#как-загружать-файлы-через-rest-api)
- [Что такое pagination в REST?](#что-такое-pagination-в-rest)

---

## Как работают interceptors в axios?

Interceptors перехватывают запросы/ответы глобально: добавление токена в заголовок, обработка 401 (refresh token), трансформация данных, логирование.

```typescript
axios.interceptors.request.use(config => {
  config.headers.Authorization = `Bearer ${getToken()}`;
  return config;
});
axios.interceptors.response.use(
  response => response,
  async error => {
    if (error.response?.status === 401) {
      await refreshToken();
      return axios(error.config); // повтор запроса
    }
    return Promise.reject(error);
  }
);
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как отменить HTTP запрос?

`AbortController` — нативный API для отмены fetch запросов. Важно для: отмены при размонтировании компонента, debounced search (отмена предыдущего запроса).

```typescript
const controller = new AbortController();
const response = await fetch("/api/search", { signal: controller.signal });
// Отмена
controller.abort();

// В React useEffect
useEffect(() => {
  const controller = new AbortController();
  fetchData(controller.signal);
  return () => controller.abort();
}, [query]);
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как правильно обрабатывать ошибки API?

Разделять типы ошибок: сетевые (fetch failed), HTTP ошибки (4xx/5xx), ошибки валидации (422), бизнес-ошибки (в теле ответа). Показывать пользователю только понятные сообщения, логировать технические детали.

```typescript
async function apiCall<T>(url: string): Promise<T> {
  const res = await fetch(url);
  if (!res.ok) {
    const error = await res.json().catch(() => ({}));
    throw new ApiError(res.status, error.message ?? "Что-то пошло не так");
  }
  return res.json();
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как реализовать retry логику?

Повторять запрос при временных ошибках (5xx, network errors). Использовать exponential backoff. TanStack Query делает это автоматически через `retry` опцию.

```typescript
async function fetchWithRetry(url: string, retries = 3): Promise<Response> {
  try {
    return await fetch(url);
  } catch (e) {
    if (retries === 0) throw e;
    await new Promise(r => setTimeout(r, 2 ** (3 - retries) * 1000));
    return fetchWithRetry(url, retries - 1);
  }
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое HTTP заголовки авторизации?

`Authorization: Bearer <token>` — JWT токен в заголовке. `Authorization: Basic base64(login:pass)` — базовая аутентификация (только HTTPS). `Cookie: session=...` — cookie-based (httpOnly, Secure). Bearer token в заголовке безопаснее localStorage от XSS.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как загружать файлы через REST API?

`multipart/form-data` через `FormData`. Прогресс через `XMLHttpRequest` или axios `onUploadProgress`.

```typescript
const formData = new FormData();
formData.append("file", file);
formData.append("name", file.name);

const { data } = await axios.post("/api/upload", formData, {
  onUploadProgress: (e) => setProgress(Math.round((e.loaded / e.total!) * 100)),
});
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое pagination в REST?

- **Offset pagination**: `?page=2&limit=20` — просто, но неэффективно для больших данных
- **Cursor pagination**: `?after=lastId&limit=20` — эффективно, без пропусков при вставке
- **Keyset pagination**: аналог cursor, через индексированное поле

Cursor предпочтителен для real-time данных (infinite scroll).

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
