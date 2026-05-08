# React — Senior Tasks

## Задачи

- [File Explorer: дерево файлов с раскрытием папок](#file-explorer-дерево-файлов-с-раскрытием-папок)
- [Offline counter с useRef и useEffect](#offline-counter-с-useref-и-useeffect)
- [Code review: LargeList + AppWithCounter](#code-review-largelist--appwithcounter)
- [Code review: хук useGetSomething — лоадер и обработка ошибок](#code-review-хук-usegetsomething--лоадер-и-обработка-ошибок)

---

## File Explorer: дерево файлов с раскрытием папок

Реализуйте компонент `Tree`, отображающий иерархическую структуру файлов и папок с возможностью раскрытия/закрытия папок по клику.

Требования:
- Узел считается папкой если у него есть поле `children`
- Папка по умолчанию показывает иконку «закрыто», при раскрытии — «открыто»
- Иконки файлов зависят от расширения: `.js`, `.tsx` имеют особые иконки, остальные — «file»
- Реализовать колбэки `onSelect` (выбор элемента) и `onExpand` (раскрытие/закрытие папки)
- Компонент должен быть мемоизирован

```tsx
const data = [
  {
    id: 1, name: "node_modules",
    children: [{ id: 2, name: "storybook", children: [{ id: 3, name: "index.js" }] }]
  },
  { id: 6, name: "public", children: [{ id: 7, name: "index.html" }] },
  { id: 8, name: "src", children: [{ id: 9, name: "App.tsx" }, { id: 10, name: "index.tsx" }] },
  { id: 11, name: "package.json" },
  { id: 12, name: "README.md" },
];

export default function App() {
  return <Tree items={data} />;
}
```

**Связанные вопросы:**

- [Как работает алгоритм reconciliation?](../../../interviews/frontend/react/3_react_senior.md#как-работает-алгоритм-reconciliation)
- [Что такое React.memo?](../../../interviews/frontend/react/2_react_middle.md#что-такое-reactmemo)

<details>
<summary>Решение</summary>

```tsx
import React, { useState, memo } from 'react';
import type { FC } from 'react';

export type TreeNode = {
  id: number;
  name: string;
  children?: TreeNode[];
};

const isFolder = (node: TreeNode) => Boolean(node.children);

const getIcon = (node: TreeNode, expanded: boolean): string => {
  if (isFolder(node)) return expanded ? '📂' : '📁';
  const ext = node.name.split('.').pop();
  if (ext === 'js') return '🟡';
  if (ext === 'tsx' || ext === 'ts') return '🔷';
  return '📄';
};

interface TreeProps {
  items: TreeNode[];
  onSelect?: (node: TreeNode) => void;
  onExpand?: (node: TreeNode, expanded: boolean) => void;
}

export const Tree: FC<TreeProps> = memo(({ items, onSelect, onExpand }) => {
  const [expanded, setExpanded] = useState<Set<number>>(new Set());

  const handleToggle = (node: TreeNode) => {
    const isExpanded = expanded.has(node.id);
    setExpanded(prev => {
      const next = new Set(prev);
      isExpanded ? next.delete(node.id) : next.add(node.id);
      return next;
    });
    onExpand?.(node, !isExpanded);
  };

  return (
    <ul style={{ listStyle: 'none', paddingLeft: '16px' }}>
      {items.map(node => {
        const isExp = expanded.has(node.id);
        const icon = getIcon(node, isExp);

        return (
          <li key={node.id}>
            <span
              onClick={() => {
                if (isFolder(node)) handleToggle(node);
                onSelect?.(node);
              }}
              style={{ cursor: 'pointer', userSelect: 'none' }}
            >
              {icon} {node.name}
            </span>

            {isFolder(node) && isExp && (
              <Tree
                items={node.children!}
                onSelect={onSelect}
                onExpand={onExpand}
              />
            )}
          </li>
        );
      })}
    </ul>
  );
});

export default function App() {
  return (
    <Tree
      items={data}
      onSelect={node => console.log('selected:', node.name)}
      onExpand={(node, exp) => console.log(node.name, exp ? 'opened' : 'closed')}
    />
  );
}
```

</details>

---

## Offline counter с useRef и useEffect

Найдите все баги в компоненте и исправьте их. При потере интернет-соединения (`offline` событие) должно запускаться ежесекундное логирование **актуального** значения `count`.

```tsx
const Component = () => {
  const [items, setItems] = useState([]);
  const [count, setCount] = useState(0);

  useLayoutEffect(() => {
    window.addEventListener('offline', () => {
      setInterval(() => {
        console.log(count);
      }, 1000);
    });
  });

  const onClick = useCallback(() => {
    setItems([
      ...items,
      { id: count + 1, title: new Date.getTime().toString() },
    ]);
    setCount(count + 1);
  });

  return (
    <>
      <button onClick={() => onClick()}>Добавить</button>
      <p>Всего: {count}</p>
      {items.map(el => {
        <li>{el.title}</li>;
      })}
    </>
  );
};
```

**Связанные вопросы:**

- [Чем useLayoutEffect отличается от useEffect?](../../../interviews/frontend/react/2_react_middle.md#чем-uselayouteffect-отличается-от-useeffect)
- [В чём разница между useMemo и useCallback?](../../../interviews/frontend/react/2_react_middle.md#в-чём-разница-между-usememo-и-usecallback)

<details>
<summary>Решение</summary>

**Найденные баги:**

1. `useLayoutEffect` без `[]` — эффект повторяется при каждом рендере, каждый раз добавляя новый `addEventListener`. Нужно `[]` + cleanup.
2. `setInterval` внутри `addEventListener` — при каждом `offline`-событии создаётся новый интервал, старые не очищаются. Нужно хранить и очищать.
3. `console.log(count)` — замыкание захватывает `count` при создании колбэка (всегда 0). Нужен `useRef` для актуального значения.
4. `new Date.getTime()` → `new Date().getTime()` — синтаксическая ошибка.
5. `useCallback` без `[]` — пересоздаётся при каждом рендере.
6. `setItems([...items, ...])` — захватывает `items` в closure, при быстрых кликах может быть stale. Нужен функциональный updater.
7. `items.map(el => { <li>... })` — стрелка с `{}` и без `return` ничего не рендерит.

```tsx
import { useState, useEffect, useCallback, useRef, memo } from 'react';

const Component = () => {
  const [items, setItems] = useState<{ id: number; title: string }[]>([]);
  const [count, setCount] = useState(0);
  const countRef = useRef(0);

  // Синхронизируем ref с актуальным count
  useEffect(() => {
    countRef.current = count;
  }, [count]);

  useEffect(() => {
    let interval: ReturnType<typeof setInterval> | null = null;

    const handleOffline = () => {
      // Очищаем предыдущий интервал если был
      if (interval) clearInterval(interval);
      interval = setInterval(() => {
        // Читаем актуальное значение через ref, не через closure
        console.log(countRef.current);
      }, 1000);
    };

    window.addEventListener('offline', handleOffline);

    return () => {
      window.removeEventListener('offline', handleOffline);
      if (interval) clearInterval(interval);
    };
  }, []); // пустой массив — эффект один раз при монтировании

  const onClick = useCallback(() => {
    setCount(prev => {
      const newCount = prev + 1;
      setItems(items => [
        ...items,
        { id: newCount, title: new Date().getTime().toString() }, // new Date()
      ]);
      return newCount;
    });
  }, []);

  return (
    <>
      <button onClick={onClick}>Добавить</button>
      <p>Всего: {count}</p>
      {items.length > 0 && (
        <ul>
          {items.map(el => (
            <li key={el.id}>{el.title}</li> // return + key
          ))}
        </ul>
      )}
    </>
  );
};
```

</details>

---

## Code review: LargeList + AppWithCounter

Проведите code review компонента и исправьте **все** найденные проблемы.

```tsx
const LargeList = ({ commentPrefix }) => {
  const [posts, setPosts] = useState([]);
  const [comments, setComments] = useState([]);

  // @ts-ignore
  useEffect(async () => {
    const postsResponse = await fetch('https://jsonplaceholder.typicode.com/posts');
    const commentsResponce = await fetch('https://jsonplaceholder.typicode.com/comments');
    setPosts(await postsResponse.json());
    setComments(await commentsResponce.json());
  }, []);

  const findRelatedComments = (postId: number) => {
    const newComments = [];
    for (let i = 0; i < comments.length; i++) {
      if (comments[i].id === postId) { // <-- баг
        newComments.push(comments[i]);
      }
    }
    return newComments;
  };

  return (
    <div>
      {posts.map(({ title, body, id }) => (
        <div className="post"> {/* нет key */}
          <h1>{title}</h1>
          <p>{body}</p>
          <ul>
            {findRelatedComments(id).map(comment => (
              <div> {/* нет key */}
                {commentPrefix} {comment.body}
              </div>
            ))}
          </ul>
        </div>
      ))}
    </div>
  );
};

let isLoading = true;

const AppWithCounter = () => {
  const [counter, setCounter] = useState(0);

  const increase = () => setCounter(prev => prev + 1);

  useEffect(() => {
    if (isLoading) {
      setInterval(increase, 1000); // нет очистки
    }
    isLoading = false;
  }); // нет deps

  return (
    <div>
      Прошло секунд: {counter}
      <LargeList commentPrefix="*" />
    </div>
  );
};
```

**Связанные вопросы:**

- [Как работает useEffect — зависимости и cleanup?](../../../interviews/frontend/react/2_react_middle.md#как-работает-useeffect--зависимости-и-cleanup)

<details>
<summary>Решение</summary>

**Список найденных проблем:**

1. `useEffect(async () => ...)` — нельзя передавать async-функцию напрямую (она возвращает Promise, а не cleanup-функцию). Нужно завернуть в IIFE или использовать внутреннюю async-функцию.
2. `comments[i].id === postId` — фильтруем по `id` комментария, а не по `postId`. Должно быть `comments[i].postId === postId`.
3. Отсутствие `key` у элементов списка (`posts.map`, `comments.map`).
4. `<div>` внутри `<ul>` — невалидный HTML, должен быть `<li>`.
5. `let isLoading = true` — глобальная переменная вне компонента, нарушает инкапсуляцию.
6. `useEffect` без `[]` — запускается при каждом рендере.
7. `setInterval` без очистки (`clearInterval`) — утечка памяти.
8. `LargeList` перерендеривается при каждом обновлении `counter` — нужен `memo`.

```tsx
import { useState, useEffect, memo } from 'react';

interface Post { id: number; userId: number; title: string; body: string; }
interface Comment { id: number; postId: number; name: string; email: string; body: string; }

const LargeList = memo(({ commentPrefix }: { commentPrefix: string }) => {
  const [posts, setPosts] = useState<Post[]>([]);
  const [comments, setComments] = useState<Comment[]>([]);

  useEffect(() => {
    // Нельзя делать useEffect async — оборачиваем в функцию
    const load = async () => {
      const [postsRes, commentsRes] = await Promise.all([
        fetch('https://jsonplaceholder.typicode.com/posts'),
        fetch('https://jsonplaceholder.typicode.com/comments'),
      ]);
      setPosts(await postsRes.json());
      setComments(await commentsRes.json());
    };
    load();
  }, []); // зависимости пустые — загрузка один раз

  // Оптимизация: Map вместо O(n) поиска в цикле
  const commentsByPost = comments.reduce<Record<number, Comment[]>>((acc, c) => {
    (acc[c.postId] ??= []).push(c); // группируем по postId (не id!)
    return acc;
  }, {});

  return (
    <div>
      {posts.map(({ title, body, id }) => (
        <div key={id} className="post"> {/* key обязателен */}
          <h1>{title}</h1>
          <p>{body}</p>
          <ul>
            {(commentsByPost[id] ?? []).map(comment => (
              <li key={comment.id}> {/* li, не div + key */}
                {commentPrefix} {comment.body}
              </li>
            ))}
          </ul>
        </div>
      ))}
    </div>
  );
});

const AppWithCounter = () => {
  const [counter, setCounter] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      setCounter(prev => prev + 1);
    }, 1000);
    return () => clearInterval(interval); // cleanup
  }, []); // [] — один раз при монтировании

  return (
    <div>
      Прошло секунд: {counter}
      <LargeList commentPrefix="*" />
    </div>
  );
};
```

</details>

---

## Code review: хук useGetSomething — лоадер и обработка ошибок

Найдите и исправьте два бага:
1. Лоадер отображается постоянно, хотя должен показываться только во время запроса
2. При нажатии «Fail fetch» оба алерта должны показывать ошибку, но второй показывает «Success from App»

```tsx
export const useGetSomething = () => {
  const [loading, setLoading] = useState(false);

  const fetch = useCallback(async (fail: boolean) => {
    try {
      await new Promise(resolve => setTimeout(resolve, 1000));
      if (fail) throw new Error('');
      alert('Request success');
    } catch (e) {
      alert('Request fail');
      // ошибка поглощается здесь!
    }
    // loading никогда не меняется на true/false
  }, []);

  return { fetch, loading };
};
```

**Связанные вопросы:**

- [Как работает useEffect — зависимости и cleanup?](../../../interviews/frontend/react/2_react_middle.md#как-работает-useeffect--зависимости-и-cleanup)

<details>
<summary>Решение</summary>

**Проблема 1: лоадер.**
`setLoading(true)` нигде не вызывается — `loading` всегда `false`. Нужно установить `true` до запроса и `false` в `finally` (чтобы сбрасывался и при ошибке).

**Проблема 2: обработка ошибки.**
`catch` поглощает ошибку — после него `.then()` в вызывающем коде считает промис успешным. Нужно в `catch` пробросить ошибку дальше через `throw e`.

```tsx
export const useGetSomething = () => {
  const [loading, setLoading] = useState(false);

  const fetch = useCallback(async (fail: boolean) => {
    setLoading(true); // показываем лоадер перед запросом
    try {
      await new Promise(resolve => setTimeout(resolve, 1000));
      if (fail) throw new Error('request failed');
      alert('Request success');
    } catch (e) {
      alert('Request fail');
      throw e; // пробрасываем ошибку — .catch() в вызывающем коде сработает
    } finally {
      setLoading(false); // скрываем лоадер в любом случае
    }
  }, []);

  return { fetch, loading };
};

// Теперь в App:
// fetch(false) → alert("Request success") → alert("Success from App")
// fetch(true)  → alert("Request fail")    → alert("Fail from App")
```

</details>
