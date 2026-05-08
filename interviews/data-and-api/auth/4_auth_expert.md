# Auth — Expert

## Вопросы

- [Что такое Zero-Trust архитектура и как она влияет на фронтенд?](#что-такое-zero-trust-архитектура-и-как-она-влияет-на-фронтенд)
- [Как реализовать WebAuthn (Passkeys)?](#как-реализовать-webauthn-passkeys)
- [Как строить multi-tenant аутентификацию?](#как-строить-multi-tenant-аутентификацию)
- [Как аудировать и мониторить аутентификацию?](#как-аудировать-и-мониторить-аутентификацию)

---

## Что такое Zero-Trust архитектура и как она влияет на фронтенд?

Zero-Trust: "никогда не доверяй, всегда проверяй". Каждый запрос аутентифицирован и авторизован независимо от источника. Для фронтенда: короткоживущие токены, частая проверка авторизации, mutual TLS (mTLS) для machine-to-machine, контекстная авторизация (device, location, risk score).

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как реализовать WebAuthn (Passkeys)?

WebAuthn — W3C стандарт для биометрической аутентификации (TouchID, FaceID, Windows Hello). Браузерный API: `navigator.credentials.create()` для регистрации, `navigator.credentials.get()` для входа.

```typescript
// Регистрация
const credential = await navigator.credentials.create({
  publicKey: {
    challenge: serverChallenge,
    rp: { name: "My App", id: "example.com" },
    user: { id: userId, name: email, displayName: name },
    pubKeyCredParams: [{ alg: -7, type: "public-key" }],
    authenticatorSelection: { residentKey: "required" },
  },
});
// Отправить credential на сервер для верификации
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как строить multi-tenant аутентификацию?

Tenant isolation: поддомен (`tenant.app.com`), путь (`app.com/tenant`), или header. JWT содержит `tenantId` в claims. Middleware (Next.js) определяет tenant из поддомена, проверяет доступ. Auth сервис может быть shared с tenant-aware конфигурацией (отдельные JWKS, провайдеры).

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как аудировать и мониторить аутентификацию?

1. Логировать все auth события (login, logout, token refresh, failed attempts)
2. Аномалии: login с нового устройства/IP → email уведомление
3. Rate limiting: lockout после N неудачных попыток
4. Security headers: HSTS, X-Frame-Options, CSP
5. Sentry + structured logging для auth errors

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
