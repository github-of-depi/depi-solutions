# Node.js — Junior

## Вопросы

- [Что такое Node.js?](#что-такое-nodejs)
- [Что такое event loop в Node.js?](#что-такое-event-loop-в-nodejs)
- [Как создать простой HTTP сервер?](#как-создать-простой-http-сервер)
- [Что такое npm пакеты и package.json?](#что-такое-npm-пакеты-и-packagejson)
- [Что такое CommonJS vs ES Modules?](#что-такое-commonjs-vs-es-modules)

---

## Что такое Node.js?

Node.js — JavaScript runtime на базе V8 движка Chrome. Позволяет запускать JS на сервере. Однопоточный, но асинхронный через event loop. Популярен для: REST API, real-time сервисов, BFF, инструментов (webpack, vite).

---

## Что такое event loop в Node.js?

Event loop позволяет Node.js выполнять неблокирующие I/O операции в одном потоке. Фазы: timers (setTimeout/setInterval), poll (I/O), check (setImmediate). Microtasks (Promises, process.nextTick) — между фазами.

---

## Как создать простой HTTP сервер?

```typescript
// Express
import express from "express";
const app = express();
app.use(express.json());

app.get("/api/users", async (req, res) => {
  const users = await db.users.findAll();
  res.json(users);
});

app.listen(3000, () => console.log("Server running on :3000"));
```

---

## Что такое npm пакеты и package.json?

`package.json` — манифест проекта: название, версия, dependencies, scripts. `npm install` → `node_modules/`. `dependencies` — runtime. `devDependencies` — только для разработки. `peerDependencies` — ожидает установки от потребителя.

---

## Что такое CommonJS vs ES Modules?

**CJS**: `require()` / `module.exports` — Node.js стандарт до ESM. Синхронный. **ESM**: `import` / `export` — стандарт браузера и современного Node. Асинхронный, tree-shakeable. `"type": "module"` в package.json для ESM в Node.
