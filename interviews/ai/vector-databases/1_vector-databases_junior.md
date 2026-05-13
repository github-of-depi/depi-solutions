# Vector Databases — 🟢 Junior

## Вопросы

- [Что такое векторная база данных и чем отличается от SQL?](#что-такое-векторная-база-данных-и-чем-отличается-от-sql)
- [Что такое cosine similarity, dot product, Euclidean distance?](#что-такое-cosine-similarity-dot-product-euclidean-distance)
- [Как сделать базовый семантический поиск?](#как-сделать-базовый-семантический-поиск)
- [Что такое sparse vs dense embeddings?](#что-такое-sparse-vs-dense-embeddings)

---

## Что такое векторная база данных и чем отличается от SQL?

Векторная БД хранит данные как числовые векторы (embeddings) и позволяет искать по семантическому сходству, а не по точному совпадению.

| | SQL Database | Vector Database |
|---|---|---|
| **Данные** | Структурированные (строки, числа) | Векторы (float[]) |
| **Поиск** | Точное совпадение (=, LIKE) | Приближённое ближайшее (ANN) |
| **Запрос** | "Найди записи где name='Иван'" | "Найди похожие на этот текст" |
| **Ключевой usecase** | Транзакции, отчёты | Семантический поиск, RAG |
| **Примеры** | PostgreSQL, MySQL | Qdrant, Pinecone, Chroma, Weaviate |

```typescript
// Пример работы с Qdrant
import { QdrantClient } from "@qdrant/js-client-rest";

const client = new QdrantClient({ url: "http://localhost:6333" });

// Создание коллекции
await client.createCollection("documents", {
  vectors: { size: 1536, distance: "Cosine" }
});

// Добавление документа
await client.upsert("documents", {
  points: [{
    id: 1,
    vector: await embed("Как оформить отпуск?"),
    payload: { text: "Для оформления отпуска...", source: "hr-doc.pdf" }
  }]
});

// Поиск похожих
const results = await client.search("documents", {
  vector: await embed("оформление отпуска в компании"),
  limit: 5
});
```

---

## Что такое cosine similarity, dot product, Euclidean distance?

Три метрики для измерения близости векторов:

```typescript
// Cosine Similarity — угол между векторами (независимо от длины)
// Значение: от -1 до 1 (1 = одинаковые, 0 = перпендикулярные)
function cosineSimilarity(a: number[], b: number[]): number {
  const dot = a.reduce((sum, ai, i) => sum + ai * b[i], 0);
  const normA = Math.sqrt(a.reduce((s, x) => s + x * x, 0));
  const normB = Math.sqrt(b.reduce((s, x) => s + x * x, 0));
  return dot / (normA * normB);
}

// Dot Product — скалярное произведение
// Быстрее cosine, но чувствителен к длине вектора
function dotProduct(a: number[], b: number[]): number {
  return a.reduce((sum, ai, i) => sum + ai * b[i], 0);
}
// При нормализованных векторах dot product = cosine similarity

// Euclidean Distance — геометрическое расстояние
// Значение: от 0 до ∞ (0 = одинаковые)
function euclideanDistance(a: number[], b: number[]): number {
  return Math.sqrt(a.reduce((sum, ai, i) => sum + (ai - b[i]) ** 2, 0));
}
```

**Когда что использовать:**
- **Cosine** — стандарт для text embeddings (не зависит от нормализации)
- **Dot product** — при нормализованных векторах, быстрее
- **Euclidean** — для визуальных/audio embeddings, пространственные задачи

---

## Как сделать базовый семантический поиск?

```typescript
// Семантический поиск без векторной БД (для прототипа)

class SimpleSemanticSearch {
  private documents: Array<{ text: string; vector: number[] }> = [];

  async addDocument(text: string): Promise<void> {
    const vector = await embed(text);
    this.documents.push({ text, vector });
  }

  async search(query: string, topK = 5): Promise<string[]> {
    const queryVector = await embed(query);

    const scored = this.documents.map(doc => ({
      text: doc.text,
      score: cosineSimilarity(queryVector, doc.vector)
    }));

    return scored
      .sort((a, b) => b.score - a.score)
      .slice(0, topK)
      .map(r => r.text);
  }
}

// Использование
const search = new SimpleSemanticSearch();
await search.addDocument("Политика отпусков компании");
await search.addDocument("Правила командировок");
await search.addDocument("Инструкция по ноутбуку");

const results = await search.search("как взять отпуск");
// → ["Политика отпусков компании", ...]
```

---

## Что такое sparse vs dense embeddings?

| | Dense Embeddings | Sparse Embeddings |
|---|---|---|
| **Вид** | Плотный вектор (все значения ≠ 0) | Разреженный вектор (большинство = 0) |
| **Размер** | 768–3072 измерений | 30,000+ измерений |
| **Создание** | Нейросеть | TF-IDF, BM25, SPLADE |
| **Поиск** | Семантический (синонимы) | Точные ключевые слова |
| **Пример** | "автомобиль" ≈ "машина" | "автомобиль" ≠ "машина" |

```typescript
// Dense embedding (semantic)
const denseVector = await embed("купить автомобиль");
// [0.23, -0.15, 0.87, 0.02, ...] (1536 чисел, все ненулевые)

// Sparse (BM25-like) — TF-IDF representation
const sparseVector = bm25.encode("купить автомобиль");
// {12453: 0.72, 8901: 0.54, ...} (только ненулевые = 2 из 30000)
```

**Гибридный поиск** = dense (для смысла) + sparse (для точных совпадений).
