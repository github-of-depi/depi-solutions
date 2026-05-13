# AI Engineer — 🟠 Senior

Включает весь уровень Junior + Middle + новое на этом уровне.

> **Фундамент:** сначала убедитесь, что знаете [Middle Road Map](middle.md).

---

## LLM Основы (исследовательский уровень)

- [ ] [LLM — L3](../../interviews/ai/llm/3_llm_senior.md) — BPE / WordPiece / SentencePiece, RoPE, Flash Attention, Mixture of Experts (MoE), dense vs sparse, model distillation, RLHF, reward hacking, causal masking, skip connections, GQA

---

## Prompt Engineering (продакшен)

- [ ] [Prompt Engineering — L3](../../interviews/ai/prompt-engineering/3_prompt-engineering_senior.md) — tree-of-thought, meta-prompts, adversarial inputs, edge cases, multilingual prompting, evaluation и итерация промптов, производственные шаблоны, prompt sensitivity

---

## RAG (продвинутый)

- [ ] [RAG — L3](../../interviews/ai/rag/3_rag_senior.md) — GraphRAG, Self-RAG, Agentic RAG, multi-hop QA, parent-child chunking, per-user access control, масштабирование до миллионов документов, версионирование knowledge base, multimodal RAG, PDF parsing

---

## AI Агенты (продвинутый)

- [ ] [Agents — L3](../../interviews/ai/agents/3_agents_senior.md) — сложная оркестрация агентов, управление бюджетом токенов, human-in-the-loop, guardrails для необратимых действий, безопасность агентных систем, code execution в sandbox, reflection agent

---

## Fine-Tuning (углублённо)

- [ ] [Fine-Tuning — L2](../../interviews/ai/fine-tuning/2_fine-tuning_middle.md) — RLHF и reward hacking, SFT vs alignment, catastrophic forgetting, RLAIF, синтетические данные для fine-tuning, LoRA для доменно-специфичных задач, merging adapters

---

## Векторные базы данных (масштаб)

- [ ] [Vector Databases — L3](../../interviews/ai/vector-databases/3_vector-databases_senior.md) — multi-tenant архитектура, масштабирование до миллионов эмбеддингов, embedding quantization, fine-tuning embedding-модели, embedding drift management

---

## LLMOps (продакшен)

- [ ] [LLMOps — L3](../../interviews/ai/llmops/3_llmops_senior.md) — guardrails и content filtering, PII handling (GDPR / CCPA), observability (TTFT, inter-token latency, GPU utilization), fallback стратегии, context compression / prefix caching, semantic routing, secrets management

---

## Оценка и тестирование

- [ ] [Evaluation — L2](../../interviews/ai/evaluation/2_evaluation_middle.md) — G-Eval, red teaming перед релизом, regression suite, benchmark суиты (MMLU, HumanEval, GSM8K), offline vs online evaluation, оценка качества агентов, golden datasets

---

## Безопасность и этика

- [ ] [Safety — L1](../../interviews/ai/safety/1_safety_junior.md) — типы prompt injection (direct / indirect), защита системного промпта, GDPR / CCPA, content safety, explainability vs interpretability

---

## AI System Design

- [ ] [Agents — L3 System Design](../../interviews/ai/agents/3_agents_senior.md) — проектирование customer support chatbot, document Q&A для enterprise, code generation & review system
- [ ] Latency vs quality trade-offs в AI-системах
- [ ] Caching стратегии (semantic cache, prefix cache)
- [ ] Rate limiting и cost management
- [ ] Failover и fallback при недоступности провайдера
- [ ] Multi-tenant AI chatbot platform

---

## Инфраструктура

- [ ] Speculative decoding, Paged Attention, Continuous batching — оптимизация инференса
- [ ] Self-hosted vs API: критерии выбора
- [ ] Quantization (INT4 / INT8 / FP16) — компромисс качество/скорость
- [ ] GPU memory management для LLM-инференса

---

## Ключевые вопросы уровня

- Спроектировать enterprise RAG-систему с per-user access control
- Как бороться с reward hacking в RLHF?
- Как предотвратить необратимые действия AI-агента?
- Объяснить Flash Attention и зачем он нужен
- Как настроить observability для LLM в продакшене?
- Как провести red teaming перед запуском LLM-продукта?
- Спроектировать multi-agent workflow для сложной задачи
- Как управлять catastrophic forgetting при fine-tuning?
