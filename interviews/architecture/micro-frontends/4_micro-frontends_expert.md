# Micro-frontends — Expert

## Вопросы

- [Как строить server-side composed micro-frontends?](#как-строить-server-side-composed-micro-frontends)
- [Как решать производительность в MFE архитектуре?](#как-решать-производительность-в-mfe-архитектуре)
- [Micro-frontends vs monolith: когда что выбрать?](#micro-frontends-vs-monolith-когда-что-выбрать)

---

## Как строить server-side composed micro-frontends?

Edge composition: каждый MFE рендерит свою часть на сервере (Edge Function/SSR). Cloudflare Workers / Vercel Edge: compose HTML фрагментов на уровне CDN. Next.js App Router + RSC: каждый team component — отдельный async Server Component.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как решать производительность в MFE архитектуре?

1. **Shared singleton зависимостей** — не грузить React несколько раз
2. **Import maps** — управление версиями зависимостей на уровне браузера
3. **Async loading с skeleton** — не блокировать shell пока грузятся remotes
4. **Preload критичных remotes** — `<link rel="modulepreload">` для top-level MFE
5. **Измерять LCP per MFE** — отдельные RUM метрики для каждого remote

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Micro-frontends vs monolith: когда что выбрать?

**Monorepo (рекомендуется по умолчанию)**: 1 команда или <50 разработчиков, единый технологический стек, нет требования независимого деплоя. **MFE**: 100+ разработчиков, разные команды с разными стеками/циклами деплоя, legacy migration, compliance/security boundaries между командами. MFE — не серебряная пуля. Сложность стоит дорого.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
