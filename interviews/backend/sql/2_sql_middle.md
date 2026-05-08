# SQL — Middle

## Вопросы

- [Как работают JOIN?](#как-работают-join)
- [Что такое GROUP BY и агрегатные функции?](#что-такое-group-by-и-агрегатные-функции)
- [Что такое индексы?](#что-такое-индексы)
- [Что такое транзакции?](#что-такое-транзакции)
- [Что такое оконные функции?](#что-такое-оконные-функции)

---

## Как работают JOIN?

```sql
-- INNER JOIN: только совпадающие строки
SELECT u.name, o.total FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT JOIN: все пользователи + их заказы (NULL если нет)
SELECT u.name, COUNT(o.id) as order_count FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id;

-- Несколько JOIN
SELECT u.name, p.name as product, oi.quantity
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id;
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое GROUP BY и агрегатные функции?

```sql
-- Количество заказов по пользователям
SELECT user_id, COUNT(*) as total_orders, SUM(total) as revenue
FROM orders
WHERE created_at > '2024-01-01'
GROUP BY user_id
HAVING COUNT(*) > 5  -- фильтр после агрегации
ORDER BY revenue DESC;
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое индексы?

Индекс ускоряет поиск за счёт дополнительной структуры. Без индекса — sequential scan всей таблицы. B-tree (default): для `=`, `>`, `<`, `BETWEEN`, `ORDER BY`.

```sql
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_user_created ON orders(user_id, created_at);
-- Уникальный
CREATE UNIQUE INDEX idx_users_email_unique ON users(email);
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое транзакции?

ACID транзакция — атомарная группа операций. Или все выполнятся, или ни одна.

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
-- Проверить ошибки
COMMIT;  -- или ROLLBACK при ошибке
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое оконные функции?

Оконные функции — вычисления по группе строк без свёртки (OVER). Мощнее GROUP BY.

```sql
SELECT 
  name, department, salary,
  AVG(salary) OVER (PARTITION BY department) as dept_avg,
  RANK() OVER (ORDER BY salary DESC) as salary_rank,
  ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as dept_rank
FROM employees;
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
