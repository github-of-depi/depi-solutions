# Архитектурные паттерны — Middle

## Вопросы

- [Как применять SOLID в фронтенде?](#как-применять-solid-в-фронтенде)
- [Что такое паттерн Strategy?](#что-такое-паттерн-strategy)
- [Что такое паттерн Decorator?](#что-такое-паттерн-decorator)
- [Что такое паттерн Facade?](#что-такое-паттерн-facade)
- [Что такое паттерн Repository?](#что-такое-паттерн-repository)
- [Что такое паттерн Command?](#что-такое-паттерн-command)
- [Что такое паттерн Composite в React?](#что-такое-паттерн-composite-в-react)
- [Что такое паттерн Observer через EventEmitter или RxJS?](#что-такое-паттерн-observer-через-eventemitter-или-rxjs)
- [Что такое паттерн Factory при создании компонентов?](#что-такое-паттерн-factory-при-создании-компонентов)
- [Что такое паттерн Adapter при смене библиотеки?](#что-такое-паттерн-adapter-при-смене-библиотеки)

---

## Как применять SOLID в фронтенде?

- **SRP**: компонент — UI, логика — хук, запросы — API слой
- **OCP**: кастомизация через props/children, не через условия внутри компонента
- **LSP**: кастомные компоненты полностью заменяют HTML элементы (Button с теми же атрибутами)
- **ISP**: не передавать весь объект если нужны 1-2 поля
- **DIP**: компонент зависит от интерфейса данных, не от конкретного API источника

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое паттерн Strategy?

Strategy — семейство алгоритмов, инкапсулированных в отдельные объекты, взаимозаменяемых. В JS: функции как стратегии.

```typescript
type SortStrategy<T> = (a: T, b: T) => number;

function sortUsers(users: User[], strategy: SortStrategy<User>): User[] {
  return [...users].sort(strategy);
}

const byName: SortStrategy<User> = (a, b) => a.name.localeCompare(b.name);
const byDate: SortStrategy<User> = (a, b) => a.createdAt.getTime() - b.createdAt.getTime();
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое паттерн Decorator?

Decorator оборачивает объект, добавляя поведение без изменения оригинала. В TypeScript: декораторы классов. В React: HOC — декоратор компонента.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое паттерн Facade?

Facade — упрощённый интерфейс для сложной системы. API слой в приложении — Facade над HTTP.

```typescript
// Facade скрывает детали fetch, auth headers, error handling
export const api = {
  getUsers: () => client.get<User[]>("/users"),
  createUser: (data: NewUser) => client.post<User>("/users", data),
};
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое паттерн Repository?

Repository абстрагирует доступ к данным. Компонент работает с репозиторием, не с конкретным API/storage. Легко мокировать в тестах, легко сменить источник данных.

```typescript
interface UserRepository { findById(id: string): Promise<User>; save(user: User): Promise<void>; }
class ApiUserRepository implements UserRepository {
  findById(id: string) { return fetch(`/api/users/${id}`).then(r => r.json()); }
  save(user: User) { return fetch(`/api/users/${user.id}`, { method: "PUT", body: JSON.stringify(user) }).then(() => {}); }
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое паттерн Command?

Command инкапсулирует запрос как объект. Поддержка undo/redo — каждая команда знает как отменить себя. Использование: редакторы, история действий.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое паттерн Composite в React?

**Composite (Компоновщик)** — компонент, который может содержать другие компоненты того же типа, образуя древовидную структуру.

**Классический пример — Compound Components:**
```tsx
// Компонент-контейнер раскрывает дочерние компоненты как собственные свойства
const Select = ({ children, value, onChange }) => (
  <SelectContext.Provider value={{ value, onChange }}>
    <div className="select">{children}</div>
  </SelectContext.Provider>
);

const Option = ({ value, children }) => {
  const { value: selected, onChange } = useSelectContext();
  return (
    <div
      className={selected === value ? 'selected' : ''}
      onClick={() => onChange(value)}
    >
      {children}
    </div>
  );
};

Select.Option = Option;

// Использование:
<Select value={val} onChange={setVal}>
  <Select.Option value="a">Опция A</Select.Option>
  <Select.Option value="b">Опция B</Select.Option>
</Select>
```

**Другие примеры Composite в React:**
- `<Form>` + `<Form.Item>` + `<Form.Input>` (Ant Design)
- `<Table>` + `<Table.Header>` + `<Table.Row>` + `<Table.Cell>`
- `<Menu>` + `<Menu.Item>` + `<Menu.SubMenu>`

**Преимущества:** гибкость без prop drilling, API напоминает HTML.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое паттерн Observer через EventEmitter или RxJS?

**Observer (Наблюдатель)** — объект публикует события, подписчики реагируют на них независимо друг от друга.

**Реализация через EventEmitter (mitt):**
```typescript
import mitt from 'mitt';

type Events = { 'user:login': { userId: string }; 'cart:update': { count: number } };

export const emitter = mitt<Events>();

// Публикация:
emitter.emit('user:login', { userId: '123' });

// Подписка:
emitter.on('user:login', ({ userId }) => console.log('Logged in:', userId));

// Отписка (важно для утечек памяти):
const handler = ({ userId }) => { /* ... */ };
emitter.on('user:login', handler);
emitter.off('user:login', handler);
```

**Реализация через RxJS Subject:**
```typescript
import { Subject } from 'rxjs';

const cartUpdates$ = new Subject<{ count: number }>();

// Публикация:
cartUpdates$.next({ count: 5 });

// Подписка в React:
useEffect(() => {
  const sub = cartUpdates$.subscribe(({ count }) => setCartCount(count));
  return () => sub.unsubscribe(); // важно!
}, []);
```

**Когда использовать:** связь между несвязанными компонентами без общего контекста; интеграция с внешними системами (WebSocket, analytics).
**Когда НЕ использовать:** если есть React Context или Redux — лучше использовать их, они предсказуемее.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое паттерн Factory при создании компонентов?

**Factory (Фабрика)** — создаёт объект/компонент нужного типа на основе входных данных, скрывая детали создания от вызывающего.

**Применение во фронтенде — компонент по prop `type`:**
```tsx
type WidgetType = 'chart' | 'table' | 'map' | 'counter';

const widgetRegistry: Record<WidgetType, React.ComponentType<WidgetProps>> = {
  chart: ChartWidget,
  table: TableWidget,
  map: MapWidget,
  counter: CounterWidget,
};

const WidgetFactory = ({ type, ...props }: { type: WidgetType } & WidgetProps) => {
  const Component = widgetRegistry[type];
  if (!Component) return <div>Неизвестный тип виджета</div>;
  return <Component {...props} />;
};

// Использование — вызывающий не знает как создаётся компонент:
<WidgetFactory type="chart" data={chartData} />
```

**Преимущества:**
- Добавить новый тип — только в реестр, без изменения вызывающего кода (Open/Closed)
- Легко тестировать каждый виджет изолированно
- Централизованная регистрация компонентов

**Другие применения:** фабрика модальных окон по типу, фабрика правил валидации для форм, фабрика иконок по имени.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое паттерн Adapter при смене библиотеки?

**Adapter (Адаптер)** — обёртка, которая преобразует интерфейс одной библиотеки в интерфейс, ожидаемый приложением. Позволяет менять реализацию без изменения потребителей.

**Пример: смена картографической библиотеки (Leaflet → Яндекс Карты):**
```typescript
// Интерфейс (контракт) который ожидает приложение:
interface MapProvider {
  init(container: HTMLElement, center: [number, number], zoom: number): void;
  addMarker(lat: number, lng: number, popup: string): void;
  destroy(): void;
}

// Адаптер для Leaflet:
class LeafletMapAdapter implements MapProvider {
  private map: L.Map | null = null;
  init(container, center, zoom) {
    this.map = L.map(container).setView(center, zoom);
    L.tileLayer('https://...').addTo(this.map);
  }
  addMarker(lat, lng, popup) {
    L.marker([lat, lng]).bindPopup(popup).addTo(this.map!);
  }
  destroy() { this.map?.remove(); }
}

// Адаптер для Яндекс Карт:
class YandexMapAdapter implements MapProvider {
  private map: ymaps.Map | null = null;
  init(container, [lat, lng], zoom) {
    this.map = new ymaps.Map(container, { center: [lat, lng], zoom });
  }
  addMarker(lat, lng, popup) {
    this.map!.geoObjects.add(new ymaps.Placemark([lat, lng], { balloonContent: popup }));
  }
  destroy() { this.map?.destroy(); }
}

// React компонент работает через интерфейс, не знает реализацию:
const MapComponent = ({ provider }: { provider: MapProvider }) => { /* ... */ };
```

**Преимущества:**
- Смена библиотеки — только новый адаптер
- Тестирование — мок адаптера вместо реальной карты
- Приложение изолировано от API конкретной библиотеки

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
