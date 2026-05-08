# Forms — Middle

## Вопросы

- [Как использовать React Hook Form?](#как-использовать-react-hook-form)
- [Как интегрировать Zod с React Hook Form?](#как-интегрировать-zod-с-react-hook-form)
- [Как работать с массивами полей (useFieldArray)?](#как-работать-с-массивами-полей-usefieldarray)
- [Как реализовать зависимые поля?](#как-реализовать-зависимые-поля)
- [Как обрабатывать ошибки валидации?](#как-обрабатывать-ошибки-валидации)
- [Как загружать файлы через форму?](#как-загружать-файлы-через-форму)

---

## Как использовать React Hook Form?

RHF управляет формой через uncontrolled компоненты + `ref`: производительность лучше чем controlled (нет ре-рендера на каждый keystroke). `register` регистрирует поле, `handleSubmit` обрабатывает submit с валидацией, `formState.errors` — ошибки.

```typescript
const { register, handleSubmit, formState: { errors } } = useForm<FormData>();

const onSubmit = (data: FormData) => console.log(data);

<form onSubmit={handleSubmit(onSubmit)}>
  <input {...register("email", { required: "Email обязателен" })} />
  {errors.email && <span>{errors.email.message}</span>}
</form>
```

---

## Как интегрировать Zod с React Hook Form?

`@hookform/resolvers` — пакет адаптеров. `zodResolver` принимает Zod схему и возвращает функцию валидации. Тип формы можно вывести из схемы через `z.infer`.

```typescript
const schema = z.object({
  email: z.string().email("Некорректный email"),
  password: z.string().min(8, "Минимум 8 символов"),
});
type FormData = z.infer<typeof schema>;

const { register, handleSubmit, formState: { errors } } = useForm<FormData>({
  resolver: zodResolver(schema),
});
```

---

## Как работать с массивами полей (useFieldArray)?

`useFieldArray` управляет динамическими массивами полей: добавление/удаление/перестановка с сохранением валидации и состояния.

```typescript
const { fields, append, remove } = useFieldArray({ control, name: "items" });

{fields.map((field, index) => (
  <div key={field.id}>
    <input {...register(`items.${index}.name`)} />
    <button onClick={() => remove(index)}>Удалить</button>
  </div>
))}
<button onClick={() => append({ name: "" })}>Добавить</button>
```

---

## Как реализовать зависимые поля?

`watch` или `useWatch` для отслеживания значения поля. Показывать/скрывать поля, изменять валидацию на основе других значений.

```typescript
const watchType = watch("type");
{watchType === "company" && (
  <input {...register("companyName", { required: true })} />
)}
```

---

## Как обрабатывать ошибки валидации?

Ошибки доступны через `formState.errors` — объект с вложенной структурой по именам полей. `setError` — программно устанавливать ошибки (например, с сервера). `clearErrors` — сбрасывать.

```typescript
// Серверные ошибки
try {
  await submit(data);
} catch (error) {
  setError("email", { message: "Email уже занят" });
}
```

---

## Как загружать файлы через форму?

Через `<input type="file">` + `register`, получать `FileList` в `onSubmit`. Для превью — `URL.createObjectURL()`. Загрузка через `FormData` + `fetch`/axios.

```typescript
const { register, handleSubmit } = useForm<{ avatar: FileList }>();

const onSubmit = async (data: { avatar: FileList }) => {
  const formData = new FormData();
  formData.append("avatar", data.avatar[0]);
  await fetch("/api/upload", { method: "POST", body: formData });
};
<input type="file" accept="image/*" {...register("avatar")} />
```
