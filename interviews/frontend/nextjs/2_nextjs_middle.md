# Next.js — Middle

## Вопросы

- [В чём разница между getStaticProps, getServerSideProps и ISR?](#в-чём-разница-между-getstaticprops-getserversideprops-и-isr)
- [Что такое API Routes в Next.js?](#что-такое-api-routes-в-nextjs)
- [Как работает middleware в Next.js?](#как-работает-middleware-в-nextjs)
- [Что такое dynamic imports в Next.js?](#что-такое-dynamic-imports-в-nextjs)
- [Как обрабатывать ошибки в App Router?](#как-обрабатывать-ошибки-в-app-router)
- [Что такое Metadata API в Next.js?](#что-такое-metadata-api-в-nextjs)
- [Как работает кэширование fetch в Next.js?](#как-работает-кэширование-fetch-в-nextjs)

---

## В чём разница между getStaticProps, getServerSideProps и ISR?

Все три — Pages Router API. `getStaticProps` — данные при сборке (SSG). `getServerSideProps` — данные при каждом запросе (SSR). ISR: `getStaticProps` + `revalidate` — страница регенерируется в фоне по истечении времени. App Router заменяет их расширенным `fetch` с опциями `cache` и `next.revalidate`.

```typescript
// ISR — обновлять каждые 60 секунд
export async function getStaticProps() {
  const data = await fetchData();
  return { props: { data }, revalidate: 60 };
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое API Routes в Next.js?

API Routes — серверные эндпоинты внутри Next.js приложения. Pages Router: файлы в `pages/api/`. App Router: файлы `route.ts` с экспортом именованных функций GET, POST и т.д.

```typescript
// app/api/users/route.ts
export async function GET() {
  const users = await db.user.findMany();
  return Response.json(users);
}

export async function POST(request: Request) {
  const body = await request.json();
  const user = await db.user.create({ data: body });
  return Response.json(user, { status: 201 });
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работает middleware в Next.js?

Middleware (`middleware.ts` в корне) запускается на edge runtime перед каждым запросом: аутентификация, редиректы, A/B тесты, геолокация. Конфигурируется через `config.matcher` — паттерн маршрутов.

```typescript
import { NextRequest, NextResponse } from "next/server";
export function middleware(request: NextRequest) {
  const token = request.cookies.get("token");
  if (!token && request.nextUrl.pathname.startsWith("/dashboard")) {
    return NextResponse.redirect(new URL("/login", request.url));
  }
  return NextResponse.next();
}
export const config = { matcher: ["/dashboard/:path*"] };
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое dynamic imports в Next.js?

`next/dynamic` — обёртка над `React.lazy` с SSR-поддержкой. Позволяет загружать компонент только при необходимости, уменьшая initial bundle. `ssr: false` — загружать только на клиенте (для компонентов с browser-only API).

```typescript
import dynamic from "next/dynamic";
const HeavyChart = dynamic(() => import("./HeavyChart"), {
  loading: () => <Skeleton />,
  ssr: false,
});
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как обрабатывать ошибки в App Router?

`error.tsx` — автоматически создаёт Error Boundary для сегмента. Получает `error` и `reset` (retry). `global-error.tsx` — обрабатывает ошибки в корневом layout. Обязательно `"use client"` — Error Boundaries только клиентские.

```typescript
// app/dashboard/error.tsx
"use client";
export default function Error({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <div>
      <p>Что-то пошло не так: {error.message}</p>
      <button onClick={reset}>Попробовать снова</button>
    </div>
  );
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое Metadata API в Next.js?

Metadata API — встроенный способ управления `<head>` в App Router. Экспорт объекта `metadata` (статический) или функции `generateMetadata` (динамический) из `page.tsx` или `layout.tsx`.

```typescript
// Статические метаданные
export const metadata = { title: "Главная", description: "Описание" };

// Динамические
export async function generateMetadata({ params }: { params: { id: string } }) {
  const product = await fetchProduct(params.id);
  return { title: product.name, openGraph: { images: [product.image] } };
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работает кэширование fetch в Next.js?

Next.js расширяет встроенный `fetch` опциями кэширования. `force-cache` (по умолчанию в Server Components) — кэшировать навсегда. `no-store` — без кэша (аналог SSR). `next.revalidate` — ISR-поведение. `next.tags` — тегированный кэш для инвалидации через `revalidateTag`.

```typescript
// Кэш навсегда (SSG)
fetch(url, { cache: "force-cache" });
// Без кэша (SSR)
fetch(url, { cache: "no-store" });
// ISR — обновлять каждые 60с
fetch(url, { next: { revalidate: 60 } });
// Тегированный кэш
fetch(url, { next: { tags: ["products"] } });
// Инвалидация в Server Action
import { revalidateTag } from "next/cache";
revalidateTag("products");
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
