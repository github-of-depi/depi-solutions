# React — Junior Tasks

## Задачи

- [Clock: компонент часов](#clock-компонент-часов)
- [Tree: отрисовка вложенного дерева из JSON](#tree-отрисовка-вложенного-дерева-из-json)

---

## Clock: компонент часов

Реализуйте компонент `Clock`, который каждую секунду отображает текущее время в формате `HH:MM:SS`. Реализуйте **два варианта** — через `setInterval` и через рекурсивный `setTimeout`. Убедитесь, что таймер очищается при размонтировании компонента.

```tsx
const Clock = () => {
  return (
    <div>
      <p>Current time: {/* текущее время */}</p>
    </div>
  );
};
```

**Связанные вопросы:**

- [Как работает useEffect — зависимости и cleanup?](../../../interviews/frontend/react/2_react_middle.md#как-работает-useeffect--зависимости-и-cleanup)
- [Способы условного рендеринга в React?](../../../interviews/frontend/react/1_react_junior.md#способы-условного-рендеринга-в-react)

<details>
<summary>Решение</summary>

```tsx
import React, { useState, useEffect } from 'react';

// Вариант 1: setInterval
const ClockInterval = () => {
  const [date, setDate] = useState(new Date());

  useEffect(() => {
    const interval = setInterval(() => {
      setDate(new Date());
    }, 1000);

    return () => clearInterval(interval); // cleanup при размонтировании
  }, []);

  return (
    <div>
      <p>Current time: {date.toLocaleTimeString()}</p>
    </div>
  );
};

// Вариант 2: рекурсивный setTimeout
const ClockTimeout = () => {
  const [time, setTime] = useState<string>('');

  useEffect(() => {
    let timeout: ReturnType<typeof setTimeout>;

    const update = () => {
      setTime(new Date().toLocaleTimeString());
      timeout = setTimeout(update, 1000);
    };

    update();

    return () => clearTimeout(timeout);
  }, []);

  return (
    <div>
      <p>Current time: {time}</p>
    </div>
  );
};
```

**Разница между вариантами:** `setInterval` вызывается строго каждые 1000 мс независимо от времени выполнения колбэка. `setTimeout` + рекурсия — следующий вызов планируется только после завершения текущего, что позволяет избежать накопления задержек при долгом выполнении.

</details>

---

## Tree: отрисовка вложенного дерева из JSON

Есть вложенная структура данных. Нужно:
1. Написать функцию `getDataFromBackend()`, которая возвращает промис с данными через 2 секунды
2. Отрисовать данные в виде дерева с отступом у каждого уровня вложенности
3. Добавить поиск с фильтрацией по имени (если дочерний элемент подходит — показать его и всех родителей)
4. Обернуть поиск в `debounce` через кастомный хук `useDebounce`

```json
[
  { "id": 1, "name": "Название один" },
  {
    "id": 2, "name": "Второе название",
    "child": [
      { "id": 6, "name": "Винни-Пух", "child": [] },
      {
        "id": 7, "name": "Крокодил Гена",
        "child": [
          { "id": 11, "name": "Глубокий элемент", "child": [] }
        ]
      }
    ]
  },
  { "id": 5, "name": "Пяточок", "child": [] }
]
```

**Связанные вопросы:**

- [Что такое хуки и зачем они появились?](../../../interviews/frontend/react/1_react_junior.md#что-такое-хуки-и-зачем-они-появились)
- [Как работает useEffect — зависимости и cleanup?](../../../interviews/frontend/react/2_react_middle.md#как-работает-useeffect--зависимости-и-cleanup)

<details>
<summary>Решение</summary>

```tsx
import React, { useState, useEffect, useMemo } from 'react';

interface TreeItem {
  id: number;
  name: string;
  child?: TreeItem[];
}

// Симуляция запроса
const getDataFromBackend = (): Promise<TreeItem[]> => {
  return new Promise(resolve =>
    setTimeout(() => resolve(data), 2000)
  );
};

// Кастомный хук debounce
const useDebounce = (value: string, delay: number) => {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debounced;
};

// Рекурсивная фильтрация: возвращает узел если он или его потомки совпадают
const filterTree = (items: TreeItem[], search: string): TreeItem[] => {
  if (!search) return items;

  return items.reduce<TreeItem[]>((acc, item) => {
    const filteredChildren = filterTree(item.child || [], search);
    const matches = item.name.toLowerCase().includes(search.toLowerCase());

    if (matches || filteredChildren.length > 0) {
      acc.push({ ...item, child: filteredChildren });
    }
    return acc;
  }, []);
};

// Компонент дерева
const Tree = ({ tree }: { tree: TreeItem[] }) => {
  if (!tree.length) return null;

  return (
    <div style={{ marginLeft: '24px' }}>
      {tree.map(item => (
        <div key={item.id}>
          <p>{item.name}</p>
          {item.child && <Tree tree={item.child} />}
        </div>
      ))}
    </div>
  );
};

export default function App() {
  const [tree, setTree] = useState<TreeItem[]>([]);
  const [search, setSearch] = useState('');
  const debouncedSearch = useDebounce(search, 400);

  useEffect(() => {
    getDataFromBackend().then(setTree);
  }, []);

  const filteredTree = useMemo(
    () => filterTree(tree, debouncedSearch),
    [tree, debouncedSearch]
  );

  return (
    <div>
      <input
        value={search}
        onChange={e => setSearch(e.target.value)}
        placeholder="Поиск..."
      />
      <Tree tree={filteredTree} />
    </div>
  );
}
```

</details>
