# LLMOps — 🟢 Junior

## Вопросы

- [Как работать с LLM API в продакшене?](#как-работать-с-llm-api-в-продакшене)
- [Как считать стоимость и токены?](#как-считать-стоимость-и-токены)
- [Как реализовать retry с exponential backoff?](#как-реализовать-retry-с-exponential-backoff)
- [Как реализовать streaming responses?](#как-реализовать-streaming-responses)
- [Как безопасно хранить API ключи?](#как-безопасно-хранить-api-ключи)

---

## Как работать с LLM API в продакшене?

```typescript
import OpenAI from "openai";

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
  timeout: 30_000,    // таймаут 30 секунд
  maxRetries: 3,      // автоматические retry при сбоях
});

// Базовый вызов с обработкой ошибок
async function callLLM(messages: Message[]): Promise<string> {
  try {
    const response = await openai.chat.completions.create({
      model: "gpt-4o-mini",
      messages,
      max_tokens: 1000,
      temperature: 0.7,
    });

    return response.choices[0].message.content ?? "";
  } catch (error) {
    if (error instanceof OpenAI.APIError) {
      console.error(`OpenAI error ${error.status}: ${error.message}`);
      throw error;
    }
    throw error;
  }
}
```

---

## Как считать стоимость и токены?

```typescript
// Подсчёт токенов до вызова API (для оценки стоимости)
import { encoding_for_model } from "tiktoken";

function countTokens(text: string, model = "gpt-4o"): number {
  const enc = encoding_for_model(model);
  const tokens = enc.encode(text);
  enc.free();
  return tokens.length;
}

// Стоимость на основе usage из ответа
const PRICES = {
  "gpt-4o": {
    input: 0.0025 / 1000,   // $2.50 / 1M tokens
    output: 0.01 / 1000     // $10 / 1M tokens
  },
  "gpt-4o-mini": {
    input: 0.00015 / 1000,  // $0.15 / 1M tokens
    output: 0.0006 / 1000   // $0.60 / 1M tokens
  }
};

function calculateCost(usage: Usage, model: string): number {
  const price = PRICES[model];
  return (
    usage.prompt_tokens * price.input +
    usage.completion_tokens * price.output
  );
}

// Логируем стоимость каждого запроса
const response = await openai.chat.completions.create({ model, messages });
const cost = calculateCost(response.usage, model);
console.log(`Request cost: $${cost.toFixed(6)}`);
```

---

## Как реализовать retry с exponential backoff?

```typescript
interface RetryOptions {
  maxRetries?: number;
  initialDelayMs?: number;
  maxDelayMs?: number;
  retryableErrors?: number[]; // HTTP status codes
}

async function withRetry<T>(
  fn: () => Promise<T>,
  options: RetryOptions = {}
): Promise<T> {
  const {
    maxRetries = 3,
    initialDelayMs = 1000,
    maxDelayMs = 60_000,
    retryableErrors = [429, 500, 502, 503, 504]
  } = options;

  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      const isLastAttempt = attempt === maxRetries;
      const isRetryable = error instanceof OpenAI.APIError
        && retryableErrors.includes(error.status);

      if (isLastAttempt || !isRetryable) throw error;

      // Exponential backoff с jitter
      const delay = Math.min(
        initialDelayMs * Math.pow(2, attempt) + Math.random() * 1000,
        maxDelayMs
      );

      // Уважаем Retry-After заголовок от провайдера
      const retryAfter = (error as any).headers?.["retry-after"];
      const waitMs = retryAfter ? parseInt(retryAfter) * 1000 : delay;

      console.log(`Attempt ${attempt + 1} failed. Retrying in ${waitMs}ms...`);
      await new Promise(resolve => setTimeout(resolve, waitMs));
    }
  }

  throw new Error("Unreachable");
}

// Использование
const response = await withRetry(() =>
  openai.chat.completions.create({ model: "gpt-4o", messages })
);
```

---

## Как реализовать streaming responses?

```typescript
// Server-Sent Events для streaming из backend в браузер

// Backend (Express)
app.get("/api/chat", async (req, res) => {
  res.setHeader("Content-Type", "text/event-stream");
  res.setHeader("Cache-Control", "no-cache");
  res.setHeader("Connection", "keep-alive");

  const stream = await openai.chat.completions.create({
    model: "gpt-4o",
    messages: req.body.messages,
    stream: true
  });

  for await (const chunk of stream) {
    const content = chunk.choices[0]?.delta?.content ?? "";
    if (content) {
      res.write(`data: ${JSON.stringify({ content })}\n\n`);
    }
  }

  res.write("data: [DONE]\n\n");
  res.end();
});

// Frontend (React)
async function streamChat(messages: Message[], onChunk: (text: string) => void) {
  const response = await fetch("/api/chat", {
    method: "POST",
    body: JSON.stringify({ messages }),
    headers: { "Content-Type": "application/json" }
  });

  const reader = response.body!.getReader();
  const decoder = new TextDecoder();

  while (true) {
    const { value, done } = await reader.read();
    if (done) break;

    const text = decoder.decode(value);
    const lines = text.split("\n").filter(l => l.startsWith("data: "));

    for (const line of lines) {
      const data = line.replace("data: ", "");
      if (data === "[DONE]") return;

      const { content } = JSON.parse(data);
      onChunk(content);
    }
  }
}
```

---

## Как безопасно хранить API ключи?

```typescript
// НИКОГДА не хардкодить ключи в коде
// BAD:
const openai = new OpenAI({ apiKey: "sk-proj-abc123..." }); // ❌

// ХОРОШО: через переменные окружения
const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY }); // ✅

// .env файл (не коммитить в git!)
// .gitignore должен содержать: .env, .env.*

// Для продакшена: используй secrets manager
// AWS Secrets Manager / GCP Secret Manager / HashiCorp Vault
import { SecretsManagerClient, GetSecretValueCommand } from "@aws-sdk/client-secrets-manager";

async function getApiKey(secretName: string): Promise<string> {
  const client = new SecretsManagerClient({ region: "eu-west-1" });
  const response = await client.send(
    new GetSecretValueCommand({ SecretId: secretName })
  );
  return JSON.parse(response.SecretString!).OPENAI_API_KEY;
}

// Проверка что ключ не утёк в логи
function sanitizeLogs(message: string): string {
  return message.replace(/sk-[a-zA-Z0-9-_]{20,}/g, "[REDACTED]");
}
```
