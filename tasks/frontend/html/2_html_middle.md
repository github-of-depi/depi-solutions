# HTML — Middle — Задачи

## Задачи

- [Адаптивное изображение](#адаптивное-изображение)
- [Форма с HTML5-валидацией](#форма-с-html5-валидацией)

---

## Адаптивное изображение

**Сложность:** Medium

**Связанные вопросы:**

- [Для чего нужны тег picture и атрибут srcset?](../../../interviews/frontend/html/2_html_middle.md#для-чего-нужны-тег-picture-и-атрибут-srcset)

**Материалы для изучения:**

- [MDN: Адаптивные изображения](https://developer.mozilla.org/ru/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images)

**Задание:**

Реализуйте адаптивное изображение с помощью `<picture>`, которое:

- Загружает **WebP** для браузеров с поддержкой этого формата, JPEG как fallback
- Показывает версию **1200×600** на экранах шире 800px и **400×200** на меньших
- Корректно отображается на экранах с **2x** плотностью пикселей

**Пример:**

```html
<picture>
  <!-- условия здесь -->
  <img src="fallback.jpg" alt="Описание" />
</picture>
```

<details>
<summary>Решение</summary>

```html
<picture>
  <!-- WebP для широких экранов -->
  <source
    media="(min-width: 800px)"
    srcset="large.webp 1x, large@2x.webp 2x"
    type="image/webp"
  />
  <!-- JPEG fallback для широких экранов -->
  <source
    media="(min-width: 800px)"
    srcset="large.jpg 1x, large@2x.jpg 2x"
  />
  <!-- WebP для малых экранов -->
  <source
    srcset="small.webp 1x, small@2x.webp 2x"
    type="image/webp"
  />
  <!-- Финальный fallback — всегда последний, обязателен -->
  <img
    src="small.jpg"
    srcset="small@2x.jpg 2x"
    alt="Баннер раздела"
    width="400"
    height="200"
    decoding="async"
  />
</picture>
```

</details>

---

## Форма с HTML5-валидацией

**Сложность:** Medium

**Связанные вопросы:**

- [Как работает встроенная валидация форм в HTML5?](../../../interviews/frontend/html/2_html_middle.md#как-работает-встроенная-валидация-форм-в-html5)

**Материалы для изучения:**

- [MDN: Валидация форм на стороне клиента](https://developer.mozilla.org/ru/docs/Learn/Forms/Form_validation)

**Задание:**

Создайте форму регистрации только средствами HTML5 (без JavaScript) с полями:

- **Имя пользователя** — только латинские буквы, цифры и `_`; от 3 до 20 символов; обязательное
- **Email** — обязательное, тип email
- **Пароль** — минимум 8 символов; обязательное
- **Подтверждение пароля** — минимум 8 символов; обязательное

Все поля должны иметь корректно связанный `<label>`. Кнопка отправки должна быть семантически правильным `<button>`.

**Пример:**

```html
<form>
  <!-- поля с HTML5-валидацией -->
</form>
```

<details>
<summary>Решение</summary>

```html
<form action="/register" method="post" autocomplete="on">

  <div>
    <label for="username">Имя пользователя</label>
    <input
      type="text"
      id="username"
      name="username"
      required
      minlength="3"
      maxlength="20"
      pattern="[A-Za-z0-9_]+"
      title="Только латинские буквы, цифры и символ _"
      autocomplete="username"
    />
  </div>

  <div>
    <label for="email">Email</label>
    <input
      type="email"
      id="email"
      name="email"
      required
      autocomplete="email"
    />
  </div>

  <div>
    <label for="password">Пароль</label>
    <input
      type="password"
      id="password"
      name="password"
      required
      minlength="8"
      autocomplete="new-password"
    />
  </div>

  <div>
    <label for="password-confirm">Подтверждение пароля</label>
    <input
      type="password"
      id="password-confirm"
      name="password_confirm"
      required
      minlength="8"
      autocomplete="new-password"
    />
    <!--
      Примечание: проверка совпадения паролей требует JavaScript.
      HTML5 не поддерживает cross-field validation нативно.
    -->
  </div>

  <button type="submit">Зарегистрироваться</button>

</form>
```

</details>
