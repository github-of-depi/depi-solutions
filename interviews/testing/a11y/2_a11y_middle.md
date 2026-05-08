# Accessibility (A11y) — Middle

## Вопросы

- [Как строить keyboard navigation?](#как-строить-keyboard-navigation)
- [Как управлять фокусом?](#как-управлять-фокусом)
- [Как тестировать a11y автоматически?](#как-тестировать-a11y-автоматически)
- [Как строить доступные формы?](#как-строить-доступные-формы)
- [Что такое aria-live regions?](#что-такое-aria-live-regions)

---

## Как строить keyboard navigation?

Все интерактивные элементы доступны через Tab. Порядок: следует DOM order. Кастомные компоненты реализуют паттерны WAI-ARIA: стрелки для списков/меню, Escape для закрытия диалогов.

---

## Как управлять фокусом?

При открытии модала — фокус перемещается внутрь. При закрытии — возвращается к триггеру. Focus trap — фокус не уходит из модала.

```typescript
function Modal({ isOpen, onClose, triggerRef }: Props) {
  const modalRef = useRef<HTMLDivElement>(null);
  
  useEffect(() => {
    if (isOpen) {
      modalRef.current?.focus();
      // Focus trap: prevent focus leaving modal
    } else {
      triggerRef.current?.focus(); // Вернуть фокус
    }
  }, [isOpen]);
  
  return <div ref={modalRef} tabIndex={-1} role="dialog" aria-modal="true">...</div>;
}
```

---

## Как тестировать a11y автоматически?

jest-axe + RTL: автоматическая проверка в unit тестах.

```typescript
import { axe, toHaveNoViolations } from "jest-axe";
expect.extend(toHaveNoViolations);

test("форма не имеет a11y нарушений", async () => {
  const { container } = render(<LoginForm />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

---

## Как строить доступные формы?

```html
<div>
  <label for="email">Email <span aria-label="обязательное поле">*</span></label>
  <input id="email" type="email" required aria-describedby="email-error" />
  <span id="email-error" role="alert" aria-live="polite">
    {error && "Введите корректный email"}
  </span>
</div>
```

---

## Что такое aria-live regions?

`aria-live` — объявлять динамические изменения screen reader без перемещения фокуса. `polite` — объявить после текущего чтения. `assertive` — прерывает (для критичных ошибок).

```typescript
<div aria-live="polite" aria-atomic="true">
  {successMessage}
</div>
```
