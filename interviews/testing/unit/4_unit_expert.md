# Unit Testing — Expert

## Вопросы

- [Как строить testing infrastructure для большого проекта?](#как-строить-testing-infrastructure-для-большого-проекта)
- [Как тестировать перформанс?](#как-тестировать-перформанс)
- [Что такое mutation testing?](#что-такое-mutation-testing)

---

## Как строить testing infrastructure для большого проекта?

1. **Vitest вместо Jest** — нативная поддержка ESM, быстрее в monorepo
2. **Test sharding** — параллельный запуск тестов в CI (vitest --shard)
3. **Coverage thresholds** — PR блокируется при падении coverage
4. **Custom matchers** — доменно-специфичные `expect(user).toBeActiveUser()`
5. **Test factories** — `createUser({ role: "admin" })` вместо дублирования моков

---

## Как тестировать перформанс?

```typescript
it("рендерит 1000 элементов за приемлемое время", () => {
  const items = Array.from({ length: 1000 }, (_, i) => ({ id: i, name: `Item ${i}` }));
  const start = performance.now();
  render(<VirtualList items={items} />);
  const duration = performance.now() - start;
  expect(duration).toBeLessThan(100); // < 100ms
});
```

Playwright `page.metrics()` для LCP/FCP в E2E. Benchmark через vitest bench.

---

## Что такое mutation testing?

Mutation testing: инструмент (Stryker) вносит мутации в код (`>` → `>=`, удаляет условия) и проверяет, что тесты их находят. Mutation score — % убитых мутантов. Высокий coverage + низкий mutation score = тесты не проверяют логику. Полезно для критической бизнес-логики.
