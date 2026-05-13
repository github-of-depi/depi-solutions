# Evaluation — 🟢 Junior

## Вопросы

- [Что такое оценка LLM-приложений и зачем она нужна?](#что-такое-оценка-llm-приложений-и-зачем-она-нужна)
- [Что такое BLEU, ROUGE и BERTScore?](#что-такое-bleu-rouge-и-bertscore)
- [Что такое LLM-as-a-Judge?](#что-такое-llm-as-a-judge)
- [Как обнаружить галлюцинации?](#как-обнаружить-галлюцинации)
- [Что такое human evaluation?](#что-такое-human-evaluation)

---

## Что такое оценка LLM-приложений и зачем она нужна?

**Оценка (evaluation)** — систематическое измерение качества LLM-системы. Без неё невозможно:
- Понять, работает ли новая версия промпта лучше старой
- Поймать регрессии при смене модели
- Доказать бизнесу, что система решает задачу

**Два уровня оценки:**

```
Offline evaluation               Online evaluation
────────────────────────         ────────────────────────
Тестовый датасет                 Реальные пользователи
До деплоя                        После деплоя
Дёшево / быстро                  Дорого / медленно
BLEU, ROUGE, LLM-judge           CTR, thumbs up/down, retention
```

**Минимальный pipeline:**
```typescript
interface EvalCase {
  input: string;
  expectedOutput?: string;  // не всегда есть
  context?: string;         // для RAG
}

interface EvalResult {
  score: number;          // 0..1
  reasoning: string;
  passed: boolean;
}

async function runEval(
  cases: EvalCase[],
  evaluate: (c: EvalCase) => Promise<EvalResult>
): Promise<EvalSummary> {
  const results = await Promise.all(cases.map(evaluate));
  const passed = results.filter(r => r.passed).length;
  return {
    totalCases: cases.length,
    passed,
    passRate: passed / cases.length,
    avgScore: results.reduce((sum, r) => sum + r.score, 0) / results.length
  };
}
```

---

## Что такое BLEU, ROUGE и BERTScore?

Метрики для сравнения сгенерированного текста с эталоном (reference).

**BLEU (Bilingual Evaluation Understudy):**
```
Измеряет n-gram precision: сколько n-грамм из гипотезы встречается в эталоне.

Гипотеза:  "The cat sat on the mat"
Эталон:    "The cat is on the mat"

1-gram precision: 5/6 = 0.83 (все слова кроме "sat" совпадают)
```

**ROUGE:**
```
Измеряет recall: сколько n-грамм из эталона встречается в гипотезе.
ROUGE-L — по длинной общей подпоследовательности (LCS).

Применяется для: суммаризация, translation
```

**BERTScore:**
```
Использует эмбеддинги — учитывает семантическое сходство.
Лучше BLEU/ROUGE для задач где важен смысл, а не точные слова.
```

```python
from bert_score import score

hyps = ["The cat sat on the mat"]
refs = ["A cat is resting on a mat"]

P, R, F1 = score(hyps, refs, lang="en")
print(f"BERTScore F1: {F1.mean().item():.3f}")  # ~0.91 (семантически близко)
```

**Ограничения:**
- BLEU/ROUGE не улавливают смысл (перефразирование = плохой score)
- Требуют reference (эталонный ответ), который не всегда есть
- Плохо работают для открытых вопросов (open-ended generation)

---

## Что такое LLM-as-a-Judge?

Использование LLM (например, GPT-4o) для оценки качества ответа другой LLM.

```typescript
// LLM-judge для оценки ответа по критериям
async function judgeResponse(
  question: string,
  answer: string,
  criteria: string[]
): Promise<JudgeResult> {
  const prompt = `
    Ты оцениваешь качество ответа AI-ассистента.
    
    Вопрос: ${question}
    
    Ответ: ${answer}
    
    Оцени ответ по следующим критериям (1-5):
    ${criteria.map((c, i) => `${i + 1}. ${c}`).join("\n")}
    
    Верни JSON: { "scores": [{"criterion": "...", "score": 1-5, "reasoning": "..."}] }
  `;

  const result = await llm(prompt, { responseFormat: "json" });
  const parsed = JSON.parse(result);

  return {
    scores: parsed.scores,
    avgScore: parsed.scores.reduce((s, c) => s + c.score, 0) / parsed.scores.length
  };
}

// Использование
const result = await judgeResponse(
  "Что такое RAG?",
  "RAG — это метод генерации...",
  ["Точность", "Полнота", "Понятность", "Структура"]
);
// Преимущество: не нужен reference, оценивает произвольные тексты
// Риск: предвзятость судьи (самооценка, позиционная предвзятость)
```

**Снижение предвзятости:**
```typescript
// 1. Случайный порядок вариантов при попарном сравнении
// 2. Несколько судей → усреднение
// 3. Чёткие рубрики в промпте
// 4. Калибровка: сравниваем LLM-judge с human evaluation
```

---

## Как обнаружить галлюцинации?

```typescript
// Галлюцинация = ответ уверен, но фактически неверен

// 1. NLI-based: модель проверяет entailment (context → answer)
async function checkGroundedness(
  context: string,
  answer: string
): Promise<GroundednessResult> {
  const verdict = await llm(`
    Дан контекст и ответ.
    Поддерживается ли каждое утверждение в ответе контекстом?
    
    Контекст: ${context}
    Ответ: ${answer}
    
    Для каждого предложения ответа: SUPPORTED / UNSUPPORTED / NOT_IN_CONTEXT
    Верни JSON: {"sentences": [{"text": "...", "verdict": "...", "evidence": "..."}]}
  `);

  const result = JSON.parse(verdict);
  const hallucinated = result.sentences.filter(s => s.verdict !== "SUPPORTED");
  
  return {
    isGrounded: hallucinated.length === 0,
    hallucinatedSentences: hallucinated,
    groundednessScore: 1 - hallucinated.length / result.sentences.length
  };
}

// 2. SelfCheckGPT: генерируем ответ N раз, сравниваем согласованность
async function selfCheckHallucination(
  question: string,
  n = 5
): Promise<number> {
  const responses = await Promise.all(
    Array.from({ length: n }, () => callLLM(question))
  );

  // Если ответы сильно расходятся → высокая вероятность галлюцинации
  const consistency = await measureConsistency(responses);
  return 1 - consistency; // халлюцинация score: 0 = достоверно, 1 = галлюцинация
}
```

---

## Что такое human evaluation?

Оценка реальными людьми — золотой стандарт, но дорогой и медленный.

**Форматы:**
```
1. Прямая оценка (Likert scale)
   "Оцените ответ от 1 до 5 по точности/полноте/стилю"

2. Попарное сравнение (A/B)
   "Какой ответ лучше: A или B?"
   → Bradley-Terry model для итогового ранжирования
   → Используется для RLHF training data

3. Thumbs Up/Down (продакшен)
   Простой, но зашумлённый сигнал
```

```typescript
// Сбор human feedback в продакшене
interface UserFeedback {
  messageId: string;
  userId: string;
  rating: "thumbs_up" | "thumbs_down";
  comment?: string;
  timestamp: Date;
}

// Анализ: какие типы запросов получают плохие оценки?
async function analyzeNegativeFeedback(): Promise<FeedbackInsights> {
  const negative = await db.feedback.find({ rating: "thumbs_down" });
  
  // Кластеризация по типам проблем
  const clusters = await clusterByEmbeddings(negative.map(f => f.question));
  
  return {
    topIssues: clusters.slice(0, 5),
    failureRate: negative.length / total,
    // → используем для создания golden test set
  };
}
```
