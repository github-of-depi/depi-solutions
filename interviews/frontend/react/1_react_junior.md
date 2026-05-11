# React — Junior

## Вопросы

- [Что такое React и какие проблемы он решает?](#что-такое-react-и-какие-проблемы-он-решает)
- [В чём разница между state и props?](#в-чём-разница-между-state-и-props)
- [Что такое JSX и как он компилируется?](#что-такое-jsx-и-как-он-компилируется)
- [Что такое виртуальный DOM?](#что-такое-виртуальный-dom)
- [Чем функциональные компоненты отличаются от классовых?](#чем-функциональные-компоненты-отличаются-от-классовых)
- [Что такое хуки и зачем они появились?](#что-такое-хуки-и-зачем-они-появились)
- [Для чего нужен prop key в списках?](#для-чего-нужен-prop-key-в-списках)
- [Что такое Fragment и чем он лучше div-обёртки?](#что-такое-fragment-и-чем-он-лучше-div-обёртки)
- [Что такое prop drilling и как его избежать?](#что-такое-prop-drilling-и-как-его-избежать)
- [Способы условного рендеринга в React?](#способы-условного-рендеринга-в-react)
- [Разница между элементом и компонентом?](#разница-между-элементом-и-компонентом)
- [Как создать компонент в React?](#как-создать-компонент-в-react)
- [Что такое prop children?](#что-такое-prop-children)
- [Почему нельзя обновлять state напрямую?](#почему-нельзя-обновлять-state-напрямую)
- [Что происходит при вызове setState?](#что-происходит-при-вызове-setstate)
- [Чистая функция в React?](#чистая-функция-в-react)
- [createElement и cloneElement?](#createelement-и-cloneelement)
- [Синтетические события?](#синтетические-события)
- [HTML vs React?](#html-vs-react)

---

## Что такое React и какие проблемы он решает?

React — JavaScript-библиотека для построения пользовательских интерфейсов. Решает три ключевые проблемы: сложность работы с DOM (виртуальный DOM для батчинга обновлений), переиспользование кода (компонентная архитектура) и предсказуемость данных (однонаправленный поток данных).

```typescript
function Greeting({ name }: { name: string }) {
  return <h1>Привет, {name}!</h1>;
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## В чём разница между state и props?

**Props** — входные данные от родителя, доступны только для чтения. **State** — внутреннее изменяемое состояние компонента, при изменении вызывает ре-рендер. Props текут сверху вниз, state локален и управляется внутри компонента.

```typescript
function Counter({ initialCount }: { initialCount: number }) {
  const [count, setCount] = useState(initialCount); // state
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
  // initialCount — props, count — state
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое JSX и как он компилируется?

JSX — синтаксическое расширение JavaScript, похожее на HTML. Компилируется Babel/SWC в вызовы `React.createElement()` или в `jsx()` из `react/jsx-runtime` (новый JSX transform, React 17+). JSX не является обязательным, но делает код читаемее.

```typescript
// JSX
const el = <h1 className="title">Привет</h1>;

// После компиляции (new JSX transform)
import { jsx } from "react/jsx-runtime";
const el = jsx("h1", { className: "title", children: "Привет" });
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое виртуальный DOM?

Виртуальный DOM — лёгкое JavaScript-представление реального DOM-дерева. При изменении состояния React создаёт новое vDOM-дерево, сравнивает с предыдущим (diffing/reconciliation) и применяет к реальному DOM только минимальный набор изменений (patching). Это значительно эффективнее прямых манипуляций с DOM.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Чем функциональные компоненты отличаются от классовых?

Функциональные компоненты — обычные функции, поддерживают хуки, имеют меньший overhead. Классовые компоненты наследуют `React.Component`, используют `this.state` и методы жизненного цикла — легаси-подход. Хуки работают только в функциональных компонентах, которые сегодня являются стандартом.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое хуки и зачем они появились?

Хуки (React 16.8+) — функции с префиксом `use`, позволяющие использовать состояние и жизненный цикл в функциональных компонентах. Появились чтобы: переиспользовать логику без HOC и render props, упростить сложные классовые компоненты, устранить путаницу с `this`.

Основные: `useState`, `useEffect`, `useContext`, `useRef`, `useMemo`, `useCallback`, `useReducer`.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Для чего нужен prop key в списках?

`key` помогает React идентифицировать каждый элемент списка при reconciliation. Без стабильного `key` React не понимает, что изменилось, и может перерисовать весь список или потерять локальное состояние элементов. Индекс массива как `key` — антипаттерн при сортировке и удалении элементов.

```typescript
// Плохо — index как key
items.map((item, i) => <Item key={i} {...item} />);

// Хорошо — уникальный стабильный id
items.map(item => <Item key={item.id} {...item} />);
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое Fragment и чем он лучше div-обёртки?

`React.Fragment` (или `<>...</>`) — специальный компонент для группировки нескольких элементов без создания лишнего DOM-узла. Преимущества перед `<div>`:

- Не засоряет DOM лишними узлами
- Не ломает семантику CSS (`flexbox`, `grid`) от родительского элемента
- Не добавляет лишних узлов в `<tr>`/`<td>`, `<ul>`/`<li>` (где `<div>` невалиден)

```tsx
// Плохо: лишний div
return (
  <div>
    <h1>Заголовок</h1>
    <p>Текст</p>
  </div>
);

// Хорошо: Fragment
return (
  <>
    <h1>Заголовок</h1>
    <p>Текст</p>
  </>
);

// Длинный синтаксис нужен когда нужен key (render list)
return (
  <React.Fragment key={item.id}>
    <dt>{item.term}</dt>
    <dd>{item.description}</dd>
  </React.Fragment>
);
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**

- [React Docs: Fragment](https://react.dev/reference/react/Fragment)

---

## Что такое prop drilling и как его избежать?

Prop drilling — передача данных через несколько уровней компонентов, даже если промежуточные компоненты эти данные не используют. Приводит к связанности и усложняет рефакторинг.

**Способы избежать:**

1. **Context API** — для глобальных данных (тема, локаль, пользователь)
2. **State manager** (Zustand, Redux) — для сложного состояния
3. **Component composition** — передача `children` или render props вместо пропсов

```tsx
// Проблема: theme просачивается через App → Layout → Sidebar → Button
// Решение через Context:
const ThemeContext = createContext('light');

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Layout />  {/* не нужно пробрасывать theme через Layout */}
    </ThemeContext.Provider>
  );
}

function Button() {
  const theme = useContext(ThemeContext); // доступ напрямую
  return <button className={theme}>...</button>;
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**

- [React Docs: Passing data deeply with Context](https://react.dev/learn/passing-data-deeply-with-context)

---

## Способы условного рендеринга в React?

Условный рендеринг — вывод разного JSX в зависимости от условий. Основные подходы:

```tsx
// 1. Тернарный оператор
const El = condition ? <A /> : <B />;

// 2. && (осторожно: 0 && рендерит 0!)
const El = isLoggedIn && <Dashboard />;
// Безопасный вариант:
const El = !!items.length && <List />;

// 3. Переменная + if
let content;
if (status === 'loading') content = <Spinner />;
else if (status === 'error') content = <Error />;
else content = <Data />;

// 4. Switch внутри функции — для сложной логики
const renderByRole = (role: string) => {
  switch (role) {
    case 'admin': return <AdminView />;
    case 'user':  return <UserView />;
    default:      return <GuestView />;
  }
};
```

**Связанные задачи:**

- [Clock: компонент часов](../../../tasks/frontend/react/1_react_junior.md#clock-компонент-часов)

**Материалы для изучения:**

- [React Docs: Conditional rendering](https://react.dev/learn/conditional-rendering)

---

## Разница между элементом и компонентом?

**Элемент** (React element) — простой неизменяемый объект, описывающий что нужно отрендерить. Результат JSX или `React.createElement()`:
```jsx
const el = <h1>Привет</h1>;
// { type: 'h1', props: { children: 'Привет' }, ... }
```

**Компонент** — функция или класс, которая принимает props и возвращает элементы (или null). Компонент — «шаблон», элемент — «экземпляр» рендера.

```jsx
// Компонент (функция)
function Greeting({ name }) {
  return <h1>Привет, {name}!</h1>; // возвращает элемент
}

// Использование компонента создаёт элемент
const el = <Greeting name="Алиса" />;
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как создать компонент в React?

**Функциональный компонент (рекомендуется):**
```tsx
// Простой
function Button({ label, onClick }: { label: string; onClick: () => void }) {
  return <button onClick={onClick}>{label}</button>;
}

// Стрелочная функция
const Card = ({ title, children }: { title: string; children: React.ReactNode }) => (
  <div className="card">
    <h2>{title}</h2>
    {children}
  </div>
);

export default Button;
```

**Правила:**
- Имя начинается с **заглавной** буквы
- Возвращает JSX или `null`
- Один дефолтный экспорт или несколько именованных

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое prop children?

`children` — специальный prop, содержащий дочерние элементы, переданные между открывающим и закрывающим тегами компонента.

```tsx
function Card({ title, children }: { title: string; children: React.ReactNode }) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="card-body">{children}</div>
    </div>
  );
}

// Использование
<Card title="Новость">
  <p>Текст статьи</p>
  <a href="/more">Читать далее</a>
</Card>
```

`React.ReactNode` — тип для children: принимает JSX, строки, числа, массивы, `null`, `undefined`.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Почему нельзя обновлять state напрямую?

Прямое изменение state не вызывает ре-рендер — React не знает, что что-то изменилось:

```jsx
// Плохо — React не знает об изменении
this.state.count = 5;
state.items.push(newItem);

// Хорошо — React планирует ре-рендер
this.setState({ count: 5 });
setItems([...items, newItem]); // новая ссылка!
```

**Иммутабельность** — React сравнивает ссылки (`===`). Для объектов и массивов нужно создавать новый объект/массив, иначе React считает, что ничего не изменилось:

```jsx
// Плохо — та же ссылка
const next = state.items;
next.push(item);
setItems(next); // React не ре-рендерит

// Хорошо — новая ссылка
setItems([...state.items, item]);
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что происходит при вызове setState?

1. React ставит обновление в **очередь** (не применяет сразу)
2. В React 18 происходит **батчинг** — несколько `setState` в одном event handler объединяются в один ре-рендер
3. React планирует ре-рендер компонента
4. На следующем рендере компонент вызывается с новым значением state
5. React сравнивает новый и предыдущий vDOM (reconciliation) → обновляет только изменившиеся части DOM

```jsx
// Оба вызова батчатся в один ре-рендер (React 18)
function handleClick() {
  setCount(c => c + 1); // не ре-рендерит сразу
  setName('Alice');     // не ре-рендерит сразу
  // ← один ре-рендер здесь
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Чистая функция в React?

**Чистая функция** (pure function) — функция, которая при одинаковых аргументах всегда возвращает одинаковый результат и не производит побочных эффектов.

React-компоненты должны быть чистыми функциями относительно props и state:

```jsx
// Чистый компонент — детерминированный рендер
function Greeting({ name }) {
  return <h1>Привет, {name}!</h1>; // только зависит от props
}

// Нечистый — побочный эффект в рендере (плохо!)
function BadComponent() {
  document.title = 'Обновление'; // ← нельзя в рендере
  return <div />;
}

// Правильно — побочные эффекты в useEffect
function GoodComponent() {
  useEffect(() => { document.title = 'Обновление'; }, []);
  return <div />;
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## createElement и cloneElement?

**`React.createElement(type, props, ...children)`** — основная функция создания элементов (в которую компилируется JSX):
```jsx
// JSX
const el = <Button color="blue">Нажми</Button>;
// Эквивалентно:
const el = React.createElement(Button, { color: 'blue' }, 'Нажми');
```

**`React.cloneElement(element, extraProps, ...children)`** — клонирует существующий элемент с дополнительными/переопределёнными props:
```jsx
function Toolbar({ children }) {
  return React.Children.map(children, child =>
    React.cloneElement(child, { size: 'sm' }) // добавляем size всем детям
  );
}

<Toolbar>
  <Button>Save</Button>   {/* получит size="sm" */}
  <Button>Cancel</Button> {/* получит size="sm" */}
</Toolbar>
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Синтетические события?

React оборачивает нативные DOM-события в **SyntheticEvent** — кросс-браузерную обёртку с единым API.

```jsx
function Form() {
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();         // работает во всех браузерах
    console.log(e.target);      // SyntheticEvent
    console.log(e.nativeEvent); // нативный DOM-event
  };

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    console.log(e.target.value);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input onChange={handleChange} />
    </form>
  );
}
```

React использует **event delegation** — обработчики вешаются на корневой DOM-узел, а не на каждый элемент. В React 17+ это корень приложения (`#root`), а не `document`.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## HTML vs React?

**Ключевые отличия JSX от HTML:**

| HTML | React/JSX |
|---|---|
| `class="..."` | `className="..."` |
| `for="..."` | `htmlFor="..."` |
| `onclick="fn()"` | `onClick={fn}` |
| `style="color: red"` | `style={{ color: 'red' }}` |
| Само-закрывающие: `<br>` | `<br />` |
| Комментарии `<!-- -->` | `{/* */}` |

**React** — декларативный подход: описываешь *что* должно отображаться. Браузерный DOM — императивный (`getElementById`, `innerHTML`).

```jsx
// HTML (императивно)
document.getElementById('counter').textContent = count;

// React (декларативно)
return <span>{count}</span>; // React сам обновит DOM
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
