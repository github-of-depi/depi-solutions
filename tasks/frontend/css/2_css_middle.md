# CSS — Middle — Задачи

## Задачи

- [Схлопывание отступов](#схлопывание-отступов)
- [CSS-анимация по производительности](#css-анимация-по-производительности)

---

## Схлопывание отступов

**Сложность:** Medium

**Связанные вопросы:**

- [Что такое схлопывание отступов (margin collapsing)?](../../../interviews/frontend/css/2_css_middle.md#что-такое-схлопывание-отступов-margin-collapsing)

**Материалы для изучения:**

- [MDN: Схлопывание внешних отступов](https://developer.mozilla.org/ru/docs/Web/CSS/CSS_box_model/Mastering_margin_collapsing)

**Задание:**

Ответьте на три вопроса о схлопывании отступов, не запуская код.

**Часть 1.** Каким будет расстояние между блоками A и B?

```html
<style>
  .a { margin-bottom: 32px; }
  .b { margin-top: 20px; }
</style>

<div class="a">A</div>
<div class="b">B</div>
```

**Часть 2.** Сколько пикселей отступа сверху у `.child` относительно страницы?

```html
<style>
  .parent { margin-top: 40px; }
  .child  { margin-top: 20px; }
</style>

<div class="parent">
  <div class="child">Контент</div>
</div>
```

**Часть 3.** Как исправить схлопывание в части 2, не меняя значения margin?

<details>
<summary>Решение</summary>

**Часть 1:** расстояние = **32px** (max из 32 и 20, не сумма)

**Часть 2:** у `.child` — **40px** сверху. Из-за схлопывания margin родителя и первого ребёнка оба margin объединяются в один (max = 40px). `.parent` как будто сам начинается от верха страницы без отступа.

**Часть 3:** исправление — создать BFC для `.parent` любым из способов:

```css
/* Вариант 1: padding-top (любой ненулевой) */
.parent { padding-top: 1px; }

/* Вариант 2: border */
.parent { border-top: 1px solid transparent; }

/* Вариант 3: overflow */
.parent { overflow: hidden; }

/* Вариант 4: display: flow-root (рекомендуется — нет побочных эффектов) */
.parent { display: flow-root; }
```

После любого из этих изменений отступы не схлопываются: `.parent` получит `margin-top: 40px`, `.child` — `margin-top: 20px` относительно родителя → итоговое смещение `.child` от верха страницы = 60px.

</details>

---

## CSS-анимация по производительности

**Сложность:** Medium

**Связанные вопросы:**

- [Когда использовать translate() вместо position: absolute?](../../../interviews/frontend/css/3_css_senior.md#когда-использовать-translate-вместо-position-absolute)
- [Как работают CSS-анимации и переходы?](../../../interviews/frontend/css/2_css_middle.md#как-работают-css-анимации-и-переходы)

**Материалы для изучения:**

- [web.dev: Stick to Compositor-Only Properties](https://web.dev/articles/stick-to-compositor-only-properties-and-manage-layer-count)

**Задание:**

Перед вами анимированное меню, реализованное через `left`. Перепишите анимацию так, чтобы она не вызывала layout/paint и работала через compositor-only свойства.

```html
<style>
  .menu {
    position: fixed;
    top: 0;
    left: -280px;   /* скрыто */
    width: 280px;
    height: 100vh;
    background: white;
    transition: left 0.3s ease;
  }
  .menu.open {
    left: 0;        /* видно */
  }
</style>

<nav class="menu">...</nav>
<button onclick="document.querySelector('.menu').classList.toggle('open')">
  Меню
</button>
```

**Требования к решению:**
- Вместо `left` использовать `transform: translateX()`
- Анимация должна оставаться плавной и не вызывать reflow
- Поведение (скрыт/показан) должно быть идентичным оригиналу

<details>
<summary>Решение</summary>

```html
<style>
  .menu {
    position: fixed;
    top: 0;
    left: 0;
    width: 280px;
    height: 100vh;
    background: white;
    /* Перемещаем за левую границу экрана через transform */
    transform: translateX(-100%);
    transition: transform 0.3s ease;
    /* Рекомендуется для явного создания compositor layer */
    will-change: transform;
  }

  .menu.open {
    transform: translateX(0);
  }
</style>

<nav class="menu">...</nav>
<button onclick="document.querySelector('.menu').classList.toggle('open')">
  Меню
</button>
```

**Почему это лучше:**
- `translateX` обрабатывается на compositor thread, не запускает layout/paint
- `-100%` — процент от ширины самого элемента (280px), поэтому всегда корректно скрывает меню
- `will-change: transform` сигнализирует браузеру создать отдельный compositor layer заранее (убрать после окончания использования если элемент долго не анимируется)

</details>
