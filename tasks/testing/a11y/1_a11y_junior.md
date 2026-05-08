# Accessibility — Junior — Задачи

## Задачи

- [Доступная кнопка-иконка](#доступная-кнопка-иконка)
- [Skip-link реализация](#skip-link-реализация)

---

## Доступная кнопка-иконка

**Сложность:** Easy

**Связанные вопросы:**

- [Как скрыть элемент от скринридеров и как сделать его видимым только для них?](../../../interviews/testing/a11y/1_a11y_junior.md#как-скрыть-элемент-от-скринридеров-и-как-сделать-его-видимым-только-для-них)
- [Что такое ARIA?](../../../interviews/testing/a11y/1_a11y_junior.md#что-такое-aria)

**Материалы для изучения:**

- [MDN: aria-label](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-label)

**Задание:**

Перед вами три варианта кнопки-иконки. Найдите проблемы доступности в каждом и исправьте их.

```html
<!-- Вариант A: кнопка удаления -->
<button>
  <svg width="16" height="16" viewBox="0 0 16 16">
    <path d="M2 4h12M5 4V2h6v2M6 7v5M10 7v5M3 4l1 10h8l1-10" />
  </svg>
</button>

<!-- Вариант B: кнопка закрытия -->
<div onclick="closeModal()">✕</div>

<!-- Вариант C: иконка + дублирующий текст -->
<button>
  <img src="save-icon.png" alt="Сохранить" />
  Сохранить
</button>
```

<details>
<summary>Решение</summary>

**Вариант A** — SVG без текстового описания, скринридер не знает назначения кнопки:

```html
<button aria-label="Удалить запись">
  <svg aria-hidden="true" width="16" height="16" viewBox="0 0 16 16" focusable="false">
    <path d="M2 4h12M5 4V2h6v2M6 7v5M10 7v5M3 4l1 10h8l1-10" />
  </svg>
</button>
```

**Вариант B** — `<div>` не интерактивен по умолчанию: нет роли, нет фокуса, нет клавиатурного управления. Лучше использовать нативный `<button>`:

```html
<button aria-label="Закрыть" onclick="closeModal()">✕</button>
```

Если `<div>` принципиален (legacy-код): добавить `role="button"`, `tabindex="0"`, обработчик `keydown` для Enter/Space. Но нативный `<button>` всегда предпочтительнее.

**Вариант C** — `alt` изображения дублирует текст кнопки. Скринридер скажет «Сохранить, Сохранить, кнопка». Декоративная иконка должна иметь `alt=""`:

```html
<button>
  <img src="save-icon.png" alt="" />
  Сохранить
</button>
```

</details>

---

## Skip-link реализация

**Сложность:** Easy

**Связанные вопросы:**

- [Что такое skip-links?](../../../interviews/testing/a11y/1_a11y_junior.md#что-такое-skip-links)

**Материалы для изучения:**

- [WebAIM: Skip Navigation Links](https://webaim.org/techniques/skipnav/)

**Задание:**

Реализуйте skip-link для следующей страницы. Ссылка должна:
1. Появляться только при фокусе (через Tab)
2. Перемещать фокус на основной контент
3. Быть первым фокусируемым элементом на странице

```html
<!-- Заготовка страницы -->
<body>
  <!-- skip-link здесь -->

  <header>
    <nav>
      <a href="/">Главная</a>
      <a href="/about">О нас</a>
      <a href="/contact">Контакты</a>
    </nav>
  </header>

  <main>
    <h1>Добро пожаловать</h1>
    <p>Основной контент страницы.</p>
  </main>
</body>
```

<details>
<summary>Решение</summary>

```html
<body>

  <a href="#main-content" class="skip-link">
    Перейти к основному содержимому
  </a>

  <header>
    <nav>
      <a href="/">Главная</a>
      <a href="/about">О нас</a>
      <a href="/contact">Контакты</a>
    </nav>
  </header>

  <!-- tabindex="-1" позволяет принять программный фокус -->
  <main id="main-content" tabindex="-1">
    <h1>Добро пожаловать</h1>
    <p>Основной контент страницы.</p>
  </main>

</body>
```

```css
.skip-link {
  position: absolute;
  top: -100%;
  left: 8px;
  z-index: 9999;
  padding: 8px 16px;
  background: #000;
  color: #fff;
  text-decoration: none;
  border-radius: 0 0 4px 4px;
  font-size: 14px;
}

/* Показываем только когда получает фокус */
.skip-link:focus {
  top: 0;
}

/* Убираем нативный outline у main — фокус только программный */
main:focus {
  outline: none;
}
```

**Важные детали:**
- Ссылка должна быть **первым элементом** в `<body>` — перед любой навигацией
- `tabindex="-1"` на `<main>`: без него браузер не переместит фокус на нефокусируемый элемент при переходе по якорной ссылке
- `main:focus { outline: none }` — убираем кольцо у `<main>`, так как это программный, не пользовательский фокус

</details>
