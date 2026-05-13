# RAG — 🟠 Senior

## Вопросы

- [Что такое GraphRAG и когда его использовать?](#что-такое-graphrag-и-когда-его-использовать)
- [Что такое Self-RAG?](#что-такое-self-rag)
- [Что такое Agentic RAG?](#что-такое-agentic-rag)
- [Как реализовать per-user access control в RAG?](#как-реализовать-per-user-access-control-в-rag)
- [Как масштабировать RAG до миллионов документов?](#как-масштабировать-rag-до-миллионов-документов)
- [Как решать multi-hop вопросы в RAG?](#как-решать-multi-hop-вопросы-в-rag)
- [Как обрабатывать multimodal контент в RAG?](#как-обрабатывать-multimodal-контент-в-rag)
- [Как версионировать knowledge base в RAG?](#как-версионировать-knowledge-base-в-rag)

---

## Что такое GraphRAG и когда его использовать?

GraphRAG (Microsoft) — подход, при котором документы представляются не только как векторы, но и как граф знаний (nodes = сущности, edges = связи). Позволяет отвечать на вопросы, требующие понимания отношений между сущностями.

```python
# Упрощённая схема GraphRAG pipeline
# 1. Извлечение сущностей и отношений из текстов
entities, relations = extract_with_llm(documents)

# Граф: ["Илон Маск"] --[CEO]--> ["Tesla"]
#        ["Tesla"] --[производит]--> ["Model S"]

# 2. Построение граф-индекса
knowledge_graph = build_graph(entities, relations)

# 3. Запрос через граф
def graph_rag_query(question: str) -> str:
    # Стандартный поиск + обход графа для связанных сущностей
    relevant_entities = extract_entities(question)
    subgraph = knowledge_graph.neighborhood(relevant_entities, depth=2)
    context = subgraph_to_text(subgraph) + vector_search(question)
    return llm(question, context)
```

**Когда использовать:**
- Вопросы о связях между сущностями ("Кто работает с кем?")
- Аналитика больших корпусов ("Какие темы встречаются во всех документах?")
- Сложные multi-hop вопросы

**Когда НЕ нужен:**
- Простые Q&A по документам — классический RAG достаточен и дешевле

**Материалы:**
- [GraphRAG: Microsoft Research](https://microsoft.github.io/graphrag/)

---

## Что такое Self-RAG?

Self-RAG — модель, обученная сама решать: нужно ли делать retrieval для ответа, и насколько найденные документы полезны.

```
Вопрос: "Когда родился Пушкин?"
Self-RAG решение: [Retrieve] = Yes (нужен факт)
   → Retrieves documents
   → [ISREL] = Yes (релевантно)
   → [ISSUP] = Yes (документ поддерживает ответ)
   → Генерирует ответ

Вопрос: "Напиши стихотворение о весне"
Self-RAG решение: [Retrieve] = No (творческая задача, данные не нужны)
   → Генерирует напрямую
```

Специальные токены, которые генерирует модель:
- `[Retrieve]` — нужен ли поиск
- `[ISREL]` — релевантен ли чанк
- `[ISSUP]` — поддерживает ли чанк ответ
- `[ISUSE]` — полезен ли ответ

**Почему важно:** избегает ненужных retrieval операций (экономия latency) и фильтрует нерелевантные результаты.

---

## Что такое Agentic RAG?

Agentic RAG — подход, при котором агент динамически решает как именно делать retrieval: может выполнять несколько поисков, уточнять запрос, использовать разные источники.

```typescript
class AgenticRAG {
  tools = [
    {
      name: "vector_search",
      description: "Поиск по семантическому сходству в базе документов",
      fn: (query: string) => vectorDB.search(query)
    },
    {
      name: "keyword_search",
      description: "Точный поиск по ключевым словам",
      fn: (query: string) => bm25.search(query)
    },
    {
      name: "sql_query",
      description: "Запрос к структурированным данным",
      fn: (sql: string) => db.query(sql)
    }
  ];

  async answer(question: string): Promise<string> {
    // Агент сам выбирает инструменты и стратегию
    const plan = await this.planRetrieval(question);
    const context = await this.executeRetrievalPlan(plan);
    return llm(buildPrompt(question, context));
  }

  private async planRetrieval(question: string): Promise<RetrievalPlan> {
    return llm(`
      Для ответа на вопрос "${question}" нужен ли:
      1. Семантический поиск
      2. Точный поиск по ключевым словам
      3. SQL запрос к структурированным данным
      4. Комбинация нескольких
      
      Верни JSON план действий.
    `);
  }
}
```

---

## Как реализовать per-user access control в RAG?

```typescript
// Стратегия: хранить ACL в metadata каждого документа
interface DocumentMetadata {
  text: string;
  source: string;
  allowedRoles: string[];     // ["admin", "manager"]
  allowedUserIds: string[];   // ["user123", "user456"]
  tenantId: string;           // для мультиарендности
}

async function secureSearch(
  query: string,
  userContext: UserContext
): Promise<SearchResult[]> {
  const queryVector = await embed(query);

  // Фильтрация на уровне векторной БД
  // Qdrant / Pinecone поддерживают metadata filtering
  return vectorDB.search(queryVector, {
    topK: 10,
    filter: {
      must: [
        { key: "tenantId", match: { value: userContext.tenantId } }
      ],
      should: [
        { key: "allowedRoles", match: { any: userContext.roles } },
        { key: "allowedUserIds", match: { value: userContext.userId } }
      ]
    }
  });
}

// Дополнительный слой проверки после retrieval
function filterByPermissions(
  results: SearchResult[],
  user: UserContext
): SearchResult[] {
  return results.filter(doc => {
    if (doc.metadata.allowedUserIds.includes(user.userId)) return true;
    if (doc.metadata.allowedRoles.some(r => user.roles.includes(r))) return true;
    return false;
  });
}
```

---

## Как масштабировать RAG до миллионов документов?

```typescript
// 1. Индексирование через очередь (не синхронно)
// Publisher
await queue.publish("index-documents", {
  documents: newDocs,
  priority: "low"
});

// Consumer (горизонтально масштабируемый)
queue.consume("index-documents", async (batch) => {
  const embeddings = await batchEmbed(batch.documents); // batching для API
  await vectorDB.batchUpsert(embeddings);
});

// 2. Шардирование по tenantId / категории
const shards = {
  legal: new VectorDB("legal-index"),
  hr: new VectorDB("hr-index"),
  finance: new VectorDB("finance-index")
};

// 3. Кэширование частых запросов (semantic cache)
const cache = new SemanticCache({ threshold: 0.95 });

async function cachedSearch(query: string): Promise<SearchResult[]> {
  const cached = await cache.get(query);
  if (cached) return cached;

  const results = await vectorDB.search(await embed(query));
  await cache.set(query, results, { ttl: 3600 });
  return results;
}

// 4. Quantization эмбеддингов для снижения памяти
// 1536 float32 → 1536 int8 = 4x меньше памяти, ~1% потеря качества
```

---

## Как решать multi-hop вопросы в RAG?

Multi-hop — вопросы, требующие нескольких шагов поиска: "Какова зарплата CEO компании, которая выпустила iPhone?"

```typescript
async function multiHopRAG(question: string, maxHops = 3): Promise<string> {
  let context = "";
  let currentQuestion = question;

  for (let hop = 0; hop < maxHops; hop++) {
    // 1. Ищем по текущему подвопросу
    const results = await retrieveContext(currentQuestion);
    context += results.join("\n\n");

    // 2. Проверяем: достаточно ли контекста?
    const assessment = await llm(`
      Вопрос: "${question}"
      Накопленный контекст: "${context}"
      
      Достаточно ли контекста для ответа? Если нет — какой дополнительный поиск нужен?
      Верни JSON: {"sufficient": boolean, "next_query": string | null}
    `);

    const { sufficient, next_query } = JSON.parse(assessment);
    if (sufficient) break;
    currentQuestion = next_query;
  }

  return llm(buildFinalPrompt(question, context));
}
```

---

## Как обрабатывать multimodal контент в RAG?

```typescript
// Multimodal RAG: текст + изображения + таблицы
async function multimodalIndex(document: Document): Promise<void> {
  // 1. Парсинг PDF с layout awareness
  const parsed = await pdfParser.parse(document.path);

  // 2. Разные стратегии для разных типов контента
  for (const element of parsed.elements) {
    if (element.type === "text") {
      await indexTextChunk(element.content);
    } else if (element.type === "table") {
      // Таблицы → текстовое описание через LLM
      const description = await llm(`Опиши данные в этой таблице:\n${element.markdown}`);
      await indexTextChunk(description, { originalTable: element.markdown });
    } else if (element.type === "image") {
      // Изображения → описание через Vision LLM
      const description = await visionLLM(`Опиши что на изображении`, element.base64);
      await indexTextChunk(description, { hasImage: true });
    }
  }
}

// Мультимодальные эмбеддинги (CLIP-подобные)
// Позволяют искать "найди документы с графиками роста продаж"
const multimodalEmbed = await voyageai.multimodalEmbed([
  { type: "text", content: query },
]);
```

---

## Как версионировать knowledge base в RAG?

```typescript
interface VersionedDocument {
  id: string;
  content: string;
  version: number;
  effectiveFrom: Date;
  effectiveTo: Date | null; // null = текущая версия
  checksum: string;
}

class VersionedVectorStore {
  async upsertDocument(doc: Document): Promise<void> {
    const existing = await this.getLatest(doc.id);
    const newChecksum = hash(doc.content);

    // Обновляем только если контент изменился
    if (existing?.checksum === newChecksum) return;

    // Помечаем старую версию как устаревшую
    if (existing) {
      await this.vectorDB.update(existing.vectorId, {
        metadata: { effectiveTo: new Date() }
      });
    }

    // Добавляем новую версию
    const embedding = await embed(doc.content);
    await this.vectorDB.upsert({
      vector: embedding,
      metadata: {
        id: doc.id,
        version: (existing?.version ?? 0) + 1,
        effectiveFrom: new Date(),
        effectiveTo: null,
        checksum: newChecksum
      }
    });
  }

  // Поиск только по актуальным документам
  async searchCurrent(query: string): Promise<SearchResult[]> {
    return this.vectorDB.search(await embed(query), {
      filter: { key: "effectiveTo", match: { value: null } }
    });
  }
}
```
