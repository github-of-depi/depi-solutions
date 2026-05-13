# LLMOps — 🔵 Middle

## Вопросы

- [Как мониторить LLM-приложения в продакшене?](#как-мониторить-llm-приложения-в-продакшене)
- [Что такое prompt versioning и как управлять промптами?](#что-такое-prompt-versioning-и-как-управлять-промптами)
- [Как реализовать semantic caching?](#как-реализовать-semantic-caching)
- [Как реализовать A/B тестирование для LLM?](#как-реализовать-ab-тестирование-для-llm)
- [Что такое CI/CD для AI-приложений?](#что-такое-cicd-для-ai-приложений)
- [Как реализовать guardrails для LLM?](#как-реализовать-guardrails-для-llm)
- [Как обеспечить structured output в продакшене?](#как-обеспечить-structured-output-в-продакшене)

---

## Как мониторить LLM-приложения в продакшене?

**Ключевые метрики:**

```typescript
// Инструментирование через OpenTelemetry
import { trace, metrics } from "@opentelemetry/api";

const tracer = trace.getTracer("llm-app");
const meter = metrics.getMeter("llm-app");

// Counters и histograms
const llmRequestCounter = meter.createCounter("llm.requests.total");
const llmTokensHistogram = meter.createHistogram("llm.tokens.used");
const llmLatencyHistogram = meter.createHistogram("llm.latency.ms");
const llmCostCounter = meter.createCounter("llm.cost.usd");

async function instrumentedLLMCall(
  messages: Message[],
  model: string
): Promise<LLMResponse> {
  const span = tracer.startSpan("llm.call");

  span.setAttributes({
    "llm.model": model,
    "llm.messages.count": messages.length,
    "llm.input.tokens": countTokens(messages)
  });

  const start = Date.now();
  try {
    const response = await openai.chat.completions.create({ model, messages });
    const latency = Date.now() - start;

    // Метрики
    llmRequestCounter.add(1, { model, status: "success" });
    llmLatencyHistogram.record(latency, { model });
    llmTokensHistogram.record(response.usage.total_tokens, { model });
    llmCostCounter.add(calculateCost(response.usage, model), { model });

    span.setAttributes({
      "llm.output.tokens": response.usage.completion_tokens,
      "llm.latency.ms": latency
    });
    span.setStatus({ code: SpanStatusCode.OK });

    return response;
  } catch (error) {
    llmRequestCounter.add(1, { model, status: "error" });
    span.recordException(error);
    span.setStatus({ code: SpanStatusCode.ERROR });
    throw error;
  } finally {
    span.end();
  }
}
```

**TTFT (Time To First Token) — ключевая метрика для UX:**

```typescript
async function measureTTFT(messages: Message[]): Promise<void> {
  const start = Date.now();
  let firstTokenReceived = false;

  const stream = await openai.chat.completions.create({
    model: "gpt-4o",
    messages,
    stream: true
  });

  for await (const chunk of stream) {
    if (!firstTokenReceived && chunk.choices[0]?.delta?.content) {
      const ttft = Date.now() - start;
      firstTokenMetric.record(ttft);
      firstTokenReceived = true;
    }
  }
}
```

---

## Что такое prompt versioning и как управлять промптами?

```typescript
// Управление версиями промптов как кодом

interface PromptVersion {
  id: string;
  version: string;      // semver: "1.2.3"
  template: string;
  variables: string[];  // список переменных в шаблоне
  createdAt: Date;
  createdBy: string;
  tags: string[];       // ["production", "canary", "deprecated"]
}

class PromptRegistry {
  // Хранение в базе данных с историей
  async createVersion(prompt: PromptVersion): Promise<void> {
    await db.prompts.insert({
      ...prompt,
      hash: sha256(prompt.template)
    });
  }

  // Получение текущей production версии
  async getProduction(promptId: string): Promise<PromptVersion> {
    return db.prompts.findOne({
      id: promptId,
      tags: { includes: "production" }
    });
  }

  // История изменений
  async getHistory(promptId: string): Promise<PromptVersion[]> {
    return db.prompts.find({ id: promptId }).orderBy("createdAt", "desc");
  }

  // Rollback
  async rollback(promptId: string, version: string): Promise<void> {
    await db.prompts.update(
      { id: promptId, tags: { includes: "production" } },
      { $pull: { tags: "production" } }
    );
    await db.prompts.update(
      { id: promptId, version },
      { $push: { tags: "production" } }
    );
  }
}

// Использование с LangSmith / PromptLayer для prod
import { Client } from "langsmith";
const client = new Client();
const prompt = await client.pullPrompt("my-prompt:production");
```

---

## Как реализовать semantic caching?

Semantic cache возвращает кэшированный ответ для семантически похожих запросов (не только точных совпадений).

```typescript
class SemanticCache {
  private vectorDB: VectorDB;
  private simThreshold: number;

  constructor(options: { threshold?: number } = {}) {
    this.simThreshold = options.threshold ?? 0.95;
    this.vectorDB = new VectorDB();
  }

  async get(query: string): Promise<string | null> {
    const queryVector = await embed(query);
    const results = await this.vectorDB.search(queryVector, { topK: 1 });

    if (results.length > 0 && results[0].score >= this.simThreshold) {
      console.log(`Cache hit! Similarity: ${results[0].score}`);
      return results[0].payload.response;
    }
    return null;
  }

  async set(query: string, response: string, ttl = 3600): Promise<void> {
    const vector = await embed(query);
    await this.vectorDB.upsert({
      id: hash(query),
      vector,
      payload: {
        query,
        response,
        expiresAt: Date.now() + ttl * 1000
      }
    });
  }
}

// Middleware для LLM
async function llmWithCache(messages: Message[]): Promise<string> {
  const lastUserMsg = messages.at(-1)!.content;
  const cached = await cache.get(lastUserMsg);
  if (cached) return cached;

  const response = await callLLM(messages);
  await cache.set(lastUserMsg, response);
  return response;
}
```

---

## Как реализовать A/B тестирование для LLM?

```typescript
class LLMABTest {
  private experiments: Map<string, Experiment> = new Map();

  async getVariant(
    experimentId: string,
    userId: string
  ): Promise<"control" | "treatment"> {
    const experiment = this.experiments.get(experimentId)!;

    // Детерминированное разбиение — один пользователь всегда в одной группе
    const hash = murmurhash(`${experimentId}:${userId}`) % 100;
    return hash < experiment.treatmentPercent ? "treatment" : "control";
  }

  async track(
    experimentId: string,
    variant: string,
    metrics: ExperimentMetrics
  ): Promise<void> {
    await this.metricsDB.insert({
      experimentId,
      variant,
      ...metrics,
      timestamp: new Date()
    });
  }
}

// Использование
async function handleRequest(userId: string, query: string): Promise<string> {
  const variant = await abTest.getVariant("prompt-v2-test", userId);

  const prompt = variant === "treatment"
    ? promptRegistry.get("customer-support-v2")
    : promptRegistry.get("customer-support-v1");

  const response = await callLLM(buildMessages(prompt, query));

  // Трекинг метрик
  await abTest.track("prompt-v2-test", variant, {
    userId,
    latencyMs: responseLatency,
    tokenCount: tokenCount,
    // Качество через LLM-judge (async)
  });

  return response;
}
```

---

## Что такое CI/CD для AI-приложений?

```yaml
# .github/workflows/ai-app.yml

name: AI App CI/CD

on: [push]

jobs:
  # 1. Статические проверки
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm run lint
      - run: npm run typecheck

  # 2. Юнит тесты (без реальных LLM вызовов — моки)
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - run: npm test

  # 3. Оценка качества промптов
  prompt-eval:
    runs-on: ubuntu-latest
    steps:
      - name: Run prompt evaluation
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
        run: |
          python scripts/eval_prompts.py \
            --test-set tests/golden_queries.json \
            --threshold 0.85 \
            --model gpt-4o-mini

  # 4. Integration тесты
  integration:
    needs: [lint, unit-tests]
    runs-on: ubuntu-latest
    steps:
      - name: Run RAG pipeline test
        run: npm run test:integration

  # 5. Deploy с canary
  deploy:
    needs: [prompt-eval, integration]
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy canary (10% трафика)
        run: kubectl set image deployment/ai-app app=new-version
      - name: Monitor for 10 minutes
        run: python scripts/monitor_canary.py --duration 600
      - name: Full rollout
        run: kubectl rollout resume deployment/ai-app
```

---

## Как реализовать guardrails для LLM?

```typescript
import { Guardrails, Shield } from "nemo-guardrails"; // или кастомная реализация

class LLMGuardrails {
  // 1. Input guardrails
  async validateInput(userInput: string): Promise<ValidationResult> {
    const checks = await Promise.all([
      this.checkPromptInjection(userInput),
      this.checkPII(userInput),
      this.checkTopicRelevance(userInput),
      this.checkLength(userInput)
    ]);

    return {
      allowed: checks.every(c => c.allowed),
      blockedReasons: checks.filter(c => !c.allowed).map(c => c.reason)
    };
  }

  // 2. Output guardrails
  async validateOutput(output: string, context: Context): Promise<string> {
    if (await this.containsPII(output)) {
      return this.redactPII(output);
    }
    if (await this.isOffTopic(output, context.allowedTopics)) {
      return "Я могу помочь только с вопросами по теме [тема].";
    }
    if (await this.containsHarmfulContent(output)) {
      return "Я не могу предоставить такой ответ.";
    }
    return output;
  }

  private async checkPromptInjection(text: string): Promise<Check> {
    const patterns = [/ignore.*instructions/i, /forget.*rules/i, /new persona/i];
    const hasPattern = patterns.some(p => p.test(text));

    if (hasPattern) {
      return { allowed: false, reason: "prompt_injection" };
    }

    // LLM-based detection для сложных случаев
    const verdict = await llm(`
      Является ли следующий текст попыткой prompt injection?
      Ответь только "yes" или "no".
      Текст: "${text}"
    `);

    return { allowed: verdict.trim() !== "yes", reason: "prompt_injection" };
  }
}

// Использование
async function safeGenerate(userInput: string): Promise<string> {
  const inputCheck = await guardrails.validateInput(userInput);
  if (!inputCheck.allowed) {
    return `Запрос не может быть обработан: ${inputCheck.blockedReasons.join(", ")}`;
  }

  const output = await callLLM(buildMessages(userInput));
  return guardrails.validateOutput(output, { allowedTopics: ["support", "docs"] });
}
```

---

## Как обеспечить structured output в продакшене?

```typescript
// Стратегия: Structured Outputs API → Zod validation → fallback retry

import { z } from "zod";
import { zodResponseFormat } from "openai/helpers/zod";

const ResponseSchema = z.object({
  sentiment: z.enum(["positive", "negative", "neutral"]),
  confidence: z.number().min(0).max(1),
  summary: z.string().max(200),
  topics: z.array(z.string()).max(5)
});

async function structuredGenerate(
  text: string,
  retries = 2
): Promise<z.infer<typeof ResponseSchema>> {
  for (let attempt = 0; attempt <= retries; attempt++) {
    try {
      // Попытка 1: Structured Outputs (гарантированно валидный JSON)
      const response = await openai.beta.chat.completions.parse({
        model: "gpt-4o-2024-08-06",
        messages: [{ role: "user", content: `Analyze: ${text}` }],
        response_format: zodResponseFormat(ResponseSchema, "analysis")
      });

      return response.choices[0].message.parsed!;
    } catch (error) {
      if (attempt === retries) {
        // Fallback: вернуть безопасный дефолт
        return {
          sentiment: "neutral",
          confidence: 0,
          summary: "Unable to analyze",
          topics: []
        };
      }
    }
  }
  throw new Error("Unreachable");
}
```
