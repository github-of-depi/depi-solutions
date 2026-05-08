# Forms — Expert

## Вопросы

- [Как реализовать schema-driven форму?](#как-реализовать-schema-driven-форму)
- [Как строить form builder для конечных пользователей?](#как-строить-form-builder-для-конечных-пользователей)
- [Как обрабатывать offline forms и optimistic submit?](#как-обрабатывать-offline-forms-и-optimistic-submit)
- [Как тестировать сложные формы?](#как-тестировать-сложные-формы)

---

## Как реализовать schema-driven форму?

JSON Schema / Zod schema → динамически генерируемые поля. Конфигурация описывает тип поля, валидацию, зависимости. Рендерер маппит тип поля на компонент. Используется в CMS, no-code платформах.

```typescript
const fieldConfig: FieldConfig[] = [
  { name: "email", type: "email", label: "Email", validation: z.string().email() },
  { name: "role", type: "select", label: "Роль", options: ["admin", "user"] },
];
function DynamicForm({ fields }: { fields: FieldConfig[] }) {
  return fields.map(f => <FieldRenderer key={f.name} config={f} />);
}
```

---

## Как строить form builder для конечных пользователей?

Drag-and-drop интерфейс (dnd-kit) + schema как state. Каждый элемент — JSON объект с типом и настройками. Preview режим рендерит форму из schema. Экспорт schema в JSON для сохранения.

Ключевые вызовы: порядок полей, условная логика (show/hide), nested groups, валидация at runtime.

---

## Как обрабатывать offline forms и optimistic submit?

TanStack Query `useMutation` + `persistQueryClient` для queue офлайн операций. При потере сети — сохранять в IndexedDB, retrying при восстановлении. Optimistic submit: немедленно обновлять UI, откатывать при ошибке.

```typescript
const mutation = useMutation({
  mutationFn: submitForm,
  onMutate: async (data) => {
    await queryClient.cancelQueries({ queryKey: ["form"] });
    const prev = queryClient.getQueryData(["form"]);
    queryClient.setQueryData(["form"], data); // optimistic
    return { prev };
  },
  onError: (err, _, ctx) => queryClient.setQueryData(["form"], ctx?.prev), // rollback
});
```

---

## Как тестировать сложные формы?

RTL + `userEvent` для реалистичной симуляции ввода. Тестировать: submit с валидными данными, отображение ошибок при невалидных, условные поля, зависимые поля, серверные ошибки.

```typescript
it("показывает ошибку email", async () => {
  render(<LoginForm />);
  await userEvent.type(screen.getByLabelText("Email"), "not-email");
  await userEvent.click(screen.getByRole("button", { name: "Войти" }));
  expect(await screen.findByText("Некорректный email")).toBeInTheDocument();
});
```
