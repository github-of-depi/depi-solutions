# Docker — Senior

## Вопросы

- [Как строить Docker workflow для frontend проекта?](#как-строить-docker-workflow-для-frontend-проекта)
- [Как работать с Docker в CI/CD?](#как-работать-с-docker-в-cicd)
- [Что такое Docker layer caching?](#что-такое-docker-layer-caching)
- [Docker security best practices?](#docker-security-best-practices)

---

## Как строить Docker workflow для frontend проекта?

1. `docker-compose.dev.yml` — локальная разработка (volumes для hot reload)
2. `Dockerfile` — production multi-stage сборка
3. Image теги: `my-app:${GIT_SHA}` — трассировка
4. Registry: GitHub Container Registry или AWS ECR
5. Semantic versioning образов + latest тег

---

## Как работать с Docker в CI/CD?

```yaml
# GitHub Actions
- name: Build and push Docker image
  uses: docker/build-push-action@v5
  with:
    context: .
    push: true
    tags: ghcr.io/org/app:${{ github.sha }}
    cache-from: type=gha    # GitHub Actions cache
    cache-to: type=gha,mode=max
```

---

## Что такое Docker layer caching?

Docker кэширует слои. Если слой не изменился — использует кэш. **Порядок важен**: копировать `package.json` раньше исходников → npm install кэшируется пока не меняется package.json.

```dockerfile
COPY package*.json ./   # редко меняется → кэшируется
RUN npm ci              # кэш если package.json не изменился
COPY . .                # часто меняется → инвалидирует кэш
```

---

## Docker security best practices?

1. **Non-root user**: `USER node` в Dockerfile
2. **Read-only filesystem**: `--read-only` при запуске
3. **Minimal image**: scratch или distroless вместо ubuntu
4. **Scan**: `docker scout cves` или Trivy для проверки CVE
5. **Secrets**: не в Dockerfile/image, через environment или Docker secrets
