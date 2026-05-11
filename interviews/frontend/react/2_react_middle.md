# React — Middle

## Вопросы

- [Как работает useState внутри React?](#как-работает-usestate-внутри-react)
- [Как работает useEffect — зависимости и cleanup?](#как-работает-useeffect--зависимости-и-cleanup)
- [В чём разница между useMemo и useCallback?](#в-чём-разница-между-usememo-и-usecallback)
- [Что такое Context API и когда его использовать?](#что-такое-context-api-и-когда-его-использовать)
- [Что такое порталы и когда они нужны?](#что-такое-порталы-и-когда-они-нужны)
- [Controlled vs uncontrolled компоненты?](#controlled-vs-uncontrolled-компоненты)
- [Как избежать лишних ре-рендеров?](#как-избежать-лишних-ре-рендеров)
- [Что такое React.memo?](#что-такое-reactmemo)
- [Чем useLayoutEffect отличается от useEffect?](#чем-uselayouteffect-отличается-от-useeffect)
- [Что такое useReducer и когда его использовать?](#что-такое-usereducer-и-когда-его-использовать)
- [Методы жизненного цикла компонента?](#методы-жизненного-цикла-компонента)
- [Что такое PureComponent?](#что-такое-purecomponent)
- [Анимация в React?](#анимация-в-react)
- [Flux архитектура?](#flux-архитектура)
- [Отладка React и линтеры?](#отладка-react-и-линтеры)
- [Ограничения React?](#ограничения-react)
- [Refs в React?](#refs-в-react)
- [Маршрутизация в React?](#маршрутизация-в-react)
- [Преимущества React?](#преимущества-react)

---

## Как работает useState внутри React?

`useState` хранит состояние в связном списке «ячеек» внутри Fiber-узла компонента. При каждом рендере хуки вызываются в том же порядке — поэтому нельзя использовать хуки внутри условий. Сеттер принимает новое значение или updater-функцию `prev => next`, которая гарантирует актуальный предыдущий state.

```typescript
const [count, setCount] = useState(0);
// Updater — безопасно при батчинге и async обновлениях
setCount(prev => prev + 1);
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работает useEffect — зависимости и cleanup?

`useEffect` запускается после рендера (браузером отрисован DOM). Массив зависимостей определяет, когда эффект перезапускается: `[]` — только при монтировании, `[dep]` — при изменении dep, без массива — на каждый рендер. Возвращаемая функция — cleanup, вызывается перед следующим запуском или размонтированием.

```typescript
useEffect(() => {
  const handler = () => console.log("resize");
  window.addEventListener("resize", handler);
  return () => window.removeEventListener("resize", handler); // cleanup
}, []); // только при монтировании
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## В чём разница между useMemo и useCallback?

`useMemo` мемоизирует **результат** вычисления, `useCallback` — **саму функцию**. Оба пересчитываются только при изменении зависимостей. Используй их только когда есть реальная проблема производительности — преждевременная оптимизация усложняет код.

```typescript
const filtered = useMemo(() => items.filter(i => i.active), [items]);
const handleClick = useCallback(() => setCount(c => c + 1), []);
// useCallback(fn, deps) === useMemo(() => fn, deps)
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое Context API и когда его использовать?

Context позволяет передавать данные вниз по дереву без явной передачи через props (props drilling). Подходит для: темы, локали, данных текущего пользователя. Не заменяет стейт-менеджер при частых обновлениях — каждое изменение context перерендеривает всех потребителей.

```typescript
const ThemeContext = createContext<"light" | "dark">("light");
// Provider
<ThemeContext.Provider value="dark"><App /></ThemeContext.Provider>
// Consumer
const theme = useContext(ThemeContext);
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое порталы и когда они нужны?

Порталы позволяют рендерить дочерний компонент в произвольный DOM-узел за пределами родительского дерева. Используются для: модальных окон, тултипов, дропдаунов — там, где нужно выйти за пределы `overflow: hidden` или `z-index` родителя.

```typescript
import { createPortal } from "react-dom";

function Modal({ children }: { children: React.ReactNode }) {
  return createPortal(children, document.getElementById("modal-root")!);
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Controlled vs uncontrolled компоненты?

**Controlled** — значение поля управляется React state, React является единственным источником правды. **Uncontrolled** — значение хранится в DOM, читается через `ref`. Controlled — стандарт для форм (легче валидировать, сбрасывать, тестировать). Uncontrolled — для интеграции со сторонними библиотеками.

```typescript
// Controlled
const [value, setValue] = useState("");
<input value={value} onChange={e => setValue(e.target.value)} />

// Uncontrolled
const ref = useRef<HTMLInputElement>(null);
<input ref={ref} defaultValue="initial" />
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как избежать лишних ре-рендеров?

1. `React.memo` — мемоизация компонента по props
2. `useMemo` / `useCallback` — мемоизация значений и функций
3. Разбивать state на независимые части
4. Выносить стабильные данные за пределы компонента
5. Использовать `useContext` только в компонентах, которым реально нужны данные

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое React.memo?

`React.memo` — HOC, который мемоизирует функциональный компонент: пропускает ре-рендер, если props не изменились (поверхностное сравнение). Полезен для компонентов, которые часто получают одни и те же props от часто обновляемого родителя. Не панацея — сравнение тоже имеет стоимость.

```typescript
const Item = React.memo(({ label, onClick }: Props) => {
  return <button onClick={onClick}>{label}</button>;
});
// onClick нужно обернуть в useCallback, иначе memo бесполезен
```

**Связанные задачи:**

- [Оптимизация ре-рендеров: memo + useCallback](../../../tasks/frontend/react/2_react_middle.md#оптимизация-ре-рендеров-memo--usecallback)

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Чем useLayoutEffect отличается от useEffect?

Оба хука запускают побочные эффекты, но в разное время относительно отрисовки:

| | `useEffect` | `useLayoutEffect` |
|---|---|---|
| Когда | После paint (асинхронно) | После DOM-мутаций, до paint (синхронно) |
| Блокирует рендер | Нет | Да |
| Применение | Запросы, подписки, логирование | Измерение DOM, синхронная анимация |

```tsx
// useEffect — не блокирует, пользователь видит промежуточный рендер
useEffect(() => {
  setHeight(ref.current.offsetHeight); // может вызвать видимый "прыжок"
}, []);

// useLayoutEffect — синхронно, до отрисовки, нет мерцания
useLayoutEffect(() => {
  setHeight(ref.current.offsetHeight); // высота установлена до paint
}, []);
```

**Правило:** используй `useEffect` по умолчанию. `useLayoutEffect` — только когда нужно измерить DOM или синхронно предотвратить мерцание перед отрисовкой.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**

- [React Docs: useLayoutEffect](https://react.dev/reference/react/useLayoutEffect)

---

## Что такое useReducer и когда его использовать?

`useReducer` — хук для управления состоянием через чистую функцию-редюсер: `(state, action) => newState`. Аналог Redux-паттерна на уровне компонента.

```tsx
type State = { count: number; step: number };
type Action =
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'setStep'; payload: number };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment': return { ...state, count: state.count + state.step };
    case 'decrement': return { ...state, count: state.count - state.step };
    case 'setStep':   return { ...state, step: action.payload };
    default: return state;
  }
}

const [state, dispatch] = useReducer(reducer, { count: 0, step: 1 });

dispatch({ type: 'increment' });
dispatch({ type: 'setStep', payload: 5 });
```

**Когда предпочесть `useReducer` перед `useState`:**
- Несколько связанных полей состояния, обновляемых вместе
- Сложная логика переходов состояния
- Следующее состояние зависит от предыдущего
- Нужно вынести логику состояния наружу для тестирования

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**

- [React Docs: useReducer](https://react.dev/reference/react/useReducer)

---

## Методы жизненного цикла компонента?

В **классовых компонентах** — методы жизненного цикла. В **функциональных** — имитируются через `useEffect`.

```
Монтирование:     constructor → render → componentDidMount
Обновление:       render → componentDidUpdate
Размонтирование:  componentWillUnmount
Ошибка:           getDerivedStateFromError / componentDidCatch
```

```jsx
// Классовый компонент
class Timer extends React.Component {
  componentDidMount()    { /* как useEffect(fn, []) */ }
  componentDidUpdate(prevProps, prevState) { /* useEffect(fn, [deps]) */ }
  componentWillUnmount() { /* return () => cleanup в useEffect */ }
}

// Функциональный эквивалент
function Timer() {
  useEffect(() => {
    // componentDidMount
    const id = setInterval(tick, 1000);
    return () => clearInterval(id); // componentWillUnmount
  }, []); // [] = только при монтировании
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое PureComponent?

`React.PureComponent` — классовый компонент, реализующий `shouldComponentUpdate` с **поверхностным сравнением** (shallow compare) props и state. Пропускает ре-рендер, если значения не изменились.

```jsx
// Устаревший способ (классовый)
class MyComponent extends React.PureComponent {
  render() { return <div>{this.props.name}</div>; }
}

// Современный эквивалент (функциональный)
const MyComponent = React.memo(({ name }) => <div>{name}</div>);
```

**Ограничение:** поверхностное сравнение — одинаковые объекты по ссылке считаются равными. Мутации объектов не будут замечены.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Анимация в React?

**CSS-переходы и анимации (простейший способ):**
```css
.fade-enter { opacity: 0; }
.fade-enter-active { opacity: 1; transition: opacity 300ms; }
```

**React Transition Group** — управляет классами при монтировании/размонтировании:
```jsx
import { CSSTransition } from 'react-transition-group';
<CSSTransition in={show} timeout={300} classNames="fade" unmountOnExit>
  <div>Контент</div>
</CSSTransition>
```

**Framer Motion** — декларативные анимации:
```jsx
import { motion } from 'framer-motion';
<motion.div
  initial={{ opacity: 0, y: -20 }}
  animate={{ opacity: 1, y: 0 }}
  exit={{ opacity: 0 }}
  transition={{ duration: 0.3 }}
>
  Контент
</motion.div>
```

**GSAP / Anime.js** — для сложных анимационных последовательностей через `useRef` + `useEffect`.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Flux архитектура?

**Flux** — архитектурный паттерн, разработанный Facebook для управления состоянием в React-приложениях. Однонаправленный поток данных.

**4 сущности:**
```
Action → Dispatcher → Store → View
  ↑________________________________|
```

- **Action** — объект с типом и данными: `{ type: 'ADD_TODO', payload: text }`
- **Dispatcher** — центральный хаб, рассылает action всем store
- **Store** — содержит состояние и логику обработки действий
- **View** — React-компоненты, подписанные на store

**Сегодня:** оригинальный Flux почти не используется. Идеи реализованы в **Redux** (единый store + чистые reducers) и **Zustand**.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Отладка React и линтеры?

**React DevTools** (расширение Chrome/Firefox):
- Инспектор компонентов: props, state, hooks
- Profiler: какие компоненты ре-рендерятся и сколько времени
- Подсветка ре-рендеров

**Линтеры:**
- **ESLint** + `eslint-plugin-react-hooks` — правила хуков:
  - Нельзя вызывать хуки внутри условий/циклов
  - `exhaustive-deps` — предупреждение о пропущенных зависимостях `useEffect`
- **eslint-plugin-react** — best practices компонентов
- **TypeScript** — типовая безопасность props

**Отладка в коде:**
```jsx
// Логирование рендеров
console.log('render', props);

// React.StrictMode — двойной вызов рендера в dev для поиска нечистых компонентов
<React.StrictMode><App /></React.StrictMode>

// why-did-you-render — библиотека для отслеживания лишних ре-рендеров
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Ограничения React?

1. **Только UI** — не full framework (нет routing, DI, HTTP-клиента из коробки)
2. **JSX** — нужна сборка (Babel/SWC/esbuild), нельзя в браузере без подготовки
3. **Производительность** — частые ре-рендеры без оптимизаций (memo, useMemo)
4. **Размер бандла** — react + react-dom ≈ 42kb gzip; Angular включает больше из коробки
5. **setState асинхронный** — нельзя читать новое состояние сразу после вызова
6. **Boilerplate** — управление состоянием, роутинг, data-fetching требуют отдельных решений
7. **Скорость обучения** — hooks, JSX, reconciliation, Fiber — кривая обучения для новичков
8. **Нет SSR из коробки** — нужен Next.js / Remix

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Refs в React?

**useRef** возвращает изменяемый объект `{ current: ... }`, который сохраняется между рендерами и **не вызывает ре-рендер** при изменении.

**Два применения:**
```jsx
// 1. Доступ к DOM-узлу
function FocusInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  const focus = () => inputRef.current?.focus();

  return (
    <>
      <input ref={inputRef} />
      <button onClick={focus}>Фокус</button>
    </>
  );
}

// 2. Хранение значения между рендерами без ре-рендера
function Timer() {
  const timerId = useRef<number>(null);

  useEffect(() => {
    timerId.current = setInterval(tick, 1000);
    return () => clearInterval(timerId.current!);
  }, []);
}
```

`forwardRef` — пробросить ref через компонент к DOM-элементу внутри него.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Маршрутизация в React?

React не имеет встроенного роутинга. Стандартный вариант — **React Router** (v6+).

```jsx
import { BrowserRouter, Routes, Route, Link, useNavigate, useParams } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Главная</Link>
        <Link to="/users">Пользователи</Link>
      </nav>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/users" element={<Users />} />
        <Route path="/users/:id" element={<UserDetail />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}

function UserDetail() {
  const { id } = useParams();
  const navigate = useNavigate();
  // ...
}
```

**Альтернативы:** TanStack Router (типобезопасный), Next.js App Router (file-based).

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Преимущества React?

1. **Virtual DOM** — эффективное обновление только изменившихся частей UI
2. **Компонентный подход** — переиспользование, изоляция, простота тестирования
3. **Декларативность** — описываешь *что*, не *как*
4. **Большая экосистема** — Redux, React Router, TanStack Query, Framer Motion и тысячи других
5. **JSX** — HTML + JS в одном месте, удобно и типобезопасно с TypeScript
6. **React Native** — тот же код для iOS и Android
7. **Server Components** (React 19) — компоненты на сервере без JS в бандле
8. **Большое комьюнити** — огромная база знаний, решений, специалистов

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Чем useLayoutEffect отличается от useEffect?

Оба хука запускают побочные эффекты, но в разное время относительно отрисовки:

| | `useEffect` | `useLayoutEffect` |
|---|---|---|
| Когда | После paint (асинхронно) | После DOM-мутаций, до paint (синхронно) |
| Блокирует рендер | Нет | Да |
| Применение | Запросы, подписки, логирование | Измерение DOM, синхронная анимация |

```tsx
// useEffect — не блокирует, пользователь видит промежуточный рендер
useEffect(() => {
  setHeight(ref.current.offsetHeight); // может вызвать видимый "прыжок"
}, []);

// useLayoutEffect — синхронно, до отрисовки, нет мерцания
useLayoutEffect(() => {
  setHeight(ref.current.offsetHeight); // высота установлена до paint
}, []);
```

**Правило:** используй `useEffect` по умолчанию. `useLayoutEffect` — только когда нужно измерить DOM или синхронно предотвратить мерцание перед отрисовкой.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**

- [React Docs: useLayoutEffect](https://react.dev/reference/react/useLayoutEffect)

---

## Что такое useReducer и когда его использовать?

`useReducer` — хук для управления состоянием через чистую функцию-редюсер: `(state, action) => newState`. Аналог Redux-паттерна на уровне компонента.

```tsx
type State = { count: number; step: number };
type Action =
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'setStep'; payload: number };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment': return { ...state, count: state.count + state.step };
    case 'decrement': return { ...state, count: state.count - state.step };
    case 'setStep':   return { ...state, step: action.payload };
    default: return state;
  }
}

const [state, dispatch] = useReducer(reducer, { count: 0, step: 1 });

dispatch({ type: 'increment' });
dispatch({ type: 'setStep', payload: 5 });
```

**Когда предпочесть `useReducer` перед `useState`:**
- Несколько связанных полей состояния, обновляемых вместе
- Сложная логика переходов состояния
- Следующее состояние зависит от предыдущего
- Нужно вынести логику состояния наружу для тестирования

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**

- [React Docs: useReducer](https://react.dev/reference/react/useReducer)
