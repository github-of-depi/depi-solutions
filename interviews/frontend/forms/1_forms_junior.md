# Forms — Junior

## Вопросы

- [Что такое controlled и uncontrolled компонент формы?](#что-такое-controlled-и-uncontrolled-компонент-формы)
- [Как обработать submit формы в React?](#как-обработать-submit-формы-в-react)
- [Что такое HTML5 валидация форм?](#что-такое-html5-валидация-форм)
- [Что такое FormData API?](#что-такое-formdata-api)
- [Как сбросить форму?](#как-сбросить-форму)

---

## Что такое controlled и uncontrolled компонент формы?

**Controlled** — значение поля хранится в React state, React — единственный источник правды. **Uncontrolled** — значение хранится в DOM, читается через `ref`. Controlled — стандарт для форм с валидацией и зависимыми полями.

```typescript
// Controlled
const [email, setEmail] = useState("");
<input value={email} onChange={e => setEmail(e.target.value)} />

// Uncontrolled
const emailRef = useRef<HTMLInputElement>(null);
<input ref={emailRef} defaultValue="" />
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как обработать submit формы в React?

Навешивать `onSubmit` на тег `<form>`, вызывать `e.preventDefault()` чтобы предотвратить перезагрузку страницы. Собирать данные из state или FormData.

```typescript
function LoginForm() {
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    // собрать данные
  };
  return <form onSubmit={handleSubmit}>...</form>;
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое HTML5 валидация форм?

HTML5 атрибуты валидации: `required`, `type="email"`, `minlength`, `maxlength`, `pattern`, `min`, `max`. Браузер показывает встроенные сообщения об ошибках. Для кастомных сообщений используй `setCustomValidity()` или отключи нативную через `novalidate`.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое FormData API?

`FormData` — встроенный API для сбора данных формы, в том числе файлов. Автоматически собирает все `name` поля формы.

```typescript
function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
  e.preventDefault();
  const formData = new FormData(e.currentTarget);
  const email = formData.get("email") as string;
  const file = formData.get("avatar") as File;
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как сбросить форму?

Через `form.reset()` (нативный DOM метод) или управляя state вручную. В React Hook Form — `reset()` метод из `useForm()`.

```typescript
const formRef = useRef<HTMLFormElement>(null);
function handleReset() { formRef.current?.reset(); }
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
