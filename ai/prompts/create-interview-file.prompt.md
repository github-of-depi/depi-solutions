---
platform: "copilot"
encoding: "UTF-8"
language: "russian"
---

# Prompt: Create Interview Question File

## Context

This repository contains interview preparation materials for a Senior+ Frontend Developer.
Files are organized by topic and skill level inside `interviews/`.

## Folder Scope

Репозиторий содержит **два параллельных дерева** папок:

### `interviews/` — вопросы и ответы (Q&A)

```
interviews/
  frontend/         → react, typescript, nextjs, state-management, css, styling, forms, performance, browser, tooling
  data-and-api/     → rest, websockets, tanstack-query, graphql, auth
  architecture/     → patterns, component-design, fsd, micro-frontends, system-design, adr
  testing/          → unit, e2e, a11y, security, code-review
  devops/           → git, docker, ci-cd, cloud
  backend/          → nodejs, sql, oop
```

### `tasks/` — практические задачи

Структура **зеркалирует** `interviews/` — те же подпапки и те же числовые префиксы уровней:

```
tasks/
  frontend/         → react, typescript, javascript, css, …
  data-and-api/     → rest, graphql, …
  algorithms/       → easy, medium, hard (не делится по уровням)
  …
```

Do NOT create level files in `behavioral/` or `companies/`.

## Naming Convention

### Q&A files (`interviews/`)

Files must use a numeric prefix and include both topic name and level so they sort correctly and are self-descriptive:

Pattern: `{number}_{topic}_{level}.md`

| File                  | Level  | Assumes knowledge of     |
| --------------------- | ------ | ------------------------ |
| `1_{topic}_junior.md` | Junior | —                        |
| `2_{topic}_middle.md` | Middle | Junior                   |
| `3_{topic}_senior.md` | Senior | Junior + Middle          |
| `4_{topic}_expert.md` | Expert | Junior + Middle + Senior |

Examples: `1_react_junior.md`, `2_typescript_middle.md`, `4_nextjs_expert.md`

### Task files (`tasks/`)

Same numeric prefix convention, same topic/level names:

Pattern: `{number}_{topic}_{level}.md`

Examples: `1_javascript_junior.md` inside `tasks/frontend/javascript/`  
Algorithms: `easy.md`, `medium.md`, `hard.md` inside `tasks/algorithms/` (no level prefix)

Topic name must match the folder name (lowercase, hyphens preserved: `state-management`, `tanstack-query`).

## File Structure Templates

### Q&A file (`interviews/`)

Each file must follow this exact structure:

```markdown
# {Topic Name} — {Level}

## Вопросы

- [Вопрос первый?](#вопрос-первый)
- [Вопрос второй?](#вопрос-второй)
- [Вопрос третий?](#вопрос-третий)

---

## Вопрос первый?

Ответ на первый вопрос.

**Связанные задачи:**

- [Название задачи](../../../tasks/frontend/{topic}/1_{topic}_junior.md#название-задачи)

**Материалы для изучения:**

- [Название ресурса](https://example.com)

---

## Вопрос второй?

Ответ на второй вопрос.

**Связанные задачи:**

<!-- Связанных задач нет -->

**Материалы для изучения:**

<!-- Материалы не добавлены -->

---

## Вопрос третий?

Ответ на третий вопрос.

**Связанные задачи:**

- [Название задачи](../../../tasks/frontend/{topic}/1_{topic}_junior.md#название-задачи)

**Материалы для изучения:**

- [MDN: Тема](https://developer.mozilla.org/ru/docs/...)
- [YouTube: Объяснение](https://youtu.be/...)
```

### Task file (`tasks/`)

Each task file must follow this exact structure:

````markdown
# {Topic Name} — {Level} — Задачи

## Задачи

