# AI Engineer — 🟢 Junior

Фундамент: понимание того, что такое LLM, как использовать готовые модели через API и строить простые AI-приложения. Все темы обязательны.

> **Middle?** → [Middle Road Map](middle.md) включает этот список + свой уровень.

---

## LLM Основы

- [ ] [LLM — L1](../../interviews/ai/llm/1_llm_junior.md) — что такое LLM, токены, контекстное окно, temperature, embeddings, hallucinations, system prompt vs user message

---

## Prompt Engineering

- [ ] [Prompt Engineering — L1](../../interviews/ai/prompt-engineering/1_prompt-engineering_junior.md) — zero-shot / one-shot / few-shot, system prompt, structured output (JSON), базовая защита от prompt injection, "не знаю"

---

## RAG

- [ ] [RAG — L1](../../interviews/ai/rag/1_rag_junior.md) — зачем нужен RAG, базовый pipeline (chunk → embed → store → retrieve → generate), понятие чанка и эмбеддинга

---

## AI Агенты

- [ ] [Agents — L1](../../interviews/ai/agents/1_agents_junior.md) — что такое AI агент, чем отличается от простого LLM-вызова, tool use / function calling, agent loop

---

## Векторные базы данных

- [ ] [Vector Databases — L1](../../interviews/ai/vector-databases/1_vector-databases_junior.md) — что такое векторная БД, cosine similarity, базовый семантический поиск, отличие от SQL-базы

---

## LLMOps

- [ ] [LLMOps — L1](../../interviews/ai/llmops/1_llmops_junior.md) — работа с API (OpenAI, Anthropic), подсчёт токенов и стоимости, retry с exponential backoff, streaming responses

---

## Coding with AI

- [ ] [Coding with AI — L1](../../interviews/ai/coding-with-ai/1_coding-with-ai_junior.md) — использование Copilot / ChatGPT для помощи в разработке, prompt-to-code, code review с AI

---

## Практические навыки

- [ ] Python или TypeScript/Node.js — основной язык для AI-приложений
- [ ] OpenAI API / Anthropic API — вызов моделей, параметры (temperature, max_tokens, top_p)
- [ ] Базовое использование LangChain или LlamaIndex
- [ ] Работа с `.env` и безопасное хранение API-ключей

---

## Ключевые вопросы уровня

- Что такое LLM и как она генерирует текст?
- Чем токен отличается от слова? Как считать стоимость запроса?
- Что такое temperature и как влияет на ответы?
- Объяснить RAG простыми словами
- Чем агент отличается от простого вызова LLM?
- Что такое prompt injection?
- Написать простой RAG pipeline (fetch → chunk → embed → search → generate)
