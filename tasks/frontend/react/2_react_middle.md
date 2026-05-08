# React — Middle Tasks

## Задачи

- [Оптимизация ре-рендеров: memo + useCallback](#оптимизация-ре-рендеров-memo--usecallback)
- [Рефакторинг Modal компонента](#рефакторинг-modal-компонента)
- [Render Props: ModalWithRender](#render-props-modalwithrender)
- [BTC конвертер с polling](#btc-конвертер-с-polling)
- [Динамические инпуты с валидацией](#динамические-инпуты-с-валидацией)
- [Счётчик с шагом: useRef для стабильных колбэков](#счётчик-с-шагом-useref-для-стабильных-колбэков)

---

## Оптимизация ре-рендеров: memo + useCallback

В компоненте `App` есть инпут, счётчик и дочерний компонент `ChildComponent`. При изменении инпута дочерний компонент ненужно перерендеривается.

**Задача 1:** Предотвратить ре-рендер `ChildComponent` при изменении инпута, если `count` не изменился.

**Задача 2:** `ChildComponent` также принимает функцию `increment`. Предотвратить ре-рендер при изменении инпута, стабилизировав ссылку на функцию.

```tsx
const App = () => {
  const [input, setInput] = useState('');
  const [count, setCount] = useState(0);

  const increment = () => setCount(count + 1);

  return (
    <div>
      <input value={input} onChange={e => setInput(e.target.value)} />
      <button onClick={increment}>Increment</button>
      <p>Count: {count}</p>
      <ChildComponent count={count} increment={increment} />
    </div>
  );
};

const ChildComponent = ({ count, increment }) => {
  console.log(':: render');
  return (
    <>
      <div>Child: {count}</div>
      <button onClick={increment}>Increment</button>
    </>
  );
};
```

**Связанные вопросы:**

- [Как избежать лишних ре-рендеров?](../../../interviews/frontend/react/2_react_middle.md#как-избежать-лишних-ре-рендеров)
- [Что такое React.memo?](../../../interviews/frontend/react/2_react_middle.md#что-такое-reactmemo)
- [В чём разница между useMemo и useCallback?](../../../interviews/frontend/react/2_react_middle.md#в-чём-разница-между-usememo-и-usecallback)
- [Техники оптимизации производительности React?](../../../interviews/frontend/react/3_react_senior.md#техники-оптимизации-производительности-react)

<details>
<summary>Решение</summary>

```tsx
import { useState, useCallback, memo } from 'react';

const App = () => {
  const [input, setInput] = useState('');
  const [count, setCount] = useState(0);

  // useCallback: стабильная ссылка на функцию между рендерами
  // Используем функциональный updater чтобы не захватывать count в closure
  const increment = useCallback(() => {
    setCount(prev => prev + 1);
  }, []); // пустые deps — функция создаётся один раз

  return (
    <div>
      <input value={input} onChange={e => setInput(e.target.value)} />
      <button onClick={increment}>Increment</button>
      <p>Count: {count}</p>
      <ChildComponent count={count} increment={increment} />
    </div>
  );
};

// memo: пропускает ре-рендер если props не изменились (поверхностное сравнение)
const ChildComponent = memo(({ count, increment }) => {
  console.log(':: render');
  return (
    <>
      <div>Child: {count}</div>
      <button onClick={increment}>Increment</button>
    </>
  );
});
```

**Почему без `useCallback` `memo` бесполезен:** при каждом рендере `App` создаётся новая функция `increment` — новая ссылка. `memo` видит изменение `increment` и перерендеривает дочерний компонент.

</details>

---

## Рефакторинг Modal компонента

Найдите и исправьте все проблемы в компоненте `Modal`:

```tsx
const Modal: React.FC<IProps> = props => {
  useEffect(() => {
    if (props.isOpen) {
      document.title = props.title + ' - ' + document.title;
    }
    document.body.classList.add('overflow--hidden');
  }, []);

  const didModalClosed = useCallback(() => props.onModalClose(), []);

  return props.isOpen && (
    <>
      <div className="fade" onClick={didModalClosed}>
        <div className="modal">
          <div className="header">
            <h3>{props.title}</h3>
            <button onClick={didModalClosed}>Close</button>
          </div>
          <div className="content">{props.children}</div>
          <div className="footer">
            <button onClick={props.onSubmit}>Submit</button>
          </div>
        </div>
      </div>
    </>
  );
};
```

**Связанные вопросы:**

- [Как работает useEffect — зависимости и cleanup?](../../../interviews/frontend/react/2_react_middle.md#как-работает-useeffect--зависимости-и-cleanup)
- [В чём разница между useMemo и useCallback?](../../../interviews/frontend/react/2_react_middle.md#в-чём-разница-между-usememo-и-usecallback)

<details>
<summary>Решение</summary>

**Проблемы в исходном коде:**
1. `useEffect` с `[]` — `document.body.classList.add` вызывается только при монтировании и **никогда не убирается** (отсутствует cleanup + зависимость от `isOpen`)
2. `useCallback` с `[]` — `onModalClose` захвачена в closure при монтировании, если пропс сменится — колбэк устарел
3. `props.isOpen && <></>` — возвращает `false` вместо `null` при закрытии (потенциальные проблемы)
4. Лишний `<>` Fragment — не нужен вокруг одного `div`

```tsx
// Выносим логику блокировки скролла в кастомный хук
const useScrollLock = (isOpen: boolean) => {
  useEffect(() => {
    if (isOpen) {
      document.body.classList.add('overflow--hidden');
    } else {
      document.body.classList.remove('overflow--hidden'); // cleanup
    }
  }, [isOpen]); // зависимость от isOpen
};

const Modal: React.FC<IProps> = (props) => {
  const { isOpen, title, onModalClose, onSubmit, children } = props;

  useScrollLock(isOpen);

  // зависимость от onModalClose чтобы всегда иметь актуальную ссылку
  const handleClose = useCallback(() => {
    onModalClose();
  }, [onModalClose]);

  if (!isOpen) return null; // явный null вместо false

  return (
    <div className="fade" onClick={handleClose}>
      <div className="modal">
        <div className="header">
          <h3>{title}</h3>
          <button onClick={handleClose}>Close</button>
        </div>
        <div className="content">{children}</div>
        <div className="footer">
          <button type="button" onClick={onSubmit}>Submit</button>
        </div>
      </div>
    </div>
  );
};
```

</details>

---

## Render Props: ModalWithRender

Реализуйте компонент `ModalWithRender`, использующий паттерн **render props**: он принимает `children` как функцию, которую вызывает с объектом `{ onOpen }`, и `content` для отображения в модалке.

```tsx
// Использование
<ModalWithRender content="Позвоните нам: 8-800-555">
  {({ onOpen }) => <button onClick={onOpen}>Открыть</button>}
</ModalWithRender>

<ModalWithRender content="О нас">
  {({ onOpen }) => <a onClick={onOpen}>Подробнее</a>}
</ModalWithRender>
```

**Связанные вопросы:**

- [Что такое HOC и как его использовать?](../../../interviews/frontend/react/3_react_senior.md#что-такое-hoc-и-как-его-использовать)
- [Что такое порталы и когда они нужны?](../../../interviews/frontend/react/2_react_middle.md#что-такое-порталы-и-когда-они-нужны)

<details>
<summary>Решение</summary>

```tsx
import { useState } from 'react';
import Modal from 'react-modal';

type RenderProps = {
  onOpen: () => void;
};

type Props = {
  content?: string;
  children: (props: RenderProps) => React.ReactNode;
};

const ModalWithRender = ({ children, content }: Props) => {
  const [isOpen, setIsOpen] = useState(false);

  const renderProps: RenderProps = {
    onOpen: () => setIsOpen(true),
  };

  return (
    <>
      {children(renderProps)}
      <Modal isOpen={isOpen} onRequestClose={() => setIsOpen(false)}>
        {content}
      </Modal>
    </>
  );
};
```

**Паттерн render props** позволяет компоненту делегировать рендеринг части UI вызывающему коду, сохраняя при этом контроль над состоянием. Альтернатива — кастомный хук `useModal`, который возвращает `{ isOpen, onOpen, onClose }`.

</details>

---

## BTC конвертер с polling

Реализуйте конвертер BTC в другие валюты:

1. Курсы получать из `https://blockchain.info/ticker` (объект `{ USD: { buy, sell, last, ... }, EUR: {...}, ... }`)
2. Автоматически обновлять курс **каждую минуту**
3. В поле ввода BTC разрешать только цифры
4. При вводе числа пересчитывать результат в выбранной валюте
5. Компоненты `ExchangeInput` и `CurrenciesSelect` — вынести отдельно, оптимизировать ре-рендеры

**Связанные вопросы:**

- [В чём разница между useMemo и useCallback?](../../../interviews/frontend/react/2_react_middle.md#в-чём-разница-между-usememo-и-usecallback)
- [Как работает useEffect — зависимости и cleanup?](../../../interviews/frontend/react/2_react_middle.md#как-работает-useeffect--зависимости-и-cleanup)

<details>
<summary>Решение</summary>

```tsx
import { useEffect, useState, useMemo, useCallback } from 'react';

type CurrencyCode = string;
type Currency = { last: number; buy: number; sell: number; symbol: string };
type Currencies = Record<CurrencyCode, Currency>;

const fetchCurrencies = (): Promise<Currencies> =>
  fetch('https://blockchain.info/ticker').then(r => r.json());

export default function App() {
  const [currencies, setCurrencies] = useState<Currencies | null>(null);
  const [selectedCurrency, setSelectedCurrency] = useState<CurrencyCode | null>(null);
  const [btcAmount, setBtcAmount] = useState(0);

  const loadRates = useCallback(async () => {
    const data = await fetchCurrencies();
    setCurrencies(data);
  }, []);

  useEffect(() => {
    loadRates();
    const interval = setInterval(loadRates, 60_000);
    return () => clearInterval(interval);
  }, [loadRates]);

  const currencyKeys = useMemo(
    () => currencies ? Object.keys(currencies) : [],
    [currencies]
  );

  const result = useMemo(() => {
    if (!selectedCurrency || !currencies) return 0;
    return btcAmount * currencies[selectedCurrency].buy;
  }, [selectedCurrency, btcAmount, currencies]);

  return (
    <div>
      <h1>BTC Конвертер</h1>
      <ExchangeInput value={btcAmount} setValue={setBtcAmount} />
      <CurrenciesSelect
        currencies={currencyKeys}
        result={result}
        onChange={setSelectedCurrency}
      />
    </div>
  );
}

const ExchangeInput = ({ value, setValue }: { value: number; setValue: (v: number) => void }) => {
  const handleChange = useCallback((e: React.ChangeEvent<HTMLInputElement>) => {
    const clean = e.target.value.replace(/[^\d]/g, '');
    setValue(Number(clean));
  }, []);

  return (
    <div>
      <span>BTC</span>
      <input type="text" value={value} onChange={handleChange} />
    </div>
  );
};

const CurrenciesSelect = ({ currencies, result, onChange }: {
  currencies: string[];
  result: number;
  onChange: (c: string) => void;
}) => {
  const handleChange = useCallback(
    (e: React.ChangeEvent<HTMLSelectElement>) => onChange(e.target.value),
    [onChange]
  );

  return (
    <div>
      <select onChange={handleChange}>
        <option>-- Выберите валюту --</option>
        {currencies.map(c => <option key={c} value={c}>{c}</option>)}
      </select>
      <input type="text" value={result} disabled />
    </div>
  );
};
```

</details>

---

## Динамические инпуты с валидацией

Реализуйте форму где:
1. По кнопке «Добавить инпут» добавляется новый `<input>`
2. Во всех инпутах ведётся валидация: значение должно быть `"react"`
3. Кнопка «Сохранить» активируется только если **все** инпуты прошли валидацию и хотя бы один есть

**Связанные вопросы:**

- [Как работает useState внутри React?](../../../interviews/frontend/react/2_react_middle.md#как-работает-usestate-внутри-react)

<details>
<summary>Решение</summary>

```tsx
import { useState, useCallback, useMemo } from 'react';

const validate = (value: string) => value === 'react';

export const App = () => {
  const [inputs, setInputs] = useState<Record<string, string>>({});

  const handleAdd = useCallback(() => {
    setInputs(prev => {
      const nextKey = String(Object.keys(prev).length + 1);
      return { ...prev, [nextKey]: '' };
    });
  }, []);

  const handleChange = useCallback((id: string, value: string) => {
    setInputs(prev => ({ ...prev, [id]: value }));
  }, []);

  const isValid = useMemo(() => {
    const values = Object.values(inputs);
    return values.length > 0 && values.every(validate);
  }, [inputs]);

  return (
    <form>
      {Object.entries(inputs).map(([key, value]) => (
        <input
          key={key}
          type="text"
          value={value}
          onChange={e => handleChange(key, e.target.value)}
          style={{ borderColor: validate(value) ? 'green' : 'red' }}
        />
      ))}
      <div>
        <button type="button" onClick={handleAdd}>Добавить инпут</button>
        <button type="button" disabled={!isValid}>Сохранить</button>
      </div>
    </form>
  );
};
```

</details>

---

## Счётчик с шагом: useRef для стабильных колбэков

Реализуйте счётчик с изменяемым шагом через `<input type="range">`. Кнопки «Increment» и «Decrement» **не должны перерендериваться** при изменении шага.

Используйте `useRef` для доступа к актуальному значению шага внутри стабильных колбэков.

```tsx
const Button = memo((props: ButtonHTMLAttributes<HTMLButtonElement>) => {
  console.log('button render', props.children);
  return <button {...props} />;
});
```

**Связанные вопросы:**

- [Как избежать лишних ре-рендеров?](../../../interviews/frontend/react/2_react_middle.md#как-избежать-лишних-ре-рендеров)
- [В чём разница между useMemo и useCallback?](../../../interviews/frontend/react/2_react_middle.md#в-чём-разница-между-usememo-и-usecallback)

<details>
<summary>Решение</summary>

```tsx
import { useState, useCallback, useRef, useEffect, memo, ButtonHTMLAttributes } from 'react';

const Button = memo((props: ButtonHTMLAttributes<HTMLButtonElement>) => {
  console.log('button render', props.children);
  return <button {...props} />;
});

export const App = () => {
  const [step, setStep] = useState(1);
  const [counter, setCounter] = useState(0);

  // Ref всегда содержит актуальное значение шага
  const stepRef = useRef(step);

  useEffect(() => {
    stepRef.current = step;
  }, [step]);

  // Стабильные колбэки с пустыми deps — читают step через ref
  const handleIncrement = useCallback(() => {
    setCounter(prev => prev + stepRef.current);
  }, []);

  const handleDecrement = useCallback(() => {
    setCounter(prev => prev - stepRef.current);
  }, []);

  const handleRangeChange = useCallback((e: React.ChangeEvent<HTMLInputElement>) => {
    setStep(Number(e.target.value));
  }, []);

  return (
    <div>
      <div>Counter: {counter}</div>
      <input value={step} onChange={handleRangeChange} type="range" min="1" max="10" />
      <span>Step: {step}</span>
      <Button onClick={handleIncrement}>Increment</Button>
      <Button onClick={handleDecrement}>Decrement</Button>
    </div>
  );
};
```

**Ключевая идея:** `useCallback` с `[]` создаёт функцию один раз — поэтому она не видит обновлений `step` из замыкания. Обходной путь — хранить актуальное значение в `ref`, который мутируется в `useEffect`. Это позволяет иметь и стабильную ссылку, и актуальное значение.

</details>
