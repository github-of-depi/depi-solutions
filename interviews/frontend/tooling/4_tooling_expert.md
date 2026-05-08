# Tooling — Expert

## Вопросы

- [Как работает esbuild изнутри?](#как-работает-esbuild-изнутри)
- [Как строить кастомный build pipeline?](#как-строить-кастомный-build-pipeline)
- [Как оптимизировать CI/CD pipeline для frontend?](#как-оптимизировать-cicd-pipeline-для-frontend)
- [Что такое import maps и как они изменят деплой?](#что-такое-import-maps-и-как-они-изменят-деплой)

---

## Как работает esbuild изнутри?

esbuild написан на Go — параллельная обработка файлов из коробки. Алгоритм: параллельный парсинг AST → link (разрешение зависимостей) → chunk assignment → параллельный print. Минификация через собственный AST-based minifier. В 10-100x быстрее Webpack/Rollup за счёт: Go vs JS, параллелизма, отсутствия type checking.

---

## Как строить кастомный build pipeline?

Rollup API для программного управления сборкой: `rollup.rollup()` → `bundle.generate()` / `bundle.write()`. Полезно для: multi-output библиотек (ESM + CJS + UMD), кастомных трансформаций, специальных условий сборки.

```typescript
const bundle = await rollup({
  input: "src/index.ts",
  plugins: [typescript(), resolve(), commonjs()],
});
await bundle.write({ format: "esm", dir: "dist/esm" });
await bundle.write({ format: "cjs", dir: "dist/cjs" });
await bundle.close();
```

---

## Как оптимизировать CI/CD pipeline для frontend?

1. **Параллелизация**: lint, test, build — параллельно
2. **Кэширование**: `node_modules`, Turborepo remote cache, Docker layer cache
3. **Turborepo Remote Cache** — один кэш на всю команду/CI
4. **Affected only**: тестировать только изменённые пакеты (Nx `affected`, Turborepo `--filter`)
5. **Preview deployments**: каждый PR → preview URL (Vercel, Netlify)
6. **Bundle size regression**: автоматическая проверка размера бандла в PR

---

## Что такое import maps и как они изменят деплой?

Import maps — нативный браузерный механизм маппинга import specifiers на URLs без bundler. `import "react"` → `https://cdn.esm.sh/react@18`. Позволяет деплоить файлы независимо, менять версии без пересборки всего приложения — основа для micro-frontends нового поколения.

```html
<script type="importmap">
{
  "imports": {
    "react": "https://esm.sh/react@18",
    "react-dom": "https://esm.sh/react-dom@18"
  }
}
</script>
```
