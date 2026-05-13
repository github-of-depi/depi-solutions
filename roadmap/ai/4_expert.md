# AI Engineer — 🔴 Expert

Включает все предыдущие уровни + экспертные знания архитектурного и исследовательского уровня.

> **Фундамент:** сначала убедитесь, что знаете [Senior Road Map](senior.md).

---

## LLM Архитектура (математический уровень)

- [ ] [LLM — L4](../../interviews/ai/llm/4_llm_expert.md) — математика масштабирования внимания (√dₖ), Grouped-Query Attention (GQA), RMSNorm, Cross-Entropy Loss, архитектурные решения (Llama, Mistral, Gemma), Vision Transformer (ViT), Small Language Models

---

## Prompt Engineering (автоматизация)

- [ ] [Prompt Engineering — L4](../../interviews/ai/prompt-engineering/4_prompt-engineering_expert.md) — prompt tuning vs fine-tuning, DSPy и автоматическая оптимизация промптов, meta-prompts для генерации промптов, statistical rigor при сравнении промптов

---

## RAG (архитектурный уровень)

- [ ] [RAG — L4](../../interviews/ai/rag/4_rag_expert.md) — enterprise RAG с конфликтующими источниками, real-time RAG для часто обновляемых данных, fine-tuning embedding-моделей под домен, кастомный re-ranker, сложные multi-tenant системы на миллиарды документов

---

## AI Агенты (распределённые системы)

- [ ] [Agents — L4](../../interviews/ai/agents/4_agents_expert.md) — распределённые multi-agent системы, custom agent frameworks, multi-modal агенты, Context Engineering, harness engineering, формальная верификация поведения агентов

---

## Fine-Tuning (исследовательский уровень)

- [ ] [Fine-Tuning — L3](../../interviews/ai/fine-tuning/3_fine-tuning_senior.md) — continual pre-training, full fine-tuning на кластере GPU, LoRA adapter merging стратегии, knowledge distillation и правовые аспекты, custom PEFT методы, alignment tax

---

## Инфраструктура (масштаб датацентра)

- [ ] [LLMOps — L4](../../interviews/ai/llmops/4_llmops_expert.md) — tensor / pipeline parallelism, FSDP vs DeepSpeed ZeRO, GPU selection для LLM-инференса, multi-region deployment, capacity planning, LLM routing по сложности запроса
- [ ] Flash Attention, speculative decoding — математика и реализация
- [ ] Serving миллиардов запросов: архитектура inference cluster

---

## Оценка и тестирование (governance)

- [ ] [Evaluation — L3](../../interviews/ai/evaluation/3_evaluation_senior.md) — evaluation-driven development, continuous evaluation в продакшене, intersectional bias, audit reproducibility, мультимодальное red teaming, conflicting audit results

---

## Безопасность и этика (compliance)

- [ ] [Safety — L2](../../interviews/ai/safety/2_safety_middle.md) — EU AI Act compliance, differential privacy в ML, federated learning и защита от data poisoning, AI watermarking, AI incident response plan, NIST AI RMF, proxy discrimination, feedback loops

---

## AI Strategy & Leadership

- [ ] [Agents — L4 Behavioral](../../interviews/ai/agents/4_agents_expert.md) — ROI AI-фичи, принятие архитектурных решений, управление ожиданиями стейкхолдеров
- [ ] Принятие решений: API vs self-hosted, RAG vs fine-tuning vs prompting — когда что выбрать
- [ ] Коммуникация ограничений AI нетехническим аудиториям
- [ ] Оценка environmental impact: снижение carbon footprint AI-тренировок
- [ ] Human over-reliance на AI: как предотвратить

---

## System Design (масштаб и сложность)

- [ ] Multi-agent workflow system — коллаборация агентов на сложных задачах
- [ ] Real-time AI transcription / live streaming content moderation
- [ ] AI dynamic pricing engine
- [ ] Fraud detection powered by LLMs
- [ ] AI resume screening: 100K заявок в неделю
- [ ] AI voice assistant архитектура

---

## Ключевые вопросы уровня

- Объяснить математику механизма внимания: зачем делим на √dₖ?
- Как работает Grouped-Query Attention и чем лучше MHA?
- Реализовать LLM inference cluster с tensor parallelism
- EU AI Act: как определить high-risk classification?
- Как автоматически оптимизировать промпты (DSPy)?
- Спроектировать AI-систему для 100K req/sec с graceful degradation
- Как бороться с proxy discrimination в AI-системах?
- Differentiating: когда RL-агент, когда RAG+агент, когда просто fine-tuning?
