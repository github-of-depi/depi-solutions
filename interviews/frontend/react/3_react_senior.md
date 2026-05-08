# React — Senior

## Вопросы

- [Как работает алгоритм reconciliation?](#как-работает-алгоритм-reconciliation)
- [Что такое Error Boundaries?](#что-такое-error-boundaries)
- [Как работает React Suspense?](#как-работает-react-suspense)
- [Что такое паттерн compound components?](#что-такое-паттерн-compound-components)
- [Как работает батчинг обновлений в React 18?](#как-работает-батчинг-обновлений-в-react-18)
- [Что такое forwardRef и useImperativeHandle?](#что-такое-forwardref-и-useimperativehandle)
- [Как виртуализировать длинные списки?](#как-виртуализировать-длинные-списки)
- [Что такое HOC и как его использовать?](#что-такое-hoc-и-как-его-использовать)
- [Техники оптимизации производительности React?](#техники-оптимизации-производительности-react)

---

## Как работает алгоритм reconciliation?

React сравнивает новое и предыдущее vDOM-деревья, используя эвристики O(n): элементы разного типа уничтожаются и создаются заново, элементы одного типа обновляются (атрибуты, children). `key` используется для сопоставления элементов в списках. Fiber переработал этот алгоритм: работа разбита на единицы (fiber nodes), что позволяет прерывать и возобновлять обход дерева.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое Error Boundaries?

Error Boundaries — классовые компоненты, перехватывающие ошибки JavaScript в дереве потомков и отображающие fallback UI вместо падения приложения. Реализуются через `getDerivedStateFromError` (для рендера fallback) и `componentDidCatch` (для логирования). Функциональных аналогов нет — нужен классовый компонент или библиотека `react-error-boundary`.

```typescript
class ErrorBoundary extends React.Component<Props, { hasError: boolean }> {
  state = { hasError: false };
  static getDerivedStateFromError() { return { hasError: true }; }
  componentDidCatch(error: Error, info: React.ErrorInfo) { logError(error, info); }
  render() { return this.state.hasError ? <Fallback /> : this.props.children; }
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работает React Suspense?

Suspense «приостанавливает» рендер поддерева, пока данные или код не готовы, и показывает `fallback`. Компонент сигнализирует о загрузке, бросая Promise (интеграция через `use()` или библиотеки типа TanStack Query). В React 18 Suspense интегрирован со streaming SSR: сервер может стримить готовые части страницы, не ожидая медленных данных.

```typescript
<Suspense fallback={<Spinner />}>
  <LazyComponent />   // React.lazy(() => import("./Heavy"))
  <DataComponent />   // использует use(fetchData())
</Suspense>
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое паттерн compound components?

Compound Components — паттерн, при котором несколько компонентов работают вместе, разделяя неявное состояние через Context. Даёт пользователю компонента гибкость в расстановке элементов, сохраняя инкапсулированную логику.

```typescript
function Tabs({ children }: { children: React.ReactNode }) {
  const [active, setActive] = useState(0);
  return <TabsContext.Provider value={{ active, setActive }}>{children}</TabsContext.Provider>;
}
Tabs.Tab = function Tab({ index, label }: { index: number; label: string }) {
  const { active, setActive } = useContext(TabsContext);
  return <button onClick={() => setActive(index)} aria-selected={active === index}>{label}</button>;
};
// Использование: <Tabs><Tabs.Tab index={0} label="A" /><Tabs.Tab index={1} label="B" /></Tabs>
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работает батчинг обновлений в React 18?

До React 18 батчинг работал только внутри обработчиков событий React — вызовы setState в setTimeout, Promise.then и нативных обработчиках вызывали отдельные ре-рендеры. В React 18 появился **automatic batching**: все обновления состояния объединяются в один ре-рендер независимо от контекста вызова. Отключить можно через `flushSync`.

```typescript
// React 18 — один ре-рендер вместо двух
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
}, 1000);

// Принудительный синхронный рендер
import { flushSync } from "react-dom";
flushSync(() => setCount(c => c + 1));
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое forwardRef и useImperativeHandle?

`forwardRef` позволяет дочернему компоненту принять `ref` от родителя и прокинуть его на DOM-узел или другой компонент. `useImperativeHandle` позволяет кастомизировать, что именно будет доступно через `ref` — вместо всего DOM-узла можно экспортировать только нужные методы.

```typescript
const Input = forwardRef<HTMLInputElement, Props>((props, ref) => {
  return <input ref={ref} {...props} />;
});

// useImperativeHandle
const FancyInput = forwardRef<{ focus: () => void }, Props>((props, ref) => {
  const inputRef = useRef<HTMLInputElement>(null);
  useImperativeHandle(ref, () => ({ focus: () => inputRef.current?.focus() }));
  return <input ref={inputRef} {...props} />;
});
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как виртуализировать длинные списки?

При рендере тысяч элементов DOM становится узким местом. Виртуализация рендерит только видимые элементы + небольшой буфер. Библиотеки: `@tanstack/react-virtual` (headless), `react-window`, `react-virtuoso`. Ключевые параметры: высота контейнера, высота строки (фиксированная или переменная), overscan.

```typescript
import { useVirtualizer } from "@tanstack/react-virtual";

const virtualizer = useVirtualizer({
  count: items.length,
  getScrollElement: () => parentRef.current,
  estimateSize: () => 50,
});
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое HOC и как его использовать?

HOC (Higher-Order Component) — функция, принимающая компонент и возвращающая **новый компонент** с расширенным поведением. Позволяет повторно использовать логику без дублирования.

```tsx
// HOC: защищает маршрут от неавторизованных пользователей
function withAuth<P extends {}>(WrappedComponent: React.FC<P>) {
  return function WithAuth(props: P) {
    const isAuth = useAuth();
    if (!isAuth) return <Navigate to="/login" />;
    return <WrappedComponent {...props} />;
  };
}

const ProtectedDashboard = withAuth(Dashboard);

// HOC для логгирования рендеров
function withLogger<P>(Component: React.FC<P>) {
  return function WithLogger(props: P) {
    useEffect(() => { console.log('render', Component.name, props); });
    return <Component {...props} />;
  };
}
```

**Когда использовать:** авторизация, перехват ошибок, фечеризация (feature flags), логгирование. Современная альтернатива — кастомные хуки, которые решают те же задачи чище.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**

- [React Docs: Higher-Order Components](https://legacy.reactjs.org/docs/higher-order-components.html)

---

## Техники оптимизации производительности React?

**1. Предотвращение лишних ре-рендеров:**
- `React.memo` — мемоизация компонента (поверхностное сравнение props)
- `useCallback` — стабильные ссылки на функции для `memo`-компонентов
- `useMemo` — мемоизация дорогостоящих вычислений

**2. Раздробление кода (ленивая загрузка):**
- `React.lazy` + `Suspense` — динамический импорт компонентов
- раздробление bundle по маршрутам

**3. Виртуализация списков:**
- `react-window`, `@tanstack/virtual` — рендеринг только видимых элементов

**4. Правильная структура состояния:**
- Колокация состояния вниз (не пропсы с объектами-значениями)
- Неизменяемые обновления (вместо мутации)

```tsx
// Пример: React.lazy + Suspense
const AdminPanel = React.lazy(() => import('./AdminPanel'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <AdminPanel />
    </Suspense>
  );
}

// Пример: мемоизация + useCallback
const Item = React.memo(({ label, onClick }) => <button onClick={onClick}>{label}</button>);

function Parent() {
  const handleClick = useCallback(() => {/* ... */}, []); // стабильная ссылка
  return <Item label="test" onClick={handleClick} />;
}
```

**Связанные задачи:**

- [Оптимизация ре-рендеров: memo + useCallback](../../../tasks/frontend/react/2_react_middle.md#оптимизация-ре-рендеров-memo--usecallback)

**Материалы для изучения:**

- [React Docs: Производительность](https://react.dev/learn/render-and-commit)
