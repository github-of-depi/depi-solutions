# HTML — Junior — Задачи

## Задачи

- [Семантическая разметка страницы](#семантическая-разметка-страницы)
- [Картинка с подписью](#картинка-с-подписью)

---

## Семантическая разметка страницы

**Сложность:** Easy

**Связанные вопросы:**

- [Что такое семантика в HTML?](../../../interviews/frontend/html/1_html_junior.md#что-такое-семантика-в-html)

**Материалы для изучения:**

- [MDN: Семантика в HTML](https://developer.mozilla.org/ru/docs/Glossary/Semantics#семантика_в_html)

**Задание:**

Разметьте типичную страницу блога, используя семантические теги HTML5. Страница должна содержать:

- шапку с логотипом и навигацией
- основной блок с тремя превью-статьями
- боковую панель с рубриками
- подвал с копирайтом

Избегайте `<div>` там, где есть подходящий семантический аналог.

**Пример структуры:**

```html
<!-- Ожидаемая структура (упрощённо) -->
<body>
  <header>...</header>
  <main>
    <section>
      <article>...</article>
      <article>...</article>
    </section>
    <aside>...</aside>
  </main>
  <footer>...</footer>
</body>
```

<details>
<summary>Решение</summary>

```html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Мой блог</title>
</head>
<body>

  <header>
    <a href="/" aria-label="На главную">
      <img src="logo.svg" alt="Логотип блога" width="120" height="40" />
    </a>
    <nav aria-label="Основная навигация">
      <ul>
        <li><a href="/articles">Статьи</a></li>
        <li><a href="/about">Об авторе</a></li>
        <li><a href="/contacts">Контакты</a></li>
      </ul>
    </nav>
  </header>

  <main>
    <section aria-label="Последние статьи">
      <article>
        <header>
          <h2><a href="/post-1">Введение в TypeScript</a></h2>
          <time datetime="2026-05-01">1 мая 2026</time>
        </header>
        <p>Краткое описание первой статьи...</p>
        <footer>
          <a href="/post-1">Читать далее</a>
        </footer>
      </article>

      <article>
        <header>
          <h2><a href="/post-2">React hooks в деталях</a></h2>
          <time datetime="2026-04-20">20 апреля 2026</time>
        </header>
        <p>Краткое описание второй статьи...</p>
        <footer>
          <a href="/post-2">Читать далее</a>
        </footer>
      </article>

      <article>
        <header>
          <h2><a href="/post-3">CSS Grid за 10 минут</a></h2>
          <time datetime="2026-04-10">10 апреля 2026</time>
        </header>
        <p>Краткое описание третьей статьи...</p>
        <footer>
          <a href="/post-3">Читать далее</a>
        </footer>
      </article>
    </section>

    <aside aria-label="Рубрики">
      <h2>Рубрики</h2>
      <ul>
        <li><a href="/category/javascript">JavaScript</a></li>
        <li><a href="/category/react">React</a></li>
        <li><a href="/category/css">CSS</a></li>
      </ul>
    </aside>
  </main>

  <footer>
    <p><small>&copy; 2026 Мой блог. Все права защищены.</small></p>
  </footer>

</body>
</html>
```

</details>

---

## Картинка с подписью

**Сложность:** Easy

**Связанные вопросы:**

- [Для чего нужен атрибут alt у изображения?](../../../interviews/frontend/html/1_html_junior.md#для-чего-нужен-атрибут-alt-у-изображения)

**Материалы для изучения:**

- [MDN: Images in HTML](https://developer.mozilla.org/ru/docs/Learn/HTML/Multimedia_and_embedding/Images_in_HTML)

**Задание:**

Разметьте три изображения с использованием семантически корректных тегов HTML:

1. Информативное изображение с текстовой подписью под ним
2. Декоративное изображение (должно быть проигнорировано скринридером)
3. Изображение, которое является ссылкой

**Пример:**

```html
<!-- Заготовки — заполните правильно -->

<!-- 1. Информативное с подписью -->
<??? >
  <img src="chart.png" alt="???" />
  <???></???>
</???>

<!-- 2. Декоративное -->
<img src="divider.svg" alt="???" />

<!-- 3. Изображение-ссылка -->
<a href="/profile">
  <img src="avatar.jpg" alt="???" />
</a>
```

<details>
<summary>Решение</summary>

```html
<!-- 1. Информативное с подписью — используем figure + figcaption -->
<figure>
  <img
    src="chart.png"
    alt="График роста аудитории за 2025 год: с 10 тыс. до 150 тыс. пользователей"
    width="800"
    height="400"
  />
  <figcaption>Рост аудитории блога в 2025 году</figcaption>
</figure>

<!-- 2. Декоративное — пустой alt, чтобы скринридер пропустил элемент -->
<img src="divider.svg" alt="" role="presentation" />

<!-- 3. Изображение-ссылка — alt описывает назначение ссылки, не внешний вид -->
<a href="/profile">
  <img
    src="avatar.jpg"
    alt="Перейти к профилю Ивана Иванова"
    width="80"
    height="80"
  />
</a>
```

</details>
