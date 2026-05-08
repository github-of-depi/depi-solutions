# Architecture Decision Records — Senior

## Вопросы

- [Как строить культуру ADR в команде?](#как-строить-культуру-adr-в-команде)
- [Как провести architecture review с ADR?](#как-провести-architecture-review-с-adr)
- [Как связывать ADR с кодом?](#как-связывать-adr-с-кодом)

---

## Как строить культуру ADR в команде?

1. **Шаблон** — готовый template в репозитории
2. **PR requirement** — значимые изменения архитектуры требуют ADR в PR
3. **Review** — ADR проходят code review как код
4. **Visibility** — список ADR в README, changelog
5. **Регулярный review** — ежеквартально проверять актуальность

---

## Как провести architecture review с ADR?

Architecture review meeting: автор презентует ADR, команда задаёт вопросы, голосует. RFC (Request for Comments) процесс: 1-2 недели для комментариев. Для значимых решений — RFC + ADR после принятия.

---

## Как связывать ADR с кодом?

Комментарии в коде с ссылкой на ADR:

```typescript
// Ref: docs/adr/ADR-003-use-zustand.md
// Zustand выбран вместо Redux для этого модуля — см. ADR
const useStore = create<State>((set) => ({...}));
```

Теги в коде позволяют найти всё связанное с решением.
