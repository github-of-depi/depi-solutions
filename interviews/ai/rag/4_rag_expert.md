# RAG — 🔴 Expert

## Вопросы

- [Как разрешать противоречия между источниками в enterprise RAG?](#как-разрешать-противоречия-между-источниками-в-enterprise-rag)
- [Как строить real-time RAG для часто обновляемых данных?](#как-строить-real-time-rag-для-часто-обновляемых-данных)
- [Как fine-tune embedding модель под специфический домен?](#как-fine-tune-embedding-модель-под-специфический-домен)
- [Архитектура multi-tenant RAG на миллиарды документов?](#архитектура-multi-tenant-rag-на-миллиарды-документов)
- [Как реализовать custom re-ranker?](#как-реализовать-custom-re-ranker)

---

## Как разрешать противоречия между источниками в enterprise RAG?

```typescript
// Стратегия 1: Источники с приоритетом
interface DocumentSource {
  id: string;
  name: string;
  trustScore: number;      // 0–1: официальный API > wiki > форум
  lastUpdated: Date;
}

async function conflictAwareRAG(question: string): Promise<string> {
  const results = await hybridSearch(question);

  // Группировка по противоречивым фактам
  const conflicts = await detectConflicts(results);

  if (conflicts.length > 0) {
    return llm(`
      Вопрос: "${question}"
      
      По этому вопросу найдены противоречивые источники:
      
      ${conflicts.map(c => `
        [Источник: ${c.source}, Доверие: ${c.trustScore}]
        ${c.text}
      `).join('\n')}
      
      Представь разные точки зрения, отдав предпочтение более достоверным источникам.
      Укажи противоречие явно.
    `);
  }

  return standardGenerate(question, results);
}

// Стратегия 2: Temporal priority — более свежий источник побеждает
function selectByRecency(sources: DocumentSource[]): DocumentSource {
  return sources.sort((a, b) => b.lastUpdated.getTime() - a.lastUpdated.getTime())[0];
}

// Стратегия 3: Source hierarchy (политика, процедура, гайдлайн)
const sourceHierarchy = ["policy", "procedure", "guideline", "faq"];
```

---

## Как строить real-time RAG для часто обновляемых данных?

```typescript
// Event-driven indexing: обновляем при изменении документа
class RealTimeRAG {
  constructor(
    private vectorDB: VectorDB,
    private eventBus: EventBus
  ) {
    // Подписываемся на события изменения документов
    this.eventBus.subscribe("document.updated", this.handleUpdate.bind(this));
    this.eventBus.subscribe("document.deleted", this.handleDelete.bind(this));
  }

  private async handleUpdate(event: DocumentEvent): Promise<void> {
    // Инкрементальное обновление — только изменённые чанки
    const newChunks = splitIntoChunks(event.document.content);
    const existingChunks = await this.vectorDB.getByDocId(event.document.id);

    const { added, removed, unchanged } = diffChunks(existingChunks, newChunks);

    await Promise.all([
      this.vectorDB.batchDelete(removed.map(c => c.id)),
      this.vectorDB.batchUpsert(await embedChunks(added))
    ]);
  }

  // Cache invalidation при обновлении
  private async handleUpdate(event: DocumentEvent): Promise<void> {
    await this.semanticCache.invalidateRelated(event.document.topics);
    await this.reindex(event.document);
  }
}

// Для данных с очень частыми обновлениями (биржевые котировки, новости):
// Hybrid: статические данные в векторной БД + динамические через API tool
const tools = [
  {
    name: "get_latest_price",
    description: "Получить актуальную цену акции (real-time)",
    fn: (ticker: string) => marketAPI.getPrice(ticker)
  }
];
```

---

## Как fine-tune embedding модель под специфический домен?

```python
# Fine-tuning через contrastive learning (sentence-transformers)
from sentence_transformers import SentenceTransformer, InputExample, losses
from torch.utils.data import DataLoader

# 1. Подготовка обучающих пар
# Positive pairs: вопрос + правильный ответ
# Negative pairs: вопрос + неправильный ответ (hard negatives важны!)
train_examples = [
    InputExample(texts=["Как оформить отпуск?",
                         "Для оформления отпуска заполните форму HR-12..."], label=1.0),
    InputExample(texts=["Как оформить отпуск?",
                         "Политика зарплат компании..."], label=0.0),
    # Hard negatives — похожие, но неправильные:
    InputExample(texts=["Как оформить отпуск?",
                         "Как оформить командировку?"], label=0.2),
]

# 2. Fine-tuning
model = SentenceTransformer("BAAI/bge-m3")
train_loss = losses.CosineSimilarityLoss(model)
dataloader = DataLoader(train_examples, batch_size=16)

model.fit(
    train_objectives=[(dataloader, train_loss)],
    epochs=3,
    warmup_steps=100
)

# 3. Оценка на domain-specific benchmark
from sentence_transformers.evaluation import EmbeddingSimilarityEvaluator
evaluator = EmbeddingSimilarityEvaluator.from_input_examples(val_examples)
score = evaluator(model)
```

**Hard negatives** — ключ к качеству: примеры, которые кажутся похожими, но не являются ответом на вопрос.

---

## Архитектура multi-tenant RAG на миллиарды документов?

```
                    ┌─────────────────────────────────┐
User Request        │         API Gateway              │
(tenant_id: T1)     │  Auth / Rate Limiting / Routing  │
                    └────────────┬────────────────────┘
                                 │
                    ┌────────────▼────────────────────┐
                    │      Query Router                 │
                    │  (по tenant + типу запроса)       │
                    └──────┬───────────────┬───────────┘
                           │               │
              ┌────────────▼──┐   ┌────────▼────────┐
              │  Shared Index  │   │  Tenant Index    │
              │  (публичные    │   │  (приватные      │
              │   документы)   │   │   документы T1)  │
              └────────────────┘   └──────────────────┘

Шардирование по tenant:
- Малые tenants (< 10K docs): shared namespace с metadata filter
- Средние tenants (10K–1M docs): dedicated namespace в shared cluster
- Крупные tenants (> 1M docs): dedicated VectorDB instance
```

```typescript
class MultiTenantVectorDB {
  async search(query: string, tenantId: string): Promise<SearchResult[]> {
    const tier = await this.getTenantTier(tenantId);

    switch (tier) {
      case "small":
        // Metadata filtering в shared index
        return this.sharedIndex.search(query, {
          filter: { tenantId, effectiveTo: null }
        });

      case "medium":
        // Dedicated namespace
        return this.getNamespace(tenantId).search(query);

      case "large":
        // Dedicated instance
        return this.getTenantInstance(tenantId).search(query);
    }
  }
}
```

---

## Как реализовать custom re-ranker?

```python
# Cross-encoder re-ranker: берёт пару (query, document) и даёт score
from transformers import AutoTokenizer, AutoModelForSequenceClassification
import torch

class CrossEncoderReranker:
    def __init__(self, model_name="cross-encoder/ms-marco-MiniLM-L-6-v2"):
        self.tokenizer = AutoTokenizer.from_pretrained(model_name)
        self.model = AutoModelForSequenceClassification.from_pretrained(model_name)

    def rerank(self, query: str, documents: list[str], top_k: int = 5) -> list[str]:
        # Формируем пары (query, document)
        pairs = [[query, doc] for doc in documents]

        features = self.tokenizer(
            pairs,
            padding=True,
            truncation=True,
            max_length=512,
            return_tensors="pt"
        )

        with torch.no_grad():
            scores = self.model(**features).logits.squeeze()

        # Сортируем по score
        ranked = sorted(zip(documents, scores.tolist()), key=lambda x: x[1], reverse=True)
        return [doc for doc, _ in ranked[:top_k]]

# Domain fine-tuning re-ranker
# Нужны labeled пары: (query, relevant_doc, irrelevant_doc)
from sentence_transformers import CrossEncoder

model = CrossEncoder("cross-encoder/ms-marco-MiniLM-L-6-v2")
model.fit(
    train_dataloader=DataLoader(train_samples),
    epochs=3,
    loss_fct=torch.nn.BCEWithLogitsLoss()
)
```

**Материалы:**
- [BEIR: Heterogeneous Retrieval Benchmark](https://github.com/beir-cellar/beir)
- [Sentence Transformers: Cross Encoders](https://sbert.net/docs/cross_encoder/pretrained_models.html)
