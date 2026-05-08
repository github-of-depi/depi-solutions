# Дизайн компонентов — Middle

## Вопросы

- [Что такое Compound Components паттерн?](#что-такое-compound-components-паттерн)
- [Что такое Headless Components?](#что-такое-headless-components)
- [Что такое Polymorphic Component?](#что-такое-polymorphic-component)
- [Как проектировать API компонента?](#как-проектировать-api-компонента)
- [Что такое Controlled vs Uncontrolled паттерн в библиотеках?](#что-такое-controlled-vs-uncontrolled-паттерн-в-библиотеках)

---

## Что такое Compound Components паттерн?

Compound Components — группа компонентов, работающих вместе через shared state (Context). Пример: `<Select>`, `<Select.Trigger>`, `<Select.Content>`, `<Select.Item>`.

```typescript
const TabsContext = createContext<TabsState>(null!);

function Tabs({ children, defaultValue }: Props) {
  const [active, setActive] = useState(defaultValue);
  return <TabsContext.Provider value={{ active, setActive }}>{children}</TabsContext.Provider>;
}
Tabs.List = TabsList;
Tabs.Tab = Tab;
Tabs.Panel = TabsPanel;

// Usage:
<Tabs defaultValue="first">
  <Tabs.List>
    <Tabs.Tab value="first">Первая</Tabs.Tab>
  </Tabs.List>
  <Tabs.Panel value="first">Контент</Tabs.Panel>
</Tabs>
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое Headless Components?

Headless — компонент предоставляет поведение (logic, a11y, state) без UI. Стилизация — задача потребителя. Примеры: Radix UI, Headless UI, React Aria.

```typescript
function useAccordion(defaultOpen?: string) {
  const [open, setOpen] = useState<string | undefined>(defaultOpen);
  const toggle = (id: string) => setOpen(prev => prev === id ? undefined : id);
  return { open, toggle };
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое Polymorphic Component?

Polymorphic Component — компонент, рендерящий разные HTML/компоненты через `as` prop. `<Button as="a" href="...">` рендерит ссылку, `<Button as={Link}>` — React Router Link.

```typescript
type PolymorphicProps<E extends ElementType> = {
  as?: E;
} & ComponentPropsWithRef<E>;

function Button<E extends ElementType = "button">({ as, ...props }: PolymorphicProps<E>) {
  const Component = as ?? "button";
  return <Component {...props} />;
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как проектировать API компонента?

1. Минимальный API — только необходимые props
2. Принцип наименьшего удивления — поведение как у нативного HTML
3. Composition-friendly — поддержка `children`, `className`, forwarded `ref`
4. Escape hatch — способ кастомизировать без форка компонента

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое Controlled vs Uncontrolled паттерн в библиотеках?

Библиотеки UI поддерживают оба режима: controlled (состояние снаружи через value+onChange), uncontrolled (defaultValue, внутреннее состояние). Реализовывать через `useControllableState`: если передан `value` — controlled mode, иначе uncontrolled.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
