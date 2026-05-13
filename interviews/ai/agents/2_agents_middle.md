# AI Agents — 🔵 Middle

## Вопросы

- [Что такое Plan-and-Execute агент?](#что-такое-plan-and-execute-агент)
- [Какие типы памяти у агента?](#какие-типы-памяти-у-агента)
- [Что такое multi-agent система?](#что-такое-multi-agent-система)
- [Что такое MCP (Model Context Protocol)?](#что-такое-mcp-model-context-protocol)
- [Как обрабатывать ошибки в агентных системах?](#как-обрабатывать-ошибки-в-агентных-системах)
- [Как управлять стоимостью токенов в агенте?](#как-управлять-стоимостью-токенов-в-агенте)

---

## Что такое Plan-and-Execute агент?

Plan-and-Execute — двухфазная архитектура:
1. **Планировщик** разбивает задачу на шаги
2. **Исполнитель** выполняет каждый шаг

```typescript
async function planAndExecute(task: string): Promise<string> {
  // Фаза 1: Планирование
  const plan = await llm(`
    Создай пошаговый план для выполнения задачи.
    Верни JSON-массив с шагами: [{step: number, action: string, tool: string}]
    
    Задача: "${task}"
  `);
  const steps = JSON.parse(plan);

  // Фаза 2: Выполнение
  const results: string[] = [];

  for (const step of steps) {
    const context = `
      Задача: ${task}
      Выполненные шаги: ${JSON.stringify(results)}
      Текущий шаг: ${step.action}
    `;

    const result = await executeStep(step, context);
    results.push(result);

    // Переплан при необходимости
    if (needsReplanning(result)) {
      return planAndExecute(task + "\nУже сделано: " + results.join("; "));
    }
  }

  return await synthesize(task, results);
}
```

**Плюсы перед ReAct:**
- Видимость: знаем весь план заранее
- Параллельное выполнение независимых шагов
- Легче проверить план перед выполнением (human-in-the-loop)

---

## Какие типы памяти у агента?

```typescript
interface AgentMemory {
  // 1. Short-term (in-context) — текущий разговор
  conversationHistory: Message[];

  // 2. Long-term (external store) — факты между сессиями
  longTermMemory: VectorDB;

  // 3. Episodic — прошлые задачи и их исходы
  episodicMemory: VectorDB;

  // 4. Semantic — знания о мире / домене
  knowledgeBase: VectorDB;

  // 5. Working memory — текущий контекст задачи
  workingMemory: { currentTask: string; intermediateResults: string[] };
}

class AgentWithMemory {
  async processMessage(userMsg: string): Promise<string> {
    // Загружаем релевантные воспоминания
    const relevantMemories = await this.memory.longTermMemory.search(
      await embed(userMsg),
      { topK: 3 }
    );

    // Добавляем к контексту
    const contextualizedMsg = `
      Воспоминания о пользователе: ${relevantMemories.map(m => m.text).join("; ")}
      
      Текущее сообщение: ${userMsg}
    `;

    const response = await this.llm(contextualizedMsg, this.memory.conversationHistory);

    // Сохраняем важные факты в долгосрочную память
    await this.extractAndStore(userMsg, response);

    return response;
  }

  private async extractAndStore(msg: string, response: string): Promise<void> {
    const facts = await this.llm(`
      Извлеки важные факты о пользователе из этого диалога.
      Верни JSON-массив строк или пустой массив если нечего запоминать.
      
      Пользователь: ${msg}
      Ассистент: ${response}
    `);

    const parsed = JSON.parse(facts);
    if (parsed.length > 0) {
      await this.memory.longTermMemory.upsert(
        parsed.map((fact: string) => ({
          text: fact,
          vector: embed(fact),
          metadata: { timestamp: new Date(), type: "user_fact" }
        }))
      );
    }
  }
}
```

---

## Что такое multi-agent система?

Multi-agent система — несколько специализированных агентов, работающих совместно под управлением оркестратора.

```typescript
// Паттерн: Supervisor (оркестратор) + Worker агенты
class SupervisorAgent {
  private workers: Map<string, WorkerAgent> = new Map([
    ["researcher", new ResearcherAgent()],
    ["writer", new WriterAgent()],
    ["reviewer", new ReviewerAgent()]
  ]);

  async execute(task: string): Promise<string> {
    // Оркестратор решает кому делегировать
    const routing = await this.llm(`
      Задача: "${task}"
      
      Доступные агенты:
      - researcher: поиск и сбор информации
      - writer: написание текста
      - reviewer: проверка качества
      
      Какой агент должен выполнить задачу первым? Верни имя агента.
    `);

    const worker = this.workers.get(routing.trim());
    const result = await worker.execute(task);

    // После reviewer может вернуть на доработку
    const review = await this.workers.get("reviewer")!.execute(result);
    if (review.includes("NEEDS_REVISION")) {
      return this.workers.get("writer")!.execute(task + "\nОтзыв: " + review);
    }

    return result;
  }
}

// Паттерн: Pipeline (последовательная обработка)
async function pipelineAgents(task: string): Promise<string> {
  const researched = await researcherAgent.execute(task);
  const drafted = await writerAgent.execute(task, researched);
  const reviewed = await reviewerAgent.execute(drafted);
  return reviewed;
}
```

---

## Что такое MCP (Model Context Protocol)?

MCP — открытый стандарт Anthropic для подключения LLM к внешним инструментам и данным через унифицированный интерфейс.

```typescript
// Без MCP: каждое приложение реализует интеграции отдельно
// С MCP: один протокол для всех — модель + инструменты + данные

// MCP Server (предоставляет инструменты)
import { Server } from "@modelcontextprotocol/sdk/server/index.js";

const server = new Server({
  name: "database-server",
  version: "1.0.0"
});

server.setRequestHandler("tools/call", async (request) => {
  if (request.params.name === "query_database") {
    const result = await db.query(request.params.arguments.sql);
    return { content: [{ type: "text", text: JSON.stringify(result) }] };
  }
});

// MCP Client (AI приложение)
import { Client } from "@modelcontextprotocol/sdk/client/index.js";

const client = new Client({ name: "my-ai-app", version: "1.0.0" });
await client.connect(transport);

const tools = await client.listTools(); // получаем все доступные инструменты
```

**Ключевые концепции MCP:**
- **Resources** — данные (файлы, БД, API)
- **Tools** — действия (функции)
- **Prompts** — шаблоны промптов
- **Sampling** — модель может запрашивать LLM вызовы

**Материалы:**
- [MCP Documentation](https://modelcontextprotocol.io/)

---

## Как обрабатывать ошибки в агентных системах?

```typescript
class ResilientAgent {
  async executeWithRetry(
    toolName: string,
    args: unknown,
    maxRetries = 3
  ): Promise<ToolResult> {
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        return await this.tools[toolName].execute(args);
      } catch (error) {
        if (attempt === maxRetries) {
          // Сообщаем LLM об ошибке — пусть решит что делать
          return {
            error: true,
            message: `Инструмент ${toolName} не смог выполниться после ${maxRetries} попыток: ${error.message}`,
            suggestion: "Попробуй другой подход"
          };
        }
        // Exponential backoff
        await sleep(Math.pow(2, attempt) * 1000);
      }
    }
  }

  // Fallback стратегия
  async executeWithFallback(
    primaryTool: string,
    fallbackTool: string,
    args: unknown
  ): Promise<ToolResult> {
    try {
      return await this.executeTool(primaryTool, args);
    } catch {
      console.warn(`Primary tool failed, using fallback`);
      return this.executeTool(fallbackTool, args);
    }
  }
}
```

**Типы ошибок в агентах:**
| Тип | Причина | Решение |
|-----|---------|---------|
| Tool failure | API недоступен | Retry + fallback |
| Wrong parameters | LLM передал неверные аргументы | Валидация + retry с исправленным промптом |
| Infinite loop | Агент циклится | Max iterations + детектор повторов |
| Context overflow | Слишком длинная история | Сжатие истории (summary) |

---

## Как управлять стоимостью токенов в агенте?

```typescript
class BudgetAwareAgent {
  private tokensBudget: number;
  private tokensUsed = 0;

  async execute(task: string): Promise<string> {
    while (this.tokensUsed < this.tokensBudget) {
      const response = await this.llmCall(this.messages);
      this.tokensUsed += response.usage.total_tokens;

      if (!response.message.tool_calls?.length) {
        return response.message.content;
      }

      // Если осталось мало бюджета — попросить завершить
      if (this.tokensUsed > this.tokensBudget * 0.8) {
        this.messages.push({
          role: "user",
          content: "Бюджет заканчивается. Дай лучший возможный ответ с текущими данными."
        });
      }

      await this.executeTools(response.message.tool_calls);
    }

    return "Бюджет токенов исчерпан.";
  }

  // Компрессия истории для экономии токенов
  private async compressHistory(): Promise<void> {
    if (this.messages.length > 20) {
      const summary = await this.llm(`
        Сожми следующую историю в краткое резюме (max 200 слов):
        ${JSON.stringify(this.messages.slice(0, -5))}
      `);
      this.messages = [
        { role: "system", content: `История: ${summary}` },
        ...this.messages.slice(-5) // оставляем последние 5 сообщений
      ];
    }
  }
}
```
