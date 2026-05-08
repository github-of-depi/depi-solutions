# Tooling — Senior

## Вопросы

- [Как писать кастомные Vite плагины?](#как-писать-кастомные-vite-плагины)
- [Как настроить monorepo с Turborepo?](#как-настроить-monorepo-с-turborepo)
- [Как оптимизировать время сборки?](#как-оптимизировать-время-сборки)
- [Как мигрировать с Webpack на Vite?](#как-мигрировать-с-webpack-на-vite)
- [Что такое Module Federation?](#что-такое-module-federation)

---

## Как писать кастомные Vite плагины?

Vite плагины — объекты с хуками (rollup + vite-specific). Хуки: `transform` — трансформация кода, `load` — кастомная загрузка модулей, `resolveId` — резолвинг импортов, `configureServer` — middleware для dev сервера.

```typescript
function myPlugin(): Plugin {
  return {
    name: "my-plugin",
    transform(code, id) {
      if (id.endsWith(".svg")) {
        return { code: `export default ${JSON.stringify(code)}` };
      }
    },
    configureServer(server) {
      server.middlewares.use("/api", (req, res) => {
        res.end(JSON.stringify({ status: "ok" }));
      });
    },
  };
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как настроить monorepo с Turborepo?

`turbo.json` описывает pipeline задач с зависимостями между пакетами. Turborepo кэширует результаты на основе хеша входных файлов — повторный запуск без изменений мгновенный.

```json
{
  "pipeline": {
    "build": { "dependsOn": ["^build"], "outputs": ["dist/**"] },
    "test": { "dependsOn": ["build"] },
    "dev": { "cache": false, "persistent": true }
  }
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как оптимизировать время сборки?

1. **SWC/esbuild** вместо Babel — 10-100x быстрее
2. **TypeScript project references** — инкрементальная компиляция
3. **Turborepo/Nx caching** — не пересобирать неизменённые пакеты
4. **Vite** вместо Webpack в dev — нет бандлинга в dev режиме
5. **CI кэширование** `node_modules` и build output
6. `skipLibCheck: true` в tsconfig

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как мигрировать с Webpack на Vite?

1. Заменить `webpack.config.js` на `vite.config.ts`
2. Обновить `import.meta.env` вместо `process.env`
3. Заменить `require()` на `import` (Vite — ESM only)
4. Обновить `public/index.html` — Vite использует его напрямую
5. Кастомные Webpack loaders → Vite плагины
6. Проверить CJS-only зависимости — нужна `optimizeDeps.include`

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое Module Federation?

Module Federation (Webpack 5, @module-federation) — runtime sharing модулей между приложениями. Host загружает federated модули из удалённых приложений без пересборки. Основа для micro-frontends. Shared dependencies — общий React между приложениями без дублирования.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
