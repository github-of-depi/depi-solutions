# Next.js — Expert

## Вопросы

- [Что такое Partial Prerendering (PPR)?](#что-такое-partial-prerendering-ppr)
- [Как работает React Server Component Payload?](#как-работает-react-server-component-payload)
- [Как настроить custom cache handler в Next.js?](#как-настроить-custom-cache-handler-в-nextjs)
- [Как организовать микрофронтенды с Next.js?](#как-организовать-микрофронтенды-с-nextjs)
- [Как профилировать и дебажить производительность Next.js приложения?](#как-профилировать-и-дебажить-производительность-nextjs-приложения)

---

## Что такое Partial Prerendering (PPR)?

PPR (Next.js 14+, экспериментально) — гибридная стратегия: статическая оболочка страницы генерируется при сборке и кэшируется на CDN, динамические части (обёрнутые в Suspense) заполняются сервером при каждом запросе через streaming. Объединяет преимущества SSG (скорость доставки) и SSR (свежие данные) без компромиссов.

```typescript
// next.config.js
experimental: { ppr: true }

// page.tsx
export default function Page() {
  return (
    <>
      <StaticHeader />     {/* статично, CDN */}
      <Suspense fallback={<Skeleton />}>
        <DynamicFeed />    {/* стримится с сервера */}
      </Suspense>
    </>
  );
}
```

---

## Как работает React Server Component Payload?

RSC Payload — компактный бинарный формат (JSON-like), описывающий дерево Server Components. Содержит: rendered output (HTML-строки для Client boundaries), props для Client Components, ссылки на Client Component chunks. Клиент использует RSC Payload для: initial render, навигации (без полного HTML), обновления конкретных частей дерева без потери клиентского состояния.

При навигации Next.js запрашивает RSC Payload для нового сегмента, не перезагружая весь документ — отсюда сохранение state в незатронутых layout-ах.

---

## Как настроить custom cache handler в Next.js?

Custom cache handler позволяет хранить кэш Next.js в Redis, Memcached или любом другом хранилище вместо файловой системы. Необходим для горизонтального масштабирования (несколько инстансов). Реализуется через класс с методами `get`, `set`, `revalidateTag`.

```typescript
// cache-handler.ts
export default class CacheHandler {
  async get(key: string) {
    const data = await redis.get(key);
    return data ? JSON.parse(data) : null;
  }
  async set(key: string, data: unknown, ctx: { tags?: string[] }) {
    await redis.set(key, JSON.stringify(data));
    if (ctx.tags) await this.tagIndex(key, ctx.tags);
  }
  async revalidateTag(tag: string) {
    const keys = await this.getKeysByTag(tag);
    await redis.del(...keys);
  }
}

// next.config.js
cacheHandler: require.resolve("./cache-handler")
```

---

## Как организовать микрофронтенды с Next.js?

**Module Federation** (через `@module-federation/nextjs-mf`): каждый Next.js app экспортирует/импортирует компоненты на лету. **Multi-zone**: несколько независимых Next.js приложений под одним доменом, объединённых через `rewrites`. Multi-zone проще в поддержке, Module Federation — для sharing state/components между командами.

```javascript
// next.config.js (host app)
async rewrites() {
  return [
    { source: "/checkout/:path*", destination: "https://checkout.example.com/:path*" }
  ];
}
```

---

## Как профилировать и дебажить производительность Next.js приложения?

1. **Bundle analysis**: `ANALYZE=true next build` с `@next/bundle-analyzer`
2. **Server timing**: Next.js добавляет `Server-Timing` headers — видно в DevTools Network
3. **React DevTools Profiler**: flamegraph ре-рендеров клиентских компонентов
4. **OpenTelemetry**: `instrumentation.ts` + `registerOTel` для tracing серверных запросов
5. **`next build --debug`**: детальный вывод статической генерации
6. **Vercel Speed Insights / Web Vitals**: RUM-метрики в production

```typescript
// instrumentation.ts
export async function register() {
  if (process.env.NEXT_RUNTIME === "nodejs") {
    const { registerOTel } = await import("@vercel/otel");
    registerOTel({ serviceName: "my-app" });
  }
}
```
