# Next.js — Senior

## Вопросы

- [Что такое Server Components и как они отличаются от Client Components?](#что-такое-server-components-и-как-они-отличаются-от-client-components)
- [Как работает streaming в Next.js App Router?](#как-работает-streaming-в-nextjs-app-router)
- [Что такое Server Actions?](#что-такое-server-actions)
- [Как работают parallel routes и intercepting routes?](#как-работают-parallel-routes-и-intercepting-routes)
- [Как оптимизировать производительность Next.js приложения?](#как-оптимизировать-производительность-nextjs-приложения)
- [Как организовать аутентификацию в App Router?](#как-организовать-аутентификацию-в-app-router)

---

## Что такое Server Components и как они отличаются от Client Components?

**Server Components** (по умолчанию в App Router) — выполняются только на сервере, нет доступа к state/effects/браузерным API, но есть прямой доступ к БД/FS/секретам. Не попадают в JS-бандл клиента. **Client Components** (`"use client"`) — выполняются на клиенте и на сервере (hydration), имеют интерактивность.

Правило: Client Components не могут напрямую использовать Server Components как children — только через props/slot паттерн.

```typescript
// Server Component — прямой доступ к БД
async function UserList() {
  const users = await db.user.findMany(); // нет useEffect!
  return <ul>{users.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

---

## Как работает streaming в Next.js App Router?

Streaming — HTML отправляется клиенту по частям (HTTP chunked transfer). Медленные Server Components оборачиваются в `Suspense`: сервер стримит готовые части сразу, заменяя Suspense fallback содержимым по готовности. Улучшает TTFB и воспринимаемую скорость загрузки.

```typescript
// app/page.tsx
export default function Page() {
  return (
    <>
      <Header />                  {/* отправляется сразу */}
      <Suspense fallback={<Skeleton />}>
        <SlowDataComponent />     {/* стримится когда готов */}
      </Suspense>
    </>
  );
}
```

---

## Что такое Server Actions?

Server Actions — функции, выполняемые на сервере, вызываемые из клиентского кода. Помечаются директивой `"use server"`. Заменяют API Routes для мутаций: submit форм, обновление данных. Интегрируются с `useFormState`, `useFormStatus` и React `form` action.

```typescript
// actions.ts
"use server";
export async function createUser(formData: FormData) {
  const name = formData.get("name") as string;
  await db.user.create({ data: { name } });
  revalidatePath("/users");
}

// Component
<form action={createUser}>
  <input name="name" />
  <button type="submit">Создать</button>
</form>
```

---

## Как работают parallel routes и intercepting routes?

**Parallel routes** (`@slot`) — одновременный рендер нескольких страниц в одном layout (например, основной контент + модальное окно). **Intercepting routes** (`(.)`, `(..)`, `(...)`) — перехват маршрута для показа контента в текущем контексте (photo gallery в Instagram стиле: клик открывает модал, но прямая ссылка — полную страницу).

```
app/
  layout.tsx           → получает @modal как prop
  @modal/
    (.)photos/[id]/
      page.tsx         → модальное окно
  photos/[id]/
    page.tsx           → полная страница фото
```

---

## Как оптимизировать производительность Next.js приложения?

1. Максимально использовать Server Components — меньше JS на клиенте
2. `next/dynamic` с `ssr: false` для тяжёлых клиентских компонентов
3. `next/image` с правильными размерами и `priority` для LCP-изображений
4. Стратегия кэширования fetch — `force-cache` + `revalidate` вместо `no-store` где возможно
5. Partial Prerendering (PPR, экспериментально) — статическая оболочка + динамические острова
6. Bundle analyzer: `@next/bundle-analyzer`

---

## Как организовать аутентификацию в App Router?

Рекомендуемый подход: **NextAuth.js v5 (Auth.js)** — официальная интеграция с App Router. Middleware проверяет сессию на edge runtime до рендера страницы.

```typescript
// middleware.ts
export { auth as middleware } from "./auth";
export const config = { matcher: ["/((?!api|_next|.*\\..*).*)"] };

// Серверный компонент
import { auth } from "./auth";
export default async function Dashboard() {
  const session = await auth();
  if (!session) redirect("/login");
  return <div>Привет, {session.user.name}</div>;
}
```
