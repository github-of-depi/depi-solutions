# Node.js — Middle

## Вопросы

- [Как строить REST API с Express?](#как-строить-rest-api-с-express)
- [Что такое middleware в Express?](#что-такое-middleware-в-express)
- [Как обрабатывать ошибки в Node.js?](#как-обрабатывать-ошибки-в-nodejs)
- [Что такое streams в Node.js?](#что-такое-streams-в-nodejs)
- [Как работать с переменными окружения?](#как-работать-с-переменными-окружения)

---

## Как строить REST API с Express?

```typescript
// routes/users.ts
const router = express.Router();

router.get("/", authenticate, async (req, res, next) => {
  try {
    const users = await userService.findAll({ page: req.query.page });
    res.json({ data: users, meta: { total: users.length } });
  } catch (err) { next(err); }
});

router.post("/", authenticate, validate(createUserSchema), async (req, res, next) => {
  try {
    const user = await userService.create(req.body);
    res.status(201).json(user);
  } catch (err) { next(err); }
});

export default router;
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое middleware в Express?

Middleware — функция `(req, res, next)` в pipeline обработки запроса. Порядок важен. `next()` — передать следующему. `next(err)` — к error handler.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как обрабатывать ошибки в Node.js?

```typescript
// Global error handler (последний middleware)
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  console.error(err);
  if (err instanceof ValidationError) return res.status(400).json({ error: err.message });
  if (err instanceof NotFoundError)   return res.status(404).json({ error: "Not found" });
  res.status(500).json({ error: "Internal server error" });
});

// Unhandled rejections
process.on("unhandledRejection", (reason) => { logger.error(reason); process.exit(1); });
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое streams в Node.js?

Streams — обработка данных по частям (не загружать всё в память). Readable, Writable, Transform. Файлы, HTTP ответы, SSE — streams. `pipe()` для chaining.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работать с переменными окружения?

`dotenv` для локальной разработки. Никогда не коммитить `.env`. Валидировать env при старте через zod.

```typescript
const env = z.object({
  PORT: z.coerce.number().default(3000),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
}).parse(process.env);
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
