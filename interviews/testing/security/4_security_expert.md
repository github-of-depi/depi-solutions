# Security (Frontend) — Expert

## Вопросы

- [Как строить threat model для фронтенд приложения?](#как-строить-threat-model-для-фронтенд-приложения)
- [Что такое Trusted Types и как они помогают?](#что-такое-trusted-types-и-как-они-помогают)

---

## Как строить threat model для фронтенд приложения?

STRIDE модель для фронтенда:
- **S**poofing: фишинг, credential theft → MFA, httpOnly cookies
- **T**ampering: XSS, MITM → CSP, HTTPS, SRI
- **R**epudiation: нет логов → audit log чувствительных действий
- **I**nformation Disclosure: sensitive data в URL/localStorage → secure storage
- **D**oS: Client-side heavy operations → rate limiting, debouncing
- **E**levation of Privilege: IDOR, broken RBAC → server-side validation

DREAD: Damage, Reproducibility, Exploitability, Affected users, Discoverability — scoring уязвимостей.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое Trusted Types и как они помогают?

Trusted Types (W3C) — браузерный API, запрещающий назначение строк в "опасные" DOM синки без явного создания Trusted Type объекта. Устраняет DOM XSS at the source.

```javascript
// CSP: require-trusted-types-for 'script';
// Без Trusted Types — CSP нарушение:
element.innerHTML = userInput; // TypeError!

// С Trusted Types:
const policy = trustedTypes.createPolicy("my-policy", {
  createHTML: (input) => DOMPurify.sanitize(input),
});
element.innerHTML = policy.createHTML(userInput); // OK
```

React поддерживает Trusted Types с v18.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
