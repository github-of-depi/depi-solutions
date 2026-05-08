# Accessibility — Middle — Задачи

## Задачи

- [Доступная навигация](#доступная-навигация)
- [Доступный список с ARIA-ролями](#доступный-список-с-aria-ролями)

---

## Доступная навигация

**Сложность:** Medium

**Связанные вопросы:**

- [Разница между role и aria-label?](../../../interviews/testing/a11y/2_a11y_middle.md#разница-между-role-и-aria-label)
- [Как строить keyboard navigation?](../../../interviews/testing/a11y/2_a11y_middle.md#как-строить-keyboard-navigation)

**Материалы для изучения:**

- [WAI-ARIA: Navigation Landmark](https://www.w3.org/TR/wai-aria-1.2/#navigation)
- [WebAIM: Navigation](https://webaim.org/techniques/hypertext/link_text)

**Задание:**

На странице есть три навигационных блока и кнопка с иконкой. Найдите проблемы доступности и исправьте их.

```html
<!-- Проблемная разметка -->

<!-- 1. Две навигации без различия -->
<nav>
  <a href="/">Главная</a>
  <a href="/products">Продукты</a>
</nav>

<nav>
  <a href="/about">О нас</a>
  <a href="/contact">Контакты</a>
</nav>

<!-- 2. Хлебные крошки -->
<nav>
  <ol>
    <li><a href="/">Главная</a></li>
    <li><a href="/products">Продукты</a></li>
    <li>Ноутбуки</li>
  </ol>
</nav>

<!-- 3. Кнопки действий в таблице -->
<table>
  <tr>
    <td>Продукт A</td>
    <td>
      <button><svg><!-- edit icon --></svg></button>
      <button><svg><!-- delete icon --></svg></button>
    </td>
  </tr>
</table>
```

<details>
<summary>Решение</summary>

**Проблема 1**: несколько `<nav>` без различия. Скринридер скажет «навигация, навигация» — пользователь не понимает разницы.

```html
<nav aria-label="Основная навигация">
  <a href="/">Главная</a>
  <a href="/products">Продукты</a>
</nav>

<nav aria-label="Вспомогательная навигация">
  <a href="/about">О нас</a>
  <a href="/contact">Контакты</a>
</nav>
```

**Проблема 2**: хлебные крошки нужно явно пометить и указать текущий элемент.

```html
<nav aria-label="Хлебные крошки">
  <ol>
    <li><a href="/">Главная</a></li>
    <li><a href="/products">Продукты</a></li>
    <!-- aria-current="page" — текущая страница -->
    <li><span aria-current="page">Ноутбуки</span></li>
  </ol>
</nav>
```

**Проблема 3**: кнопки-иконки без описания и без контекста строки.

```html
<table>
  <tr>
    <td>Продукт A</td>
    <td>
      <!-- Имя включает название продукта для контекста -->
      <button aria-label="Редактировать Продукт A">
        <svg aria-hidden="true" focusable="false"><!-- edit icon --></svg>
      </button>
      <button aria-label="Удалить Продукт A">
        <svg aria-hidden="true" focusable="false"><!-- delete icon --></svg>
      </button>
    </td>
  </tr>
</table>
```

</details>

---

## Доступный список с ARIA-ролями

**Сложность:** Medium

**Связанные вопросы:**

- [Разница между role и aria-label?](../../../interviews/testing/a11y/2_a11y_middle.md#разница-между-role-и-aria-label)
- [Что такое ARIA?](../../../interviews/testing/a11y/1_a11y_junior.md#что-такое-aria)

**Материалы для изучения:**

- [MDN: ARIA listbox role](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/listbox_role)
- [WAI-ARIA Authoring Practices: Listbox](https://www.w3.org/WAI/ARIA/apg/patterns/listbox/)

**Задание:**

Реализуйте кастомный список выбора (аналог `<select>`) с помощью `<div>` и ARIA-атрибутов. Список должен быть полностью доступен для скринридеров и клавиатурных пользователей.

Требования:
- Скринридер должен объявлять элемент как «список», а каждый пункт — как «вариант»
- Выбранный вариант должен быть помечен
- Список должен фокусироваться и быть доступен с клавиатуры
- Название списка должно быть доступно

```html
<!-- Заготовка без ARIA -->
<div class="custom-select">
  <span>Выберите язык программирования</span>
  <div class="options">
    <div class="option selected">JavaScript</div>
    <div class="option">TypeScript</div>
    <div class="option">Python</div>
  </div>
</div>
```

<details>
<summary>Решение</summary>

```html
<div class="custom-select">
  <!-- Лейбл — связываем через id -->
  <span id="lang-label">Выберите язык программирования</span>

  <!--
    role="listbox" — сообщает скринридеру, что это список выбора
    aria-labelledby — привязывает видимый лейбл
    tabindex="0" — делает контейнер фокусируемым
    aria-activedescendant — указывает текущий активный элемент
  -->
  <div
    role="listbox"
    aria-labelledby="lang-label"
    aria-activedescendant="opt-js"
    tabindex="0"
    class="options"
  >
    <!--
      role="option" — каждый пункт
      aria-selected="true" — выбранный вариант
      id — нужен для aria-activedescendant
    -->
    <div role="option" id="opt-js" aria-selected="true" class="option selected">
      JavaScript
    </div>
    <div role="option" id="opt-ts" aria-selected="false" class="option">
      TypeScript
    </div>
    <div role="option" id="opt-py" aria-selected="false" class="option">
      Python
    </div>
  </div>
</div>
```

```javascript
// Минимальная клавиатурная навигация
const listbox = document.querySelector('[role="listbox"]');
const options = [...listbox.querySelectorAll('[role="option"]')];

listbox.addEventListener('keydown', (e) => {
  const currentId = listbox.getAttribute('aria-activedescendant');
  const currentIndex = options.findIndex(o => o.id === currentId);

  if (e.key === 'ArrowDown') {
    const next = options[Math.min(currentIndex + 1, options.length - 1)];
    listbox.setAttribute('aria-activedescendant', next.id);
    next.scrollIntoView({ block: 'nearest' });
    e.preventDefault();
  }
  if (e.key === 'ArrowUp') {
    const prev = options[Math.max(currentIndex - 1, 0)];
    listbox.setAttribute('aria-activedescendant', prev.id);
    e.preventDefault();
  }
  if (e.key === 'Enter' || e.key === ' ') {
    options.forEach(o => o.setAttribute('aria-selected', 'false'));
    const active = document.getElementById(listbox.getAttribute('aria-activedescendant')!);
    active?.setAttribute('aria-selected', 'true');
    e.preventDefault();
  }
});
```

> **Примечание**: в реальных проектах предпочтительнее использовать нативный `<select>` (доступен из коробки) или библиотеки Radix UI / React Aria, которые реализуют паттерн корректно во всех edge cases.

</details>
