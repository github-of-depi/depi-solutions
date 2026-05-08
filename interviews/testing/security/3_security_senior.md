# Security (Frontend) — Senior

## Вопросы

- [Как строить security review процесс для фронтенда?](#как-строить-security-review-процесс-для-фронтенда)
- [Как защититься от supply chain атак?](#как-защититься-от-supply-chain-атак)
- [Что такое CORS и как правильно настроить?](#что-такое-cors-и-как-правильно-настроить)
- [Как безопасно обрабатывать файловую загрузку?](#как-безопасно-обрабатывать-файловую-загрузку)

---

## Как строить security review процесс для фронтенда?

1. **SAST**: ESLint security rules (`eslint-plugin-security`), CodeQL
2. **DAST**: автоматическое сканирование (OWASP ZAP) в CI
3. **Dependency audit**: `npm audit`, Snyk, Dependabot
4. **Code review checklist**: проверка XSS, CSRF, auth endpoints
5. **Security headers**: автоматическая проверка (securityheaders.com)

---

## Как защититься от supply chain атак?

Supply chain атака — компрометация npm пакета. Защита:
- `package-lock.json` / `pnpm-lock.yaml` — точные версии
- `npm audit` в CI — проверка известных CVE
- Snyk / GitHub Dependabot — мониторинг
- SRI для CDN ресурсов
- Ограниченные permissions для npm скриптов (`ignore-scripts`)

---

## Что такое CORS и как правильно настроить?

CORS — браузерный механизм контроля cross-origin запросов. `Access-Control-Allow-Origin: *` опасно для endpoints с cookies. Правильно: конкретные origins, `withCredentials: true` требует точного origin (не `*`).

---

## Как безопасно обрабатывать файловую загрузку?

1. **Типы**: whitelist MIME типов на клиенте + сервере (не только расширение)
2. **Размер**: `maxSize` ограничение
3. **Сканирование**: вирусное сканирование на сервере (ClamAV, cloud service)
4. **Хранение**: отдельный домен/CDN для загруженных файлов (изоляция от основного)
5. **Content-Disposition**: `attachment` для скачивания, не `inline`
