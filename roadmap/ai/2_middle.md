# AI Engineer — 🔵 Middle

Включает весь уровень Junior + новое на этом уровне.

> **Фундамент:** сначала убедитесь, что знаете [Junior Road Map](junior.md).

---

## LLM Основы (глубоко)

- [ ] [LLM — L2](../../interviews/ai/llm/2_llm_middle.md) — Transformer архитектура, self-attention Q/K/V, multi-head attention, KV cache, encoder-only vs decoder-only vs encoder-decoder, positional encoding, open-source vs closed-source

---

## Prompt Engineering

- [ ] [Prompt Engineering — L2](../../interviews/ai/prompt-engineering/2_prompt-engineering_middle.md) — Chain-of-Thought (CoT), self-consistency, ReAct prompting, output parsers, "lost in the middle", prompt chaining, multi-turn conversations, role prompting

---

## RAG (системный уровень)

- [ ] [RAG — L2](../../interviews/ai/rag/2_rag_middle.md) — стратегии чанкинга (fixed / recursive / semantic), выбор embedding-модели, hybrid search (vector + keyword), re-ranking, оценка RAG (faithfulness / relevance / precision / recall), HyDE, query decomposition

---

## AI Агенты

- [ ] [Agents — L2](../../interviews/ai/agents/2_agents_middle.md) — ReAct архитектура, Plan-and-Execute, типы памяти агента (short-term / long-term / episodic), multi-agent системы, MCP (Model Context Protocol), error recovery

---

## Fine-Tuning

- [ ] [Fine-Tuning — L1](../../interviews/ai/fine-tuning/1_fine-tuning_junior.md) — когда нужен fine-tuning vs RAG vs prompt engineering, LoRA, QLoRA, подготовка датасета, instruction tuning

---

## Векторные базы данных

- [ ] [Vector Databases — L2](../../interviews/ai/vector-databases/2_vector-databases_middle.md) — алгоритмы ANN (HNSW, IVF), hybrid search, metadata filtering, dimensionality, sparse vs dense embeddings, embedding drift

---

## LLMOps

- [ ] [LLMOps — L2](../../interviews/ai/llmops/2_llmops_middle.md) — мониторинг LLM-приложений, prompt versioning, A/B тестирование, CI/CD для AI, semantic caching, rate limiting, structured output в продакшене

---

## Оценка (Evaluation)

- [ ] [Evaluation — L1](../../interviews/ai/evaluation/1_evaluation_junior.md) — BLEU, ROUGE, BERTScore, human evaluation, LLM-as-a-judge, hallucination detection, оценка RAG end-to-end

---

## Coding with AI

- [ ] [Coding with AI — L2](../../interviews/ai/coding-with-ai/2_coding-with-ai_middle.md)

---

## Практические навыки

- [ ] LangChain / LlamaIndex — продвинутое использование: chains, agents, RAG pipelines
- [ ] Chroma / Qdrant / Pinecone — работа с векторными базами данных в продакшене
- [ ] Quantization (INT8 / FP16 / BF16) — основы оптимизации инференса
- [ ] Реализация RAG pipeline с нуля (embedding + vector search + generation)
- [ ] Function calling / tool use через OpenAI / Anthropic API

---

## Ключевые вопросы уровня

- Объяснить механизм attention: Q, K, V
- Зачем нужен KV cache и как он ускоряет инференс?
- Сравнить стратегии чанкинга: fixed vs recursive vs semantic
- Что такое hybrid search и почему он лучше pure vector search?
- Как устроен ReAct агент?
- Когда выбирать LoRA, а когда полный fine-tuning?
- Реализовать conversation memory с sliding window
- Объяснить LLM-as-a-judge: что это и какие ограничения?
