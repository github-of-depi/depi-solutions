# RAG (Retrieval-Augmented Generation) — 🟢 Junior

## Вопросы

- [Что такое RAG и зачем он нужен?](#что-такое-rag-и-зачем-он-нужен)
- [Как устроен базовый RAG pipeline?](#как-устроен-базовый-rag-pipeline)
- [Что такое чанкинг и почему размер чанка важен?](#что-такое-чанкинг-и-почему-размер-чанка-важен)
- [Что такое embedding-модель и как она работает?](#что-такое-embedding-модель-и-как-она-работает)
- [Чем RAG отличается от fine-tuning?](#чем-rag-отличается-от-fine-tuning)

---

## Что такое RAG и зачем он нужен?

RAG (Retrieval-Augmented Generation) — подход, при котором LLM получает дополнительный контекст из внешней базы знаний перед генерацией ответа.

**Проблемы без RAG:**
- LLM знает только то, что было в тренировочных данных
- Нет доступа к свежей или приватной информации
- Высокая вероятность галлюцинаций на специфических вопросах

**RAG решает:**
- Даёт LLM доступ к актуальным корпоративным документам
- Снижает галлюцинации (ответ основан на реальных источниках)
- Дешевле и быстрее, чем fine-tuning для обновляемых данных

```
Пользователь: "Какова политика отпусков в нашей компании?"
   ↓
[Поиск в базе документов] → Находит релевантные страницы из HR-документации
   ↓
LLM получает: вопрос + найденные документы → генерирует ответ
```

**Материалы:**
- [RAG paper: Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks](https://arxiv.org/abs/2005.11401)

---

## Как устроен базовый RAG pipeline?

RAG pipeline состоит из двух фаз:

### Фаза 1: Индексирование (offline)
```typescript
async function indexDocuments(docs: Document[]) {
  for (const doc of docs) {
    // 1. Разбиваем документ на чанки
    const chunks = splitIntoChunks(doc.text, { size: 512, overlap: 50 });

    // 2. Генерируем эмбеддинги для каждого чанка
    const embeddings = await embedModel.embed(chunks.map(c => c.text));

    // 3. Сохраняем в векторную базу данных
    await vectorDB.upsert(
      chunks.map((chunk, i) => ({
        id: `${doc.id}-${i}`,
        vector: embeddings[i],
        metadata: { text: chunk.text, source: doc.source }
      }))
    );
  }
}
```

### Фаза 2: Получение ответа (online)
```typescript
async function queryRAG(question: string): Promise<string> {
  // 1. Получаем эмбеддинг вопроса
  const queryEmbedding = await embedModel.embed(question);

  // 2. Ищем топ-K релевантных чанков
  const results = await vectorDB.search(queryEmbedding, { topK: 5 });
  const context = results.map(r => r.metadata.text).join("\n\n");

  // 3. Передаём вопрос + контекст в LLM
  const answer = await llm(`
    Ответь на вопрос, используя только предоставленный контекст.
    Если ответа нет — скажи "не знаю".

    Контекст:
    ${context}

    Вопрос: ${question}
  `);

  return answer;
}
```

---

## Что такое чанкинг и почему размер чанка важен?

Чанкинг — разбивка документов на небольшие фрагменты перед векторизацией.

**Почему размер важен:**
- Слишком **маленькие чанки** (< 100 токенов): теряется контекст, поиск находит осколки
- Слишком **большие чанки** (> 1000 токенов): много нерелевантного шума, превышает лимит контекста LLM
- Оптимально: **256–512 токенов** с **перекрытием (overlap) 10–15%**

```typescript
function splitIntoChunks(
  text: string,
  options: { size: number; overlap: number }
): Chunk[] {
  const { size, overlap } = options;
  const chunks: Chunk[] = [];
  let start = 0;

  while (start < text.length) {
    const end = Math.min(start + size, text.length);
    chunks.push({
      text: text.slice(start, end),
      startIndex: start
    });
    start += size - overlap; // перекрытие для сохранения контекста
  }

  return chunks;
}
```

**Типы чанкинга:**
| Тип | Описание | Когда |
|-----|----------|-------|
| Fixed-size | Разбивка по фиксированному числу символов/токенов | Быстро, базовый вариант |
| Recursive | Разбивка по иерархии: абзац → предложение → слово | Структурированные тексты |
| Semantic | Разбивка по смысловым границам (embeddings) | Лучшее качество |

---

## Что такое embedding-модель и как она работает?

Embedding-модель преобразует текст в вектор чисел (обычно 768–3072 измерений), где семантически близкие тексты имеют близкие векторы.

```typescript
// Получение эмбеддинга через OpenAI API
const response = await openai.embeddings.create({
  model: "text-embedding-3-small", // 1536 dim, дешевле
  // или "text-embedding-3-large"  // 3072 dim, точнее
  input: "Как оформить отпуск?"
});

const vector = response.data[0].embedding; // float[]

// Косинусное сходство — мера близости двух векторов
function cosineSimilarity(a: number[], b: number[]): number {
  const dot = a.reduce((sum, ai, i) => sum + ai * b[i], 0);
  const normA = Math.sqrt(a.reduce((sum, ai) => sum + ai * ai, 0));
  const normB = Math.sqrt(b.reduce((sum, bi) => sum + bi * bi, 0));
  return dot / (normA * normB);
}
// Результат: от -1 до 1, где 1 = идентичные тексты
```

**Популярные модели:**
- `text-embedding-3-small` (OpenAI) — баланс цена/качество
- `text-embedding-3-large` (OpenAI) — лучшее качество
- `sentence-transformers/all-MiniLM-L6-v2` — open-source, быстрая
- `BAAI/bge-m3` — мультиязычная, open-source

---

## Чем RAG отличается от fine-tuning?

| | RAG | Fine-tuning |
|---|---|---|
| **Стоимость** | Низкая | Высокая (GPU) |
| **Время обновления знаний** | Мгновенно (добавить документ) | Переобучение модели |
| **Прозрачность** | Можно показать источник | Знания в весах |
| **Объём знаний** | Неограничен | Ограничен контекстом |
| **Точность формата** | Зависит от промпта | Высокая |
| **Когда использовать** | Внешние знания, FAQ, документация | Стиль, формат, поведение |

**Правило большого пальца:**
- Нужны свежие или корпоративные данные → **RAG**
- Нужно изменить стиль/тон/формат ответов → **Fine-tuning**
- Нужно и то и другое → **RAG + Fine-tuning**

**Материалы:**
- [Choosing between RAG and Fine-tuning](https://www.pinecone.io/learn/rag-vs-fine-tuning/)