- [Название первой задачи](#название-первой-задачи)
- [Название второй задачи](#название-второй-задачи)

---

## Название первой задачи

**Сложность:** Easy / Medium / Hard

**Связанные вопросы:**

- [Вопрос из Q&A](../../../interviews/frontend/{topic}/1_{topic}_junior.md#вопрос)

**Материалы для изучения:**

- [Название ресурса](https://example.com)
  <!-- Если материала нет: -->
  <!-- Материалы не добавлены -->

**Задание:**

Описание задачи...

**Пример:**

```typescript
// входные данные
// ожидаемый результат
```
````

<details>
<summary>Решение</summary>

```typescript
// реализация
```

</details>

---

## Название второй задачи

**Сложность:** Medium

**Связанные вопросы:**

<!-- Связанных вопросов нет -->

**Материалы для изучения:**

<!-- Материалы не добавлены -->

**Задание:**

...

`````

## Rules

1. **H1 heading**: `# {Topic Name} — {Level}` — e.g. `# React — Junior`, `# TypeScript — Senior`
2. **`## Вопросы`** section contains a bullet list of anchor links to every question in the file
3. **Anchor format**: GitHub Flavored Markdown style — lowercase, spaces → hyphens, special chars removed
   - `Что такое замыкание?` → `#что-такое-замыкание`
   - `Как работает useEffect?` → `#как-работает-useeffect`
   - **Long questions (anchor > 50 chars)**: shorten to the key concept only
     - Full heading: `## Что такое useMemo и useCallback — в чём разница и когда что использовать?`
     - Shortened anchor: `(#usememo-и-usecallback-разница)`
     - The H2 heading stays full — only the anchor link in the list is shortened
4. Each question is an **H2 heading** — this creates the anchor target automatically
5. Separate each Q&A block with `---`
6. **No question repetition** across levels within the same topic folder — each level adds new depth
7. **Question count guidelines**:
   - Junior: 5–8 questions — fundamentals, definitions, basic usage
   - Middle: 6–10 questions — patterns, trade-offs, real-world scenarios
   - Senior: 6–10 questions — architecture, edge cases, performance, internals
   - Expert: 4–7 questions — deep internals, system design, custom implementations
8. **Language**: All questions and answers must be written in **Russian**
9. Code blocks must use the full language name: ` ```typescript `, ` ```javascript `, ` ```bash `
10. **Материалы для изучения** block is **required** after every answer — include real links (MDN, official docs, YouTube) when available, otherwise use the empty comment placeholder `<!-- Материалы не добавлены -->`
11. **Связанные задачи** block is **required** after every answer — link to the corresponding task in `tasks/` when one exists, otherwise use `<!-- Связанных задач нет -->`
12. **Task file H1**: `# {Topic Name} — {Level} — Задачи` — e.g. `# JavaScript — Junior — Задачи`
13. **Task difficulty** is one of: `Easy`, `Medium`, `Hard` — choose based on expected level
14. **Mapping is bidirectional** — if a Q&A question links to a task, that task must link back to the question and vice versa

## Answer Guidelines

- **Junior**: 2–3 предложения + простой пример кода если применимо
- **Middle**: 3–5 предложений + пример с пояснениями, trade-offs
- **Senior**: 4–6 предложений + архитектурная диаграмма (Mermaid) или сложный пример
- **Expert**: 6–8 предложений + ссылки на исходный код / спецификации где уместно
- **Если ответ не применим к уровню**: явно укажи `> Не применимо к этому уровню.` и не оставляй блок пустым
- После **каждого** ответа обязательно добавь блоки `**Связанные задачи:**` и `**Материалы для изучения:**` — даже если они пустые (используй comment-placeholder)

## Mermaid Usage

Для Senior+ вопросов по архитектуре предпочитай Mermaid вместо ASCII-диаграмм.
Оборачивай в блок с языком `mermaid`:

````markdown
```mermaid
sequenceDiagram
    participant U as User
    participant C as Client
    participant S as Server

    U->>C: click
    C->>S: API call
    S-->>C: response
    C-->>U: render
`````

````

Типы диаграмм по ситуации:

- `sequenceDiagram` — жизненный цикл запроса, взаимодействие компонентов
- `graph TD` — архитектура системы, дерево зависимостей
- `flowchart LR` — алгоритмы, flow принятия решений

## Edge Cases

1. **Existing files**:
   - Add new questions at the end of the file
   - If an existing question is outdated or incorrect: keep it, but add a `> **Обновление**: ...` callout below the answer
   - Never delete questions automatically
2. **Missing levels**: If a topic genuinely has no Junior-level questions (e.g., `micro-frontends`), start from Middle and add a note in H1: `# Micro-Frontends — Middle (нет Junior уровня)`
3. **Same-topic deduplication**: Before adding a question, verify it doesn't appear in other level files of the same topic folder.
4. **Cross-topic deduplication**: If a question logically belongs to multiple topics (e.g., «Что такое мемоизация?» fits both `react` and `performance`) — add it to the **most relevant** topic only. In other topics add a reference:
   > См. также: [Что такое мемоизация?](../performance/3_performance_senior.md#что-такое-мемоизация)
5. **No matching task**: If there is no task in `tasks/` for a given question, use the placeholder `<!-- Связанных задач нет -->` — do **not** fabricate task links
6. **No learning material**: If you don't know a reliable link for a question, use `<!-- Материалы не добавлены -->` — do **not** fabricate URLs
7. **Tasks without a matching question**: A task file may exist independently (e.g., algorithms). The `**Связанные вопросы:**` block is still required — use `<!-- Связанных вопросов нет -->` if empty

## Knowledge Dependencies

When creating questions, assume the candidate knows prerequisite topics:

| Topic              | Prerequisites               |
| ------------------ | --------------------------- |
| `typescript`       | JavaScript (any level)      |
| `nextjs`           | React (same level or lower) |
| `graphql`          | REST basics                 |
| `tanstack-query`   | REST basics                 |
| `testing/*`        | The technology being tested |
| `state-management` | React basics                |

If a question requires knowledge outside the assumed prerequisites, add:

> **Prerequisite**: понимание [topic]

## Anchor Validation

Every `[text](#anchor)` must have a matching `## Heading` in the same file.

Slug rules: lowercase → trim → spaces to hyphens → remove all special characters except hyphens.

Examples:

- `Что такое Virtual DOM?` → `#что-такое-virtual-dom`
- `Как работает useEffect?` → `#как-работает-useeffect`

Quick check — find broken anchors with this regex (match = problem):

```
\[.+?\]\(#[^)]*[^\w\-а-яёА-ЯЁ][^)]*\)
```

Valid anchor characters: lowercase letters (including Cyrillic), digits, hyphens only.

## Quality Checklist

Each generated file must pass:

- [ ] Questions match the depth of the declared level (no Junior questions in a Senior file)
- [ ] No question duplicated across files in the same topic folder
- [ ] No question duplicated in a more relevant topic folder (cross-topic check)
- [ ] All anchor links correctly resolve to H2 headings in the same file
- [ ] Long anchors (> 50 chars) are shortened in the link but the H2 heading stays full
- [ ] Each answer has a code example or Mermaid diagram where applicable
- [ ] Questions are realistic interview questions — not trivial or too academic
- [ ] Empty/inapplicable answers use `> Не применимо к этому уровню.` instead of blank
- [ ] Every Q&A answer has a `**Связанные задачи:**` block (with link or placeholder comment)
- [ ] Every Q&A answer has a `**Материалы для изучения:**` block (with link or placeholder comment)
- [ ] Every task has a `**Связанные вопросы:**` block (with link or placeholder comment)
- [ ] Every task has a `**Материалы для изучения:**` block (with link or placeholder comment)
- [ ] Cross-references are bidirectional — if Q links to Task, Task links back to Q
- [ ] No fabricated URLs in materials — use placeholder comment if link is unknown

## Anti-Example (DO NOT do this)

```markdown
## Что такое React?

React — это библиотека. (слишком коротко, нет глубины)
```

**Correct:**

```markdown
## Что такое React и какие проблемы он решает?

React — это JavaScript-библиотека для построения пользовательских интерфейсов.
Решает три ключевые проблемы:

1. **Сложность работы с DOM** → виртуальный DOM для батчинга обновлений
2. **Управление состоянием** → однонаправленный поток данных
3. **Переиспользование кода** → компонентная архитектура
```

---

## Example: React — Junior

```markdown
# React — Junior

## Вопросы

- [Что такое React и зачем он нужен?](#что-такое-react-и-зачем-он-нужен)
- [В чём разница между state и props?](#в-чём-разница-между-state-и-props)
- [Что такое JSX?](#что-такое-jsx)

---

## Что такое React и зачем он нужен?

React — это JavaScript-библиотека для построения UI. Использует компонентную архитектуру
и виртуальный DOM для эффективного обновления интерфейса при изменении данных.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
- [React: официальная документация](https://react.dev/learn)
- [YouTube: Что такое React за 5 минут](https://youtu.be/7TvS0iKR3_c?t=638)

---

## В чём разница между state и props?

- **Props** — входные данные, передаются от родителя к дочернему компоненту, только для чтения.
- **State** — внутреннее изменяемое состояние компонента, управляется им самим.

Props текут вниз, state локален и вызывает ре-рендер при изменении.

**Связанные задачи:**
- [Компонент счётчика с props и state](../../../tasks/frontend/react/1_react_junior.md#компонент-счётчика-с-props-и-state)

**Материалы для изучения:**
- [React: State — A Component's Memory](https://react.dev/learn/state-a-components-memory)

---

## Что такое JSX?

JSX — расширение синтаксиса JavaScript, похожее на HTML. Компилируется Babel в вызовы `React.createElement()`.

\`\`\`typescript
const element = <h1>Hello</h1>;
// компилируется в:
const element = React.createElement('h1', null, 'Hello');
\`\`\`

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
- [React: Writing Markup with JSX](https://react.dev/learn/writing-markup-with-jsx)
```

---

## Example: React — Expert

```markdown
# React — Expert

## Вопросы

- [Как реализовать кастомный React-рендерер?](#как-реализовать-кастомный-react-рендерер)
- [Что такое Fiber и как работает алгоритм согласования?](#что-такое-fiber-и-как-работает-алгоритм-согласования)

---

## Как реализовать кастомный React-рендерер?

Кастомный рендерер реализуется через пакет `react-reconciler`. Необходимо предоставить
host config — объект с методами для работы с целевой платформой.

Ключевые методы host config:

- `createInstance()` — создание узла платформы
- `appendChild()` / `insertBefore()` — добавление в дерево
- `commitUpdate()` — применение изменений после диффинга

\`\`\`typescript
import Reconciler from 'react-reconciler';

const hostConfig = {
  createInstance(type, props) { /* создаём узел */ },
  appendChildToContainer(container, child) { /* монтируем */ },
  commitUpdate(instance, updatePayload, type, oldProps, newProps) { /* патчим */ },
  // ... остальные методы
};

const renderer = Reconciler(hostConfig);
\`\`\`

Именно так реализованы `react-native`, `react-three-fiber`, `ink` (React для терминала).

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
- [react-reconciler: npm](https://www.npmjs.com/package/react-reconciler)
- [Building a Custom React Renderer — Sophie Alpert](https://youtu.be/CGpMlWVcHok)

---

## Что такое Fiber и как работает алгоритм согласования?

Fiber — внутренняя архитектура React (с версии 16), заменившая рекурсивный алгоритм
на итеративный с возможностью прерывания.

Каждый компонент — Fiber-узел со связным списком: `child`, `sibling`, `return`.
Поля `lanes` определяют приоритет обновления.

Работа разделена на две фазы:

1. **Render (прерываемая)** — обход дерева, вычисление diff, без side effects
2. **Commit (синхронная)** — применение изменений к DOM

Это позволяет React прерывать рендеринг низкоприоритетных обновлений в пользу срочных
(анимации, ввод пользователя) — основа Concurrent Mode и Transitions.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
- [React Fiber Architecture — Andrew Clark (GitHub)](https://github.com/acdlite/react-fiber-architecture)
- [Inside Fiber: in-depth overview — Max Koretskyi](https://medium.com/react-in-depth/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-e1c4700ef6f3)
```

## Example: JavaScript — Junior — Задачи (task file)

```markdown
# JavaScript — Junior — Задачи

## Задачи

- [Функция проверки палиндрома](#функция-проверки-палиндрома)
- [Функция поиска гласных в строке](#функция-поиска-гласных-в-строке)

---

## Функция проверки палиндрома

**Сложность:** Easy

**Связанные вопросы:**
- [Что такое чистые функции?](../../../interviews/frontend/javascript/2_javascript_middle.md#что-такое-чистые-функции)

**Материалы для изучения:**
- [YouTube: Разбор задачи](https://youtu.be/ycYp7CYOnO0?t=683)

**Задание:**

Напишите функцию `isPalindrome(str: string): boolean`, которая возвращает `true` если строка является палиндромом (читается одинаково слева и справа, без учёта регистра и пробелов).

**Пример:**

\`\`\`typescript
isPalindrome("racecar")   // true
isPalindrome("A man a plan a canal Panama") // true
isPalindrome("hello")     // false
\`\`\`

<details>
<summary>Решение</summary>

\`\`\`typescript
function isPalindrome(str: string): boolean {
  const clean = str.toLowerCase().replace(/[^a-z0-9]/g, '');
  return clean === clean.split('').reverse().join('');
}
\`\`\`

</details>

---

## Функция поиска гласных в строке

**Сложность:** Easy

**Связанные вопросы:**
<!-- Связанных вопросов нет -->

**Материалы для изучения:**
- [YouTube: Разбор задачи](https://youtu.be/7TvS0iKR3_c?t=807)

**Задание:**

Напишите функцию `findVowels(str: string): number`, которая возвращает количество гласных букв (a, e, i, o, u) в строке.

**Пример:**

\`\`\`typescript
findVowels("hello") // 2
findVowels("why")   // 0
\`\`\`

<details>
<summary>Решение</summary>

\`\`\`typescript
function findVowels(str: string): number {
  return (str.match(/[aeiou]/gi) ?? []).length;
}
\`\`\`

</details>
```
````
