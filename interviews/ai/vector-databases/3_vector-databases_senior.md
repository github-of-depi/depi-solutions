# Vector Databases — 🟠 Senior

## Вопросы

- [Как масштабировать векторную БД до миллионов документов?](#как-масштабировать-векторную-бд-до-миллионов-документов)
- [Как строить multi-tenant индекс?](#как-строить-multi-tenant-индекс)
- [Как измерить качество embedding-модели?](#как-измерить-качество-embedding-модели)
- [Как fine-tune embedding-модель под домен?](#как-fine-tune-embedding-модель-под-домен)
- [Как снизить потребление памяти векторной БД?](#как-снизить-потребление-памяти-векторной-бд)

---

## Как масштабировать векторную БД до миллионов документов?

```typescript
// 1. Шардирование — разбивка индекса по нескольким нодам
// Qdrant: автоматическое шардирование
await client.createCollection("large-docs", {
  vectors: { size: 1536, distance: "Cosine" },
  shard_number: 6,          // 6 шардов на кластер из 3 нод = 2 шарда/нода
  replication_factor: 2     // каждый шард реплицирован на 2 ноды
});

// 2. Батчевое индексирование
async function batchIndex(documents: Document[]): Promise<void> {
  const BATCH_SIZE = 100;

  for (let i = 0; i < documents.length; i += BATCH_SIZE) {
    const batch = documents.slice(i, i + BATCH_SIZE);

    // Параллельная генерация эмбеддингов
    const embeddings = await Promise.all(
      batch.map(doc => embed(doc.text))
    );

    await client.upsert("large-docs", {
      wait: false, // асинхронно — не ждём подтверждения
      points: batch.map((doc, j) => ({
        id: doc.id,
        vector: embeddings[j],
        payload: { text: doc.text, source: doc.source }
      }))
    });

    console.log(`Indexed ${i + batch.length} / ${documents.length}`);
  }
}

// 3. Индексирование через очередь
async function queueBasedIndexing(): Promise<void> {
  const queue = new Queue("doc-indexing");

  queue.process(async (job) => {
    const { documents } = job.data;
    const embeddings = await batchEmbed(documents);
    await vectorDB.batchUpsert(embeddings);
  });
}

// 4. Payload индексы для быстрой фильтрации
await client.createPayloadIndex("large-docs", {
  field_name: "department",
  field_schema: "keyword"
});
await client.createPayloadIndex("large-docs", {
  field_name: "created_at",
  field_schema: "datetime"
});
```

---

## Как строить multi-tenant индекс?

```typescript
// Стратегия зависит от размера tenant

// Малые tenants (< 10K docs): один индекс, фильтрация по tenant_id
async function searchSmallTenant(
  query: string,
  tenantId: string
): Promise<SearchResult[]> {
  return client.search("shared-index", {
    vector: await embed(query),
    limit: 10,
    filter: {
      must: [{ key: "tenant_id", match: { value: tenantId } }]
    }
  });
}

// Средние tenants (10K–500K docs): отдельные коллекции
async function searchMediumTenant(
  query: string,
  tenantId: string
): Promise<SearchResult[]> {
  const collectionName = `tenant-${tenantId}`;
  return client.search(collectionName, {
    vector: await embed(query),
    limit: 10
  });
}

// Крупные tenants (500K+ docs): отдельные Qdrant инстансы
class TenantRouter {
  private instances: Map<string, QdrantClient> = new Map();

  async getClient(tenantId: string): Promise<QdrantClient> {
    if (!this.instances.has(tenantId)) {
      const config = await this.getTenantConfig(tenantId);
      this.instances.set(tenantId, new QdrantClient({
        url: config.dedicatedInstanceUrl
      }));
    }
    return this.instances.get(tenantId)!;
  }
}

// Изоляция данных: гарантируем что tenant не видит данные других
async function validateTenantIsolation(
  results: SearchResult[],
  tenantId: string
): Promise<SearchResult[]> {
  // Второй уровень проверки прав доступа
  return results.filter(r => r.payload.tenant_id === tenantId);
}
```

---

## Как измерить качество embedding-модели?

```python
# Метрики для retrieval качества

# 1. Recall@K — доля правильных документов в топ-K результатах
def recall_at_k(retrieved: list, relevant: list, k: int) -> float:
    retrieved_k = retrieved[:k]
    relevant_found = sum(1 for doc in retrieved_k if doc in relevant)
    return relevant_found / len(relevant) if relevant else 0

# 2. NDCG@K (Normalized Discounted Cumulative Gain)
# Учитывает позицию правильного результата (выше = лучше)
def ndcg_at_k(retrieved: list, relevant: list, k: int) -> float:
    dcg = sum(
        1 / math.log2(i + 2)
        for i, doc in enumerate(retrieved[:k])
        if doc in relevant
    )
    ideal_dcg = sum(1 / math.log2(i + 2) for i in range(min(k, len(relevant))))
    return dcg / ideal_dcg if ideal_dcg > 0 else 0

# 3. MRR (Mean Reciprocal Rank)
def mrr(all_queries_results: list[tuple[list, str]]) -> float:
    reciprocal_ranks = []
    for retrieved, relevant in all_queries_results:
        rank = next(
            (1/(i+1) for i, doc in enumerate(retrieved) if doc == relevant),
            0
        )
        reciprocal_ranks.append(rank)
    return sum(reciprocal_ranks) / len(reciprocal_ranks)

# Benchmark через MTEB
from mteb import MTEB

evaluation = MTEB(tasks=["NFCorpus", "TREC-COVID"])
results = evaluation.run(your_model)
```

---

## Как fine-tune embedding-модель под домен?

```python
from sentence_transformers import SentenceTransformer, InputExample, losses
from sentence_transformers.evaluation import InformationRetrievalEvaluator
from torch.utils.data import DataLoader

# 1. Подготовка обучающих данных
# Нужны: запрос + релевантный документ (positive)
# Hard negatives: похожие, но нерелевантные документы

train_examples = [
    InputExample(
        texts=[
            "Какова доза парацетамола для взрослых?",       # query
            "Стандартная доза парацетамола: 500–1000 мг каждые 4–6 часов..."  # positive
        ],
        label=1.0
    ),
    InputExample(
        texts=[
            "Какова доза парацетамола для взрослых?",       # query
            "Парацетамол применяется для снижения температуры..."  # hard negative
        ],
        label=0.1  # похожий, но не отвечает на вопрос
    )
]

# 2. Обучение через MultipleNegativesRankingLoss
model = SentenceTransformer("BAAI/bge-m3")
dataloader = DataLoader(train_examples, batch_size=16)
loss = losses.MultipleNegativesRankingLoss(model)

# 3. Evaluation на hold-out set
evaluator = InformationRetrievalEvaluator(
    queries=test_queries,
    corpus=test_corpus,
    relevant_docs=test_relevant
)

model.fit(
    train_objectives=[(dataloader, loss)],
    evaluator=evaluator,
    epochs=3,
    evaluation_steps=500
)
```

---

## Как снизить потребление памяти векторной БД?

```typescript
// 1. Quantization (самый эффективный метод)
const collection = await client.createCollection("docs", {
  vectors: { size: 1536, distance: "Cosine" },
  quantization_config: {
    scalar: {
      type: "int8",       // float32 → int8: 4x экономия
      quantile: 0.99,
      always_ram: true    // квантизованные в RAM, оригиналы на диске
    }
  },
  optimizers_config: {
    memmap_threshold: 20000  // векторы > 20K точек → mmap (диск)
  }
});

// 2. Матрёшкины эмбеддинги — уменьшить размерность
const embedding = await openai.embeddings.create({
  model: "text-embedding-3-small",
  input: text,
  dimensions: 512  // вместо 1536 → 3x меньше памяти
});

// 3. Хранить только индекс в RAM, оригинальные векторы — на диске
await client.updateCollectionParams("docs", {
  optimizers_config: {
    indexing_threshold: 20000,
    memmap_threshold: 0  // все векторы через mmap
  }
});

// 4. Удалять неиспользуемые payload поля
// Хранить в векторной БД только то, что нужно для фильтрации
// Остальные данные — в обычной БД по ID

// 5. TTL для устаревших документов
await client.setPayloadIndex("docs", {
  field_name: "expires_at",
  field_schema: "datetime"
});
// Регулярная чистка через scroll + delete
```
