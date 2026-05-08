# Browser — Middle

## Вопросы

- [Что такое event loop?](#что-такое-event-loop)
- [Чем microtasks отличаются от macrotasks?](#чем-microtasks-отличаются-от-macrotasks)
- [Что такое repaint и reflow?](#что-такое-repaint-и-reflow)
- [Что такое Service Worker?](#что-такое-service-worker)
- [Что такое CORS и как он работает?](#что-такое-cors-и-как-он-работает)
- [Что такое Web Storage vs Cookies vs IndexedDB?](#что-такое-web-storage-vs-cookies-vs-indexeddb)
- [Что такое requestAnimationFrame?](#что-такое-requestanimationframe)
- [Canvas vs SVG — в чём разница?](#canvas-vs-svg--в-чём-разница)

---

## Что такое event loop?

Event loop — механизм выполнения JavaScript. Call stack — текущий код, один поток. Web APIs (setTimeout, fetch) — выполняются браузером параллельно. По завершению — callback в Task Queue (macrotask) или Microtask Queue. Event loop: выполнить весь call stack → очистить microtask queue → взять одну macrotask → повторить.

---

## Чем microtasks отличаются от macrotasks?

**Microtasks** (Promise.then, queueMicrotask, MutationObserver) — обрабатываются полностью после каждой macrotask, до следующего рендера. **Macrotasks** (setTimeout, setInterval, I/O, события) — одна за итерацию event loop. Бесконечный цикл через Promise — заблокирует рендер, через setTimeout — нет.

```javascript
console.log("1");
setTimeout(() => console.log("4 macro"), 0);
Promise.resolve().then(() => console.log("2 micro")).then(() => console.log("3 micro"));
// 1, 2, 3, 4
```

---

## Что такое repaint и reflow?

**Reflow** (layout) — пересчёт геометрии всех элементов, дорогая операция. Вызывается: изменением размеров, шрифтов, добавлением DOM-элементов. **Repaint** — перерисовка пикселей без изменения геометрии. Batching: React и современные браузеры батчат изменения. `requestAnimationFrame` — синхронизация с рендером.

---

## Что такое Service Worker?

Service Worker — JavaScript файл, работающий в отдельном потоке, без доступа к DOM. Перехватывает сетевые запросы, управляет кэшем, обеспечивает offline работу, Push-уведомления. Жизненный цикл: install → activate → fetch. Workbox — высокоуровневая библиотека для SW.

---

## Что такое CORS и как он работает?

CORS (Cross-Origin Resource Sharing) — браузерный механизм безопасности: запросы к другому origin блокируются, если сервер явно не разрешил. Сервер отвечает заголовком `Access-Control-Allow-Origin`. Preflight (`OPTIONS`) — для non-simple запросов (custom headers, POST с JSON).

---

## Что такое Web Storage vs Cookies vs IndexedDB?

| | localStorage | Cookies | IndexedDB |
|---|---|---|---|
| Размер | ~5MB | ~4KB | Гигабайты |
| Тип данных | Строки | Строки | Любые (structured clone) |
| Сервер | Нет | Да (httpOnly) | Нет |
| Worker | Нет | Нет | Да |
| API | Sync | Sync | Async |

---

## Что такое requestAnimationFrame?

`rAF` — callback вызывается браузером перед следующим кадром (~60 раз в секунду). Используется для плавных JS-анимаций синхронизированных с refresh rate. Лучше setTimeout(fn, 16) — браузер оптимизирует частоту и паузит при скрытой вкладке.

---

## Canvas vs SVG — в чём разница?

| | **Canvas** | **SVG** |
|---|---|---|
| Тип | Растровый (пиксели) | Векторный (XML DOM) |
| API | Imperative (JS-команды) | Declarative (теги) |
| Масштабирование | Теряет качество | Без потерь (векторный) |
| DOM | Нет DOM у объектов | Каждый элемент — DOM-узел |
| События | Только на `<canvas>` целиком | На каждом элементе (`click`, `hover`) |
| Производительность | Быстрый при большом кол-ве объектов | Медленный при >1000 элементов |
| Анимации | requestAnimationFrame + ручной redraw | CSS/SMIL анимации, легко |
| Доступность | Требует ручного ARIA | SVG семантически доступнее |

```html
<!-- Canvas — bitmap, рисуем командами -->
<canvas id="c" width="400" height="300"></canvas>
<script>
  const ctx = document.getElementById("c").getContext("2d");
  ctx.fillStyle = "blue";
  ctx.fillRect(10, 10, 100, 50);
  ctx.beginPath();
  ctx.arc(200, 150, 50, 0, Math.PI * 2);
  ctx.fill();
</script>

<!-- SVG — векторный, декларативный -->
<svg width="400" height="300">
  <rect x="10" y="10" width="100" height="50" fill="blue" />
  <circle cx="200" cy="150" r="50" fill="red" onclick="alert('clicked!')" />
  <text x="50" y="200" font-size="20">SVG текст</text>
</svg>
```

Когда использовать:
- **Canvas**: игры, real-time визуализации, image processing, WebGL, большое число объектов
- **SVG**: иконки, диаграммы, инфографика, интерактивные карты, логотипы (масштабирование без потерь)
