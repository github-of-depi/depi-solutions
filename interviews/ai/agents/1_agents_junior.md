# AI Agents — 🟢 Junior

## Вопросы

- [Что такое AI агент и чем он отличается от обычного LLM-вызова?](#что-такое-ai-агент-и-чем-он-отличается-от-обычного-llm-вызова)
- [Что такое tool use / function calling?](#что-такое-tool-use--function-calling)
- [Как устроен agent loop?](#как-устроен-agent-loop)
- [Что такое ReAct агент?](#что-такое-react-агент)

---

## Что такое AI агент и чем он отличается от обычного LLM-вызова?

**Обычный LLM вызов:** один запрос → один ответ. Модель не может предпринять действия во внешнем мире.

**AI агент:** LLM + инструменты + цикл рассуждений. Агент может:
- Принимать решения, что делать дальше
- Вызывать инструменты (поиск, код, API)
- Использовать результаты инструментов для следующих шагов
- Итерировать пока задача не решена

```
Одиночный LLM:    User → LLM → Response

AI Agent:         User → LLM → Action → Tool → Observation
                              ↑__________________________|
                         (цикл пока задача не решена)
```

**Ключевые компоненты агента:**
1. **LLM** — "мозг" (принимает решения)
2. **Tools** — доступные действия (поиск, калькулятор, API, код)
3. **Memory** — контекст и история
4. **Agent Loop** — цикл "думаю → действую → наблюдаю"


---

## Что такое tool use / function calling?

Function calling — механизм, при котором LLM может запросить выполнение внешней функции, передав ей структурированные параметры.

```typescript
// 1. Описываем инструменты (что модель может делать)
const tools = [
  {
    type: "function",
    function: {
      name: "get_weather",
      description: "Получить текущую погоду в городе",
      parameters: {
        type: "object",
        properties: {
          city: { type: "string", description: "Название города" },
          units: { type: "string", enum: ["celsius", "fahrenheit"] }
        },
        required: ["city"]
      }
    }
  }
];

// 2. Вызываем LLM с инструментами
const response = await openai.chat.completions.create({
  model: "gpt-4o",
  messages: [{ role: "user", content: "Какая погода в Москве?" }],
  tools
});

// 3. Если модель решила вызвать инструмент
const toolCall = response.choices[0].message.tool_calls?.[0];
if (toolCall) {
  const args = JSON.parse(toolCall.function.arguments);
  // args = { city: "Москва", units: "celsius" }

  // 4. Выполняем реальную функцию
  const weatherData = await getWeather(args.city, args.units);

  // 5. Возвращаем результат модели
  const finalResponse = await openai.chat.completions.create({
    model: "gpt-4o",
    messages: [
      { role: "user", content: "Какая погода в Москве?" },
      response.choices[0].message, // ответ с tool_call
      { role: "tool", tool_call_id: toolCall.id, content: JSON.stringify(weatherData) }
    ]
  });
}
```

---

## Как устроен agent loop?

Agent loop — цикл "думаю → действую → наблюдаю", который повторяется пока агент не решит задачу или не достигнет лимита итераций.

```typescript
async function agentLoop(
  userTask: string,
  tools: Tool[],
  maxIterations = 10
): Promise<string> {
  const messages: Message[] = [
    { role: "user", content: userTask }
  ];

  for (let i = 0; i < maxIterations; i++) {
    // 1. LLM решает что делать
    const response = await llm({ messages, tools });
    messages.push(response.message);

    // 2. Если нет tool_calls — агент закончил, возвращаем ответ
    if (!response.message.tool_calls?.length) {
      return response.message.content;
    }

    // 3. Выполняем все запрошенные инструменты
    for (const toolCall of response.message.tool_calls) {
      const result = await executeTool(toolCall);
      messages.push({
        role: "tool",
        tool_call_id: toolCall.id,
        content: JSON.stringify(result)
      });
    }
  }

  // Достигнут лимит итераций
  return "Задача не была завершена за отведённое количество шагов.";
}
```

**Условия остановки:**
- Модель не вызывает инструменты (ответила напрямую)
- Достигнут `maxIterations`
- Явный сигнал завершения (`finish` инструмент)
- Таймаут или бюджет токенов исчерпан

---

## Что такое ReAct агент?

ReAct (Reasoning + Acting) — архитектура агента, при которой модель явно чередует:
- **Thought** — внутреннее рассуждение
- **Action** — вызов инструмента
- **Observation** — результат инструмента

```typescript
const reactSystemPrompt = `
Ты — агент-ассистент. Решай задачи пошагово.

Доступные инструменты:
- search(query: string): поиск информации
- calculator(expression: string): вычисления
- finish(answer: string): вернуть финальный ответ

Формат ОБЯЗАТЕЛЬНЫЙ:
Thought: [твоё рассуждение о следующем шаге]
Action: tool_name(arguments)

После получения Observation продолжай до вызова finish().
`;

// Пример работы:
// User: "Сколько будет корень из числа дней в 2024 году?"
//
// Thought: Мне нужно знать количество дней в 2024 году.
// Action: calculator("365 + 1") // 2024 - високосный
// Observation: 366
//
// Thought: Теперь нужно вычислить квадратный корень из 366.
// Action: calculator("sqrt(366)")
// Observation: 19.13
//
// Thought: У меня есть ответ.
// Action: finish("Корень из числа дней 2024 года (366) ≈ 19.13")
```
