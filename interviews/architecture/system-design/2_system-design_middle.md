# System Design (Frontend) — Middle

## Вопросы

- [Как спроектировать ленту (Feed) в соцсети?](#как-спроектировать-ленту-feed-в-соцсети)
- [Как проектировать систему уведомлений?](#как-проектировать-систему-уведомлений)
- [Как строить autocomplete?](#как-строить-autocomplete)
- [Как проектировать offline-capable приложение?](#как-проектировать-offline-capable-приложение)

---

## Как спроектировать ленту (Feed) в соцсети?

Компоненты решения:
- **API**: пагинация (cursor-based), prefetching следующей страницы
- **Кэш**: TanStack Query, staleTime для фоновых обновлений
- **Рендеринг**: виртуализация списка (react-virtual) — тысячи постов
- **Real-time**: WebSocket/SSE для новых постов — badge "Показать N новых"
- **Медиа**: lazy loading изображений, адаптивные размеры через CDN

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как проектировать систему уведомлений?

- **Transport**: WebSocket (push) или polling (простой)
- **State**: TanStack Query + optimistic read marking
- **Storage**: последние N уведомлений в памяти, история — pagination
- **UX**: bell icon с badge, dropdown, notification center page
- **Push**: Service Worker + Web Push API для браузерных уведомлений

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как строить autocomplete?

- **Debounce**: 200-300ms после ввода
- **Cancellation**: AbortController — отменять предыдущий запрос
- **Кэш**: запомнить последние N запросов (Map)
- **Клавиатура**: ArrowUp/Down/Enter/Escape — a11y
- **Highlight**: подсветка введённой части в результатах

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как проектировать offline-capable приложение?

- **Service Worker**: cache-first для статики, network-first для API
- **Offline detection**: `navigator.onLine` + `online`/`offline` events
- **Background sync**: Service Worker Background Sync API — ставить запросы в очередь
- **Optimistic UI**: мгновенные обновления, синхронизация при онлайне
- **IndexedDB**: хранение данных для offline работы

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
