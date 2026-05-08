# Docker — Middle

## Вопросы

- [Как создать оптимальный Dockerfile для фронтенда?](#как-создать-оптимальный-dockerfile-для-фронтенда)
- [Что такое multi-stage build?](#что-такое-multi-stage-build)
- [Что такое docker-compose?](#что-такое-docker-compose)
- [Как оптимизировать размер Docker image?](#как-оптимизировать-размер-docker-image)

---

## Как создать оптимальный Dockerfile для фронтенда?

```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production=false
COPY . .
RUN npm run build

# Production stage — только nginx + статика
FROM nginx:alpine AS runner
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

## Что такое multi-stage build?

Multi-stage — несколько FROM в одном Dockerfile. Итоговый image содержит только нужное (нет node_modules, devDependencies). Значительно уменьшает размер image.

---

## Что такое docker-compose?

docker-compose — запуск нескольких контейнеров вместе.

```yaml
# docker-compose.yml
services:
  frontend:
    build: .
    ports: ["3000:80"]
  backend:
    image: my-api:latest
    ports: ["4000:4000"]
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes: [pgdata:/var/lib/postgresql/data]
volumes:
  pgdata:
```

---

## Как оптимизировать размер Docker image?

1. Alpine базовые образы (node:20-alpine vs node:20 — в 3x меньше)
2. Multi-stage builds
3. `.dockerignore` — исключить node_modules, .git, .env
4. `npm ci --only=production` — только production зависимости
5. Объединять `RUN` команды (меньше слоёв)
