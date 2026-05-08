# SQL — Expert

## Вопросы

- [Как строить масштабируемую БД архитектуру?](#как-строить-масштабируемую-бд-архитектуру)
- [Что такое шардирование?](#что-такое-шардирование)

---

## Как строить масштабируемую БД архитектуру?

1. **Read replicas**: мастер для writes, реплики для reads
2. **Connection pooling**: PgBouncer — тысячи клиентов, ограниченный pool БД
3. **Partitioning**: по дате (range), по user_id (hash) для больших таблиц
4. **Caching**: Redis для горячих данных, materialized views для отчётов
5. **VACUUM/ANALYZE**: auto vacuum настроен, не accumulating dead tuples

---

## Что такое шардирование?

Sharding — горизонтальное разбиение данных между несколькими БД. Каждый шард — независимая БД с подмножеством данных. Сложность: cross-shard queries, resharding, global unique IDs. Инструменты: Citus (PostgreSQL), Vitess (MySQL).
