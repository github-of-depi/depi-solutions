# SQL — Junior

## Вопросы

- [Что такое реляционная база данных?](#что-такое-реляционная-база-данных)
- [Как использовать SELECT и WHERE?](#как-использовать-select-и-where)
- [Что такое PRIMARY KEY и FOREIGN KEY?](#что-такое-primary-key-и-foreign-key)
- [Как вставить, обновить и удалить данные?](#как-вставить-обновить-и-удалить-данные)
- [Что такое NULL?](#что-такое-null)

---

## Что такое реляционная база данных?

Реляционная БД — хранит данные в таблицах со строками и столбцами. Связи между таблицами через ключи. SQL — язык запросов. Примеры: PostgreSQL, MySQL, SQLite.

---

## Как использовать SELECT и WHERE?

```sql
-- Все пользователи
SELECT * FROM users;

-- Конкретные поля
SELECT id, name, email FROM users;

-- Фильтрация
SELECT * FROM users WHERE role = 'admin' AND is_active = true;

-- Поиск по паттерну
SELECT * FROM users WHERE name ILIKE '%alice%';

-- Сортировка и лимит
SELECT * FROM users ORDER BY created_at DESC LIMIT 10 OFFSET 20;
```

---

## Что такое PRIMARY KEY и FOREIGN KEY?

**PRIMARY KEY** — уникальный идентификатор строки, не NULL. **FOREIGN KEY** — ссылка на PRIMARY KEY другой таблицы. Обеспечивает referential integrity — нельзя добавить запись с несуществующим FK.

---

## Как вставить, обновить и удалить данные?

```sql
-- INSERT
INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com');

-- UPDATE
UPDATE users SET role = 'admin' WHERE id = 1;

-- DELETE
DELETE FROM users WHERE id = 1;

-- Soft delete (лучше чем DELETE)
UPDATE users SET deleted_at = NOW() WHERE id = 1;
```

---

## Что такое NULL?

NULL — отсутствие значения (не пустая строка, не 0). `WHERE email IS NULL` / `IS NOT NULL`. `COALESCE(value, default)` — вернуть default если NULL.
