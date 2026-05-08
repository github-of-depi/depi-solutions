# Git — Senior

## Вопросы

- [rebase vs merge: когда что использовать?](#rebase-vs-merge-когда-что-использовать)
- [Как разрешать сложные конфликты?](#как-разрешать-сложные-конфликты)
- [Что такое git bisect?](#что-такое-git-bisect)
- [Как организовать git workflow для команды?](#как-организовать-git-workflow-для-команды)

---

## rebase vs merge: когда что использовать?

**Rebase**: feature ветки перед merge в main (чистая история). `git rebase -i HEAD~3` — интерактивный rebase (squash, reorder). **Merge**: при объединении в main/develop (сохранить контекст). **Правило**: rebase локальных веток, merge публичных. Никогда не rebase main/develop.

---

## Как разрешать сложные конфликты?

```bash
git merge feature/x  # конфликт
git status           # посмотреть conflicted файлы
# Открыть в IDE — <<<< HEAD ... ==== ... >>>> feature/x
git add resolved.ts  # после разрешения
git merge --continue # завершить
git merge --abort    # отменить если всё плохо
```

VS Code и IntelliJ — удобный diff для конфликтов.

---

## Что такое git bisect?

`git bisect` — бинарный поиск коммита, введшего баг.

```bash
git bisect start
git bisect bad           # текущий HEAD — с багом
git bisect good v1.0.0   # v1.0.0 — без бага
# Git переключает на средний коммит, проверяешь
git bisect good/bad      # отмечаешь, git продолжает поиск
git bisect reset         # завершить
```

---

## Как организовать git workflow для команды?

1. **Trunk-Based Development** (рекомендуется): частые малые коммиты в main, feature flags, CI на каждый commit
2. **Branch naming**: `feat/jira-123-user-profile`, `fix/jira-456-login-error`
3. **PR size**: <400 строк правило
4. **Protected branches**: main требует PR + review + CI green
5. **Commit signing**: GPG подпись коммитов для верификации
