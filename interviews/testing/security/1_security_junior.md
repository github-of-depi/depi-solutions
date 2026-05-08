# Security (Frontend) — Junior

## Вопросы

- [Что такое XSS?](#что-такое-xss)
- [Что такое CSRF?](#что-такое-csrf)
- [Что такое Content Security Policy?](#что-такое-content-security-policy)
- [Что такое OWASP Top 10?](#что-такое-owasp-top-10)
- [Как безопасно хранить токены?](#как-безопасно-хранить-токены)

---

## Что такое XSS?

XSS (Cross-Site Scripting) — внедрение вредоносного JS кода на страницу. Злоумышленник крадёт cookies/tokens, делает действия от имени пользователя. Типы: Reflected (в URL), Stored (в БД), DOM-based. Защита: экранирование вывода, CSP, React делает это автоматически (но опасны `dangerouslySetInnerHTML`).

---

## Что такое CSRF?

CSRF (Cross-Site Request Forgery) — вредоносный сайт заставляет браузер пользователя отправлять запросы к легитимному сайту с его cookies. Защита: SameSite cookie, CSRF-токены, проверка Origin заголовка.

---

## Что такое Content Security Policy?

CSP — HTTP заголовок, ограничивающий откуда браузер может загружать ресурсы (скрипты, стили, изображения). Защита от XSS и data injection атак.

```
Content-Security-Policy: default-src 'self'; script-src 'self' cdn.example.com; style-src 'self' 'unsafe-inline'
```

---

## Что такое OWASP Top 10?

OWASP Top 10 — список наиболее критичных уязвимостей веб-приложений. Для фронтенда релевантны: Injection (XSS), Broken Authentication, Security Misconfiguration, Vulnerable Components.

---

## Как безопасно хранить токены?

- **httpOnly Cookie** — защищена от JS (XSS-safe)
- **In-memory** — не переживает обновление страницы
- **Не localStorage** — доступна любому JS коду на странице

Refresh token → httpOnly cookie. Access token → in-memory.
