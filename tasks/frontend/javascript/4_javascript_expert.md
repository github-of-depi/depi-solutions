# JavaScript — Expert Tasks

## Задачи

- [Параллельная очередь задач (Task Queue)](#параллельная-очередь-задач-task-queue)
- [Функция spyOn](#функция-spyon)

---

## Параллельная очередь задач (Task Queue)

Реализуйте класс `Queue`, который принимает:
1. `processTask(task, resolve)` — обработчик одной задачи; задача считается выполненной после вызова `resolve`
2. `parallelLimit` — максимальное количество одновременно обрабатываемых задач
3. `onEmpty` — колбэк, вызываемый когда все задачи обработаны

Требования:
- Метод `add(task)` — добавить задачу в очередь
- Метод `start()` — запустить обработку
- Не более `parallelLimit` задач одновременно
- При завершении всех задач вызвать `onEmpty`

```javascript
const processTask = (task, resolve) => {
  const workTime = 500 + Math.random() * 500;
  setTimeout(() => { console.log(task); resolve(); }, workTime);
};

const queue = new Queue(processTask, 2, () => console.log('Queue is empty'));
queue.add('task 1');
queue.add('task 2');
queue.add('task 3');
queue.start();

// Возможный вывод (порядок task 1/2 может меняться):
// task 2
// task 1
// task 3
// Queue is empty
```

**Связанные вопросы:**

- [Async/Await: как работает под капотом?](../../../interviews/frontend/javascript/3_javascript_senior.md#asyncawait-как-работает-под-капотом)

<details>
<summary>Решение</summary>

```javascript
class Queue {
  constructor(processTask, parallelLimit, onEmpty) {
    this.processTask = processTask;
    this.parallelLimit = parallelLimit;
    this.onEmpty = onEmpty;
    this.tasks = [];
    this.running = 0;
    this.completed = 0;
    this.total = 0;
  }

  add(task) {
    this.tasks.push(task);
    this.total++;
  }

  start() {
    // Запускаем до parallelLimit задач одновременно
    for (let i = 0; i < this.parallelLimit; i++) {
      this._next();
    }
  }

  _next() {
    if (this.tasks.length === 0) {
      // Если нечего обрабатывать — проверяем, всё ли завершено
      if (this.running === 0 && this.completed === this.total) {
        this.onEmpty();
      }
      return;
    }

    const task = this.tasks.shift();
    this.running++;

    this.processTask(task, () => {
      this.running--;
      this.completed++;
      this._next(); // берём следующую задачу
    });
  }
}
```

</details>

---

## Функция spyOn

Реализуйте функцию `spyOn(obj, methodName)`, которая отслеживает вызовы метода объекта, **не изменяя** его поведение. Функция должна возвращать объект `spy` со свойством `calls` — массивом аргументов каждого вызова.

```javascript
const person = {
  firstName: '',
  lastName: '',
  update(fullName) {
    const [first, last] = fullName.split(' ');
    this.firstName = first;
    this.lastName = last;
  }
};

const spy = spyOn(person, 'update');

person.update('Иван Иванов');
console.log(person.firstName, person.lastName); // Иван Иванов

person.update('Пётр Петров');
console.log(person.firstName, person.lastName); // Пётр Петров

console.log(spy.calls); // [['Иван Иванов'], ['Пётр Петров']]
```

**Связанные вопросы:**

- [Что такое Proxy и Reflect?](../../../interviews/frontend/javascript/4_javascript_expert.md#что-такое-proxy-и-reflect)

<details>
<summary>Решение</summary>

```javascript
// Вариант 1: подмена метода
function spyOn(obj, methodName) {
  const originalFn = obj[methodName];
  const spy = { calls: [] };

  obj[methodName] = function(...args) {
    spy.calls.push(args);
    return originalFn.apply(this, args);
  };

  return spy;
}

// Вариант 2: через Proxy (без изменения оригинального объекта)
function spyOnProxy(obj, methodName) {
  const spy = { calls: [] };

  const handler = {
    get(target, prop) {
      if (prop === methodName) {
        return function(...args) {
          spy.calls.push(args);
          return target[methodName].apply(target, args);
        };
      }
      return target[prop];
    }
  };

  return { proxy: new Proxy(obj, handler), spy };
}

// Использование варианта 1
const spy = spyOn(person, 'update');
person.update('Иван Иванов');
console.log(spy.calls); // [['Иван Иванов']]
```

</details>
