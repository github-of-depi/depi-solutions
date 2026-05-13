# LLMOps — 🔴 Expert

## Вопросы

- [Как реализовать LLM Gateway для организации?](#как-реализовать-llm-gateway-для-организации)
- [LLM Routing на уровне инфраструктуры](#llm-routing-на-уровне-инфраструктуры)
- [Speculative Decoding и Continuous Batching](#speculative-decoding-и-continuous-batching)
- [Multi-region deployment для AI систем](#multi-region-deployment-для-ai-систем)
- [Capacity planning для AI workloads](#capacity-planning-для-ai-workloads)
- [Cloud vs On-Device deployment](#cloud-vs-on-device-deployment)

---

## Как реализовать LLM Gateway для организации?

LLM Gateway — централизованный прокси между приложениями и LLM-провайдерами. Даёт единую точку для управления auth, cost, rate limiting, logging.

```typescript
// Kong / custom gateway для LLM

class LLMGateway {
  async handleRequest(req: GatewayRequest): Promise<GatewayResponse> {
    const context = await this.buildContext(req);

    // 1. Authentication & Authorization
    const identity = await this.auth.verify(req.apiKey);
    await this.authz.checkPermission(identity, "llm:generate");

    // 2. Budget enforcement
    await this.budget.checkAndReserve(identity.teamId, estimatedCost(req));

    // 3. Route к оптимальной модели
    const model = await this.router.selectModel(req, identity.tier);

    // 4. Rate limiting (per team, per model)
    await this.rateLimiter.acquire(identity.teamId, model, req.estimatedTokens);

    // 5. PII scrubbing
    const sanitizedReq = await this.piiScrubber.sanitize(req);

    // 6. Cache check
    const cached = await this.semanticCache.get(sanitizedReq);
    if (cached) {
      await this.metrics.track({ ...context, cacheHit: true });
      return cached;
    }

    // 7. Forward to provider
    const response = await this.providerPool.generate(sanitizedReq, model);

    // 8. Output guardrails
    const safeResponse = await this.guardrails.validate(response);

    // 9. Logging & billing
    await Promise.all([
      this.logger.log({ request: sanitizedReq, response: safeResponse, ...context }),
      this.billing.record(identity.teamId, response.usage)
    ]);

    // 10. Cache store
    await this.semanticCache.set(sanitizedReq, safeResponse);

    return safeResponse;
  }
}

// Конфигурация через YAML (как Kong плагины)
const gatewayConfig = {
  routes: [
    {
      path: "/v1/chat",
      plugins: [
        { name: "auth", config: { provider: "jwt" } },
        { name: "rate-limit", config: { rpm: 1000, tpm: 500000 } },
        { name: "semantic-cache", config: { threshold: 0.95, ttl: 3600 } },
        { name: "llm-router", config: { strategy: "cost-optimized" } }
      ]
    }
  ]
};
```

---

## LLM Routing на уровне инфраструктуры

```typescript
// Умная маршрутизация по нескольким критериям
class IntelligentRouter {
  async route(request: LLMRequest): Promise<ModelEndpoint> {
    const factors = await this.analyzeRequest(request);

    // Матрица принятия решений
    if (factors.requiresReasoning && !factors.latencySensitive) {
      return this.endpoints.o3; // медленно, но точно
    }

    if (factors.isCodeTask) {
      return this.endpoints["claude-opus-4-5-code"]; // специализация
    }

    if (factors.isSimple && factors.costSensitive) {
      return this.endpoints["gpt-4o-mini"]; // дёшево
    }

    if (factors.latencySensitive) {
      // Выбираем по текущей latency провайдеров (real-time health check)
      return await this.selectByLatency(["groq-llama-70b", "together-qwen"]);
    }

    return this.endpoints["gpt-4o"]; // default
  }

  // Canary routing: 5% трафика на новую модель
  async canaryRoute(request: LLMRequest): Promise<ModelEndpoint> {
    const isCanary = Math.random() < 0.05;
    if (isCanary) {
      this.metrics.tag(request.id, "canary");
      return this.endpoints["gpt-5-preview"];
    }
    return this.endpoints["gpt-4o"];
  }

  // Shadow routing: основной запрос + теневой для сравнения
  async shadowRoute(request: LLMRequest): Promise<Response> {
    const [primary, shadow] = await Promise.allSettled([
      this.endpoints.primary.generate(request),
      this.endpoints.shadow.generate(request)  // результат игнорируем
    ]);

    // Логируем оба ответа для сравнения
    if (shadow.status === "fulfilled") {
      await this.compareResponses(primary.value, shadow.value, request);
    }

    return primary.value;
  }
}
```

---

## Speculative Decoding и Continuous Batching

**Speculative Decoding:**
```python
# Идея: маленькая (draft) модель генерирует несколько токенов,
# большая (target) модель проверяет их за один проход

def speculative_decode(prompt: str, draft_model, target_model, k=4):
    """
    k = количество токенов, которые draft генерирует за раз
    Ускорение: ~2-3x при хорошем acceptance rate
    """
    draft_tokens = draft_model.generate(prompt, n=k)
    # Target модель за ОДИН forward pass проверяет все k токенов
    accepted = target_model.verify(prompt, draft_tokens)
    # Отклонённые токены → перегенерировать с target
    return accepted
```

**Continuous Batching (vLLM):**
```python
# Проблема: статичный batching неэффективен
# (короткие запросы ждут пока длинные завершатся)

# Continuous batching: итерация за итерацией
# На каждом шаге генерации добавляем новые запросы в batch,
# удаляем завершённые

from vllm import LLM, SamplingParams

llm = LLM(
    model="meta-llama/Llama-3.1-8B-Instruct",
    max_num_seqs=256,      # параллельно обрабатываем 256 запросов
    max_num_batched_tokens=32768,
    # Paged Attention: KV cache разбит на страницы — нет фрагментации
)

# При правильной настройке: 10-20x больший throughput vs простой batching
```

---

## Multi-region deployment для AI систем

```
Принципы:
1. Data locality: данные обрабатываются в регионе пользователя (GDPR)
2. Latency: ближайший регион = меньше задержка
3. Failover: при падении региона → автоматически другой
```

```typescript
class MultiRegionAI {
  private regions = {
    "eu-west": { endpoint: "eu.api.openai.com", latency: 50 },
    "us-east": { endpoint: "us.api.openai.com", latency: 80 },
    "ap-southeast": { endpoint: "ap.api.openai.com", latency: 120 }
  };

  async generate(request: Request): Promise<Response> {
    const userRegion = this.getRegion(request.userCountry);

    // GDPR: данные европейских пользователей не должны покидать EU
    const allowedRegions = this.isGDPRUser(request.userCountry)
      ? ["eu-west"]  // только EU регион
      : Object.keys(this.regions);

    // Выбор ближайшего доступного региона
    for (const region of this.rankByLatency(allowedRegions, userRegion)) {
      try {
        return await this.callRegion(region, request);
      } catch (error) {
        console.error(`Region ${region} failed, trying next`);
      }
    }

    throw new Error("All regions unavailable");
  }
}
```

---

## Capacity Planning для AI Workloads

```typescript
// Оценка ресурсов для AI системы

interface CapacityModel {
  requestsPerDay: number;
  avgInputTokens: number;
  avgOutputTokens: number;
  peakMultiplier: number;   // пик = среднее × этот коэффициент
}

function estimateCapacity(model: CapacityModel) {
  const {
    requestsPerDay,
    avgInputTokens,
    avgOutputTokens,
    peakMultiplier = 3
  } = model;

  const totalTokensPerDay = requestsPerDay * (avgInputTokens + avgOutputTokens);
  const peakRPM = (requestsPerDay / 24 / 60) * peakMultiplier;
  const peakTPM = peakRPM * (avgInputTokens + avgOutputTokens);

  return {
    tokensPerDay: totalTokensPerDay,
    monthlyCost: estimateMonthlyCost(totalTokensPerDay * 30),
    requiredTPM: peakTPM,

    // Для self-hosted: GPU requirements
    gpusNeeded: Math.ceil(peakTPM / THROUGHPUT_PER_GPU),

    // Для API: нужен ли enterprise plan?
    needsEnterprisePlan: peakTPM > 500_000
  };
}

// Пример: 10K пользователей, 20 запросов/день, 500 in + 300 out токенов
const capacity = estimateCapacity({
  requestsPerDay: 200_000,
  avgInputTokens: 500,
  avgOutputTokens: 300,
  peakMultiplier: 5
});
// → 160M токенов/день, ~$400/день (gpt-4o-mini), 167K peak TPM
```

---

## Cloud vs On-Device deployment

| | Cloud (API) | Self-Hosted | On-Device |
|---|---|---|---|
| **Стоимость** | Per token | GPU аренда | Устройство |
| **Latency** | 100-2000ms | 50-500ms | 10-100ms |
| **Privacy** | Данные уходят | Данные у тебя | Данные на устройстве |
| **Размер модели** | Неограничен | 7B-70B+ | 1B-7B |
| **Обновление** | Автоматическое | Ручное | Через app update |

```typescript
// On-device inference через WebLLM (браузер) или llama.cpp (native)
import { CreateMLCEngine } from "@mlc-ai/web-llm";

// Запуск 7B модели прямо в браузере (WebGPU)
const engine = await CreateMLCEngine("Llama-3.1-8B-Instruct-q4f32_1-MLC");

const response = await engine.chat.completions.create({
  messages: [{ role: "user", content: "Hello!" }],
  // Данные не покидают браузер — 100% приватность
});
```

**Когда on-device:**
- Строгие требования к приватности (мед. данные, финансы)
- Offline режим
- Latency < 100ms критична
- Стоимость масштаба делает API невыгодным

**Материалы:**
- [vLLM: Production-ready LLM serving](https://vllm.ai/)
- [WebLLM: Run LLMs in browser](https://webllm.mlc.ai/)
