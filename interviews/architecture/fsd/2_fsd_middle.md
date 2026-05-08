# Feature-Sliced Design — Middle

## Вопросы

- [Как применять FSD в реальном проекте?](#как-применять-fsd-в-реальном-проекте)
- [Как организовать shared слой?](#как-организовать-shared-слой)
- [Как работают entities в FSD?](#как-работают-entities-в-fsd)
- [Как организовать features слой?](#как-организовать-features-слой)
- [Как настроить eslint-plugin-fsd?](#как-настроить-eslint-plugin-fsd)

---

## Как применять FSD в реальном проекте?

Структура проекта:

```
src/
  app/         # providers, router, styles
  pages/       # /home, /profile, /cart
  widgets/     # Header, Sidebar, ProductCard
  features/    # auth/login, cart/add-item, profile/edit
  entities/    # user, product, order
  shared/      # ui/, api/, lib/, config/
```

Публичный API каждого slice через `index.ts` — только то, что нужно снаружи.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как организовать shared слой?

```
shared/
  ui/          # переиспользуемые UI компоненты (Button, Input, Modal)
  api/         # базовый HTTP клиент
  lib/         # утилиты, хелперы
  config/      # константы, env переменные
  types/       # общие типы
```

shared — исключение из правила слоёв: импортируется отовсюду, но сам ничего не импортирует из выше.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работают entities в FSD?

Entity — бизнес-сущность. Каждая entity: `ui/` (карточки, аватары), `model/` (store slice, selectors, types), `api/` (CRUD операции).

```
entities/user/
  ui/
    UserCard.tsx
    UserAvatar.tsx
  model/
    types.ts      # User interface
    store.ts      # Redux slice или Zustand store
    selectors.ts
  api/
    userApi.ts
  index.ts        # публичный API
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как организовать features слой?

Feature — действие пользователя: login, add-to-cart, edit-profile. Может использовать entities, но entity не использует feature.

```
features/auth/login/
  ui/        # LoginForm.tsx
  model/     # loginMutation, form schema
  api/       # loginApi.ts
  index.ts
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как настроить eslint-plugin-fsd?

```javascript
// .eslintrc
{
  "plugins": ["@feature-sliced"],
  "rules": {
    "@feature-sliced/layers-slices": "error",  // запрет импортов выше по слоям
    "@feature-sliced/public-api": "error",      // только через index.ts
  }
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
