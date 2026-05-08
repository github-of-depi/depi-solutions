# System Design (Frontend) — Expert

## Вопросы

- [Как проектировать Google Docs-масштаба систему?](#как-проектировать-google-docs-масштаба-систему)
- [Как строить глобально распределённый фронтенд?](#как-строить-глобально-распределённый-фронтенд)
- [Как проектировать систему для 10M+ пользователей?](#как-проектировать-систему-для-10m-пользователей)

---

## Как проектировать Google Docs-масштаба систему?

Ключевые решения:
1. **CRDT** (Diamond Types / Yjs) — децентрализованный merge, нет single point of authority
2. **Snapshot + Operations log** — эффективное хранение истории
3. **Presence service** — отдельный сервис для cursor awareness (не через основной WS)
4. **Permissions** — документ-level ACL, real-time permission changes
5. **Versioning** — named versions, restore, history timeline

---

## Как строить глобально распределённый фронтенд?

- **Multi-region deployment**: Vercel / Cloudflare Workers — code near users
- **Edge caching**: HTML кэшируется на Edge с smart invalidation
- **Geo-routing**: пользователи роутятся в ближайший регион
- **Data residency**: GDPR — данные EU пользователей остаются в EU
- **Latency budgets**: target <100ms TTFB для любого региона

---

## Как проектировать систему для 10M+ пользователей?

1. **Bundle splitting**: code split по роутам, lazy components
2. **CDN**: все статические ресурсы, Edge HTML caching
3. **Database reads**: Read replicas, Redis кэш для популярных данных
4. **WebSocket scaling**: Redis Pub/Sub, Kafka для событий
5. **Graceful degradation**: core features работают при деградации сервисов
6. **Feature flags**: постепенный rollout, kill switches
