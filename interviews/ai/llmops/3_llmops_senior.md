# LLMOps — 🟠 Senior

## Вопросы

- [Как реализовать fallback стратегии при недоступности модели?](#как-реализовать-fallback-стратегии-при-недоступности-модели)
- [Как реализовать semantic routing?](#как-реализовать-semantic-routing)
- [Как управлять PII в LLM-приложениях?](#как-управлять-pii-в-llm-приложениях)
- [Как реализовать context compression и prefix caching?](#как-реализовать-context-compression-и-prefix-caching)
- [Как реализовать rate limiting для AI API?](#как-реализовать-rate-limiting-для-ai-api)
- [Как масштабировать AI-сервис под высокую нагрузку?](#как-масштабировать-ai-сервис-под-высокую-нагрузку)

---

## Как реализовать fallback стратегии при недоступности модели?

```typescript
// Multi-provider fallback с автоматическим переключением
class LLMRouter {
  private providers: LLMProvider[] = [
    { name: "openai", client: openaiClient, priority: 1 },
    { name: "anthropic", client: anthropicClient, priority: 2 },
    { name: "groq", client: groqClient, priority: 3 }  // дешевле, быстрее
  ];

  private circuitBreakers: Map<string, CircuitBreaker> = new Map(
    this.providers.map(p => [p.name, new CircuitBreaker({
      failureThreshold: 5,
      timeout: 60_000,
      halfOpenRequests: 1
    })])
  );

  async generate(request: LLMRequest): Promise<string> {
    for (const provider of this.providers) {
      const breaker = this.circuitBreakers.get(provider.name)!;

      if (breaker.state === "OPEN") {
        console.log(`Skipping ${provider.name}: circuit breaker OPEN`);
        continue;
      }

      try {
        const result = await breaker.execute(() =>
          provider.client.generate(request)
        );
        return result;
      } catch (error) {
        console.error(`${provider.name} failed: ${error.message}`);
        // Переходим к следующему провайдеру
      }
    }

    // Все провайдеры недоступны — graceful degradation
    return this.generateFallbackResponse(request);
  }

  private generateFallbackResponse(request: LLMRequest): string {
    // Кэшированный ответ или статический fallback
    const cached = this.cache.getSimilar(request.prompt);
    if (cached) return cached;

    return "Сервис временно недоступен. Попробуйте позже или обратитесь в поддержку.";
  }
}
```

---

## Как реализовать semantic routing?

Semantic routing — маршрутизация запросов на разные модели в зависимости от типа задачи (сложности, домена, стоимости).

```typescript
// Классификация запроса → выбор оптимальной модели

enum RequestComplexity {
  SIMPLE = "simple",    // gpt-4o-mini
  MEDIUM = "medium",    // gpt-4o
  COMPLEX = "complex"   // o1 / o3
}

class SemanticRouter {
  // Быстрая классификация через эмбеддинги
  private routeEmbeddings: Map<string, number[]> = new Map();

  async initialize(): Promise<void> {
    // Предвычисляем эмбеддинги для примеров каждого типа
    const examples = {
      simple: ["переведи текст", "исправь опечатки", "summarize this"],
      coding: ["write a function", "debug this code", "implement algorithm"],
      reasoning: ["solve this math problem", "analyze this business case", "plan a strategy"]
    };

    for (const [route, texts] of Object.entries(examples)) {
      const vectors = await Promise.all(texts.map(embed));
      const centroid = computeCentroid(vectors);
      this.routeEmbeddings.set(route, centroid);
    }
  }

  async route(query: string): Promise<ModelConfig> {
    const queryVector = await embed(query);

    // Находим ближайший маршрут
    let bestRoute = "simple";
    let bestScore = -Infinity;

    for (const [route, routeVector] of this.routeEmbeddings) {
      const score = cosineSimilarity(queryVector, routeVector);
      if (score > bestScore) { bestScore = score; bestRoute = route; }
    }

    // Маппинг маршрут → модель
    return {
      simple:    { model: "gpt-4o-mini", temperature: 0.3 },
      coding:    { model: "gpt-4o", temperature: 0.1 },
      reasoning: { model: "o3-mini", temperature: 1 }
    }[bestRoute];
  }
}

// Экономия: 80% запросов → дешёвая модель, 20% → дорогая
```

---

## Как управлять PII в LLM-приложениях?

```typescript
import presidio from "presidio-analyzer"; // или кастомные regex

class PIIManager {
  // 1. Обнаружение PII перед отправкой в LLM
  async detectPII(text: string): Promise<PIIResult[]> {
    const results = await presidio.analyze({
      text,
      entities: ["PERSON", "EMAIL_ADDRESS", "PHONE_NUMBER",
                 "CREDIT_CARD", "IP_ADDRESS", "LOCATION"]
    });
    return results;
  }

  // 2. Анонимизация: замена реальных данных на заглушки
  async anonymize(text: string): Promise<{ anonymized: string; mapping: PIIMapping }> {
    const piiItems = await this.detectPII(text);
    let anonymized = text;
    const mapping: PIIMapping = {};

    for (const item of piiItems) {
      const placeholder = `<${item.entity_type}_${hash(item.text).slice(0, 6)}>`;
      mapping[placeholder] = item.text;
      anonymized = anonymized.replace(item.text, placeholder);
    }

    return { anonymized, mapping };
  }

  // 3. Деанонимизация ответа (восстановление реальных данных)
  deanonymize(response: string, mapping: PIIMapping): string {
    return Object.entries(mapping).reduce(
      (text, [placeholder, original]) => text.replace(placeholder, original),
      response
    );
  }
}

// Middleware: анонимизируем перед LLM, деанонимизируем после
async function piiSafeGenerate(userInput: string): Promise<string> {
  const { anonymized, mapping } = await piiManager.anonymize(userInput);
  const llmResponse = await callLLM(anonymized);
  return piiManager.deanonymize(llmResponse, mapping);
}

// 4. Логирование: не логировать PII!
function sanitizeForLogs(data: Record<string, unknown>): Record<string, unknown> {
  const PII_FIELDS = ["email", "phone", "name", "address"];
  return Object.fromEntries(
    Object.entries(data).map(([k, v]) =>
      PII_FIELDS.includes(k.toLowerCase()) ? [k, "[REDACTED]"] : [k, v]
    )
  );
}
```

---

## Как реализовать context compression и prefix caching?

```typescript
// Context compression: сжимаем историю когда она близка к лимиту
class ContextManager {
  private maxTokens: number;

  async compressIfNeeded(messages: Message[]): Promise<Message[]> {
    const currentTokens = countTokens(messages);

    if (currentTokens > this.maxTokens * 0.8) {
      return this.compress(messages);
    }
    return messages;
  }

  private async compress(messages: Message[]): Promise<Message[]> {
    const [systemMsg, ...history] = messages;
    const recent = history.slice(-4); // сохраняем 4 последних
    const toSummarize = history.slice(0, -4);

    const summary = await llm(`
      Сожми историю диалога в краткое резюме (max 100 слов).
      Сохрани ключевые факты и контекст.
      История: ${JSON.stringify(toSummarize)}
    `);

    return [
      systemMsg,
      { role: "system", content: `Предыдущий контекст: ${summary}` },
      ...recent
    ];
  }
}

// Prefix caching (Anthropic / OpenAI): кэшировать длинный system prompt
// Экономит до 90% input token стоимости при повторных запросах

// Anthropic: явная разметка кэша
const response = await anthropic.messages.create({
  model: "claude-opus-4-5",
  system: [
    {
      type: "text",
      text: longSystemPrompt,  // ~10K токенов
      cache_control: { type: "ephemeral" }  // кэшируем этот блок
    }
  ],
  messages: [{ role: "user", content: userMessage }]
});
// Повторные запросы: cache_read_input_tokens вместо input_tokens (90% дешевле)
```

---

## Как реализовать rate limiting для AI API?

```typescript
import { RateLimiter } from "limiter";

class AIRateLimiter {
  // Per-user лимиты
  private userLimiters: Map<string, TokenBucketLimiter> = new Map();

  // Глобальный лимит провайдера
  private globalLimiter = new RateLimiter({
    tokensPerInterval: 90_000,  // 90K TPM (tokens per minute)
    interval: "minute"
  });

  async acquire(userId: string, estimatedTokens: number): Promise<void> {
    // 1. Проверяем глобальный лимит
    const globalOk = await this.globalLimiter.removeTokens(estimatedTokens);
    if (!globalOk) throw new RateLimitError("Global rate limit exceeded");

    // 2. Проверяем лимит пользователя
    const userLimiter = this.getUserLimiter(userId);
    const userOk = await userLimiter.removeTokens(estimatedTokens);
    if (!userOk) throw new RateLimitError("User rate limit exceeded");
  }

  private getUserLimiter(userId: string): TokenBucketLimiter {
    if (!this.userLimiters.has(userId)) {
      this.userLimiters.set(userId, new RateLimiter({
        tokensPerInterval: 10_000,  // 10K TPM per user
        interval: "minute"
      }));
    }
    return this.userLimiters.get(userId)!;
  }
}

// HTTP middleware
app.use("/api/chat", async (req, res, next) => {
  try {
    const estimatedTokens = countTokens(req.body.messages) + 500;
    await rateLimiter.acquire(req.user.id, estimatedTokens);
    next();
  } catch (error) {
    if (error instanceof RateLimitError) {
      res.status(429).json({
        error: "Rate limit exceeded",
        retryAfter: 60,
        message: "Слишком много запросов. Попробуйте через минуту."
      });
    }
  }
});
```

---

## Как масштабировать AI-сервис под высокую нагрузку?

```typescript
// Стратегии горизонтального масштабирования

// 1. Request queuing с приоритетами
class AIRequestQueue {
  private queues = {
    high: new PriorityQueue(),    // платные пользователи
    medium: new PriorityQueue(),  // бесплатные
    low: new PriorityQueue()      // batch обработка
  };

  async process(): Promise<void> {
    // Обрабатываем в порядке приоритета
    while (true) {
      const request =
        this.queues.high.dequeue() ??
        this.queues.medium.dequeue() ??
        this.queues.low.dequeue();

      if (request) {
        await this.processRequest(request);
      } else {
        await sleep(100);
      }
    }
  }
}

// 2. Load balancing между несколькими API ключами
class APIKeyPool {
  private keys: APIKeyState[];

  async getKey(): Promise<string> {
    // Round-robin с учётом текущего использования
    const available = this.keys
      .filter(k => k.currentRPM < k.maxRPM)
      .sort((a, b) => a.currentRPM - b.currentRPM);

    if (available.length === 0) throw new Error("All API keys rate limited");
    return available[0].key;
  }
}

// 3. Async обработка через очередь для не-срочных задач
// Синхронный: пользователь ждёт ответа
// Асинхронный: пользователь получает job_id, потом поллит статус
app.post("/api/summarize", async (req, res) => {
  const jobId = await queue.add("summarize", {
    text: req.body.text,
    userId: req.user.id
  });
  res.json({ jobId, status: "pending" });
});
```
