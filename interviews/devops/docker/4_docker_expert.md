# Docker — Expert

## Вопросы

- [Как строить production-ready Container strategy?](#как-строить-production-ready-container-strategy)
- [Docker vs другие контейнерные технологии?](#docker-vs-другие-контейнерные-технологии)

---

## Как строить production-ready Container strategy?

1. **Kubernetes**: production orchestration — Deployment, Service, Ingress, HPA
2. **Image registry**: immutable tags (SHA), никогда latest в prod
3. **Health checks**: `HEALTHCHECK` в Dockerfile, liveness/readiness probes в K8s
4. **Resource limits**: CPU и memory limits в K8s (защита от runaway containers)
5. **Rolling updates**: zero-downtime deployment через K8s Deployment strategy
6. **Monitoring**: container metrics (Prometheus/Grafana), centralized logging (Loki)

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Docker vs другие контейнерные технологии?

- **Podman** — rootless альтернатива Docker, совместимый API, нет daemon
- **containerd** — low-level runtime (Docker использует внутри)
- **Buildpacks** (Heroku/Cloud Native) — автоматическая сборка без Dockerfile
- **Nix** — reproducible builds, альтернатива Docker для dev environments
- **Wasm** — WebAssembly как альтернатива контейнерам для edge computing (размер, скорость старта)

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
