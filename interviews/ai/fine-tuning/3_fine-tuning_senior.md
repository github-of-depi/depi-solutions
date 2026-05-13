# Fine-Tuning — 🟠 Senior

## Вопросы

- [Что такое RLAIF и чем отличается от RLHF?](#что-такое-rlaif-и-чем-отличается-от-rlhf)
- [Что такое DPO как альтернатива RLHF?](#что-такое-dpo-как-альтернатива-rlhf)
- [Что такое continual pre-training?](#что-такое-continual-pre-training)
- [Как fine-tune под медицинский / юридический / финансовый домен?](#как-fine-tune-под-медицинский--юридический--финансовый-домен)
- [Как решить overfitting при fine-tuning?](#как-решить-overfitting-при-fine-tuning)
- [Правовые аспекты knowledge distillation](#правовые-аспекты-knowledge-distillation)

---

## Что такое RLAIF и чем отличается от RLHF?

RLAIF (Reinforcement Learning from AI Feedback) — вместо людей-аннотаторов используется LLM для оценки качества ответов.

```python
# RLHF: люди выбирают лучший ответ
human_preferences = collect_human_comparisons(prompt_pairs)

# RLAIF: LLM оценивает ответы
async def ai_judge_comparison(
    prompt: str,
    response_a: str,
    response_b: str
) -> float:
    """Возвращает вероятность что A лучше B"""
    judgment = await claude_opus.generate(f"""
        Оцени два ответа на запрос. Какой из них лучше?
        
        Критерии: полезность, точность, безопасность, ясность.
        
        Запрос: {prompt}
        
        Ответ A: {response_a}
        Ответ B: {response_b}
        
        Верни JSON: {{"winner": "A" or "B", "confidence": 0.0-1.0, "reasoning": "..."}}
    """)
    return json.loads(judgment)["confidence"] if json.loads(judgment)["winner"] == "A" else 0.0
```

| | RLHF | RLAIF |
|---|---|---|
| **Стоимость** | Высокая (люди) | Низкая (AI) |
| **Масштаб** | Ограничен числом аннотаторов | Неограничен |
| **Качество** | Отражает человеческие ценности | Может унаследовать предвзятости LLM |
| **Скорость** | Медленно | Быстро |

**Материалы:**
- [Constitutional AI (Anthropic)](https://arxiv.org/abs/2212.08073)

---

## Что такое DPO как альтернатива RLHF?

DPO (Direct Preference Optimization) — более простой метод выравнивания: обучает модель напрямую на парах предпочтений без отдельной reward model и PPO.

```python
from trl import DPOTrainer, DPOConfig

# Датасет DPO: тройки (prompt, chosen, rejected)
dpo_dataset = [
    {
        "prompt": "Как взломать систему?",
        "chosen": "Я не могу помочь с незаконными действиями.",
        "rejected": "Вот инструкция: ..."
    },
    {
        "prompt": "Напиши резюме статьи",
        "chosen": "Статья рассматривает три ключевых аспекта...",
        "rejected": "Статья хорошая. Автор молодец."
    }
]

# Training
dpo_config = DPOConfig(
    beta=0.1,           # KL penalty coefficient (насколько далеко можем уйти от ref model)
    learning_rate=5e-5,
    per_device_train_batch_size=2,
    num_train_epochs=3
)

dpo_trainer = DPOTrainer(
    model=sft_model,
    ref_model=sft_model_ref,  # замороженная референсная копия
    args=dpo_config,
    train_dataset=dpo_dataset,
    tokenizer=tokenizer
)

dpo_trainer.train()
```

**DPO vs RLHF:**
- DPO: проще, нет reward model, нет PPO, обычно достаточно хорошо
- RLHF/PPO: сложнее, но более гибкий, лучше для сложных требований

**Материалы:**
- [DPO paper](https://arxiv.org/abs/2305.18290)

---

## Что такое continual pre-training?

Continual pre-training — продолжение предобучения модели на доменных данных (без instruction format), чтобы модель "узнала" специфическую терминологию и знания.

```python
# Когда нужен:
# - Домен с уникальной лексикой (медицина, право, химия)
# - Большой корпус свежих данных (новости, статьи)
# - Языки с малым представлением в исходной модели

# Pipeline: continual pre-training → instruction tuning → alignment

from transformers import Trainer, TrainingArguments, DataCollatorForLanguageModeling

# Датасет: просто сырой текст, без instruction format
corpus = [
    "Атеросклероз — хроническое заболевание артерий, характеризующееся...",
    "Гипертензия определяется как систолическое АД ≥ 140 мм рт. ст...",
    # ... миллионы примеров медицинского текста
]

# Обучение в режиме next-token prediction (как GPT pretraining)
data_collator = DataCollatorForLanguageModeling(
    tokenizer=tokenizer,
    mlm=False  # causal LM (предсказание следующего токена)
)

trainer = Trainer(
    model=base_model,
    args=TrainingArguments(
        learning_rate=1e-4,   # выше чем при SFT
        num_train_epochs=1,   # обычно 1-3 прохода
        per_device_train_batch_size=8
    ),
    train_dataset=corpus,
    data_collator=data_collator
)
```

---

## Как fine-tune под медицинский / юридический / финансовый домен?

```python
# Общая стратегия для regulated domains

# 1. Сбор данных
domain_corpus = {
    "medical": ["PubMed abstracts", "clinical notes", "drug interactions"],
    "legal": ["court decisions", "legislation", "legal commentaries"],
    "financial": ["SEC filings", "earnings calls", "financial reports"]
}

# 2. Continual pre-training на доменном корпусе
model = continue_pretraining(base_model, domain_corpus[domain])

# 3. SFT на quality examples с domain experts
expert_examples = load_expert_annotated_data()
model = supervised_finetune(model, expert_examples)

# 4. Alignment с domain-specific safety
# Medical: не ставить диагнозы, направлять к врачу
# Legal: не давать юридических советов без квалификации
# Financial: предупреждать о рисках

alignment_rules = """
Ты — медицинский ассистент для справочных целей.
НИКОГДА не ставь диагнозы и не заменяй консультацию врача.
Всегда добавляй: "Проконсультируйтесь с врачом."
"""

# 5. Evaluation с domain experts + специфические бенчмарки
# Medical: MedQA, PubMedQA
# Legal: LegalBench
# Financial: FinanceBench
```

---

## Как решить overfitting при fine-tuning?

```python
# Признаки overfitting:
# - Training loss падает, validation loss растёт
# - Модель заучила примеры дословно (verbatim memorization)
# - Плохо работает на новых данных

# Методы решения:

# 1. Уменьшить количество эпох
training_args = TrainingArguments(
    num_train_epochs=1,       # вместо 5
    load_best_model_at_end=True
)

# 2. Уменьшить learning rate
learning_rate=5e-5  # вместо 2e-4

# 3. Добавить регуляризацию
training_args = TrainingArguments(
    weight_decay=0.01,         # L2 регуляризация
    max_grad_norm=0.3,         # gradient clipping
)

# 4. Уменьшить LoRA rank
lora_config = LoraConfig(r=8)  # вместо r=64

# 5. Добавить dropout
lora_config = LoraConfig(lora_dropout=0.1)

# 6. Увеличить датасет (data augmentation)
augmented = [paraphrase(example) for example in train_data]
train_data.extend(augmented)

# Мониторинг через eval loss
trainer = Trainer(
    evaluation_strategy="steps",
    eval_steps=50,
    metric_for_best_model="eval_loss",
    greater_is_better=False,
    early_stopping_patience=3  # остановить если нет улучшений 3 шага подряд
)
```

---

## Правовые аспекты knowledge distillation

Knowledge distillation — обучение малой "студенческой" модели воспроизводить поведение большой "учительской" модели.

**Правовые риски:**

```
1. Нарушение Terms of Service
   - OpenAI TOS: запрещает использовать выходы GPT для обучения конкурирующих моделей
   - Anthropic TOS: аналогично
   → Проверяй TOS провайдера перед использованием выходов для обучения!

2. Copyright
   - Если учительская модель генерирует контент защищённый авторским правом
   → Используй только данные, на которые у тебя есть права

3. GDPR
   - Данные пользователей в синтетическом датасете могут нарушать GDPR
   → Анонимизируй все персональные данные перед обучением
```

**Безопасные подходы:**
- Использовать open-source модели (Llama, Mistral) в качестве учителя — нет ограничений TOS
- Distillation на собственных корпоративных данных
- Проконсультироваться с юридическим отделом перед дистилляцией из коммерческих API
