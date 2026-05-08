# Cloud — Senior

## Вопросы

- [Как строить multi-region frontend deployment?](#как-строить-multi-region-frontend-deployment)
- [Как оптимизировать CDN кэширование?](#как-оптимизировать-cdn-кэширование)
- [Как строить observability для cloud frontend?](#как-строить-observability-для-cloud-frontend)

---

## Как строить multi-region frontend deployment?

Статика: CDN поставляет везде автоматически. SSR: несколько регионов через cloud provider (Vercel auto, Fly.io). Database: Read replicas в каждом регионе. Georouting: Route53 Latency-based routing → ближайший регион.

---

## Как оптимизировать CDN кэширование?

Стратегия по типу ресурса:
- `main.abc123.js` (hash в имени) → `Cache-Control: max-age=31536000, immutable`
- `index.html` → `Cache-Control: no-cache` (всегда свежий, но условный)
- API ответы → `Cache-Control: public, s-maxage=60, stale-while-revalidate=3600`

Cache invalidation: при деплое инвалидировать CDN (`aws cloudfront create-invalidation`).

---

## Как строить observability для cloud frontend?

1. **RUM**: web-vitals → ваш analytics endpoint / Datadog RUM / New Relic Browser
2. **Error tracking**: Sentry с source maps (загружать при деплое)
3. **Uptime**: Cloudflare Healthchecks, Checkly synthetics
4. **Alerts**: PagerDuty / OpsGenie при выходе метрик за пороги
5. **Cost monitoring**: AWS Cost Explorer, Vercel usage alerts
