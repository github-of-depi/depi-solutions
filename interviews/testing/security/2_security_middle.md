# Security (Frontend) — Middle

## Вопросы

- [Как реализовать защиту от XSS?](#как-реализовать-защиту-от-xss)
- [Как настроить CSP без 'unsafe-inline'?](#как-настроить-csp-без-unsafe-inline)
- [Что такое Subresource Integrity (SRI)?](#что-такое-subresource-integrity-sri)
- [Как безопасно работать с пользовательским HTML?](#как-безопасно-работать-с-пользовательским-html)
- [Что такое clickjacking и как защититься?](#что-такое-clickjacking-и-как-защититься)

---

## Как реализовать защиту от XSS?

1. **React** — не использовать `dangerouslySetInnerHTML` без санитизации
2. **Санитизация** — DOMPurify для пользовательского HTML
3. **CSP** — ограничить источники скриптов
4. **httpOnly cookies** — токены недоступны JS
5. **Trusted Types API** — ограничить создание опасных DOM операций

```typescript
import DOMPurify from "dompurify";
const clean = DOMPurify.sanitize(userContent, { ALLOWED_TAGS: ["b", "i", "a"] });
<div dangerouslySetInnerHTML={{ __html: clean }} />
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как настроить CSP без 'unsafe-inline'?

Nonces: сервер генерирует случайный nonce, добавляет к скриптам и CSP заголовку. Inline скрипты только с matching nonce.

```html
<!-- Сервер генерирует nonce на каждый запрос -->
<script nonce="abc123">/* inline script */</script>
```
```
Content-Security-Policy: script-src 'nonce-abc123'
```

Next.js поддерживает CSP nonces через middleware.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое Subresource Integrity (SRI)?

SRI — проверка целостности внешних ресурсов через hash. Браузер откажется загружать ресурс, если hash не совпадает (защита от CDN компрометации).

```html
<script src="https://cdn.example.com/lib.js"
  integrity="sha384-abc123..."
  crossorigin="anonymous"></script>
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как безопасно работать с пользовательским HTML?

DOMPurify — whitelist-based HTML санитайзер. Удаляет опасные теги/атрибуты. Настраивать allowlist минимально необходимый (не `ALLOWED_TAGS: "*"`).

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое clickjacking и как защититься?

Clickjacking — злоумышленник встраивает ваш сайт в `<iframe>`, пользователь кликает не на то что видит. Защита: `X-Frame-Options: DENY` или CSP `frame-ancestors 'none'`.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
