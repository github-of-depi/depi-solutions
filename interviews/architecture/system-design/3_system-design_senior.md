# System Design (Frontend) — Senior

## Вопросы

- [Как выбирать архитектуру для нового проекта?](#как-выбирать-архитектуру-для-нового-проекта)
- [Как проектировать real-time collaborative editor?](#как-проектировать-real-time-collaborative-editor)
- [Как строить систему мониторинга фронтенда?](#как-строить-систему-мониторинга-фронтенда)
- [Как проектировать масштабируемый e-commerce фронтенд?](#как-проектировать-масштабируемый-e-commerce-фронтенд)

---

## Как выбирать архитектуру для нового проекта?

Вопросы для принятия решений:

| Критерий | Влияние |
|----------|---------|
| SEO важен? | SSR/SSG (Next.js) |
| Частые обновления контента? | SSR или ISR |
| Статичный контент? | SSG + CDN |
| Много интерактивности? | SPA |
| Несколько команд? | Micro-frontends |
| Бюджет на инфраструктуру? | SSG дешевле |

---

## Как проектировать real-time collaborative editor?

- **Синхронизация**: CRDT (Yjs) — merge без конфликтов
- **Transport**: WebSocket + Yjs WebSocket provider
- **Awareness**: курсоры других пользователей через Yjs Awareness
- **Persistence**: сохранение Y.Doc snapshot в БД, восстановление
- **Offline**: Yjs работает офлайн, синхронизируется при подключении

---

## Как строить систему мониторинга фронтенда?

- **RUM** (Real User Monitoring): Core Web Vitals в продакшне (web-vitals library → аналитика)
- **Error tracking**: Sentry с source maps, breadcrumbs
- **Session replay**: FullStory/LogRocket для debugging UX issues
- **Custom metrics**: Performance API для бизнес-метрик (время загрузки checkout)
- **Alerting**: пороги по p75/p95 метрик

---

## Как проектировать масштабируемый e-commerce фронтенд?

- **Рендеринг**: ISR для product pages (SEO + fresh content)
- **Edge**: персонализация на Edge Functions (цены, рекомендации)
- **Кэширование**: CDN с vari-by-cookie для A/B тестов
- **Search**: Algolia/ElasticSearch для product search с autocomplete
- **Performance**: image optimization (Next.js Image), preload критичных ресурсов
