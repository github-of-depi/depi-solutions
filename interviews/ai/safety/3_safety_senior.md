# Safety & Ethics — 🟠 Senior

## Вопросы

- [EU AI Act: что должен знать инженер?](#eu-ai-act-что-должен-знать-инженер)
- [Что такое Responsible AI Framework?](#что-такое-responsible-ai-framework)
- [Как предотвратить misuse AI-системы?](#как-предотвратить-misuse-ai-системы)
- [Differential Privacy в AI-системах](#differential-privacy-в-ai-системах)
- [Model Cards и Datasheets](#model-cards-и-datasheets)
- [Audit Trails для AI-систем](#audit-trails-для-ai-систем)

---

## EU AI Act: что должен знать инженер?

EU AI Act (2024) — первый в мире комплексный закон об AI.

**Классификация рисков:**
```
Неприемлемый риск (запрещено):
  - Социальный скоринг граждан государством
  - Real-time биометрическое наблюдение в публичных местах
  - Манипуляция сознанием через subliminal методы

Высокий риск (строгие требования):
  - Биометрические системы
  - Критическая инфраструктура (энергетика, транспорт)
  - Образование (оценки, допуск к учёбе)
  - Найм и увольнение
  - Кредитный скоринг
  - Медицинские устройства

Ограниченный риск (transparency obligations):
  - Чат-боты → обязательно сообщать что это AI
  - Deepfakes → обязательная маркировка

Минимальный риск (без ограничений):
  - Фильтры спама, AI в видеоиграх
```

**Практические требования для High-Risk:**
```typescript
interface AIActCompliance {
  // 1. Техническая документация
  technicalDocumentation: {
    modelDescription: string;
    trainingData: DataDescription;
    performance: BenchmarkResults;
    limitations: string[];
  };

  // 2. Управление рисками
  riskManagement: {
    identifiedRisks: Risk[];
    mitigationMeasures: Measure[];
    residualRisks: Risk[];
  };

  // 3. Human oversight: человек должен иметь возможность вмешаться
  humanOversight: {
    overrideCapability: boolean;    // можно ли отключить AI
    reviewProcess: string;          // как решения проверяются людьми
    escalationPath: string;         // куда эскалировать проблемы
  };

  // 4. Логирование для аудита (минимум 6 месяцев)
  auditLogs: {
    retentionPeriodDays: 180;
    loggedEvents: string[];         // входы, выходы, решения
  };
}
```

---

## Что такое Responsible AI Framework?

```typescript
// Microsoft RAI Framework (адаптация)
const responsibleAIFramework = {
  // 6 принципов
  principles: {
    fairness: "Система не дискриминирует по защищённым характеристикам",
    reliability: "Система работает предсказуемо при разных условиях",
    privacy: "Персональные данные защищены",
    inclusiveness: "Система доступна для всех групп пользователей",
    transparency: "Пользователи знают что взаимодействуют с AI",
    accountability: "Люди несут ответственность за AI-решения"
  },

  // Практические шаги
  implementation: {
    design: [
      "Define use case с ограничениями",
      "Провести fairness assessment на этапе проектирования",
      "Включить diverse stakeholders в дизайн"
    ],
    development: [
      "Тестирование на bias",
      "Red teaming",
      "Evaluation с разными демографическими группами"
    ],
    deployment: [
      "Transparent AI disclosure (пользователи знают об AI)",
      "Human oversight mechanism",
      "Incident response plan для AI failures"
    ],
    monitoring: [
      "Bias drift monitoring",
      "Continuous evaluation",
      "Feedback loop от пользователей"
    ]
  }
};

// Impact Assessment перед деплоем высокорискового AI
async function conductAIImpactAssessment(): Promise<ImpactReport> {
  return {
    affectedGroups: identifyAffectedStakeholders(),
    potentialHarms: identifyPotentialHarms(),
    mitigations: proposeMitigations(),
    residualRisks: identifyResidualRisks(),
    reviewedBy: ["legal", "ethics-board", "domain-experts"],
    approvedForDeployment: false  // нужна подпись
  };
}
```

---

## Как предотвратить misuse AI-системы?

```typescript
// Misuse = использование системы способами, которые не предусмотрены

// 1. Принцип минимальных привилегий для AI
// Агент должен иметь только необходимые инструменты
const minimalAgent = {
  tools: [
    "read_customer_orders",    // только чтение
    "send_confirmation_email", // конкретное действие
    // НЕТ: delete_all_records, admin_access, payment_processing
  ]
};

// 2. Rate limiting + anomaly detection
class MisuseDetector {
  async detect(userId: string, request: Request): Promise<MisuseAlert | null> {
    const pattern = await this.analyzePattern(userId, request);

    // Паттерны злоупотребления
    if (pattern.requestRate > 100) { // >100 запросов/минуту
      return { type: "rate_abuse", userId, severity: "medium" };
    }

    if (pattern.uniqueTopics < 2 && pattern.requestCount > 50) {
      // Много однотипных запросов — возможно автоматизация
      return { type: "automated_scraping", userId, severity: "low" };
    }

    if (pattern.jailbreakAttempts > 3) {
      return { type: "jailbreak_attempt", userId, severity: "high" };
    }

    return null;
  }
}

// 3. Terms of Service + использование API
// Явно запрещаем в ToS:
// - Автоматизированное создание misleading content
// - Surveillance
// - Academic fraud
// - Weapons development

// 4. Output watermarking
// Invisible watermarks в generated text → можно отследить источник
// SynthID (Google), AEGIS и другие
async function watermarkOutput(text: string, userId: string): Promise<string> {
  // Добавляем невидимый паттерн в статистику токенов
  return await watermarker.embed(text, { userId, timestamp: Date.now() });
}
```

---

## Differential Privacy в AI-системах

```python
# Differential Privacy (DP): гарантирует, что включение/исключение
# одного человека из данных не изменит вывод значимо

# Применения в LLM:
# 1. Private fine-tuning: обучаем на приватных данных с DP гарантиями
# 2. Private inference: запросы не раскрывают данные о других пользователях

from opacus import PrivacyEngine
import torch

# DP-SGD: добавляем шум к градиентам при обучении
model = MyModel()
optimizer = torch.optim.Adam(model.parameters())

privacy_engine = PrivacyEngine()
model, optimizer, data_loader = privacy_engine.make_private(
    module=model,
    optimizer=optimizer,
    data_loader=data_loader,
    noise_multiplier=1.0,  # больше = больше приватности = хуже качество
    max_grad_norm=1.0      # clipping градиентов
)

# После обучения: epsilon (ε) = privacy budget
epsilon = privacy_engine.get_epsilon(delta=1e-5)
print(f"Privacy budget: ε={epsilon:.2f}")
# ε < 1 = очень сильная гарантия; ε < 10 = приемлемо для ML

# Trade-off: privacy ↑ → accuracy ↓ → нужен баланс под конкретный use case
```

---

## Model Cards и Datasheets

```markdown
# Model Card: Customer Support Assistant v2.1

## Intended Use
- Primary: Automated tier-1 customer support
- Out-of-scope: Medical advice, financial decisions, legal guidance

## Training Data
- Source: Internal support tickets (2020-2024), 500K samples
- Languages: English, Russian
- Known biases: Underrepresented: non-native English speakers

## Performance
| Metric              | Value  |
|---------------------|--------|
| Answer Relevance    | 87%    |
| Faithfulness        | 91%    |
| Avg TTFT            | 1.2s   |
| Jailbreak resistance| 98.3%  |

## Limitations
- May hallucinate product specifications not in knowledge base
- Performance degrades on highly technical queries
- Not recommended for: financial refund decisions >$1000 (require human review)

## Ethical Considerations
- Bias testing conducted across: gender, region, language
- Human oversight required for: escalations, refunds, account changes

## Version History
- v2.1: Improved faithfulness (+4%), reduced latency (-200ms)
- v2.0: Added multilingual support
```

---

## Audit Trails для AI-систем

```typescript
// Audit trail = неизменяемая запись всех AI-решений

interface AIAuditEvent {
  eventId: string;          // UUID
  timestamp: Date;
  systemVersion: string;
  promptVersion: string;

  // Who
  userId?: string;          // анонимизированный
  sessionId: string;

  // What
  action: "generation" | "tool_call" | "decision" | "refusal";
  inputHash: string;        // SHA256 входа (не сам вход — может содержать PII)
  outputHash: string;       // SHA256 выхода

  // Why (для high-risk decisions)
  reasoning?: string;       // CoT trace
  confidenceScore?: number;

  // Risk
  riskLevel: "low" | "medium" | "high";
  guardrailsTriggered: string[];
}

class AIAuditLogger {
  // Write-once хранилище (append-only)
  async log(event: AIAuditEvent): Promise<void> {
    await db.auditLog.insert({
      ...event,
      // Криптографическая цепочка для обнаружения tampering
      prevHash: await this.getLastHash(),
      hash: sha256(JSON.stringify(event))
    });
  }

  // Экспорт для регуляторной проверки
  async exportForAudit(
    from: Date,
    to: Date,
    userId?: string
  ): Promise<AuditExport> {
    const events = await db.auditLog.find({
      timestamp: { $gte: from, $lte: to },
      ...(userId ? { userId } : {})
    });

    return {
      events,
      integrity: await this.verifyChain(events),
      exportedAt: new Date(),
      exportedBy: "audit_system"
    };
  }
}
```
