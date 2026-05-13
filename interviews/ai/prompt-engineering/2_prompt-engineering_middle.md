# Prompt Engineering — 🔵 Middle

## Вопросы

- [Что такое Chain-of-Thought (CoT) и когда его применять?](#что-такое-chain-of-thought-cot-и-когда-его-применять)
- [Что такое self-consistency prompting?](#что-такое-self-consistency-prompting)
- [Что такое ReAct prompting?](#что-такое-react-prompting)
- [Что такое "lost in the middle" и как с этим бороться?](#что-такое-lost-in-the-middle-и-как-с-этим-бороться)
- [Что такое output parser и зачем он нужен?](#что-такое-output-parser-и-зачем-он-нужен)
- [Что такое prompt chaining и как строить цепочки промптов?](#что-такое-prompt-chaining-и-как-строить-цепочки-промптов)
- [Как вести multi-turn диалог с LLM?](#как-вести-multi-turn-диалог-с-llm)
- [Как оптимизировать промпт по стоимости и задержке?](#как-оптимизировать-промпт-по-стоимости-и-задержке)

---

## Что такое Chain-of-Thought (CoT) и когда его применять?

Chain-of-Thought (CoT) — техника, при которой модели предлагают думать пошагово перед тем, как дать ответ. Значительно улучшает точность на задачах логики, математики и рассуждений.

```typescript
// Без CoT (хуже на сложных задачах)
const prompt = `Сколько будет 17 × 24?`;

// Zero-shot CoT — просто добавь фразу
const zeroShotCoT = `
Сколько будет 17 × 24?
Давай подумаем пошагово:
`;

// Few-shot CoT — показываем пример рассуждения
const fewShotCoT = `
Вопрос: У Алисы 5 яблок, у Боба в 3 раза больше. Сколько всего?
Рассуждение: Боб имеет 5 × 3 = 15 яблок. Всего: 5 + 15 = 20.
Ответ: 20

Вопрос: ${question}
Рассуждение:
`;
```

**Когда использовать:**
- Математические задачи и логические рассуждения
- Многошаговые задачи (анализ, планирование)
- Когда нужен прозрачный ход рассуждений

**Когда НЕ нужен:**
- Простые задачи (классификация, извлечение данных) — CoT увеличивает токены без пользы

**Материалы:**
- [Prompting Guide: Chain-of-Thought](https://www.promptingguide.ai/techniques/cot)

---

## Что такое self-consistency prompting?

Self-consistency — техника улучшения CoT: запрашиваем несколько рассуждений с высокой temperature, затем берём ответ большинством голосов.

```typescript
async function selfConsistency(question: string, n = 5): Promise<string> {
  const answers = await Promise.all(
    Array.from({ length: n }).map(() =>
      openai.chat.completions.create({
        model: "gpt-4o",
        messages: [{ role: "user", content: `${question}\nДумай пошагово:` }],
        temperature: 0.7 // высокая вариативность
      })
    )
  );

  // Voting: берём самый частый ответ
  const results = answers.map(r => extractFinalAnswer(r.choices[0].message.content));
  return mostCommon(results);
}
```

Повышает точность на ~10–20% на задачах рассуждений, но увеличивает стоимость в N раз.

---

## Что такое ReAct prompting?

ReAct (Reasoning + Acting) — паттерн, при котором LLM чередует мысли (Thought), действия (Action) и наблюдения (Observation) для решения задач через инструменты.

```
Thought: Мне нужно узнать текущую температуру в Москве.
Action: search("текущая погода Москва")
Observation: 12°C, облачно

Thought: Теперь у меня есть данные, могу ответить.
Action: answer("Сейчас в Москве 12°C, облачно.")
```

```typescript
const reactPrompt = `
Ты агент-ассистент с доступом к инструментам.

Доступные инструменты:
- search(query): поиск в интернете
- calculator(expr): вычислить математическое выражение
- answer(text): дать финальный ответ

Формат:
Thought: [твоя мысль]
Action: [инструмент(аргумент)]
Observation: [результат инструмента]
... повторяй пока не готов к финальному ответу ...
Action: answer(финальный ответ)

Вопрос: ${question}
`;
```

**Материалы:**
- [ReAct: Synergizing Reasoning and Acting in LLMs](https://arxiv.org/abs/2210.03629)

---

## Что такое "lost in the middle" и как с этим бороться?

Исследования показывают, что LLM лучше запоминают информацию в начале и конце контекста, хуже — в середине. При длинном контексте важный контент, помещённый в середину, часто игнорируется.

**Стратегии:**
```typescript
// 1. Помещать самую важную информацию В НАЧАЛО или В КОНЕЦ
const prompt = `
ВАЖНЫЙ КОНТЕКСТ (начало): ${mostRelevantChunk}

${lessRelevantChunks}

НАПОМИНАНИЕ о ключевой информации: ${mostRelevantChunk}

Вопрос: ${question}
`;

// 2. Re-ranking: ставить наиболее релевантные чанки первыми и последними
function reorderChunks(chunks: Chunk[]): Chunk[] {
  const sorted = rankByRelevance(chunks);
  // Первый — самый релевантный, последний — второй по релевантности
  return [sorted[0], ...sorted.slice(2), sorted[1]];
}

// 3. Ограничивать количество чанков в контексте
const MAX_CHUNKS = 5; // не передавать 20 чанков, брать топ-5
```

**Материалы:**
- ["Lost in the Middle" paper](https://arxiv.org/abs/2307.03172)

---

## Что такое output parser и зачем он нужен?

Output parser — компонент, который берёт сырой текстовый ответ LLM и преобразует его в структурированный объект. Нужен, потому что LLM может вернуть JSON с лишним текстом, markdown-обёрткой или с ошибками.

```typescript
// Простой JSON parser с очисткой
function parseJsonResponse(raw: string): unknown {
  // Убираем markdown ```json ... ```
  const cleaned = raw
    .replace(/```json\n?/g, '')
    .replace(/```\n?/g, '')
    .trim();

  try {
    return JSON.parse(cleaned);
  } catch {
    // Попытка извлечь JSON из текста
    const match = cleaned.match(/\{[\s\S]*\}/);
    if (match) return JSON.parse(match[0]);
    throw new Error(`Failed to parse LLM response: ${raw}`);
  }
}

// С валидацией через Zod
import { z } from "zod";

const OutputSchema = z.object({
  sentiment: z.enum(["positive", "negative", "neutral"]),
  confidence: z.number().min(0).max(1),
  summary: z.string()
});

function parseAndValidate(raw: string) {
  const parsed = parseJsonResponse(raw);
  return OutputSchema.parse(parsed); // throws if invalid
}
```

---

## Что такое prompt chaining и как строить цепочки промптов?

Prompt chaining — разбивка сложной задачи на последовательность простых промптов, где вывод каждого шага становится вводом следующего.

```typescript
async function analyzeDocument(doc: string) {
  // Шаг 1: Извлечь ключевые тезисы
  const keyPoints = await llm(`
    Извлеки 3–5 ключевых тезиса из документа. Верни JSON-массив строк.
    Документ: ${doc}
  `);

  // Шаг 2: Оценить риски на основе тезисов
  const risks = await llm(`
    На основе тезисов оцени бизнес-риски. Верни JSON с полями risk и severity.
    Тезисы: ${keyPoints}
  `);

  // Шаг 3: Сформировать итоговый отчёт
  const report = await llm(`
    Напиши краткий executive summary на основе тезисов и рисков.
    Тезисы: ${keyPoints}
    Риски: ${risks}
  `);

  return report;
}
```

**Преимущества:** каждый шаг проще, ошибки локализованы, легче тестировать.
**Минус:** увеличивается latency и стоимость.

---

## Как вести multi-turn диалог с LLM?

LLM stateless — она не помнит предыдущие сообщения. Диалог поддерживается передачей всей истории при каждом запросе.

```typescript
type Message = { role: "user" | "assistant" | "system"; content: string };

class ChatSession {
  private history: Message[] = [];
  private systemPrompt: string;

  constructor(systemPrompt: string) {
    this.systemPrompt = systemPrompt;
  }

  async send(userMessage: string): Promise<string> {
    this.history.push({ role: "user", content: userMessage });

    const response = await openai.chat.completions.create({
      model: "gpt-4o",
      messages: [
        { role: "system", content: this.systemPrompt },
        ...this.history
      ]
    });

    const reply = response.choices[0].message.content!;
    this.history.push({ role: "assistant", content: reply });
    return reply;
  }

  // Обрезка истории при превышении лимита токенов
  trimHistory(maxMessages = 20): void {
    if (this.history.length > maxMessages) {
      this.history = this.history.slice(-maxMessages);
    }
  }
}
```

**Стратегии управления историей:**
- **Sliding window** — хранить последние N сообщений
- **Summary** — периодически сжимать историю в резюме
- **Selective** — хранить только релевантные сообщения

---

## Как оптимизировать промпт по стоимости и задержке?

```typescript
// 1. Использовать меньшую модель для простых задач
const model = isComplexTask ? "gpt-4o" : "gpt-4o-mini";

// 2. Ограничивать max_tokens
const response = await openai.chat.completions.create({
  model: "gpt-4o-mini",
  messages,
  max_tokens: 150, // для коротких ответов не нужно больше
});

// 3. Убирать лишние токены из промпта
// BAD: длинные объяснения, дублирующиеся инструкции
// GOOD: чёткие, лаконичные инструкции

// 4. Кэшировать ответы на повторяющиеся запросы
const cache = new Map<string, string>();

async function cachedLLM(prompt: string): Promise<string> {
  if (cache.has(prompt)) return cache.get(prompt)!;
  const result = await callLLM(prompt);
  cache.set(prompt, result);
  return result;
}

// 5. Prompt caching (Anthropic) — кэшировать длинный system prompt
// Экономит до 90% стоимости при повторных запросах с одним system prompt
```

**Материалы:**
- [Anthropic: Prompt caching](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching)
- [OpenAI: Prompt caching](https://platform.openai.com/docs/guides/prompt-caching)
