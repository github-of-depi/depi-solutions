# RAG — 🔵 Middle

## Вопросы

- [Сравни стратегии чанкинга: fixed, recursive, semantic](#сравни-стратегии-чанкинга-fixed-recursive-semantic)
- [Что такое hybrid search и почему он лучше pure vector search?](#что-такое-hybrid-search-и-почему-он-лучше-pure-vector-search)
- [Что такое re-ranking и как он улучшает качество RAG?](#что-такое-re-ranking-и-как-он-улучшает-качество-rag)
- [Как оценивать RAG систему?](#как-оценивать-rag-систему)
- [Что такое query transformation (HyDE, decomposition)?](#что-такое-query-transformation-hyde-decomposition)
- [Что такое parent-child chunking?](#что-такое-parent-child-chunking)
- [Какие частые ошибки RAG и как их диагностировать?](#какие-частые-ошибки-rag-и-как-их-диагностировать)

---

## Сравни стратегии чанкинга: fixed, recursive, semantic

| Стратегия | Принцип | Плюсы | Минусы |
|-----------|---------|-------|--------|
| **Fixed-size** | Режет по N символов/токенов | Быстро, просто | Рвёт предложения/абзацы |
| **Recursive** | Режет по иерархии (\n\n → \n → . → ,) | Сохраняет структуру | Неравный размер чанков |
| **Semantic** | Режет по смысловым границам (сходство эмбеддингов) | Лучшее качество | Медленно, дорого |

```typescript
// Recursive character text splitter (LangChain-style)
function recursiveSplit(
  text: string,
  separators = ["\n\n", "\n", ". ", " "],
  chunkSize = 512,
  overlap = 50
): string[] {
  const sep = separators[0];
  const parts = text.split(sep);

  const chunks: string[] = [];
  let current = "";

  for (const part of parts) {
    if ((current + sep + part).length <= chunkSize) {
      current += (current ? sep : "") + part;
    } else {
      if (current) chunks.push(current);
      // Рекурсивно делим часть, если она слишком большая
      if (part.length > chunkSize && separators.length > 1) {
        chunks.push(...recursiveSplit(part, separators.slice(1), chunkSize, overlap));
      } else {
        current = part;
      }
    }
  }
  if (current) chunks.push(current);
  return chunks;
}
```

**Практика:** начинай с recursive (баланс качества и скорости), переходи к semantic если качество недостаточно.

---

## Что такое hybrid search и почему он лучше pure vector search?

Hybrid search — комбинация векторного (semantic) и ключевого (BM25) поиска.

**Проблемы pure vector search:**
- Плохо работает с точными совпадениями (артикулы, имена, коды)
- Может пропустить документ с нужным словом, если семантически не близко

**BM25 (keyword search):** хорошо находит точные совпадения, плохо работает с синонимами.

**Hybrid = лучшее из двух:**

```typescript
async function hybridSearch(
  query: string,
  topK = 10
): Promise<SearchResult[]> {
  // Параллельный поиск
  const [vectorResults, bm25Results] = await Promise.all([
    vectorDB.search(await embed(query), { topK }),
    bm25Index.search(query, { topK })
  ]);

  // Reciprocal Rank Fusion (RRF) для объединения результатов
  return reciprocalRankFusion(vectorResults, bm25Results, { k: 60 });
}

function reciprocalRankFusion(
  list1: SearchResult[],
  list2: SearchResult[],
  { k = 60 } = {}
): SearchResult[] {
  const scores = new Map<string, number>();

  list1.forEach((doc, rank) => {
    const prev = scores.get(doc.id) ?? 0;
    scores.set(doc.id, prev + 1 / (k + rank + 1));
  });

  list2.forEach((doc, rank) => {
    const prev = scores.get(doc.id) ?? 0;
    scores.set(doc.id, prev + 1 / (k + rank + 1));
  });

  return [...scores.entries()]
    .sort((a, b) => b[1] - a[1])
    .map(([id, score]) => ({ id, score }));
}
```

---

## Что такое re-ranking и как он улучшает качество RAG?

Re-ranking — второй проход поиска: сначала быстро получаем кандидатов (top-20), потом переранжируем их более точной моделью.

```
Retrieval (быстро, но грубо) → top-20 кандидатов
    ↓
Re-ranker (точнее, но медленнее) → top-5 для LLM
```

```typescript
import { Cohere } from "cohere-ai";

async function rerankResults(
  query: string,
  documents: string[],
  topN = 5
): Promise<string[]> {
  const cohere = new Cohere({ token: process.env.COHERE_API_KEY });

  const response = await cohere.rerank({
    model: "rerank-multilingual-v3.0",
    query,
    documents,
    topN
  });

  return response.results
    .sort((a, b) => b.relevanceScore - a.relevanceScore)
    .map(r => documents[r.index]);
}

// Pipeline: retrieve → rerank → generate
async function advancedRAG(question: string): Promise<string> {
  const candidates = await vectorDB.search(await embed(question), { topK: 20 });
  const reranked = await rerankResults(question, candidates.map(c => c.text));
  return llm(buildPrompt(question, reranked.slice(0, 5)));
}
```

**Улучшение:** re-ranking обычно даёт +10–20% к качеству при небольшом росте latency.

---

## Как оценивать RAG систему?

Ключевые метрики:

| Метрика | Что измеряет | Как считать |
|---------|-------------|-------------|
| **Faithfulness** | Ответ основан на контексте? | LLM-judge: проверяет, нет ли фактов не из контекста |
| **Answer Relevance** | Ответ отвечает на вопрос? | LLM-judge: насколько ответ релевантен вопросу |
| **Context Precision** | Все найденные чанки полезны? | Доля полезных чанков среди retrieved |
| **Context Recall** | Всё нужное найдено? | Доля нужных чанков среди всех нужных |

```typescript
// Оценка через RAGAS (Python)
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevancy, context_precision, context_recall

result = evaluate(
    dataset=test_dataset,
    metrics=[faithfulness, answer_relevancy, context_precision, context_recall]
)
print(result)
# {'faithfulness': 0.82, 'answer_relevancy': 0.91, ...}
```

```typescript
// LLM-as-judge для faithfulness
async function checkFaithfulness(
  answer: string,
  context: string
): Promise<number> {
  const score = await llm(`
    Оцени от 1 до 5, насколько ответ основан ТОЛЬКО на предоставленном контексте.
    5 = все факты из контекста, 1 = есть факты не из контекста.
    
    Контекст: ${context}
    Ответ: ${answer}
    
    Верни только число.
  `);
  return parseInt(score);
}
```

**Материалы:**
- [RAGAS: RAG Assessment](https://docs.ragas.io/)

---

## Что такое query transformation (HyDE, decomposition)?

**HyDE (Hypothetical Document Embeddings):**
Вместо того чтобы искать по вопросу, LLM генерирует гипотетический ответ и ищет по нему. Работает лучше для вопросов с малым семантическим сходством с ответами.

```typescript
async function hydeSearch(question: string): Promise<SearchResult[]> {
  // 1. LLM генерирует гипотетический ответ
  const hypotheticalDoc = await llm(`
    Напиши подробный параграф, который был бы правильным ответом на этот вопрос:
    "${question}"
  `);

  // 2. Ищем по гипотетическому ответу (не по вопросу!)
  const embedding = await embed(hypotheticalDoc);
  return vectorDB.search(embedding, { topK: 5 });
}
```

**Query Decomposition:**
Разбивка сложного вопроса на простые подвопросы.

```typescript
async function decomposedSearch(complexQuestion: string): Promise<string> {
  // 1. Разбить на подвопросы
  const subQuestions = await llm(`
    Разбей вопрос на 2–4 простых подвопроса для поиска в базе знаний.
    Верни JSON-массив строк.
    Вопрос: "${complexQuestion}"
  `);

  // 2. Искать по каждому подвопросу
  const results = await Promise.all(
    JSON.parse(subQuestions).map((q: string) => retrieveContext(q))
  );

  // 3. Объединить контексты и ответить
  const combinedContext = results.flat().join("\n\n");
  return llm(buildPrompt(complexQuestion, combinedContext));
}
```

---

## Что такое parent-child chunking?

Parent-child chunking — техника, при которой хранятся два уровня чанков: маленькие (child, ~128 токенов) для точного поиска, большие (parent, ~512 токенов) для передачи в LLM.

```typescript
interface ChunkStore {
  parentChunks: Map<string, string>;   // id → полный текст
  childChunks: VectorDB;               // маленькие чанки для поиска
}

async function indexWithParentChild(doc: Document): Promise<void> {
  const parentChunks = splitIntoChunks(doc.text, { size: 512 });

  for (const parent of parentChunks) {
    const parentId = uuid();
    store.parentChunks.set(parentId, parent.text);

    // Делим parent на дочерние чанки
    const childChunks = splitIntoChunks(parent.text, { size: 128 });

    await store.childChunks.upsert(
      childChunks.map(child => ({
        vector: await embed(child.text),
        metadata: { text: child.text, parentId } // ссылка на parent
      }))
    );
  }
}

async function parentChildSearch(query: string): Promise<string[]> {
  // 1. Ищем по маленьким чанкам (точный поиск)
  const childResults = await store.childChunks.search(await embed(query));

  // 2. Возвращаем родительские чанки (больше контекста для LLM)
  const parentIds = [...new Set(childResults.map(r => r.metadata.parentId))];
  return parentIds.map(id => store.parentChunks.get(id)!);
}
```

---

## Какие частые ошибки RAG и как их диагностировать?

| Проблема | Симптом | Причина | Решение |
|---------|---------|---------|---------|
| **Retrieval miss** | Модель говорит "не знаю", хотя ответ есть | Плохой чанкинг или embedding | Проверь retrieved чанки напрямую; улучши чанкинг |
| **Hallucination** | Ответ не из контекста | Слабый промпт | Усиль инструкцию "отвечай ТОЛЬКО из контекста" |
| **Noise** | Ответ смешивает нужное с ненужным | Слишком много нерелевантных чанков | Добавь re-ranking; уменьши topK |
| **Redundancy** | Дублирующиеся ответы | Дублированные чанки | Дедупликация через cosine similarity |
| **Stale data** | Устаревшие ответы | База не обновляется | Настрой регулярное переиндексирование |

```typescript
// Диагностика: логируй retrieved chunks
async function debugRAG(question: string) {
  const queryEmbed = await embed(question);
  const chunks = await vectorDB.search(queryEmbed, { topK: 5 });

  console.log("Retrieved chunks:");
  chunks.forEach((chunk, i) => {
    console.log(`[${i+1}] score=${chunk.score.toFixed(3)} | ${chunk.text.slice(0, 100)}...`);
  });

  // Если chunks нерелевантны → проблема в retrieval
  // Если chunks релевантны, но ответ плохой → проблема в generation/prompt
}
```
