# Fine-Tuning — 🟢 Junior

## Вопросы

- [Что такое fine-tuning и когда он нужен?](#что-такое-fine-tuning-и-когда-он-нужен)
- [Когда выбирать fine-tuning, а когда RAG или prompt engineering?](#когда-выбирать-fine-tuning-а-когда-rag-или-prompt-engineering)
- [Что такое LoRA и как он работает?](#что-такое-lora-и-как-он-работает)
- [Как подготовить датасет для fine-tuning?](#как-подготовить-датасет-для-fine-tuning)
- [Что такое instruction tuning?](#что-такое-instruction-tuning)

---

## Что такое fine-tuning и когда он нужен?

Fine-tuning — дообучение предобученной модели на специфическом датасете для адаптации к конкретной задаче или домену.

**Типы fine-tuning:**
- **Full fine-tuning** — обновляются все веса модели. Дорого, требует много GPU.
- **PEFT (Parameter-Efficient Fine-Tuning)** — обновляется малая часть параметров (LoRA, адаптеры). Доступно на consumer hardware.

**Когда нужен fine-tuning:**
- Нужен специфический стиль/тон/формат ответов
- Доменная лексика, которую модель плохо понимает (медицина, право, нишевые технологии)
- Нужна конкретная структура вывода, которую сложно получить промптингом
- Снижение задержки (меньше токенов в промпте)

**Когда НЕ нужен:**
- Нужны актуальные или корпоративные данные → RAG
- Достаточно изменить поведение промптом → prompt engineering
- Нет достаточного бюджета и датасета

---

## Когда выбирать fine-tuning, а когда RAG или prompt engineering?

| Метод | Стоимость | Время обновления | Прозрачность | Объём знаний |
|-------|-----------|-----------------|-------------|-------------|
| **Prompt Engineering** | Минимальная | Мгновенно | Высокая | Ограничен контекстом |
| **RAG** | Низкая | Мгновенно | Высокая | Неограничен |
| **Fine-tuning** | Высокая | Дни/недели | Низкая | В весах модели |

**Алгоритм выбора:**
```
1. Сначала попробуй prompt engineering
   ↓ (не работает)
2. Добавь RAG (если нужны внешние знания)
   ↓ (стиль/формат всё равно плохой)
3. Рассмотри fine-tuning (для стиля/поведения)
   ↓ (нужно и знания и поведение)
4. RAG + Fine-tuning
```

**Материалы:**
- [OpenAI: When to fine-tune](https://platform.openai.com/docs/guides/fine-tuning/when-to-use-fine-tuning)

---

## Что такое LoRA и как он работает?

LoRA (Low-Rank Adaptation) — метод PEFT, при котором вместо изменения исходных весов добавляются небольшие матрицы низкого ранга.

**Идея:** большая матрица весов $W$ (например, 4096×4096) имеет низкий эффективный ранг при адаптации. Вместо обновления всей $W$ обучаем $\Delta W = A \cdot B$, где $A$ и $B$ — маленькие матрицы.

```
Оригинальная матрица W:   4096 × 4096 = 16,777,216 параметров
LoRA матрицы (rank=16):   4096×16 + 16×4096 = 131,072 параметров
                          → в 128 раз меньше!
```

```python
# Использование через PEFT (HuggingFace)
from peft import LoraConfig, get_peft_model
from transformers import AutoModelForCausalLM

model = AutoModelForCausalLM.from_pretrained("meta-llama/Llama-3.1-8B")

lora_config = LoraConfig(
    r=16,               # ранг матриц (обычно 4–64)
    lora_alpha=32,      # scaling factor = alpha/r
    target_modules=["q_proj", "v_proj"],  # какие слои адаптировать
    lora_dropout=0.1,
    bias="none"
)

model = get_peft_model(model, lora_config)
model.print_trainable_parameters()
# trainable params: 6,815,744 || all params: 7,248,547,840 || trainable%: 0.09%
```

**Материалы:**
- [LoRA paper: Low-Rank Adaptation of LLMs](https://arxiv.org/abs/2106.09685)

---

## Как подготовить датасет для fine-tuning?

```python
# Формат датасета для instruction tuning (JSONL)
# Каждая строка — один пример обучения

# Формат OpenAI fine-tuning
examples = [
    {
        "messages": [
            {"role": "system", "content": "Ты эксперт по корпоративному праву."},
            {"role": "user", "content": "Что такое акционерное соглашение?"},
            {"role": "assistant", "content": "Акционерное соглашение (SHA) — это договор..."}
        ]
    },
    # ... минимум 10–50 примеров, лучше 100–1000
]

# Сохранение в JSONL
import jsonlines
with jsonlines.open("train.jsonl", "w") as writer:
    writer.write_all(examples)
```

**Требования к датасету:**

| Критерий | Требование |
|----------|-----------|
| Минимальный размер | 10–50 примеров (для OpenAI), 1000+ для серьёзного обучения |
| Качество | Высокое — один плохой пример хуже чем его отсутствие |
| Разнообразие | Покрывай edge cases и вариации |
| Баланс | Равномерное распределение по типам задач |
| Формат | Тот же формат, который хочешь получать на выходе |

**Генерация синтетических данных:**
```python
# Используй сильную модель для создания обучающих примеров для слабой
async def generate_training_examples(topic: str, count: int):
    return await gpt4.generate(f"""
        Создай {count} разнообразных примеров вопрос-ответ на тему "{topic}".
        Формат JSON. Ответы должны быть в стиле: кратко, профессионально, с примерами.
    """)
```

---

## Что такое instruction tuning?

Instruction tuning — вид supervised fine-tuning, при котором модель обучается следовать инструкциям в формате вопрос-ответ или задача-ответ.

**До instruction tuning:** модель просто продолжает текст.
**После instruction tuning:** модель понимает задание и выполняет его.

```python
# Пример instruction tuning датасета
instruction_examples = [
    {
        "instruction": "Переведи текст на английский язык.",
        "input": "Привет, как дела?",
        "output": "Hello, how are you?"
    },
    {
        "instruction": "Классифицируй тональность отзыва.",
        "input": "Доставка была быстрой, но упаковка помялась.",
        "output": "нейтральный"
    },
    {
        "instruction": "Напиши SQL запрос для получения топ-10 клиентов по выручке.",
        "input": "Таблица: orders (customer_id, amount, date)",
        "output": "SELECT customer_id, SUM(amount) as total FROM orders GROUP BY customer_id ORDER BY total DESC LIMIT 10"
    }
]
```

Популярные датасеты для instruction tuning: **Alpaca**, **FLAN**, **OpenHermes**, **ShareGPT**.

**Материалы:**
- [HuggingFace: Fine-tuning guide](https://huggingface.co/docs/transformers/training)
- [Unsloth: Fast LoRA fine-tuning](https://github.com/unslothai/unsloth)
