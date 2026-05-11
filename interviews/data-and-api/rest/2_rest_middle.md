# REST — Middle

## Вопросы

- [Как работают interceptors в axios?](#как-работают-interceptors-в-axios)
- [Как отменить HTTP запрос?](#как-отменить-http-запрос)
- [Как правильно обрабатывать ошибки API?](#как-правильно-обрабатывать-ошибки-api)
- [Как реализовать retry логику?](#как-реализовать-retry-логику)
- [Что такое HTTP заголовки авторизации?](#что-такое-http-заголовки-авторизации)
- [Как загружать файлы через REST API?](#как-загружать-файлы-через-rest-api)
- [Что такое pagination в REST?](#что-такое-pagination-в-rest)
- [JSON Schema?](#json-schema)
- [GET vs HEAD?](#get-vs-head)
- [POST vs DELETE?](#post-vs-delete)
- [Cookies в HTTP?](#cookies-в-http)
- [Стили создания API?](#стили-создания-api)
- [API vs веб-сервисы?](#api-vs-веб-сервисы)
- [Что такое SOAP?](#что-такое-soap)
- [SOAP vs REST?](#soap-vs-rest)
- [Преимущества и недостатки REST API?](#преимущества-и-недостатки-rest-api)
- [Разница AJAX и REST?](#разница-ajax-и-rest)
- [Характеристики RESTful?](#характеристики-restful)
- [Как тестировать RESTful API?](#как-тестировать-restful-api)
- [Postman?](#postman)
- [Полезная нагрузка (payload)?](#полезная-нагрузка-payload)

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

---

## JSON Schema?

**JSON Schema** — стандарт описания структуры и валидации JSON-документов. Позволяет задать типы, обязательные поля, форматы.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "required": ["id", "name"],
  "properties": {
    "id":    { "type": "integer" },
    "name":  { "type": "string", "minLength": 1 },
    "email": { "type": "string", "format": "email" },
    "age":   { "type": "integer", "minimum": 0, "maximum": 150 },
    "tags":  { "type": "array", "items": { "type": "string" } }
  }
}
```

Используется для: документирования API (OpenAPI/Swagger использует JSON Schema), валидации входных данных на сервере, кодогенерации TypeScript-типов.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## GET vs HEAD?

**HEAD** — идентичен GET, но сервер возвращает только заголовки, без тела ответа. Используется для:
- Проверить существование ресурса (статус 200/404) без скачивания тела
- Узнать размер файла через `Content-Length` перед загрузкой
- Проверить актуальность кэша через `Last-Modified` / `ETag`

```javascript
const res = await fetch('/api/big-file', { method: 'HEAD' });
const size = res.headers.get('content-length');
// тело res не содержит данных
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## POST vs DELETE?

| | POST | DELETE |
|---|---|---|
| Семантика | Создание ресурса / действие | Удаление ресурса |
| Идемпотентный | ✗ нет | ✓ да |
| Тело запроса | ✓ есть | редко |
| Кэшируется | ✗ нет | ✗ нет |

```http
POST /api/users          → создать пользователя → 201 Created
DELETE /api/users/42     → удалить пользователя → 204 No Content
```

**Идемпотентность DELETE**: повторный вызов даёт тот же результат (ресурс уже удалён → `404` или `204`).

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Cookies в HTTP?

Cookie — небольшие данные, хранящиеся в браузере и автоматически отправляемые с каждым HTTP-запросом к соответствующему домену.

```http
HTTP/1.1 200 OK
Set-Cookie: token=abc123; HttpOnly; Secure; SameSite=Strict; Max-Age=86400; Path=/
```

**Атрибуты:**
- `HttpOnly` — недоступен из JS (защита от XSS)
- `Secure` — только HTTPS
- `SameSite=Strict/Lax/None` — защита от CSRF
- `Max-Age` / `Expires` — срок жизни
- `Domain` / `Path` — область действия

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Стили создания API?

| Стиль | Транспорт | Формат | Когда |
|---|---|---|---|
| **REST** | HTTP | JSON | Большинство веб-API |
| **GraphQL** | HTTP | JSON | Гибкие запросы, много сущностей |
| **SOAP** | HTTP/SMTP | XML | Корпоративные, legacy системы |
| **gRPC** | HTTP/2 | Protobuf | Микросервисы, высокая нагрузка |
| **WebSocket** | TCP | Любой | Realtime, двунаправленная связь |
| **SSE** | HTTP | text/event-stream | Серверный push |

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## API vs веб-сервисы?

**Веб-сервис** — подмножество API, доступное через сеть (HTTP). Все веб-сервисы — API, но не все API — веб-сервисы (библиотечный API не является веб-сервисом).

| | API | Веб-сервис |
|---|---|---|
| Транспорт | Любой (файл, HTTP, IPC) | Только сеть/HTTP |
| Протоколы | Любые | HTTP, SOAP, REST, GraphQL |
| Примеры | fs.readFile, Array.map | GitHub API, Payment Gateway |

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое SOAP?

**SOAP** (Simple Object Access Protocol) — протокол обмена сообщениями на основе XML. Строго типизирован, описывается WSDL-файлом.

```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <GetUserRequest xmlns="http://example.com/users">
      <UserId>42</UserId>
    </GetUserRequest>
  </soap:Body>
</soap:Envelope>
```

**Характеристики:** WS-Security для безопасности, транзакции (WS-AtomicTransaction), стандартизация — плюс для корпоративных систем. Минус: многословен, сложнее разработка.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## SOAP vs REST?

| | REST | SOAP |
|---|---|---|
| Формат | JSON (обычно) | XML (всегда) |
| Протокол | HTTP | HTTP, SMTP, TCP |
| Тип | Архитектурный стиль | Протокол |
| Типизация | Слабая | Строгая (WSDL) |
| Производительность | Выше (JSON легче) | Ниже (XML тяжелее) |
| Сложность | Низкая | Высокая |
| Применение | Веб, мобайл | Банки, корпоративные системы |

REST предпочтителен для большинства новых API. SOAP — там, где нужны встроенные стандарты WS-Security, надёжная доставка, enterprise интеграции.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Преимущества и недостатки REST API?

**Преимущества:**
- Простота и читаемость (JSON + HTTP методы)
- Stateless — масштабируется горизонтально
- Кэшируемость (GET-запросы)
- Независимость клиента и сервера
- Широкая поддержка инструментов

**Недостатки:**
- Overfetching — лишние данные в ответе
- Underfetching — несколько запросов для одного экрана
- Нет стандарта описания (но есть OpenAPI)
- Отсутствует realtime (нужен WebSocket/SSE)
- Версионирование API — усложняет поддержку

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Разница AJAX и REST?

**AJAX** — техника асинхронных запросов из браузера без перезагрузки страницы. Это способ коммуникации.

**REST** — архитектурный стиль проектирования API. Это набор принципов.

AJAX может обращаться к REST API, но также к SOAP, GraphQL или любому HTTP-эндпоинту. REST API могут вызываться не только через AJAX, но и через curl, мобильные приложения, серверный код.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Характеристики RESTful?

Рой Филдинг выделил 6 ограничений (constraints) REST:

1. **Client-Server** — разделение ответственности: UI отдельно от данных
2. **Stateless** — каждый запрос самодостаточен, сервер не хранит состояние клиента
3. **Cacheable** — ответы помечаются как кэшируемые или нет
4. **Uniform Interface** — единый интерфейс: ресурсы по URL, действия через HTTP методы
5. **Layered System** — клиент не знает о промежуточных слоях (proxy, CDN, gateway)
6. **Code on Demand** *(опционально)* — сервер может отдавать исполняемый код (JS)

API, следующий всем ограничениям, называется **RESTful**.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как тестировать RESTful API?

**Ручное тестирование:**
- **Postman** — GUI для отправки запросов, коллекции, тесты на JavaScript
- **Insomnia** — альтернатива Postman
- **curl** — командная строка

**Автоматизированное:**
```typescript
// Supertest (Node.js)
import request from 'supertest';
const res = await request(app).get('/api/users').set('Authorization', `Bearer ${token}`);
expect(res.status).toBe(200);
expect(res.body).toHaveLength(10);
```

**Инструменты:** Jest + Supertest для интеграционных тестов; Playwright/Cypress для E2E с API; contract testing через Pact.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Postman?

**Postman** — инструмент для работы с API: отправка HTTP-запросов, тестирование, документирование.

**Ключевые возможности:**
- **Collections** — группировка запросов по проекту
- **Environments** — переменные (`{{baseUrl}}`, `{{token}}`) для разных сред
- **Tests** — JavaScript-тесты после запроса: `pm.expect(pm.response.code).to.equal(200)`
- **Pre-request scripts** — подготовка данных перед запросом
- **Mock Server** — имитация API без бэкенда
- **Newman** — запуск Postman-коллекций в CI/CD

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Полезная нагрузка (payload)?

**Payload** — данные в теле (body) HTTP-запроса или ответа. Это «полезная» часть в отличие от служебных заголовков.

```http
POST /api/users HTTP/1.1
Content-Type: application/json

{ "name": "Alice", "email": "alice@example.com" }
↑ это payload запроса
```

```http
HTTP/1.1 201 Created
Content-Type: application/json

{ "id": 42, "name": "Alice", "createdAt": "2024-01-01" }
↑ это payload ответа
```

GET и DELETE обычно не имеют payload. Размер payload ограничивается настройками сервера (nginx: `client_max_body_size`).

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
