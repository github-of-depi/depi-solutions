# HTML — Middle

## Вопросы

- [Что такое категории контента в HTML5?](#что-такое-категории-контента-в-html5)
- [Чем отличается article от section?](#чем-отличается-article-от-section)
- [Для чего нужны тег picture и атрибут srcset?](#для-чего-нужны-тег-picture-и-атрибут-srcset)
- [В чём разница между canvas и svg?](#в-чём-разница-между-canvas-и-svg)
- [Как работает встроенная валидация форм в HTML5?](#как-работает-встроенная-валидация-форм-в-html5)
- [Для чего используются data-атрибуты?](#для-чего-используются-data-атрибуты)
- [Плюсы и минусы использования iframe?](#плюсы-и-минусы-использования-iframe)
- [Для чего нужен meta viewport?](#для-чего-нужен-meta-viewport)
- [HTML5 API — обзор?](#html5-api--обзор)
- [Image map — что это и как работает?](#image-map--что-это-и-как-работает)
- [HTML5 Web Workers?](#html5-web-workers)
- [Что такое DOM?](#что-такое-dom)
- [SSE (Server-Sent Events)?](#sse-server-sent-events)
- [Как сделать кастомный чекбокс?](#как-сделать-кастомный-чекбокс)
- [Drag and Drop API?](#drag-and-drop-api)
- [HTML-шаблонизаторы (Pug)?](#html-шаблонизаторы-pug)

---

## Что такое категории контента в HTML5?

HTML5 классифицирует все элементы по категориям контента, определяя где и какие элементы могут находиться. Это позволяет браузеру правильно интерпретировать вложенность и является основой валидации документа.

Основные категории:

- **Flow content** — большинство элементов тела документа (`<div>`, `<p>`, `<section>`)
- **Phrasing content** — строчные элементы внутри текста (`<span>`, `<a>`, `<strong>`)
- **Interactive content** — элементы взаимодействия с пользователем (`<a href>`, `<button>`, `<input>`)
- **Sectioning content** — элементы структуры документа (`<article>`, `<section>`, `<nav>`, `<aside>`)
- **Heading content** — заголовки (`<h1>`–`<h6>`, `<hgroup>`)
- **Embedded content** — внешние ресурсы (`<img>`, `<video>`, `<iframe>`, `<canvas>`)
- **Metadata content** — метаданные (`<meta>`, `<link>`, `<script>`, `<style>`)

**Связанные задачи:**

<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: Категории контента](https://developer.mozilla.org/ru/docs/Web/HTML/Content_categories)

---

## Чем отличается article от section?

`<article>` — самодостаточный блок контента, который имеет смысл вне контекста страницы: статья, пост в блоге, комментарий, карточка товара. Его можно вырвать из страницы и опубликовать отдельно.

`<section>` — тематическая группировка контента внутри документа без претензии на самодостаточность. Как правило, имеет свой заголовок и объединяет части одного целого.

Простой тест: если контент можно опубликовать отдельно — `<article>`, если это просто раздел большего целого — `<section>`.

```html
<article>
  <h2>Как освоить TypeScript</h2>
  <section>
    <h3>Базовые типы</h3>
    <!-- ... -->
  </section>
  <section>
    <h3>Generics</h3>
    <!-- ... -->
  </section>
</article>
```

**Связанные задачи:**

<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: article](https://developer.mozilla.org/ru/docs/Web/HTML/Element/article)
- [MDN: section](https://developer.mozilla.org/ru/docs/Web/HTML/Element/section)

---

## Для чего нужны тег picture и атрибут srcset?

`srcset` позволяет браузеру выбрать наиболее подходящее изображение из набора — по плотности пикселей или ширине viewport. Тег `<picture>` расширяет это: через `<source media="...">` можно указать разные изображения для разных условий, а также задать альтернативные форматы (WebP с JPEG-fallback).

```html
<!-- srcset для экранов с разной плотностью пикселей -->
<img
  src="photo.jpg"
  srcset="photo@2x.jpg 2x, photo@3x.jpg 3x"
  alt="Фото"
/>

<!-- picture для разных форматов и размеров экрана -->
<picture>
  <source media="(min-width: 800px)" srcset="large.webp" type="image/webp" />
  <source media="(min-width: 800px)" srcset="large.jpg" />
  <img src="small.jpg" alt="Адаптивное фото" />
</picture>
```

**Связанные задачи:**

- [Адаптивное изображение](../../../tasks/frontend/html/2_html_middle.md#адаптивное-изображение)

**Материалы для изучения:**

- [MDN: Адаптивные изображения](https://developer.mozilla.org/ru/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images)
- [MDN: picture](https://developer.mozilla.org/ru/docs/Web/HTML/Element/picture)

---

## В чём разница между canvas и svg?

`<canvas>` — растровый холст: рисование происходит пиксель за пикселем через JavaScript Canvas API. После отрисовки информация об объектах «забывается», нет DOM-узлов для отдельных фигур. Подходит для игр, анимаций, обработки изображений.

`<svg>` — векторный формат: каждая фигура является DOM-узлом, масштабируется без потери качества, поддерживает CSS-стилизацию, обработчики событий и анимации через CSS/JS. Хорошо подходит для иконок, диаграмм, интерактивных иллюстраций.

| Критерий | `<canvas>` | `<svg>` |
|---|---|---|
| Тип | Растр | Вектор |
| DOM | Нет | Да |
| Масштабируемость | Теряет качество | Без потерь |
| Применение | Игры, пиксельная обработка | Иконки, диаграммы, UI |

**Связанные задачи:**

<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: Canvas API](https://developer.mozilla.org/ru/docs/Web/API/Canvas_API)
- [MDN: SVG](https://developer.mozilla.org/ru/docs/Web/SVG)

---

## Как работает встроенная валидация форм в HTML5?

HTML5 предоставляет встроенные механизмы валидации без JavaScript. Браузер проверяет условия при попытке отправки формы и показывает стандартные подсказки на языке ОС. Ключевые атрибуты: `required`, `type`, `min`/`max`, `minlength`/`maxlength`, `pattern`.

```html
<form>
  <input type="email" required placeholder="example@mail.com" />
  <input type="password" minlength="8" required />
  <input type="number" min="1" max="100" />
  <input type="text" pattern="[A-Za-z]{3,}" title="Минимум 3 буквы" />
  <button type="submit">Отправить</button>
</form>
```

Состояния стилизуются через псевдоклассы `:valid` / `:invalid`. Чтобы отключить стандартные подсказки браузера (для кастомного UX) — добавить `novalidate` на `<form>` и реализовать валидацию через Constraint Validation API.

**Связанные задачи:**

- [Форма с HTML5-валидацией](../../../tasks/frontend/html/2_html_middle.md#форма-с-html5-валидацией)

**Материалы для изучения:**

- [MDN: Валидация форм на стороне клиента](https://developer.mozilla.org/ru/docs/Learn/Forms/Form_validation)

---

## Для чего используются data-атрибуты?

`data-*` — механизм хранения произвольных данных прямо в HTML-элементе без использования нестандартных атрибутов. Данные доступны через `element.dataset` в JavaScript. Удобны для передачи конфигурации из шаблона в скрипт или для хранения состояния прямо в разметке.

```html
<button data-user-id="42" data-action="delete">Удалить</button>
```

```javascript
const btn = document.querySelector('button');
console.log(btn.dataset.userId);  // "42"
console.log(btn.dataset.action);  // "delete"
```

**Связанные задачи:**

<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: Использование data-атрибутов](https://developer.mozilla.org/ru/docs/Learn/HTML/Howto/Use_data_attributes)

---

## Плюсы и минусы использования iframe?

`<iframe>` встраивает внешнюю HTML-страницу или ресурс внутрь текущей. Используется для вставки карт, видео, платёжных форм, OAuth-виджетов.

**Плюсы:**
- Изоляция внешнего контента в отдельном контексте
- Сторонний виджет не может напрямую влиять на стили и скрипты основной страницы
- Атрибут `sandbox` ограничивает возможности встроенного документа

**Минусы:**
- Отдельный HTTP-запрос и дополнительный контекст браузера
- Контент внутри не индексируется поисковиками в контексте основной страницы
- Риск clickjacking — требует правильных заголовков `X-Frame-Options` / CSP на стороне сервера
- Сложности с динамической высотой при изменении содержимого

**Связанные задачи:**

<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: iframe](https://developer.mozilla.org/ru/docs/Web/HTML/Element/iframe)

---

## Для чего нужен meta viewport?

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
```

Без этого тега мобильные браузеры рендерят страницу в виртуальном широком окне (обычно 980px) и уменьшают её под экран, делая текст нечитаемым. `width=device-width` устанавливает ширину viewport равной физической ширине устройства. `initial-scale=1.0` предотвращает начальное масштабирование. Без этого тега CSS медиа-запросы не срабатывают корректно на мобильных.

**Связанные задачи:**

<!-- Связанных задач нет -->

**Материалы для изучения:**

- [MDN: Мета-тег viewport](https://developer.mozilla.org/ru/docs/Web/HTML/Viewport_meta_tag)

---

## HTML5 API — обзор?

HTML5 принёс широкий спектр браузерных API:

| API | Назначение |
|---|---|
| **Geolocation** | `navigator.geolocation.getCurrentPosition()` |
| **Web Storage** | `localStorage`, `sessionStorage` |
| **IndexedDB** | База данных на клиенте |
| **Web Workers** | Фоновые потоки |
| **WebSockets** | Двухсторонняя связь |
| **SSE** | Server-Sent Events — односторонний поток с сервера |
| **Canvas API** | Рисование через JS |
| **History API** | `pushState`, `replaceState` |
| **Notifications** | `Notification.requestPermission()` |
| **Drag and Drop** | Нативный драг |
| **File API** | `FileReader`, `File`, `Blob` |
| **Clipboard API** | `navigator.clipboard.readText/writeText` |
| **Fullscreen API** | `element.requestFullscreen()` |
| **Page Visibility** | `document.visibilityState`, `visibilitychange` |
| **Web Speech** | Распознавание / синтез речи |
| **Service Worker** | Фоновой скрипт, оффлайн, push |

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Image map — что это и как работает?

**Image map** — изображение с кликабельными областями. Каждая область ведёт на отдельный URL.

```html
<img src="map.png" alt="Карта" usemap="#regions">

<map name="regions">
  <!-- rect: x1,y1, x2,y2 -->
  <area shape="rect" coords="0,0,100,100"
        href="/north" alt="Север">
  <!-- circle: cx,cy,r -->
  <area shape="circle" coords="200,150,50"
        href="/center" alt="Центр">
  <!-- poly: x1,y1,x2,y2,... -->
  <area shape="poly" coords="120,50,180,50,150,100"
        href="/south" alt="Юг">
</map>
```

Сейчас практически не используется — заменяется SVG с `<a>` или CSS clip-path + position.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## HTML5 Web Workers?

**Web Worker** — скрипт, выполняющийся в отдельном потоке (background thread), не блокируя главный поток UI. Доступа к DOM нет. Общение через `postMessage`/`onmessage`.

```javascript
// main.js
const worker = new Worker('worker.js');
worker.postMessage({ data: [1, 2, 3, 4, 5] });
worker.onmessage = (e) => console.log('Result:', e.data);
worker.onerror = (e) => console.error(e);
// worker.terminate(); // стоп

// worker.js
self.onmessage = (e) => {
  const result = e.data.data.reduce((a, b) => a + b, 0);
  self.postMessage(result);
};
```

**Виды:**
- `Worker` — обычный, для тяжёлых вычислений
- `SharedWorker` — один воркер на несколько вкладок
- `ServiceWorker` — фоновой, перехватывает сеть, оффлайн, push-уведомления

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое DOM?

**DOM** (Document Object Model) — древовидная модель HTML/XML-документа в виде объектов. Браузер парсит HTML и строит DOM в памяти. JS манипулирует DOM через API `document`.

```javascript
// Выборка
document.getElementById('id')
document.querySelector('.class')
document.querySelectorAll('div')

// Навигация
el.parentNode, el.children, el.firstElementChild
el.nextElementSibling, el.previousElementSibling

// Изменение
el.textContent = 'text'
el.innerHTML = '<b>html</b>'
el.setAttribute('class', 'active')
el.classList.add('open')
el.style.color = 'red'

// Создание / удаление
const p = document.createElement('p')
parent.appendChild(p)
parent.removeChild(p)
p.remove() // современный путь
```

**CSSOM** — аналогичное дерево для CSS. Браузер объединяет DOM + CSSOM в Render Tree для рендеринга.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## SSE (Server-Sent Events)?

**SSE** — односторонный поток данных от сервера к клиенту по протоколу HTTP. Проще WebSocket, но только в одну сторону.

```javascript
// Клиент
const es = new EventSource('/api/stream');

es.onmessage = (e) => console.log(e.data);

es.addEventListener('update', (e) => {
  console.log('Событие update:', e.data);
});

es.onerror = () => console.error('SSE error');
es.close(); // закрыть
```

```
// Сервер: Content-Type: text/event-stream

data: Привет\n\n
event: update\ndata: {"time": 123}\n\n
id: 42\nretry: 3000\ndata: переподключение через 3 с\n\n
```

**SSE vs WebSocket:** SSE проще (только HTTP), автопереподключение, подходит для пуш-уведомлений, ленты новостей. WebSocket — двусторонний, чаты/игры.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как сделать кастомный чекбокс?

CSS-стилизация нативного `<input type="checkbox">` ограничена. Решение: скрываем нативный чекбокс, стилизуем `<label>` как визуальный чекбокс через `::before`.

```html
<label class="checkbox">
  <input type="checkbox" class="checkbox__input">
  <span class="checkbox__mark"></span>
  Принять условия
</label>
```

```css
.checkbox__input {
  position: absolute;
  opacity: 0;         /* скрыть, но оставить доступным для a11y */
  width: 0; height: 0;
}

.checkbox__mark {
  display: inline-block;
  width: 18px; height: 18px;
  border: 2px solid #999;
  border-radius: 3px;
  transition: background 0.2s;
}

/* Состояние checked через соседний селектор */
.checkbox__input:checked + .checkbox__mark {
  background: #2563eb;
  border-color: #2563eb;
}

/* Галочка через ::after */
.checkbox__input:checked + .checkbox__mark::after {
  content: '';
  display: block;
  width: 5px; height: 10px;
  border: 2px solid white;
  border-top: none; border-left: none;
  transform: rotate(45deg);
  margin: 1px 0 0 4px;
}

/* Фокус для a11y */
.checkbox__input:focus-visible + .checkbox__mark {
  outline: 2px solid #2563eb;
  outline-offset: 2px;
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Drag and Drop API?

Нативный HTML5 Drag and Drop API позволяет перетаскивать элементы.

```html
<div draggable="true" id="item">Перетащи</div>
<div id="target">Зона сброса</div>
```

```javascript
const item = document.getElementById('item');
const target = document.getElementById('target');

// Источник
item.addEventListener('dragstart', (e) => {
  e.dataTransfer.setData('text/plain', item.id);
});

// Цель
target.addEventListener('dragover', (e) => {
  e.preventDefault(); // обязательно!
});

target.addEventListener('drop', (e) => {
  e.preventDefault();
  const id = e.dataTransfer.getData('text/plain');
  target.appendChild(document.getElementById(id));
});
```

**События:** `dragstart`, `drag`, `dragend` (на источнике); `dragenter`, `dragover`, `dragleave`, `drop` (на цели).

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## HTML-шаблонизаторы (Pug)?

**HTML-шаблонизатор** — препроцессор, компилирующий свой синтаксис в чистый HTML. **Pug** (раньше Jade) — самый популярный.

```pug
//- Pug
doctype html
html(lang="ru")
  head
    title Пример
    link(rel="stylesheet" href="style.css")
  body
    header.site-header
      nav
        ul
          each item in ['Home', 'About', 'Contact']
            li: a(href=`/${item.toLowerCase()}`)= item
    main
      article
        h1 Заголовок
        p Текст #{вариабльные}
        if condition
          p Да
        else
          p Нет
    include footer.pug
```

**Преимущества:** меньше бойлерплейта, вносимые `include`/`extends`/`block`, петли и циклы. Популярен в Node.js/Express. Сейчас компонентные фреймворки (React/Vue) почти вытеснили Pug.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
