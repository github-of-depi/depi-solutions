# Git — Expert

## Вопросы

- [Что такое Git internals?](#что-такое-git-internals)
- [Как строить автоматизацию поверх Git?](#как-строить-автоматизацию-поверх-git)

---

## Что такое Git internals?

Git хранит объекты: **blob** (содержимое файла), **tree** (директория), **commit** (snapshot + metadata), **tag**. Все адресованы SHA-1 hash содержимого. `.git/objects/` — object database. Refs (ветки, теги) — указатели на commit объекты.

```bash
git cat-file -t abc123  # тип объекта (blob/tree/commit)
git cat-file -p abc123  # содержимое объекта
git ls-tree HEAD        # дерево файлов
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как строить автоматизацию поверх Git?

1. **GitHub Actions** — CI/CD на git events (push, PR, release)
2. **Semantic Release** — автоматическое версионирование из Conventional Commits
3. **Changesets** — changelog и версионирование в monorepo
4. **Release Please** — Google tool для automated releases
5. **Git hooks CI** — pre-commit (lint), commit-msg (conventional), pre-push (tests)

```yaml
# .github/workflows/release.yml
on:
  push:
    branches: [main]
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: google-github-actions/release-please-action@v3
        with:
          release-type: node
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
