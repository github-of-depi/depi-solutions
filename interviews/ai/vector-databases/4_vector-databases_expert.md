# Vector Databases — 🔴 Expert

## Вопросы

- [Архитектура векторной БД для 1B+ векторов](#архитектура-векторной-бд-для-1b-векторов)
- [Как реализовать online learning для векторного индекса?](#как-реализовать-online-learning-для-векторного-индекса)
- [Гибридный поиск с поддержкой streaming updates](#гибридный-поиск-с-поддержкой-streaming-updates)
- [Мониторинг и профилирование векторного поиска](#мониторинг-и-профилирование-векторного-поиска)

---

## Архитектура векторной БД для 1B+ векторов

```
Проблема: 1B векторов × 1536 float32 = ~6TB памяти
Невозможно держать всё в RAM на одной машине.

Решение: Distributed + Hierarchical Storage
```

```
                    ┌─────────────────────────────────┐
                    │         Load Balancer            │
                    └──────────────┬──────────────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              │                    │                    │
    ┌─────────▼──────┐  ┌──────────▼─────┐  ┌──────────▼─────┐
    │   Shard 1       │  │   Shard 2       │  │   Shard 3       │
    │  (hot data)     │  │  (warm data)    │  │  (cold data)    │
    │  RAM: 64GB      │  │  RAM: 32GB      │  │  SSD: 1TB       │
    │  Latest docs    │  │  Last month     │  │  Archive        │
    └─────────────────┘  └─────────────────┘  └─────────────────┘

Tiered storage:
- HOT:  Last 30 days, float32, полностью в RAM
- WARM: 30-180 days, int8 quantized, частично в RAM
- COLD: 180+ days, binary quantized, SSD + mmap
```

```python
# Product Quantization (PQ) для экстремального сжатия
import faiss

# Инициализация PQ индекса
d = 1536      # размерность
m = 96        # количество подвекторов
nbits = 8     # бит на подвектор
nlist = 4096  # кластеры IVF

# IVF + PQ = компромисс между скоростью, памятью и качеством
quantizer = faiss.IndexFlatL2(d)
index = faiss.IndexIVFPQ(quantizer, d, nlist, m, nbits)

# 1536 float32 (6144 bytes) → 96 bytes = 64x сжатие!
# При этом recall@10 ≈ 80-90% от точного поиска

# Обучение
index.train(training_vectors)  # нужно 100K+ векторов для обучения
index.add(all_vectors)

# Поиск
index.nprobe = 64  # искать в 64 из 4096 кластеров
D, I = index.search(query_vectors, k=10)
```

---

## Как реализовать online learning для векторного индекса?

Задача: модель эмбеддингов обновляется, нужно переиндексировать без downtime.

```typescript
class ZeroDowntimeReindex {
  async reindex(
    oldCollection: string,
    newModel: EmbedModel
  ): Promise<void> {
    const newCollection = `${oldCollection}-v2`;

    // 1. Создаём новую коллекцию
    await this.vectorDB.createCollection(newCollection, {
      vectors: { size: newModel.dimensions, distance: "Cosine" }
    });

    // 2. Параллельно: все новые документы индексируем в ОБЕ коллекции
    this.startDualWrite(oldCollection, newCollection);

    // 3. Переиндексируем старые документы в фоне (throttled)
    await this.backgroundReindex(oldCollection, newCollection, newModel);

    // 4. После завершения — переключаем трафик
    await this.switchTraffic(oldCollection, newCollection);

    // 5. Останавливаем dual write
    this.stopDualWrite();

    // 6. Удаляем старую коллекцию
    await this.vectorDB.deleteCollection(oldCollection);
  }

  private async backgroundReindex(
    source: string,
    target: string,
    model: EmbedModel,
    batchSize = 100
  ): Promise<void> {
    let offset = 0;

    while (true) {
      const batch = await this.vectorDB.scroll(source, {
        limit: batchSize,
        offset,
        with_payload: true
      });

      if (batch.points.length === 0) break;

      const newVectors = await model.batchEmbed(
        batch.points.map(p => p.payload.text)
      );

      await this.vectorDB.batchUpsert(target, {
        points: batch.points.map((p, i) => ({
          ...p,
          vector: newVectors[i]
        }))
      });

      offset += batchSize;
      await sleep(100); // throttle — не перегружаем систему
    }
  }
}
```

---

## Гибридный поиск с поддержкой streaming updates

```typescript
class StreamingHybridSearch {
  private sparseIndex: BM25Index;
  private denseIndex: VectorDB;
  private streamProcessor: StreamProcessor;

  constructor() {
    // Подписываемся на поток изменений документов
    this.streamProcessor = new StreamProcessor("document-changes");
    this.streamProcessor.on("create", this.handleCreate.bind(this));
    this.streamProcessor.on("update", this.handleUpdate.bind(this));
    this.streamProcessor.on("delete", this.handleDelete.bind(this));
  }

  private async handleCreate(doc: Document): Promise<void> {
    // Обновляем оба индекса атомарно (с retry логикой)
    await Promise.all([
      this.sparseIndex.add(doc.id, doc.text),
      this.denseIndex.upsert({
        id: doc.id,
        vector: await embed(doc.text),
        payload: { text: doc.text }
      })
    ]);
  }

  async search(query: string, topK = 10): Promise<SearchResult[]> {
    const [denseResults, sparseResults] = await Promise.all([
      this.denseIndex.search(await embed(query), { topK: topK * 2 }),
      this.sparseIndex.search(query, { topK: topK * 2 })
    ]);

    return reciprocalRankFusion(denseResults, sparseResults)
      .slice(0, topK);
  }
}
```

---

## Мониторинг и профилирование векторного поиска

```typescript
// Ключевые метрики для production

class VectorSearchMonitor {
  // 1. Latency breakdown
  async monitorSearch(query: string): Promise<MonitoredResult> {
    const embedStart = Date.now();
    const queryVector = await embed(query);
    const embedLatency = Date.now() - embedStart;

    const searchStart = Date.now();
    const results = await vectorDB.search(queryVector);
    const searchLatency = Date.now() - searchStart;

    this.metrics.record({
      "embed_latency_ms": embedLatency,
      "search_latency_ms": searchLatency,
      "total_latency_ms": embedLatency + searchLatency,
      "results_count": results.length,
      "top_score": results[0]?.score
    });

    return { results, embedLatency, searchLatency };
  }

  // 2. Recall мониторинг (по golden queries)
  async checkRecall(): Promise<number> {
    const goldenQueries = this.loadGoldenQueries();
    let hits = 0;

    for (const { query, expectedDocId } of goldenQueries) {
      const results = await this.search(query, 10);
      if (results.some(r => r.id === expectedDocId)) hits++;
    }

    const recall = hits / goldenQueries.length;

    if (recall < 0.85) {
      this.alertOps(`Search recall dropped to ${recall}!`);
    }

    return recall;
  }

  // 3. Index health
  async checkIndexHealth(): Promise<void> {
    const info = await vectorDB.getCollectionInfo("docs");

    this.metrics.gauge("indexed_vectors", info.vectorsCount);
    this.metrics.gauge("index_ram_bytes", info.indexedVectorsCount * 1536 * 4);

    // Проверяем что индекс не сильно отстаёт (unindexed points)
    const unindexedRatio = info.pointsCount / info.indexedVectorsCount;
    if (unindexedRatio > 1.1) {
      this.alertOps("Too many unindexed vectors — search quality may degrade");
    }
  }
}

// SLO для векторного поиска
const SLOs = {
  p50_latency_ms: 20,
  p95_latency_ms: 100,
  p99_latency_ms: 300,
  recall_at_10: 0.90,
  availability: 0.999
};
```

**Материалы:**
- [Qdrant: Performance tips](https://qdrant.tech/documentation/guides/performance/)
- [Faiss: Guidelines for choosing an index](https://github.com/facebookresearch/faiss/wiki/Guidelines-to-choose-an-index)
- [ANN Benchmarks](https://ann-benchmarks.com/)
