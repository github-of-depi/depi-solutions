# Safety & Ethics — 🔴 Expert

## Вопросы

- [NIST AI Risk Management Framework](#nist-ai-risk-management-framework)
- [AI Watermarking: методы и ограничения](#ai-watermarking-методы-и-ограничения)
- [Proxy discrimination и как его выявить](#proxy-discrimination-и-как-его-выявить)
- [Federated Learning для приватных AI-систем](#federated-learning-для-приватных-ai-систем)
- [AI Incident Response Plan](#ai-incident-response-plan)
- [Environmental impact AI-систем](#environmental-impact-ai-систем)

---

## NIST AI Risk Management Framework

NIST AI RMF (2023) — американский добровольный стандарт управления рисками AI.

**4 функции:**
```
GOVERN → MAP → MEASURE → MANAGE
   ↑__________________________|
```

```typescript
// Реализация NIST AI RMF в организации

const nistAIRMF = {
  // 1. GOVERN: политики и культура
  govern: {
    policies: [
      "AI Acceptable Use Policy",
      "AI Risk Appetite Statement",
      "AI Ethics Principles"
    ],
    roles: {
      aiRiskOwner: "CISO или Chief AI Officer",
      aiEthicsBoard: "Cross-functional committee",
      mlEngineer: "Implement controls"
    },
    training: "Все работники с AI должны пройти AI ethics training"
  },

  // 2. MAP: идентификация рисков
  map: {
    aiSystemInventory: async () => {
      return await db.aiSystems.findAll({
        fields: ["name", "purpose", "dataUsed", "riskLevel", "owner"]
      });
    },
    riskCategories: [
      "Bias and discrimination",
      "Privacy violations",
      "Security vulnerabilities",
      "Reliability failures",
      "Misuse potential"
    ]
  },

  // 3. MEASURE: количественная оценка рисков
  measure: {
    biasMetrics: ["demographic_parity", "equal_opportunity", "calibration"],
    reliabilityMetrics: ["uptime", "accuracy_over_time", "distributional_shift"],
    securityMetrics: ["jailbreak_resistance", "pii_leakage_rate", "adversarial_robustness"]
  },

  // 4. MANAGE: снижение рисков
  manage: {
    controls: {
      preventive: ["input guardrails", "output filtering", "rate limiting"],
      detective: ["monitoring", "anomaly detection", "audit logs"],
      corrective: ["incident response", "model rollback", "emergency shutdown"]
    }
  }
};
```

---

## AI Watermarking: методы и ограничения

Watermarking позволяет отследить AI-сгенерированный контент.

```python
# Два основных подхода к watermarking текста

# 1. Token-level watermarking (Kirchenbauer et al., 2023)
# Идея: при генерации делим словарь на "green list" и "red list"
# и предпочитаем green токены

class TextWatermarker:
    def __init__(self, secret_key: str, green_ratio: float = 0.5):
        self.secret_key = secret_key
        self.green_ratio = green_ratio

    def get_green_tokens(self, prev_token_id: int, vocab_size: int) -> set:
        """Детерминированно определяем green list для данного контекста"""
        import hashlib
        seed = int(hashlib.sha256(
            f"{self.secret_key}{prev_token_id}".encode()
        ).hexdigest(), 16) % (2**32)
        rng = np.random.RandomState(seed)
        n_green = int(vocab_size * self.green_ratio)
        return set(rng.choice(vocab_size, n_green, replace=False))

    def detect(self, text: str, z_threshold: float = 4.0) -> WatermarkResult:
        """Детектируем watermark через z-test"""
        tokens = tokenize(text)
        green_count = 0

        for i in range(1, len(tokens)):
            green_list = self.get_green_tokens(tokens[i-1], VOCAB_SIZE)
            if tokens[i] in green_list:
                green_count += 1

        # Z-score: много green токенов = watermark присутствует
        z_score = (green_count - len(tokens) * self.green_ratio) / \
                  np.sqrt(len(tokens) * self.green_ratio * (1 - self.green_ratio))

        return WatermarkResult(
            detected=z_score > z_threshold,
            z_score=z_score,
            confidence=norm.cdf(z_score)
        )

# 2. SynthID (Google DeepMind) — для аудио и изображений
# Встраивает невидимые паттерны в latent space при генерации

# Ограничения watermarking:
# - Paraphrasing attack: перефразирование удаляет watermark
# - Translation attack: перевод и обратный перевод
# - Не работает для очень коротких текстов
# - Не доказывает "это создал ChatGPT" — только "этот конкретный провайдер"
```

---

## Proxy discrimination и как его выявить

```python
# Proxy discrimination: модель не использует защищённый атрибут напрямую,
# но использует коррелирующий признак (прокси)

# Пример: кредитный скоринг
# Защищённый атрибут: раса
# Прокси: почтовый индекс (исторически коррелирует с расой)

class ProxyDiscriminationDetector:
    def analyze(self, model, dataset: DataFrame) -> ProxyReport:
        protected_attrs = ["race", "gender", "age", "religion"]
        features = [c for c in dataset.columns if c not in protected_attrs + ["target"]]

        proxies = []

        for feature in features:
            for protected in protected_attrs:
                if protected not in dataset.columns:
                    continue

                # 1. Корреляция между feature и protected attr
                correlation = dataset[feature].corr(dataset[protected])

                # 2. SHAP: насколько важен feature для предсказания
                shap_importance = self.get_shap_importance(model, feature)

                # 3. Если высокая корреляция И высокая важность → прокси
                if abs(correlation) > 0.3 and shap_importance > 0.1:
                    proxies.append(ProxyFeature(
                        feature=feature,
                        protected_attr=protected,
                        correlation=correlation,
                        shap_importance=shap_importance,
                        severity="high" if abs(correlation) > 0.6 else "medium"
                    ))

        # 4. Fairness through awareness: явно добавляем protected attr
        # чтобы модель могла "разделить" его влияние
        # (paradoxically helps в некоторых подходах)

        return ProxyReport(proxies=proxies, recommendations=self.recommend(proxies))

    def recommend(self, proxies: list) -> list[str]:
        recs = []
        for proxy in proxies:
            recs.append(
                f"Consider removing or transforming '{proxy.feature}' "
                f"(corr={proxy.correlation:.2f} with {proxy.protected_attr})"
            )
        return recs
```

---

## Federated Learning для приватных AI-систем

```python
# Federated Learning: обучение модели без централизации данных
# Данные остаются на устройствах / в организациях

# Architecture:
# Central Server ← агрегирует градиенты
#       ↕
# Client 1 (hospital A)  Client 2 (hospital B)  Client 3 (hospital C)
# локальные данные        локальные данные        локальные данные

class FederatedServer:
    def __init__(self, global_model):
        self.global_model = global_model

    async def federated_round(self, clients: list[Client]) -> None:
        # 1. Рассылаем текущие веса клиентам
        global_weights = self.global_model.get_weights()

        # 2. Клиенты обучают на своих данных (параллельно)
        local_updates = await asyncio.gather(*[
            client.train_locally(global_weights, epochs=1)
            for client in clients
        ])

        # 3. FedAvg: усредняем веса с учётом размера датасета
        total_samples = sum(u.n_samples for u in local_updates)
        aggregated = weighted_average(
            [u.weights for u in local_updates],
            [u.n_samples / total_samples for u in local_updates]
        )

        self.global_model.set_weights(aggregated)

class FederatedClient:
    def __init__(self, local_data, dp_epsilon: float = None):
        self.data = local_data
        self.dp_epsilon = dp_epsilon  # опционально: DP для доп. гарантий

    async def train_locally(self, global_weights, epochs: int):
        model = Model()
        model.set_weights(global_weights)

        # Обучаем на локальных данных
        if self.dp_epsilon:
            # Добавляем DP гарантии к локальному обучению
            model = apply_dp_sgd(model, self.dp_epsilon)

        model.fit(self.data, epochs=epochs)

        return LocalUpdate(
            weights=model.get_weights(),
            n_samples=len(self.data)
            # Данные НИКОГДА не покидают клиента!
        )

# Use cases: медицина (больницы), финансы (банки), мобильные устройства
# Challenges: communication overhead, non-IID data, free-rider clients
```

---

## AI Incident Response Plan

```typescript
// AI система может отказать специфическими способами:
// галлюцинации, bias spike, adversarial exploitation, data leakage

interface AIIncident {
  id: string;
  severity: "P1" | "P2" | "P3" | "P4";
  type: "safety" | "bias" | "performance" | "security" | "compliance";
  detectedAt: Date;
  description: string;
  affectedUsers?: number;
  evidenceLogs: string[];
}

const aiIncidentPlaybook = {
  // P1: Критический (например: PII leakage, jailbreak успешен в prod)
  P1: {
    sla: "15 минут до реакции",
    immediateActions: [
      "Немедленно отключить модель (feature flag / kill switch)",
      "Уведомить: CISO, Legal, AI Lead, Data Protection Officer",
      "Сохранить forensic artifacts (логи, traces)",
      "Оценить scope: сколько пользователей затронуто?"
    ],
    within24h: [
      "Root cause analysis",
      "Уведомить регулятора если required (GDPR: 72 часа)",
      "Уведомить пострадавших пользователей"
    ]
  },

  // P2: Высокий (degraded performance, bias spike >20%)
  P2: {
    sla: "1 час",
    actions: [
      "Rollback к предыдущей версии",
      "Усиленный мониторинг",
      "Анализ причин"
    ]
  }
};

// Kill switch в продакшене
class AIEmergencyStop {
  async triggerKillSwitch(reason: string, triggeredBy: string): Promise<void> {
    // 1. Немедленно отключаем AI-функционал
    await featureFlags.disable("ai_responses");

    // 2. Все новые запросы → статический fallback
    await cache.set("ai_fallback_mode", true, { ttl: 86400 });

    // 3. Уведомление
    await pagerDuty.triggerIncident({
      title: `AI Emergency Stop: ${reason}`,
      severity: "critical",
      triggeredBy
    });

    // 4. Лог для аудита
    await auditLog.record({
      action: "emergency_stop",
      reason,
      triggeredBy,
      timestamp: new Date()
    });
  }
}
```

---

## Environmental Impact AI-систем

```typescript
// Обучение GPT-3: ~1300 МВт·ч электроэнергии = ~550 тонн CO2
// Один запрос к GPT-4: ~0.001 кВт·ч = ~0.0004 кг CO2 (зависит от региона)

interface CarbonMetrics {
  requestsPerMonth: number;
  avgEnergyPerRequest: number; // кВт·ч
  gridCarbonIntensity: number;  // гCO2/кВт·ч (US avg ~400, France ~70, Iceland ~30)
}

function estimateCarbonFootprint(metrics: CarbonMetrics): CarbonEstimate {
  const monthlyEnergy = metrics.requestsPerMonth * metrics.avgEnergyPerRequest;
  const monthlyCO2kg = monthlyEnergy * metrics.gridCarbonIntensity / 1000;

  return {
    monthlyEnergyKwh: monthlyEnergy,
    monthlyCO2kg,
    equivalentCarKm: monthlyCO2kg / 0.21, // авто: ~210г CO2/км
    annualCO2tonnes: monthlyCO2kg * 12 / 1000
  };
}

// Стратегии снижения
const greenAIStrategies = {
  // 1. Выбор датацентра с зелёной энергетикой
  provider: "Azure с carbon-neutral регионом (North Europe)",

  // 2. Оптимизация: меньше токенов = меньше энергии
  optimization: [
    "Semantic caching: не генерируем повторно",
    "Маршрутизация на меньшую модель для простых задач",
    "Compression input context",
    "Batching запросов"
  ],

  // 3. Temporal shifting: запускать batch jobs ночью
  // (когда в сети больше возобновляемой энергии)
  scheduling: "Run batch evaluations during low-carbon hours",

  // 4. Carbon accounting: отслеживать и offset
  tracking: "tools: CodeCarbon (Python), electricitymaps.com API"
};
```
