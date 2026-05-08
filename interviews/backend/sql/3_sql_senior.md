# SQL — Senior

## Вопросы

- [Как оптимизировать медленные запросы?](#как-оптимизировать-медленные-запросы)
- [Что такое EXPLAIN ANALYZE?](#что-такое-explain-analyze)
- [Как строить эффективную схему БД?](#как-строить-эффективную-схему-бд)
- [Что такое materialized views?](#что-такое-materialized-views)

---

## Как оптимизировать медленные запросы?

1. Добавить нужные индексы (смотреть на WHERE, JOIN, ORDER BY колонки)
2. Избегать `SELECT *` — только нужные поля
3. `LIMIT` где возможно
4. Избегать функций на индексированных колонках в WHERE (`WHERE LOWER(email)` → нет индекса)
5. N+1 → JOIN или подзапрос
6. Покрывающие индексы: все поля запроса в индексе → index-only scan

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое EXPLAIN ANALYZE?

`EXPLAIN ANALYZE` — план выполнения запроса с реальным временем.

```sql
EXPLAIN ANALYZE
SELECT * FROM orders WHERE user_id = 1 ORDER BY created_at DESC;
```

Что смотреть: `Seq Scan` (медленно для больших таблиц) → `Index Scan` (с индексом). `cost`, `actual time`, `rows`. `Nested Loop` при JOIN без индекса → `Hash Join`.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как строить эффективную схему БД?

1. **Нормализация** (3NF) — устранить дублирование данных
2. **UUID vs serial**: UUID — уникален глобально, serial — меньше хранилища
3. **Timestamp**: `created_at DEFAULT NOW()`, `updated_at` — всегда добавлять
4. **Soft delete**: `deleted_at TIMESTAMP` вместо DELETE
5. **Enum vs reference table**: enum для стабильных значений, таблица — для динамических

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое materialized views?

Materialized view — результат запроса сохраняется как таблица. Быстрое чтение, но данные устаревают. `REFRESH MATERIALIZED VIEW` — обновление (можно CONCURRENTLY без блокировки).

```sql
CREATE MATERIALIZED VIEW monthly_revenue AS
SELECT DATE_TRUNC('month', created_at) as month, SUM(total) as revenue
FROM orders GROUP BY 1;

REFRESH MATERIALIZED VIEW CONCURRENTLY monthly_revenue;
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
