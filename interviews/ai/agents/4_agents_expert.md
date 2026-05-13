# AI Agents — 🔴 Expert

## Вопросы

- [Как проектировать распределённые multi-agent системы?](#как-проектировать-распределённые-multi-agent-системы)
- [Что такое Harness Engineering в AI?](#что-такое-harness-engineering-в-ai)
- [Как строить multi-modal агентов?](#как-строить-multi-modal-агентов)
- [Формальная верификация поведения агентов](#формальная-верификация-поведения-агентов)
- [Паттерны оркестрации агентов](#паттерны-оркестрации-агентов)

---

## Как проектировать распределённые multi-agent системы?

```typescript
// Архитектура: Message Bus + Agent Registry
class DistributedAgentSystem {
  private messageBus: MessageBus;
  private registry: AgentRegistry;

  async routeTask(task: Task): Promise<AgentResult> {
    // 1. Декомпозиция задачи на подзадачи
    const subtasks = await this.orchestrator.decompose(task);

    // 2. Параллельное выполнение независимых задач
    const independent = subtasks.filter(t => !t.dependencies.length);
    const parallelResults = await Promise.all(
      independent.map(subtask => this.dispatchToAgent(subtask))
    );

    // 3. Последовательное выполнение зависимых задач
    const dependent = subtasks.filter(t => t.dependencies.length > 0);
    const sequentialResults = await this.executeWithDependencies(
      dependent,
      parallelResults
    );

    return this.synthesize(task, [...parallelResults, ...sequentialResults]);
  }

  // Agent discovery через registry
  async dispatchToAgent(subtask: SubTask): Promise<AgentResult> {
    const capable = await this.registry.findAgents({
      capabilities: subtask.requiredCapabilities,
      status: "available",
      maxLatency: subtask.latencyBudget
    });

    // Load balancing: выбираем наименее загруженного
    const agent = capable.sort((a, b) => a.currentLoad - b.currentLoad)[0];

    return this.messageBus.send(agent.id, subtask);
  }
}

// Обмен сообщениями между агентами
interface AgentMessage {
  id: string;
  from: string;
  to: string;
  type: "task" | "result" | "error" | "status";
  payload: unknown;
  correlationId: string; // для трассировки задачи через агентов
  timestamp: Date;
}
```

---

## Что такое Harness Engineering в AI?

Harness Engineering — создание инфраструктурного слоя (harness) вокруг AI агента для надёжной работы в продакшене: оркестрация, мониторинг, retry, state management.

```typescript
class AgentHarness {
  // 1. State management: восстановление после сбоев
  async executeWithCheckpoints(task: Task): Promise<TaskResult> {
    const checkpoint = await this.storage.loadCheckpoint(task.id);
    const startFrom = checkpoint?.lastCompletedStep ?? 0;

    for (let step = startFrom; step < task.steps.length; step++) {
      const result = await this.executeStep(task.steps[step]);

      // Сохраняем checkpoint после каждого шага
      await this.storage.saveCheckpoint(task.id, {
        lastCompletedStep: step,
        stepResults: [...(checkpoint?.stepResults ?? []), result]
      });
    }
  }

  // 2. Distributed tracing
  async executeWithTracing(task: Task): Promise<TaskResult> {
    const span = this.tracer.startSpan("agent.task", {
      attributes: {
        "task.id": task.id,
        "task.type": task.type,
        "agent.model": this.modelConfig.name
      }
    });

    try {
      const result = await this.agent.execute(task);
      span.setStatus({ code: SpanStatusCode.OK });
      return result;
    } catch (error) {
      span.recordException(error);
      span.setStatus({ code: SpanStatusCode.ERROR });
      throw error;
    } finally {
      span.end();
    }
  }

  // 3. Rate limiting per agent
  private rateLimiter = new RateLimiter({
    requestsPerMinute: 60,
    tokensPerMinute: 100_000,
    strategy: "sliding_window"
  });

  // 4. Circuit breaker: защита от каскадных сбоев
  private circuitBreaker = new CircuitBreaker({
    failureThreshold: 5,
    recoveryTimeout: 30_000,
    onOpen: () => this.alertOps("Agent circuit breaker opened")
  });
}
```

---

## Как строить multi-modal агентов?

```typescript
class MultiModalAgent {
  // Унифицированный обработчик разных типов входных данных
  async processInput(input: MultiModalInput): Promise<AgentAction> {
    const normalized = await this.normalizeInput(input);
    return this.llm(normalized, this.tools);
  }

  private async normalizeInput(input: MultiModalInput): Promise<Message[]> {
    const content: ContentPart[] = [];

    for (const part of input.parts) {
      switch (part.type) {
        case "text":
          content.push({ type: "text", text: part.content });
          break;

        case "image":
          content.push({
            type: "image_url",
            image_url: { url: `data:image/jpeg;base64,${part.base64}` }
          });
          break;

        case "audio":
          // Транскрибируем аудио через Whisper
          const transcript = await this.whisper.transcribe(part.audioData);
          content.push({ type: "text", text: `[Аудио]: ${transcript}` });
          break;

        case "document":
          // Извлекаем текст + описываем изображения/таблицы
          const extracted = await this.documentParser.extract(part.fileData);
          content.push({ type: "text", text: extracted });
          break;
      }
    }

    return [{ role: "user", content }];
  }

  // Multi-modal tools
  tools = [
    {
      name: "analyze_image",
      description: "Детальный анализ изображения",
      fn: async (imageBase64: string) => {
        return this.visionLLM.analyze(imageBase64, "Подробно опиши изображение");
      }
    },
    {
      name: "generate_image",
      description: "Создать изображение по описанию",
      fn: (prompt: string) => this.imageGen.generate(prompt)
    }
  ];
}
```

---

## Формальная верификация поведения агентов

```typescript
// Property-based testing для агентов
import { fc } from "fast-check";

describe("Agent Safety Properties", () => {
  // Свойство: агент никогда не выполняет irreversible actions без подтверждения
  it("never executes irreversible actions without approval", async () => {
    await fc.assert(
      fc.asyncProperty(
        fc.record({
          task: fc.string(),
          context: fc.record({ userApproved: fc.boolean() })
        }),
        async ({ task, context }) => {
          const actions = await agent.planActions(task);
          const irreversible = actions.filter(a => a.isIrreversible);

          for (const action of irreversible) {
            if (!context.userApproved) {
              await expect(agent.execute(action)).rejects.toThrow("Requires approval");
            }
          }
        }
      )
    );
  });

  // Свойство: агент всегда останавливается (нет infinite loop)
  it("always terminates within maxIterations", async () => {
    await fc.assert(
      fc.asyncProperty(
        fc.string({ minLength: 1 }),
        async (task) => {
          const startTime = Date.now();
          await agent.execute(task);
          const duration = Date.now() - startTime;

          // Должен завершиться в пределах бюджета
          expect(duration).toBeLessThan(MAX_AGENT_TIMEOUT);
        }
      )
    );
  });
});

// Контрактное тестирование инструментов
class ToolContractValidator {
  validate(toolDefinition: ToolDefinition, testCases: ToolTestCase[]): void {
    for (const tc of testCases) {
      // Проверяем что инструмент соответствует своей документации
      const result = toolDefinition.fn(tc.input);
      expect(result).toMatchSchema(toolDefinition.outputSchema);
    }
  }
}
```

---

## Паттерны оркестрации агентов

```typescript
// Паттерн 1: Supervisor-Worker
// Один координатор + специализированные исполнители
const supervisorPattern = {
  supervisor: orchestratorAgent,
  workers: { researcher, analyst, writer, reviewer }
};

// Паттерн 2: Peer-to-Peer (Gossip)
// Агенты общаются напрямую, нет центрального координатора
class PeerAgent {
  async collaborate(task: SubTask): Promise<void> {
    const peers = await this.registry.findPeers(task.requiredSkills);
    const consensus = await this.proposeAndVote(peers, task);
    await this.execute(consensus);
  }
}

// Паттерн 3: Blackboard
// Общее пространство знаний, агенты читают/пишут туда
class BlackboardSystem {
  private board = new SharedKnowledgeStore();

  async runAgents(problem: Problem): Promise<Solution> {
    while (!this.board.hasSolution(problem)) {
      // Каждый агент смотрит на доску и вносит вклад если может
      for (const agent of this.agents) {
        if (agent.canContribute(this.board.current())) {
          await agent.contribute(this.board);
        }
      }
    }
    return this.board.getSolution(problem);
  }
}

// Паттерн 4: Map-Reduce для параллельной обработки
async function mapReduceAgents<T>(
  items: T[],
  mapAgent: Agent,
  reduceAgent: Agent
): Promise<string> {
  // Map: параллельная обработка каждого элемента
  const mappedResults = await Promise.all(
    items.map(item => mapAgent.process(item))
  );

  // Reduce: объединение результатов
  return reduceAgent.synthesize(mappedResults);
}
```

**Материалы:**
- [LangGraph: Multi-Agent Workflows](https://langchain-ai.github.io/langgraph/)
- [AutoGen: Multi-Agent Conversation Framework](https://microsoft.github.io/autogen/)
