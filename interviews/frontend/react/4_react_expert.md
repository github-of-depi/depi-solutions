# React — Expert

## Вопросы

- [Что такое Fiber архитектура и как она работает?](#что-такое-fiber-архитектура-и-как-она-работает)
- [Что такое Concurrent Mode и React Transitions?](#что-такое-concurrent-mode-и-react-transitions)
- [Что такое React Server Components?](#что-такое-react-server-components)
- [Как реализовать кастомный React-рендерер?](#как-реализовать-кастомный-react-рендерер)
- [Как работает React Scheduler?](#как-работает-react-scheduler)

---

## Что такое Fiber архитектура и как она работает?

Fiber (React 16+) заменил рекурсивный алгоритм reconciliation на итеративный с возможностью прерывания. Каждый компонент — Fiber-узел: объект со ссылками `child`, `sibling`, `return` (связный список), полями `lanes` (приоритет), `memoizedState` (хуки), `pendingProps`/`memoizedProps`.

Работа разделена на две фазы:
1. **Render (прерываемая)** — обход дерева, вычисление diff, построение «work in progress» дерева
2. **Commit (синхронная)** — применение изменений к DOM в три под-фазы: before mutation, mutation, layout

React хранит два дерева: current (текущее) и work-in-progress (новое). После commit они меняются местами.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое Concurrent Mode и React Transitions?

Concurrent Mode позволяет React прерывать низкоприоритетный рендер в пользу срочных обновлений (ввод пользователя, анимации). Реализован через Fiber Scheduler с приоритетами (lanes).

`startTransition` / `useTransition` помечают обновление как некритическое — React может отложить его, сохраняя отзывчивость UI. `useDeferredValue` аналогичен для отдельных значений.

```typescript
const [isPending, startTransition] = useTransition();
function handleSearch(query: string) {
  setInputValue(query); // немедленно (высокий приоритет)
  startTransition(() => {
    setResults(computeResults(query)); // отложено (низкий приоритет)
  });
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое React Server Components?

RSC — компоненты, выполняемые только на сервере: нет доступа к state/effects/браузерным API, но есть прямой доступ к БД, файловой системе, секретам. Результат — сериализованное дерево (React Server Component Payload), которое клиент может получить без JS-бандла компонента. Клиентские компоненты помечаются директивой `"use client"`.

RSC снижают размер JS-бандла, устраняют waterfalls (данные прямо на сервере) и позволяют streaming рендеринг (Suspense + RSC). Лежат в основе Next.js App Router.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как реализовать кастомный React-рендерер?

Кастомный рендерер реализуется через пакет `react-reconciler`. Нужно предоставить `hostConfig` — объект с методами для работы с целевой платформой: `createInstance`, `appendChild`, `commitUpdate`, `removeChild` и другими.

```typescript
import Reconciler from "react-reconciler";

const hostConfig = {
  createInstance(type: string, props: Record<string, unknown>) {
    // создаём узел целевой платформы
    return { type, props, children: [] };
  },
  appendChildToContainer(container: unknown, child: unknown) {
    // добавляем в корень
  },
  commitUpdate(instance: unknown, updatePayload: unknown, type: string, oldProps: unknown, newProps: unknown) {
    // применяем изменения
  },
  // ... ~20 других методов
  supportsMutation: true,
  isPrimaryRenderer: true,
};

const renderer = Reconciler(hostConfig);
```

Именно так устроены `react-native`, `react-three-fiber`, `ink` (React для терминала).

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работает React Scheduler?

React Scheduler (`scheduler` package) управляет приоритетом и порядком выполнения работы. Использует `MessageChannel` для yield (не setTimeout — слишком медленно) и делит работу на 5-мс временные слоты, уступая управление браузеру между ними.

Приоритеты (lanes): ImmediatePriority (синхронно), UserBlockingPriority (100мс), NormalPriority (250мс), LowPriority, IdlePriority. `startTransition` переводит обновление в NormalPriority, давая Scheduler возможность отложить его в пользу UserBlocking обновлений.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
