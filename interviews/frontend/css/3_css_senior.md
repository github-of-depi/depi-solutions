# CSS — Senior

## Вопросы

- [Как CSS влияет на производительность рендеринга?](#как-css-влияет-на-производительность-рендеринга)
- [Что такое CSS containment?](#что-такое-css-containment)
- [Что такое CSS Layers (@layer)?](#что-такое-css-layers-layer)
- [Как работает will-change и когда его применять?](#как-работает-will-change-и-когда-его-применять)
- [Что такое критический CSS?](#что-такое-критический-css)
- [Как обеспечить кроссбраузерность в 2025 году?](#как-обеспечить-кроссбраузерность-в-2025-году)

---

## Как CSS влияет на производительность рендеринга?

Браузер строит Render Tree из DOM + CSSOM. Изменение свойств вызывает: **Layout** (reflow) — пересчёт геометрии (width, height, margin, top), **Paint** — перерисовка пикселей (color, background), **Composite** — объединение слоёв (transform, opacity). Анимируй только transform и opacity — они пропускают layout и paint.

```css
/* Только composite — GPU, без layout/paint */
.fast { transform: translateX(100px); opacity: 0.5; }

/* Вызывает layout на каждом кадре — медленно */
.slow { left: 100px; }
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое CSS containment?

`contain` изолирует элемент от влияния на остальную страницу. `contain: layout` — изменения внутри не влияют на внешний layout. `contain: paint` — содержимое не выходит за границы. `content` = `layout paint`. Используется браузером для оптимизации: не нужно пересчитывать всё дерево при изменении изолированного элемента.

```css
.card { contain: layout paint; }
/* content-visibility — lazy rendering off-screen элементов */
.card-list-item { content-visibility: auto; contain-intrinsic-size: 0 200px; }
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое CSS Layers (@layer)?

`@layer` управляет порядком каскада между группами стилей, не опираясь на специфичность. Стили в ранних слоях проигрывают поздним. Решает конфликты при интеграции сторонних библиотек.

```css
@layer reset, base, components, utilities;

@layer reset { * { box-sizing: border-box; margin: 0; } }
@layer components { .btn { padding: 8px 16px; } }
@layer utilities { .mt-4 { margin-top: 16px; } } /* выигрывает у components */
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работает will-change и когда его применять?

`will-change` подсказывает браузеру заранее создать compositor layer для элемента. Ускоряет анимации, но потребляет память. Применяй только для элементов с реальными анимациями, добавляй перед анимацией и убирай после.

```css
/* Плохо — на всех элементах */
* { will-change: transform; }

/* Хорошо — только на анимируемых, через JavaScript */
element.addEventListener("mouseenter", () => el.style.willChange = "transform");
element.addEventListener("animationend", () => el.style.willChange = "auto");
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое критический CSS?

Критический CSS — инлайн-стили, необходимые для рендера above-the-fold контента. Вставляется в `<style>` тег в `<head>`, остальной CSS загружается асинхронно. Улучшает FCP и LCP, устраняя render-blocking ресурсы. Инструменты: `critical`, `Penthouse`, Vite плагины.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как обеспечить кроссбраузерность в 2025 году?

1. **Baseline** (web.dev/baseline) — проверять поддержку фич
2. **Can I Use** для конкретных свойств
3. **Autoprefixer** — автоматические vendor prefixes
4. **Feature detection** через `@supports` вместо browser detection
5. **Graceful degradation** — продвинутые фичи как enhancement

```css
/* Feature detection */
@supports (display: grid) {
  .container { display: grid; }
}
@supports not (display: grid) {
  .container { display: flex; flex-wrap: wrap; }
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
