# Git — Middle

## Вопросы

- [Что такое Git Flow?](#что-такое-git-flow)
- [Как работать с git stash?](#как-работать-с-git-stash)
- [Что такое cherry-pick?](#что-такое-cherry-pick)
- [Как исправить последний коммит?](#как-исправить-последний-коммит)
- [Что такое git hooks?](#что-такое-git-hooks)
- [Как написать хороший commit message?](#как-написать-хороший-commit-message)

---

## Что такое Git Flow?

Git Flow — branching стратегия: `main` (производство), `develop` (интеграция), `feature/*`, `release/*`, `hotfix/*`. Для команд с регулярными релизами. Альтернативы: Trunk-Based Development (частые малые коммиты в main), GitHub Flow (main + feature branches).

---

## Как работать с git stash?

Stash — временное сохранение незакомиченных изменений.

```bash
git stash                  # сохранить изменения
git stash pop              # восстановить последние
git stash list             # список stash-ей
git stash apply stash@{1}  # применить конкретный
git stash drop stash@{0}   # удалить
```

---

## Что такое cherry-pick?

`git cherry-pick` — применить конкретный коммит к другой ветке.

```bash
git cherry-pick abc123  # применить коммит abc123 к текущей ветке
```

Использование: hotfix в main → cherry-pick в develop, выборочный перенос фичи.

---

## Как исправить последний коммит?

```bash
git commit --amend -m "исправленное сообщение"  # изменить сообщение
git add file.ts ; git commit --amend             # добавить файл в последний коммит
# Только для не отправленных коммитов!
```

---

## Что такое git hooks?

Git hooks — скрипты, выполняемые при git событиях. `pre-commit`: lint + format. `commit-msg`: проверка формата сообщения. `pre-push`: запуск тестов. Husky — удобная настройка hooks через npm.

---

## Как написать хороший commit message?

Conventional Commits:
```
feat: добавить страницу профиля пользователя
fix: исправить ошибку валидации email
refactor: разбить компонент Header на подкомпоненты
docs: обновить README с инструкциями установки
```
Формат: `type(scope): description`. Imperative mood. 72 символа на первой строке.
