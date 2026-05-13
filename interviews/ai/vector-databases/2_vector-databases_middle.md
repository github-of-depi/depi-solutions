# Vector Databases — 🔵 Middle

## Вопросы

- [Как работают ANN алгоритмы (HNSW, IVF)?](#как-работают-ann-алгоритмы-hnsw-ivf)
- [Что такое embedding dimensionality и как влияет на качество?](#что-такое-embedding-dimensionality-и-как-влияет-на-качество)
- [Что такое metadata filtering в векторной БД?](#что-такое-metadata-filtering-в-векторной-бд)
- [Что такое embedding quantization?](#что-такое-embedding-quantization)
- [Как выбрать embedding-модель для use case?](#как-выбрать-embedding-модель-для-use-case)
- [Что такое embedding drift?](#что-такое-embedding-drift)

---

## Как работают ANN алгоритмы (HNSW, IVF)?

Точный поиск ближайшего соседа среди миллионов векторов занимает O(N). ANN (Approximate Nearest Neighbor) — компромисс между точностью и скоростью.

**HNSW (Hierarchical Navigable Small World):**
```
Уровень 3: ●─────────────────────●   (мало связей, быстрый переход)
Уровень 2: ●───●─────────●───────●   
Уровень 1: ●─●─●─●─────●─●──●───●   
Уровень 0: ●●●●●●●●●●●●●●●●●●●●●●   (все векторы, много связей)

Поиск: начинаем сверху, жадно идём вниз к ближайшему соседу
```

```typescript
// Создание HNSW индекса в Qdrant
await client.createCollection("docs", {
  vectors: {
    size: 1536,
    distance: "Cosine",
  },
  hnsw_config: {
    m: 16,           // количество связей на узел (выше = точнее, медленнее)
    ef_construct: 100 // размер очереди при построении (выше = лучше индекс)
  }
});

// При поиске
await client.search("docs", {
  vector: queryVector,
  limit: 10,
  params: { hnsw_ef: 128 } // размер очереди при поиске (recall vs speed)
});
```

**IVF (Inverted File Index):**
```
1. Кластеризация векторов (K-means): создаём N кластеров (centroids)
2. При поиске: находим ближайшие M кластеров, ищем только в них

nlist = 1024  # количество кластеров
nprobe = 64   # количество кластеров для поиска
```

| | HNSW | IVF |
|---|---|---|
| **Скорость поиска** | Очень быстро | Быстро |
| **Память** | Высокая | Ниже |
| **Поддержка удалений** | Сложно | Проще |
| **Использование** | Qdrant, Weaviate | Faiss, Pinecone |

---

## Что такое embedding dimensionality и как влияет на качество?

```typescript
// Сравнение моделей по размерности

const embeddingModels = [
  {
    model: "text-embedding-3-small",
    dimensions: 1536,
    cost: "$0.02 / 1M tokens",
    quality: "good",
    storage: "6KB per embedding"
  },
  {
    model: "text-embedding-3-large",
    dimensions: 3072,
    cost: "$0.13 / 1M tokens",
    quality: "best",
    storage: "12KB per embedding"
  },
  {
    model: "BAAI/bge-m3",
    dimensions: 1024,
    cost: "free (self-hosted)",
    quality: "very good",
    storage: "4KB per embedding"
  }
];

// Matryoshka Representation Learning (MRL):
// Можно уменьшить размерность без переобучения
// text-embedding-3-small поддерживает усечение до 256 измерений
const smallerEmbedding = await openai.embeddings.create({
  model: "text-embedding-3-small",
  input: text,
  dimensions: 256  // вместо 1536 → 6x меньше памяти, небольшая потеря качества
});
```

**Правило:** начинай с меньшей размерности, переходи к большей только если качество недостаточно.

---

## Что такое metadata filtering в векторной БД?

Metadata filtering позволяет сочетать векторный поиск с фильтрацией по структурированным полям.

```typescript
// Qdrant: поиск с фильтрацией
const results = await client.search("documents", {
  vector: queryVector,
  limit: 5,
  filter: {
    must: [
      // Только документы из HR отдела
      { key: "department", match: { value: "HR" } },
      // Только актуальные
      { key: "status", match: { value: "active" } }
    ],
    should: [
      { key: "language", match: { value: "ru" } },
      { key: "language", match: { value: "en" } }
    ],
    must_not: [
      // Исключить конфиденциальные
      { key: "classification", match: { value: "confidential" } }
    ],
    // Диапазон дат
    range: {
      key: "created_at",
      gte: "2024-01-01",
      lte: "2025-12-31"
    }
  }
});
```

**Важно:** качество metadata filtering зависит от индексов. Для часто используемых полей создавай payload индексы:

```typescript
await client.createPayloadIndex("documents", {
  field_name: "department",
  field_schema: "keyword"
});
```

---

## Что такое embedding quantization?

Quantization эмбеддингов — снижение точности чисел для экономии памяти.

```typescript
// Float32 → Int8: 4x меньше памяти, ~1% потеря recall

// 1. Scalar quantization (float32 → int8)
await client.createCollection("docs", {
  vectors: {
    size: 1536,
    distance: "Cosine",
  },
  quantization_config: {
    scalar: {
      type: "int8",
      quantile: 0.99,      // процент векторов для калибровки
      always_ram: true     // квантизованные векторы в RAM
    }
  }
});

// 2. Binary quantization (float32 → 1 бит): 32x сжатие!
// Только для специально обученных моделей (Cohere embed-v3, OpenAI embedding-3)
quantization_config: {
  binary: {
    always_ram: true
  }
}

// 3. Product quantization (PQ): сжатие с кластеризацией
// Используется в Faiss для очень больших индексов
```

**Компромисс:**
- Float32: 100% recall, максимум памяти
- Int8: ~99% recall, 4x меньше памяти
- Binary: ~95% recall, 32x меньше памяти (но нужна rescore)

---

## Как выбрать embedding-модель для use case?

```typescript
// Бенчмарк через MTEB (Massive Text Embedding Benchmark)
// https://huggingface.co/spaces/mteb/leaderboard

const criteria = {
  language: "multilingual",  // если нужна русская поддержка
  task: "retrieval",         // или: clustering, classification, sts
  maxSeqLen: 512,            // ограничение по длине документа
  maxDimensions: 1536,       // ограничение памяти
  selfHosted: true           // или API (OpenAI, Cohere)
};

// Топ-модели на 2025:
const recommendations = {
  "multilingual + self-hosted": "BAAI/bge-m3",       // 1024 dim
  "english + best quality":     "text-embedding-3-large", // 3072 dim
  "english + cost effective":   "text-embedding-3-small", // 1536 dim
  "code search":                "jinaai/jina-embeddings-v3",
  "low latency":                "BAAI/bge-small-en"   // 384 dim
};
```

**Процесс выбора:**
1. Определить задачу и языки
2. Посмотреть MTEB leaderboard
3. Протестировать топ-3 на своих данных
4. Выбрать по соотношению recall/стоимость/latency

---

## Что такое embedding drift?

Embedding drift — когда обновляется embedding-модель, новые векторы несовместимы со старыми из БД. Поиск деградирует.

```typescript
// Стратегии обработки drift

// 1. Версионирование: хранить версию модели в metadata
await vectorDB.upsert({
  vector: newEmbedding,
  payload: {
    text: doc.text,
    embedding_model: "text-embedding-3-small",
    embedding_model_version: "2024-02-15",  // версия модели
    indexed_at: new Date()
  }
});

// 2. Полное переиндексирование (если база небольшая)
async function reindexAll(newEmbedModel: EmbedModel): Promise<void> {
  const allDocs = await vectorDB.scroll({ limit: 1000 });
  // Batch reindex
  const newVectors = await batchEmbed(allDocs.map(d => d.payload.text));
  await vectorDB.batchUpsert(newVectors);
}

// 3. Миграция по частям (для больших баз)
// Новые документы → новая коллекция с новой моделью
// Поиск: hybrid из обеих коллекций
// Постепенно переносить старые документы

// 4. Мониторинг: отслеживать recall@K на test set
async function monitorSearchQuality(): Promise<void> {
  const testQueries = loadGoldenQueries();
  const recall = await computeRecall(testQueries);

  if (recall < RECALL_THRESHOLD) {
    alert("Search quality degraded! Check embedding model.");
  }
}
```
