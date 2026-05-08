# Micro-frontends — Middle

## Вопросы

- [Как настроить Module Federation в Vite?](#как-настроить-module-federation-в-vite)
- [Как организовать роутинг в micro-frontend архитектуре?](#как-организовать-роутинг-в-micro-frontend-архитектуре)
- [Как решать проблему дублирования зависимостей?](#как-решать-проблему-дублирования-зависимостей)
- [Как организовать shared state между micro-frontends?](#как-организовать-shared-state-между-micro-frontends)
- [Что такое Web Components как подход к MFE?](#что-такое-web-components-как-подход-к-mfe)

---

## Как настроить Module Federation в Vite?

```typescript
// vite.config.ts (remote — Team B)
import federation from "@originjs/vite-plugin-federation";
export default { plugins: [federation({
  name: "team-b",
  filename: "remoteEntry.js",
  exposes: { "./ProductList": "./src/features/products/ProductList" },
  shared: ["react", "react-dom"],
})] };

// vite.config.ts (host — Shell)
export default { plugins: [federation({
  name: "shell",
  remotes: { "team-b": "http://team-b.example.com/assets/remoteEntry.js" },
  shared: ["react", "react-dom"],
})] };

// В коде shell
const ProductList = React.lazy(() => import("team-b/ProductList"));
```

---

## Как организовать роутинг в micro-frontend архитектуре?

Shell управляет top-level роутингом (какой MFE грузить). Каждый MFE управляет своим sub-routing. Варианты: hash-based (изоляция), pathname prefix (`/team-b/...`), или shell routing с lazy MFE.

---

## Как решать проблему дублирования зависимостей?

`shared` в Module Federation — singleton зависимости (react, react-dom). Оба приложения используют один экземпляр. Проблема версий: `requiredVersion: "^18.0.0"` — проверка совместимости. ImportMap — альтернативный подход для браузерного шаринга зависимостей.

---

## Как организовать shared state между micro-frontends?

1. **Нет shared state** — лучший вариант (MFE изолированы)
2. **URL как state** — передача через query params/hash
3. **Custom events** — `window.dispatchEvent(new CustomEvent("user:login", { detail }))`
4. **Shared library** — minimal state store в shared пакете
5. Redux — один store в shell, MFE читают через `window.__REDUX_STORE__` (антипаттерн)

---

## Что такое Web Components как подход к MFE?

Web Components (Custom Elements) — нативный стандарт браузера. MFE как Custom Elements: фреймворко-независимы, стандартный API. Недостатки: сложная интеграция с React tree, нет SSR, производительность.
