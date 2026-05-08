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
