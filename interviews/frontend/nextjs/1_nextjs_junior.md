# Next.js — Junior

## Вопросы

- [Что такое Next.js и чем он отличается от React?](#что-такое-nextjs-и-чем-он-отличается-от-react)
- [Что такое SSR, SSG и CSR?](#что-такое-ssr-ssg-и-csr)
- [Как работает файловая система роутинга в Next.js?](#как-работает-файловая-система-роутинга-в-nextjs)
- [Что такое компонент Link в Next.js?](#что-такое-компонент-link-в-nextjs)
- [Как подключить переменные окружения в Next.js?](#как-подключить-переменные-окружения-в-nextjs)
- [Что такое компонент Image в Next.js?](#что-такое-компонент-image-в-nextjs)

---

## Что такое Next.js и чем он отличается от React?

Next.js — фреймворк на основе React с встроенными возможностями: серверный рендеринг (SSR/SSG), файловая маршрутизация, оптимизация изображений, API routes. React — библиотека только для UI, Next.js решает задачи production-приложения: SEO, производительность, роутинг.

---

## Что такое SSR, SSG и CSR?

- **CSR** (Client-Side Rendering) — HTML пустой, данные и рендер на клиенте. Плохо для SEO.
- **SSR** (Server-Side Rendering) — HTML генерируется на сервере при каждом запросе. Свежие данные, нагрузка на сервер.
- **SSG** (Static Site Generation) — HTML генерируется при сборке. Максимальная скорость, подходит для статичного контента.
- **ISR** — SSG с периодическим обновлением без пересборки.

---

## Как работает файловая система роутинга в Next.js?

App Router (Next.js 13+): структура папок в `app/` определяет маршруты. `page.tsx` — страница, `layout.tsx` — обёртка, `loading.tsx` — Suspense fallback, `error.tsx` — Error Boundary, `route.ts` — API handler.

```
app/
  layout.tsx          → корневой layout
  page.tsx            → /
  users/
    page.tsx          → /users
    [id]/
      page.tsx        → /users/123
```

---

## Что такое компонент Link в Next.js?

`Link` — замена `<a>` с клиентской навигацией (без перезагрузки страницы). Автоматически prefetch-ит страницы, видимые в viewport (в production). Сохраняет состояние React между переходами.

```typescript
import Link from "next/link";
<Link href="/about">О нас</Link>
<Link href={`/users/${id}`}>Профиль</Link>
```

---

## Как подключить переменные окружения в Next.js?

Файлы: `.env.local` (не в git), `.env`, `.env.production`. Переменные с префиксом `NEXT_PUBLIC_` доступны в браузере, без префикса — только на сервере.

```bash
# .env.local
DATABASE_URL=postgres://...        # только сервер
NEXT_PUBLIC_API_URL=https://api.example.com  # доступно в браузере
```

---

## Что такое компонент Image в Next.js?

`Image` — оптимизированный `<img>`: автоматический WebP/AVIF, lazy loading, предотвращение CLS (обязательный width/height или fill). Размер изображения оптимизируется под устройство через srcset.

```typescript
import Image from "next/image";
<Image src="/hero.jpg" alt="Hero" width={1200} height={600} priority />
```
