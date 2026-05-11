# HTML — Senior

## Вопросы

- [Как HTML влияет на производительность страницы?](#как-html-влияет-на-производительность-страницы)
- [Для чего используется тег template?](#для-чего-используется-тег-template)
- [Для чего используется тег dialog?](#для-чего-используется-тег-dialog)
- [Оптимизация загрузки изображений в HTML?](#оптимизация-загрузки-изображений-в-html)
- [Атрибуты inputmode, enterkeyhint и capture?](#атрибуты-inputmode-enterkeyhint-и-capture)
- [Особенности стилизации SVG в HTML?](#особенности-стилизации-svg-в-html)
- [Чем отличается iframe от embed?](#чем-отличается-iframe-от-embed)
- [Что такое PWA (прогрессивное веб-приложение)?](#что-такое-pwa-прогрессивное-веб-приложение)
- [Email-вёрстка — особенности?](#email-вёрстка--особенности)
- [Мультиязычность (i18n) в HTML?](#мультиязычность-i18n-в-html)
- [Как происходит загрузка сайта?](#как-происходит-загрузка-сайта)
- [SSR — преимущества и недостатки?](#ssr--преимущества-и-недостатки)
- [Изящная деградация и постепенное улучшение?](#изящная-деградация-и-постепенное-улучшение)
- [Google Lighthouse — что такое?](#google-lighthouse--что-такое)

---

## Как HTML влияет на производительность страницы?

HTML-разметка непосредственно влияет на скорость загрузки и рендеринга. Ключевые практики:

1. **Загрузка скриптов** — `<script defer>` не блокирует парсинг HTML и выполняется после построения DOM; `async` подходит для независимых скриптов без зависимостей.
2. **Resource hints** — `<link rel="preload">` загружает критические ресурсы текущей страницы заранее; `<link rel="prefetch">` заготавливает ресурсы для следующей страницы.
3. **Изображения** — `loading="lazy"` откладывает загрузку внеэкранных изображений; `fetchpriority="high"` повышает приоритет LCP-изображения; `decoding="async"` выносит декодирование из основного потока.
4. **Размеры медиа** — явные `width`/`height` у `<img>` и `<video>` позволяют браузеру резервировать место заранее, предотвращая CLS (Cumulative Layout Shift).
5. **Порядок ресурсов** — критический CSS до скриптов, шрифты через `<link rel="preload">` с `as="font"`.

```html
<head>
  <link rel="preload" href="critical.css" as="style" />
  <link rel="preload" href="hero.webp" as="image" />
  <link rel="prefetch" href="/next-page.js" />
</head>

<img src="hero.webp" fetchpriority="high" decoding="async" width="1200" height="600" alt="Hero" />
<img src="card.webp" loading="lazy" decoding="async" width="400" height="300" alt="Карточка" />

<script src="app.js" defer></script>
```

**Связанные задачи:**

<!-- Связанных задач нет -->

**Материалы для изучения:**

- [web.dev: Resource hints](https://web.dev/learn/performance/resource-hints)
- [MDN: Оптимизация производительности](https://developer.mozilla.org/ru/docs/Web/Performance)

---

## Для чего используется тег template?

`<template>` — контейнер для HTML-разметки, которая не рендерится при загрузке страницы и не активирует загрузку ресурсов (изображений, скриптов). Содержимое хранится как инертный `DocumentFragment` и клонируется через JavaScript в нужный момент. Это основа Web Components и паттернов с повторяемой динамической вставкой контента.

```html
<template id="card-template">
  <div class="card">
    <h2 class="title"></h2>
    <p class="description"></p>
  </div>
</template>
```

```javascript
const template = document.getElementById('card-template');
const clone = template.content.cloneNode(true);

clone.querySelector('.title').textContent = 'Название карточки';
clone.querySelector('.description').textContent = 'Описание';

document.querySelector('.grid').appendChild(clone);
```

В отличие от скрытого через CSS `<div>`, `<template>` не порождает DOM-узлов до клонирования, что исключает любые побочные эффекты.

**Связанные задачи:**

<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: template](https://developer.mozilla.org/ru/docs/Web/HTML/Element/template)
- [MDN: Web Components](https://developer.mozilla.org/ru/docs/Web/API/Web_components)

---

## Для чего используется тег dialog?

`<dialog>` — нативный HTML-элемент для модальных окон и диалогов, заменяющий кастомные реализации на `<div>`. Даёт встроенную ловушку фокуса, закрытие по `Escape`, корректную роль ARIA `dialog` и псевдоэлемент `::backdrop` для затемнения фона. Открывается методами `showModal()` (модальный) или `show()` (немодальный); закрывается `close()`.

```html
<dialog id="confirm-dialog">
  <p>Вы уверены, что хотите удалить запись?</p>
  <button onclick="this.closest('dialog').close('yes')">Да</button>
  <button onclick="this.closest('dialog').close('no')">Отмена</button>
</dialog>
```

```javascript
const dialog = document.getElementById('confirm-dialog');

dialog.showModal();

dialog.addEventListener('close', () => {
  if (dialog.returnValue === 'yes') deleteRecord();
});
```

Преимущество перед кастомными решениями: нет необходимости вручную управлять `tabindex`, `aria-modal` и обработчиком `Escape`.

**Связанные задачи:**

<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: dialog](https://developer.mozilla.org/ru/docs/Web/HTML/Element/dialog)
- [web.dev: Building a dialog component](https://web.dev/articles/building/a-dialog-component)

---

## Оптимизация загрузки изображений в HTML?

Несколько атрибутов значительно улучшают производительность без JavaScript:

- **`loading="lazy"`** — откладывает загрузку внеэкранных изображений до приближения к viewport. Не применять к LCP-изображению выше сгиба.
- **`fetchpriority="high"`** — повышает приоритет загрузки конкретного ресурса. Используется для главного изображения страницы.
- **`decoding="async"`** — браузер декодирует изображение в фоне, не блокируя основной поток. `"sync"` нужен только если изображение должно появиться синхронно с рендером.
- **`width` / `height`** — задают соотношение сторон заранее, предотвращая CLS.

```html
<!-- LCP-изображение: высокий приоритет, без lazy -->
<img
  src="hero.webp"
  alt="Баннер"
  width="1200"
  height="600"
  fetchpriority="high"
  decoding="async"
/>

<!-- Изображения ниже сгиба: lazy -->
<img
  src="product.webp"
  alt="Товар"
  width="400"
  height="300"
  loading="lazy"
  decoding="async"
/>
```

**Связанные задачи:**

<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: loading attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img#loading)
- [web.dev: Image performance](https://web.dev/learn/performance/image-performance)

---

## Атрибуты inputmode, enterkeyhint и capture?

Три атрибута, которые улучшают UX ввода на мобильных устройствах.

**`inputmode`** задаёт тип виртуальной клавиатуры, не меняя поведение поля (в отличие от `type`):

```html
<input inputmode="numeric" />   <!-- цифровая клавиатура -->
<input inputmode="email" />     <!-- клавиатура с @ и .com -->
<input inputmode="decimal" />   <!-- числа с десятичной точкой -->
```

**`enterkeyhint`** меняет надпись на кнопке Enter виртуальной клавиатуры:

```html
<input enterkeyhint="search" />   <!-- «Найти» -->
<input enterkeyhint="send" />     <!-- «Отправить» -->
<input enterkeyhint="next" />     <!-- «Далее» -->
```

**`capture`** в `<input type="file">` открывает напрямую камеру или микрофон вместо файлового менеджера:

```html
<input type="file" capture="user" accept="image/*" />         <!-- фронтальная камера -->
<input type="file" capture="environment" accept="video/*" />  <!-- основная камера -->
```

**Связанные задачи:**

<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: inputmode](https://developer.mozilla.org/ru/docs/Web/HTML/Global_attributes/inputmode)
- [MDN: enterkeyhint](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/enterkeyhint)
- [MDN: capture](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/capture)

---

## Особенности стилизации SVG в HTML?

Способ стилизации SVG зависит от того, как он встроен:

- **Инлайновый `<svg>`** в HTML — полный доступ из внешних CSS-файлов: можно использовать CSS-переменные, псевдоклассы, свойства `fill` и `stroke`.
- **`<img src="file.svg">`** и `background-image` — SVG изолирован, внешние стили не проникают; стили должны быть прописаны прямо внутри SVG-файла.

Ключевые SVG-свойства в CSS:

```css
/* fill и stroke — основные свойства окраски */
svg path {
  fill: #333;
  stroke: #666;
  stroke-width: 1.5;
}

/* currentColor наследует цвет текста родителя — удобно для иконок в компонентах */
.icon svg path {
  fill: currentColor;
}
```

```html
<!-- Иконка автоматически наследует цвет кнопки -->
<button style="color: blue;">
  <svg>...</svg> Нажать
</button>
```

`currentColor` особенно ценен: иконка автоматически меняет цвет вместе с текстом родителя без дополнительных CSS-правил.

**Связанные задачи:**

<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: Применение CSS к SVG](https://developer.mozilla.org/ru/docs/Web/SVG/Tutorial/SVG_and_CSS)
- [MDN: currentColor](https://developer.mozilla.org/ru/docs/Web/CSS/color_value#currentcolor_keyword)

---

## Чем отличается iframe от embed?

**`<iframe>`** — полноценный вложенный документ с собственным контекстом навигации. Поддерживает двустороннюю коммуникацию через `postMessage`, атрибуты безопасности `sandbox` и `allow`. Это стандартный способ встройки сторонних сервисов: карты, видео, платёжные формы, OAuth-виджеты.

**`<embed>`** — встройка внешнего контента или плагина. Нет своего контекста навигации, ограниченные возможности взаимодействия. Изначально создавался для Flash и бинарных плагинов; сегодня практически вытеснен — иногда используется для PDF.

```html
<!-- iframe — карта с ограничениями безопасности -->
<iframe
  src="https://maps.google.com/..."
  sandbox="allow-scripts allow-same-origin"
  loading="lazy"
  title="Карта офиса"
></iframe>

<!-- embed — PDF -->
<embed src="document.pdf" type="application/pdf" width="600" height="400" />
```

**Связанные задачи:**

<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: iframe](https://developer.mozilla.org/ru/docs/Web/HTML/Element/iframe)
- [MDN: embed](https://developer.mozilla.org/ru/docs/Web/HTML/Element/embed)

---

## Что такое PWA (прогрессивное веб-приложение)?

**PWA** (Progressive Web App) — веб-приложение, использующее современные браузерные API для работы как нативное приложение: оффлайн режим, инсталляция, push-уведомления.

**Основные компоненты:**
1. **HTTPS** — обязательное условие
2. **Web App Manifest** (`manifest.json`) — метаданные приложения
3. **Service Worker** — фоновой скрипт, кэширование, оффлайн

```json
// manifest.json
{
  "name": "Моё Приложение",
  "short_name": "App",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#2563eb",
  "icons": [{ "src": "/icon-192.png", "sizes": "192x192", "type": "image/png" }]
}
```

```html
<link rel="manifest" href="/manifest.json">
<meta name="theme-color" content="#2563eb">
```

**Преимущества:** работа оффлайн, добавление на рабочий стол, push-уведомления, один кодовая база. **Недостатки:** ограниченный доступ к аппаратным API, не в App Store (Apple), хуже на iOS.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**

- [web.dev: Progressive Web Apps](https://web.dev/progressive-web-apps/)

---

## Email-вёрстка — особенности?

Email-клиенты (Гмайл, Аутлук) имеют очень слабую поддержку CSS. Основные правила:

- **Только инлайн-стили** — `style="..."` на каждом элементе
- **Таблицы для разметки** — flexbox/grid не поддерживаются в старых клиентах
- **Ширина 600-640px** — оптимально для десктопа
- **Атрибут `bgcolor`**, `align`, `cellpadding` — всё ещё работают
- **`<img>` с явным шириной/высотой** — избегать разъезжаний
- **Fallback шрифты** — `Arial`, `Georgia` (веб-шрифты не гарантированы)

```html
<!-- Базовая структура -->
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <!--[if mso]><style>td{font-family:Arial,sans-serif}</style><![endif]-->
</head>
<body>
  <table width="600" cellpadding="0" cellspacing="0" align="center">
    <tr>
      <td bgcolor="#ffffff" style="padding:20px;font-family:Arial,sans-serif">
        <h1 style="color:#333;font-size:24px">Заголовок</h1>
      </td>
    </tr>
  </table>
</body>
</html>
```

Инструменты: MJML, Foundation for Emails — преобразуют высокоуровневый шаблон в совместимый HTML.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Мультиязычность (i18n) в HTML?

```html
<!-- lang на html — основной язык -->
<html lang="ru">

<!-- lang на отдельном фрагменте -->
<p lang="en">Hello, World!</p>

<!-- hreflang: альтернативные версии страницы для SEO -->
<link rel="alternate" hreflang="en" href="https://example.com/en/">
<link rel="alternate" hreflang="ru" href="https://example.com/ru/">
<link rel="alternate" hreflang="x-default" href="https://example.com/">

<!-- dir: направление текста -->
<html lang="ar" dir="rtl"> <!-- арабский - справа налево -->
```

**Практики i18n:**
- `Intl.DateTimeFormat`, `Intl.NumberFormat` — локализация дат и чисел
- `i18next`, `react-i18next` — популярные библиотеки
- `<meta charset="UTF-8">` — обязательно для мультиязычных сайтов
- Pluralization — множественное число отличается по языкам
- Картинки без встроенного текста (пусть шаблон принимает любой длины текст)

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как происходит загрузка сайта?

1. **DNS lookup** — разрешение домена в IP
2. **TCP handshake** — установка соединения
3. **TLS handshake** — HTTPS шифрование (если HTTPS)
4. **HTTP request** — запрос HTML-документа
5. **Парсинг HTML** — браузер строит DOM-дерево
6. **Загрузка CSS** — строится CSSOM
7. **Загрузка JS** — выполняется скрипт
8. **Render Tree** — DOM + CSSOM → дерево отображаемых элементов
9. **Layout** — расчёт геометрии
10. **Paint + Composite** — отрисовка пикселей на экран

**Ключевые метрики:** FCP (First Contentful Paint), LCP (Largest Contentful Paint), TTI (Time to Interactive), CLS (Cumulative Layout Shift) — часть Core Web Vitals.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## SSR — преимущества и недостатки?

**SSR** (Server-Side Rendering) — HTML-страница генерируется на сервере при каждом запросе.

| | SSR | CSR |
|---|---|---|
| **First load** | Быстрый FCP | Медленный FCP |
| **SEO** | Отличное | Требует усилий |
| **Нагрузка сервера** | Высокая | Низкая |
| **Навигация** | Перезагрузка страниц | Instant (SPA) |

**Гибридные подходы:**
- **SSG** (Static Site Generation) — HTML генерируется при сборке, Next.js `getStaticProps`
- **ISR** (Incremental Static Regeneration) — перегенерация по запросу
- **Hydration** — привязка JS к готовому HTML (задержка TTI!)

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Изящная деградация и постепенное улучшение?

**Изящная деградация** (Graceful Degradation) — сначала разрабатывается полная версия для современных браузеров, затем добавляются фоллбэки для старых.

**Постепенное улучшение** (Progressive Enhancement) — базовый функционал работает всюду, современные браузеры получают улучшенный UX.

```html
<!-- Постепенное улучшение: -->
<!-- Базовый HTML ссылка -->
<a href="/purchase">Купить</a>
<!-- JavaScript добавляет модальное окно, если доступен -->
```

```css
/* Изящная деградация: */
/* Новые браузеры используют container queries */
@container (min-width: 400px) { .card { flex-direction: row; } }
/* Старые браузеры просто покажут одну колонку */
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Google Lighthouse — что такое?

**Google Lighthouse** — автоматизированный инструмент аудита качества веб-страниц. Встроен в Chrome DevTools (вкладка "Lighthouse").

**5 категорий оценки (0–100):**

| Категория | Что проверяет |
|---|---|
| **Performance** | LCP, FID/INP, CLS, FCP, TTFB |
| **Accessibility** | ARIA, контраст, alt, фокус |
| **Best Practices** | HTTPS, уязвимости, API |
| **SEO** | meta, hreflang, индексирование |
| **PWA** | manifest, service worker |

```bash
# CLI
npx lighthouse https://example.com --output=html --output-path=report.html
```

**Core Web Vitals** (ключевые метрики для Google):
- **LCP** (≤ 2.5s) — Largest Contentful Paint
- **INP** (≤ 200ms) — Interaction to Next Paint (заменил FID)
- **CLS** (≤ 0.1) — Cumulative Layout Shift

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**

- [web.dev: Lighthouse](https://web.dev/lighthouse-performance/)
