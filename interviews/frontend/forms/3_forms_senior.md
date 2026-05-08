# Forms — Senior

## Вопросы

- [Как оптимизировать производительность больших форм?](#как-оптимизировать-производительность-больших-форм)
- [Как реализовать многошаговую форму (wizard)?](#как-реализовать-многошаговую-форму-wizard)
- [Как обеспечить доступность форм (a11y)?](#как-обеспечить-доступность-форм-a11y)
- [Как валидировать данные на сервере и показывать ошибки?](#как-валидировать-данные-на-сервере-и-показывать-ошибки)
- [Как строить переиспользуемые компоненты полей формы?](#как-строить-переиспользуемые-компоненты-полей-формы)

---

## Как оптимизировать производительность больших форм?

RHF по умолчанию лучше чем controlled state — uncontrolled подход. Дополнительно: `useWatch` вместо `watch` (локальная подписка), разбивать большую форму на секции через `FormProvider` + `useFormContext`, `shouldUnregister: false` для сохранения данных при скрытии полей.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как реализовать многошаговую форму (wizard)?

Хранить шаг в state, данные накапливать через `FormProvider`. Валидировать только текущий шаг перед переходом через `trigger(fieldsOfStep)`.

```typescript
const methods = useForm({ mode: "onTouched" });
const [step, setStep] = useState(0);

const goNext = async () => {
  const valid = await methods.trigger(stepFields[step]);
  if (valid) setStep(s => s + 1);
};
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как обеспечить доступность форм (a11y)?

1. `<label htmlFor>` связан с `id` поля — обязательно
2. `aria-describedby` — ссылка на блок с ошибкой
3. `aria-invalid="true"` при ошибке
4. `aria-required="true"` для обязательных полей
5. Фокус на первое поле с ошибкой после неудачного submit

```typescript
<label htmlFor="email">Email</label>
<input
  id="email"
  aria-invalid={!!errors.email}
  aria-describedby={errors.email ? "email-error" : undefined}
  {...register("email")}
/>
{errors.email && <span id="email-error" role="alert">{errors.email.message}</span>}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как валидировать данные на сервере и показывать ошибки?

Zod на сервере для валидации входных данных. Ошибки возвращать в стандартной структуре. На клиенте маппить ошибки через `setError`.

```typescript
// Server Action
const result = schema.safeParse(formData);
if (!result.success) {
  return { errors: result.error.flatten().fieldErrors };
}
// Client — маппинг ошибок
if (result.errors) {
  Object.entries(result.errors).forEach(([field, messages]) => {
    setError(field as keyof FormData, { message: messages[0] });
  });
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как строить переиспользуемые компоненты полей формы?

`Controller` из RHF для интеграции кастомных UI компонентов. Обёртка принимает `name`, `control`, `label` и рендерит поле с ошибкой.

```typescript
function FormField({ name, control, label }: Props) {
  return (
    <Controller
      name={name}
      control={control}
      render={({ field, fieldState }) => (
        <div>
          <label>{label}</label>
          <input {...field} aria-invalid={!!fieldState.error} />
          {fieldState.error && <span>{fieldState.error.message}</span>}
        </div>
      )}
    />
  );
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
