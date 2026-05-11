# CSS — Senior

## Вопросы

- [Как CSS влияет на производительность рендеринга?](#как-css-влияет-на-производительность-рендеринга)
- [Что такое CSS containment?](#что-такое-css-containment)
- [Что такое CSS Layers (@layer)?](#что-такое-css-layers-layer)
- [Как работает will-change и когда его применять?](#как-работает-will-change-и-когда-его-применять)
- [Что такое критический CSS?](#что-такое-критический-css)
- [Как обеспечить кроссбраузерность в 2025 году?](#как-обеспечить-кроссбраузерность-в-2025-году)
- [Когда использовать translate() вместо position: absolute?](#когда-использовать-translate-вместо-position-absolute)
- [Что такое CSS filter и как он влияет на stacking context?](#что-такое-css-filter-и-как-он-влияет-на-stacking-context)
- [Минимизация и сжатие CSS?](#минимизация-и-сжатие-css)
- [Уменьшение HTTP-запросов?](#уменьшение-http-запросов)
- [Кэширование общих файлов?](#кэширование-общих-файлов)
- [Lazy-loading ресурсов?](#lazy-loading-ресурсов)
- [Затратные CSS-свойства для браузера?](#затратные-css-свойства-для-браузера)
- [@page — стили для печати?](#page--стили-для-печати)
- [Что такое CSSOM?](#что-такое-cssom)
- [Отладка CSS?](#отладка-css)

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

---

## Когда использовать translate() вместо position: absolute?

`transform: translate()` перемещает элемент без выхода из потока и не вызывает layout — смещение происходит на этапе composite, что в разы дешевле. `position: absolute` меняет геометрию страницы и при анимации вызывает layout на каждом кадре.

```css
/* Плохо для анимации — вызывает reflow на каждом кадре */
@keyframes slide-bad {
  from { left: 0; }
  to { left: 300px; }
}

/* Хорошо — только composite, GPU-ускорение */
@keyframes slide-good {
  from { transform: translateX(0); }
  to { transform: translateX(300px); }
}
```

Типичный паттерн центрирования через `translate` также предпочтительнее `position` + отрицательных `margin`:

```css
/* Современное центрирование */
.centered {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
```

Используй `position: absolute` для размещения элементов в DOM-структуре, `translate` — для анимаций и визуальных смещений.

**Связанные задачи:**

- [CSS-анимация по производительности](../../../tasks/frontend/css/2_css_middle.md#css-анимация-по-производительности)

**Материалы для изучения:**

- [MDN: transform](https://developer.mozilla.org/ru/docs/Web/CSS/transform)
- [web.dev: Stick to Compositor-Only Properties](https://web.dev/articles/stick-to-compositor-only-properties-and-manage-layer-count)

---

## Что такое CSS filter и как он влияет на stacking context?

`filter` применяет графические эффекты к элементу и его потомкам: размытие, насыщенность, контрастность, тени, оттенки. Ключевой побочный эффект: **любое значение `filter` кроме `none` создаёт новый stacking context**, что может сломать ожидаемое поведение `z-index` у дочерних элементов.

```css
.blur       { filter: blur(4px); }
.grayscale  { filter: grayscale(100%); }
.dark-mode  { filter: invert(90%) hue-rotate(180deg); }

/* Комбинирование фильтров */
.card:hover {
  filter: brightness(1.1) drop-shadow(0 4px 12px rgba(0,0,0,0.3));
}
```

`drop-shadow()` в отличие от `box-shadow` обтекает фактическую форму элемента (включая прозрачные части PNG), а не его прямоугольник.

```css
/* box-shadow — тень вокруг прямоугольника */
.icon { box-shadow: 0 4px 8px black; }

/* drop-shadow — тень по контуру изображения */
.icon { filter: drop-shadow(0 4px 8px black); }
```

**Связанные задачи:**

<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: filter](https://developer.mozilla.org/ru/docs/Web/CSS/filter)

---

## Минимизация и сжатие CSS?

**Минификация** — удаление пробелов, комментариев, сокращение цветов (`#ffffff` → `#fff`). Делают: PostCSS cssnano, LightningCSS, esbuild.

**Сжатие** — Gzip/Brotli на уровне сервера/CDN. Brotli даёт 15–25% лучше чем Gzip.

```bash
# Проверить сжатие
curl -H "Accept-Encoding: br" -I https://example.com/style.css
# Content-Encoding: br

# Build pipeline
vite build  # встроенная минификация + tree-shaking
```

**Дополнительно:**
- **PurgeCSS / UnCSS** — удаление неиспользуемых правил
- **Critical CSS** — встраивать только необходимое inline, остальное async
- **CSS-in-JS** — автоматически отправляет только используемые стили

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Уменьшение HTTP-запросов?

1. **Объединение файлов** — bundler собирает CSS в один файл
2. **CSS-спрайты** — иконки в одном изображении
3. **Inline критические стили** — `<style>` в head
4. **Данные в Base64** — маленькие иконки прямо в CSS
5. **HTTP/2** — мультиплексирование, отдельные запросы менее критичны
6. **CDN** — кэширование общих библиотек

```css
/* Inline небольшого SVG в CSS */
.icon {
  background-image: url("data:image/svg+xml,%3Csvg...%3E%3C/svg%3E");
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Кэширование общих файлов?

**Content-based hashing** — `styles.abc123.css`. При изменении хэш меняется, браузер скачивает заново. Без изменений — берёт из кэша бесконечно.

```html
<!-- Bundler генерирует: -->
<link rel="stylesheet" href="/assets/index.a1b2c3d4.css">
```

**Cache-Control заголовки:**
```
Cache-Control: max-age=31536000, immutable  # для хэшированных файлов
Cache-Control: no-cache                      # для index.html
```

**Service Worker** — программный контроль кэша, offline-поддержка.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Lazy-loading ресурсов?

**Изображения:**
```html
<img src="image.jpg" loading="lazy" alt="">
<!-- loading="lazy" — нативная поддержка Chrome/Firefox/Safari -->
```

**CSS-фоны (через Intersection Observer):**
```javascript
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.classList.add('loaded');
      observer.unobserve(entry.target);
    }
  });
});
document.querySelectorAll('.lazy-bg').forEach(el => observer.observe(el));
```

```css
.lazy-bg { background: #f0f0f0; }
.lazy-bg.loaded { background-image: url('hero.jpg'); }
```

**Шрифты:** `font-display: optional` — не загружать если медленно.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Затратные CSS-свойства для браузера?

**Дорогие** (вызывают layout/reflow):
- `width`, `height`, `top`, `left`, `margin`, `padding`, `border`
- `font-size`, `font-weight`, `overflow`, `display`, `position`

**Средние** (только paint):
- `color`, `background-color`, `border-color`, `box-shadow`
- `visibility`, `outline`

**Дешёвые** (только composite — GPU):
- `transform`, `opacity`, `filter` (с некоторыми функциями)

```css
/* Плохо — анимировать через top/left */
.moving { top: 100px; transition: top 0.3s; }

/* Хорошо — анимировать через transform */
.moving { transform: translateY(100px); transition: transform 0.3s; }
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## @page — стили для печати?

```css
/* Базовые print-стили */
@media print {
  nav, .sidebar, .ads { display: none; }
  body { font-size: 12pt; color: black; }
  a::after { content: ' (' attr(href) ')'; }
}

/* @page — настройки страницы */
@page {
  size: A4 portrait;
  margin: 2cm;
}

@page :first {
  margin-top: 5cm; /* особые отступы для первой страницы */
}

/* Управление разрывами */
h2 { break-before: page; }        /* новая страница перед h2 */
.keep-together { break-inside: avoid; } /* не разрывать блок */
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое CSSOM?

**CSSOM** (CSS Object Model) — дерево CSS-правил в памяти, аналог DOM для HTML. Браузер строит CSSOM при парсинге CSS. Вместе с DOM формирует **Render Tree**.

```javascript
// Доступ к CSSOM через JS
const sheets = document.styleSheets;
const rules = sheets[0].cssRules;

// Чтение вычисленных стилей
const computed = window.getComputedStyle(element);
computed.getPropertyValue('color'); // 'rgb(0, 0, 0)'

// Динамическое изменение
element.style.color = 'red'; // inline стиль

// CSSStyleDeclaration
document.styleSheets[0].insertRule('.new { color: blue; }', 0);
```

**Блокировка рендеринга:** браузер не рендерит страницу, пока не построит CSSOM. Поэтому CSS должен загружаться быстро. `<link>` в `<head>` — страница ждёт загрузки CSS перед рендерингом.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Отладка CSS?

**Chrome DevTools:**
- **Elements → Styles** — все применённые правила с источником
- **Computed** — итоговые вычисленные значения
- **Layout** — Box Model визуально
- **Layers** — compositor layers
- **Coverage** — неиспользуемые CSS-правила (Coverage tab)

**Полезные приёмы:**
```css
/* Временная подсветка всех блоков */
* { outline: 1px solid red !important; }

/* Найти overflow */
* { outline: 1px solid rgba(255,0,0,0.2); }
```

**Специфичность в DevTools:** зачёркнутые правила = перебиты правилом с большей специфичностью.

**CSS Custom Properties отладка:**
```javascript
getComputedStyle(el).getPropertyValue('--color'); // читать
el.style.setProperty('--color', 'blue');           // писать
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
