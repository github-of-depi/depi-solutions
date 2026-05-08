# Tooling — Middle

## Вопросы

- [Как настроить Vite для production?](#как-настроить-vite-для-production)
- [Как работает HMR (Hot Module Replacement)?](#как-работает-hmr-hot-module-replacement)
- [Что такое tree shaking и как его обеспечить?](#что-такое-tree-shaking-и-как-его-обеспечить)
- [Как настроить ESLint и Prettier?](#как-настроить-eslint-и-prettier)
- [Что такое path aliases?](#что-такое-path-aliases)
- [Как работает source maps?](#как-работает-source-maps)

---

## Как настроить Vite для production?

`vite.config.ts`: `build.rollupOptions` для кастомных chunks, `build.minify: "esbuild"`, `build.sourcemap`, `plugins` (React, SVG, etc). Разделение вендорного кода в отдельный chunk улучшает кэширование.

```typescript
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: { vendor: ["react", "react-dom"] },
      },
    },
  },
  resolve: { alias: { "@": path.resolve(__dirname, "src") } },
});
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работает HMR (Hot Module Replacement)?

HMR обновляет только изменённые модули без перезагрузки страницы, сохраняя состояние. Vite: dev сервер через WebSocket отправляет invalidation сообщения клиенту. Клиент запрашивает только изменённый модуль. React Fast Refresh — React-специфичный HMR с сохранением state компонентов.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое tree shaking и как его обеспечить?

Tree shaking удаляет неиспользуемый код при сборке. Требования: ES modules (`import`/`export`), `"sideEffects": false` в package.json. Использовать именованные импорты: `import { format } from "date-fns"` вместо `import dayjs from "dayjs"`.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как настроить ESLint и Prettier?

ESLint — статический анализ кода (ошибки, антипаттерны). Prettier — форматирование. Не дублировать: `eslint-config-prettier` отключает ESLint правила конфликтующие с Prettier. `husky` + `lint-staged` — запуск в pre-commit hook.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое path aliases?

Aliases заменяют относительные пути на абсолютные псевдонимы. `@` часто алиас для `src/`. Настраивается в `vite.config.ts` (resolve.alias) и `tsconfig.json` (paths — для TypeScript).

```typescript
// tsconfig.json
{ "paths": { "@/*": ["src/*"] } }
// Использование
import { Button } from "@/components/Button";
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работает source maps?

Source maps — файлы, маппирующие скомпилированный/минифицированный код обратно на исходный. Браузер DevTools показывает оригинальный TypeScript вместо минифицированного JS. Типы: `inline` (всё в одном файле), `external` (.map файл). В production: hidden (maps на сервере для Sentry, не публичные).

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
