# CSS — Junior — Задачи

## Задачи

- [Позиционирование элементов](#позиционирование-элементов)
- [Скрытие элементов](#скрытие-элементов)
- [Специфичность селекторов](#специфичность-селекторов)

---

## Позиционирование элементов

**Сложность:** Easy

**Связанные вопросы:**

- [Типы позиционирования в CSS?](../../../interviews/frontend/css/1_css_junior.md#типы-позиционирования-в-css)

**Материалы для изучения:**

- [MDN: position](https://developer.mozilla.org/ru/docs/Web/CSS/position)

**Задание:**

Реализуйте три компонента только на CSS (без изменения HTML):

1. **Тултип** — абсолютно позиционированный блок `.tooltip`, появляющийся над кнопкой при наведении (`:hover`). Кнопка является точкой отсчёта для тултипа.
2. **Плавающая кнопка** — элемент `.fab`, зафиксированный в нижнем правом углу экрана (24px от краёв) и остающийся на месте при прокрутке.
3. **Sticky-заголовок** — `<thead>` таблицы, который прилипает к верхнему краю viewport при прокрутке содержимого.

**Пример HTML:**

```html
<!-- 1. Тултип -->
<div class="tooltip-wrapper">
  <button>Наведи на меня</button>
  <div class="tooltip">Подсказка</div>
</div>

<!-- 2. FAB -->
<button class="fab">+</button>

<!-- 3. Sticky thead -->
<div class="table-wrapper" style="height: 300px; overflow-y: auto;">
  <table>
    <thead>
      <tr><th>Имя</th><th>Возраст</th></tr>
    </thead>
    <tbody><!-- строки --></tbody>
  </table>
</div>
```

<details>
<summary>Решение</summary>

```css
/* 1. Тултип */
.tooltip-wrapper {
  position: relative;
  display: inline-block;
}

.tooltip {
  position: absolute;
  bottom: calc(100% + 8px);
  left: 50%;
  transform: translateX(-50%);
  background: #333;
  color: #fff;
  padding: 4px 8px;
  border-radius: 4px;
  white-space: nowrap;
  /* Скрыт по умолчанию */
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.2s;
}

.tooltip-wrapper:hover .tooltip {
  opacity: 1;
}

/* 2. FAB */
.fab {
  position: fixed;
  bottom: 24px;
  right: 24px;
  width: 56px;
  height: 56px;
  border-radius: 50%;
  background: #3b82f6;
  color: #fff;
  font-size: 24px;
  border: none;
  cursor: pointer;
  z-index: 100;
}

/* 3. Sticky thead */
thead th {
  position: sticky;
  top: 0;
  background: white;
  z-index: 1; /* поверх содержимого tbody */
}
```

</details>

---

## Скрытие элементов

**Сложность:** Easy

**Связанные вопросы:**

- [Разница между display: none, visibility: hidden и opacity: 0?](../../../interviews/frontend/css/1_css_junior.md#разница-между-display-none-visibility-hidden-и-opacity-0)

**Материалы для изучения:**

- [MDN: visibility](https://developer.mozilla.org/ru/docs/Web/CSS/visibility)

**Задание:**

Напишите CSS для четырёх вариантов скрытия элемента. Для каждого определите:
- остаётся ли место в документе?
- доступен ли элемент для скринридеров?
- кликабелен ли он?

Затем реализуйте **доступное скрытие** (visually-hidden) — элемент невидим, но доступен для скринридеров.

**Пример:**

```html
<div class="hidden-display">A</div>
<div class="hidden-visibility">B</div>
<div class="hidden-opacity">C</div>
<div class="visually-hidden">Текст только для скринридера</div>
```

<details>
<summary>Решение</summary>

```css
/* display:none — убирает из потока и из accessibility tree */
.hidden-display { display: none; }

/* visibility:hidden — место занято, скринридеры не читают, клики не проходят */
.hidden-visibility { visibility: hidden; }

/* opacity:0 — место занято, скринридеры ЧИТАЮТ, клики ПРОХОДЯТ */
.hidden-opacity { opacity: 0; }

/* Visually hidden — невидимо, но доступно скринридерам (стандартный паттерн) */
.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

| Класс | Место в потоке | Скринридер | Клики |
|---|---|---|---|
| `.hidden-display` | Нет | Нет | Нет |
| `.hidden-visibility` | Да | Нет | Нет |
| `.hidden-opacity` | Да | Да | Да |
| `.visually-hidden` | Нет (1px) | **Да** | Нет |

</details>

---

## Специфичность селекторов

**Сложность:** Easy

**Связанные вопросы:**

- [Что такое специфичность (specificity) CSS?](../../../interviews/frontend/css/1_css_junior.md#что-такое-специфичность-specificity-css)

**Материалы для изучения:**

- [MDN: Specificity](https://developer.mozilla.org/ru/docs/Web/CSS/Specificity)

**Задание:**

Определите, какой цвет получит каждый элемент и почему. Объясните через (a, b, c) веса.

```html
<style>
  .block { color: red; }
  .block.active { color: blue; }
  div.block { color: green; }
  #unique { color: purple; }
</style>

<!-- Вопрос 1: какой цвет? -->
<div class="block">Текст 1</div>

<!-- Вопрос 2: какой цвет? -->
<div class="block active">Текст 2</div>

<!-- Вопрос 3: какой цвет? -->
<div id="unique" class="block active">Текст 3</div>
```

Дополнительный вопрос: если добавить `color: orange !important` к `.block` — что изменится?

<details>
<summary>Решение</summary>

Веса:
- `.block` → (0,0,1,0) = 10
- `.block.active` → (0,0,2,0) = 20
- `div.block` → (0,0,1,1) = 11
- `#unique` → (0,1,0,0) = 100

**Текст 1** — применяются `.block` (10) и `div.block` (11) → `div.block` выигрывает → **зелёный**

**Текст 2** — применяются все три без `#unique`: `.block` (10), `div.block` (11), `.block.active` (20) → `.block.active` выигрывает → **синий**

**Текст 3** — `#unique` (100) выигрывает у всех → **фиолетовый**

Если добавить `color: orange !important` к `.block`: `!important` перебивает всё кроме другого `!important`. Все три элемента станут **оранжевыми**, независимо от специфичности остальных правил.

</details>
