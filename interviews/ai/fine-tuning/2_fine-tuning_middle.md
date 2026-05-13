# Fine-Tuning — 🔵 Middle

## Вопросы

- [Что такое QLoRA и как он работает на consumer hardware?](#что-такое-qlora-и-как-он-работает-на-consumer-hardware)
- [Что такое RLHF и как выравнивать LLM?](#что-такое-rlhf-и-как-выравнивать-llm)
- [Что такое catastrophic forgetting и как его предотвратить?](#что-такое-catastrophic-forgetting-и-как-его-предотвратить)
- [Ключевые гиперпараметры fine-tuning](#ключевые-гиперпараметры-fine-tuning)
- [Как генерировать синтетические данные для fine-tuning?](#как-генерировать-синтетические-данные-для-fine-tuning)
- [Как объединять LoRA адаптеры?](#как-объединять-lora-адаптеры)

---

## Что такое QLoRA и как он работает на consumer hardware?

QLoRA (Quantized LoRA) = LoRA + 4-bit quantization исходной модели. Позволяет fine-tune 70B моделей на одной GPU с 24GB VRAM.

```python
from transformers import AutoModelForCausalLM, BitsAndBytesConfig
from peft import LoraConfig, get_peft_model

# 1. Загружаем модель в 4-bit
quantization_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",       # NormalFloat4 — лучше для весов
    bnb_4bit_compute_dtype=torch.bfloat16,  # вычисления в bf16
    bnb_4bit_use_double_quant=True   # двойная квантизация для экономии памяти
)

model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Llama-3.1-8B",
    quantization_config=quantization_config,
    device_map="auto"
)

# 2. Применяем LoRA поверх квантизованной модели
model = prepare_model_for_kbit_training(model)

lora_config = LoraConfig(
    r=64,
    lora_alpha=16,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj",
                    "gate_proj", "up_proj", "down_proj"],
    lora_dropout=0.05,
    task_type="CAUSAL_LM"
)

model = get_peft_model(model, lora_config)
```

**Экономия памяти:**
- Llama 3.1 8B в fp16: ~16GB VRAM
- Llama 3.1 8B в 4bit (QLoRA): ~5GB VRAM

**Материалы:**
- [QLoRA paper](https://arxiv.org/abs/2305.14314)
- [Unsloth: 2x faster QLoRA](https://github.com/unslothai/unsloth)

---

## Что такое RLHF и как выравнивать LLM?

RLHF (Reinforcement Learning from Human Feedback) — трёхэтапный процесс выравнивания модели с человеческими предпочтениями.

**Этап 1: SFT (Supervised Fine-Tuning)**
```python
# Обучаем на примерах "хороших" ответов
sft_model = fine_tune(base_model, high_quality_demonstrations)
```

**Этап 2: Reward Model Training**
```python
# Люди выбирают лучший ответ из пары
# Обучаем reward model предсказывать человеческие предпочтения
reward_model = train_reward_model(comparison_dataset)
# Входы: (prompt, response_A, response_B)
# Выходы: вероятность что A лучше B
```

**Этап 3: RL fine-tuning (PPO)**
```python
from trl import PPOTrainer

ppo_trainer = PPOTrainer(
    model=sft_model,
    ref_model=ref_sft_model,  # референсная модель для KL-дивергенции
    reward_model=reward_model,
    tokenizer=tokenizer,
    config=ppo_config
)

# Обучение через PPO
for batch in dataset:
    query_tensors = tokenize(batch["queries"])
    response_tensors = ppo_trainer.generate(query_tensors)

    # Получаем reward от reward model
    rewards = reward_model.score(query_tensors, response_tensors)

    # PPO шаг
    stats = ppo_trainer.step(query_tensors, response_tensors, rewards)
```

**Проблемы RLHF:**
- **Reward hacking** — модель оптимизирует reward model, а не реальную полезность
- **Alignment tax** — после RLHF модель может потерять часть capabilities
- **Annotation bias** — качество зависит от аннотаторов

---

## Что такое catastrophic forgetting и как его предотвратить?

Catastrophic forgetting — при fine-tuning на узкой задаче модель "забывает" общие знания из предобучения.

```python
# Метод 1: LoRA — заморожены оригинальные веса, обновляются только адаптеры
# Модель сохраняет базовые знания

# Метод 2: Rehearsal — добавляем примеры из оригинальных данных
def create_training_dataset(domain_data, general_data, mix_ratio=0.1):
    """Смешиваем 90% доменных + 10% общих данных"""
    n_general = int(len(domain_data) * mix_ratio)
    general_sample = random.sample(general_data, n_general)
    return domain_data + general_sample

# Метод 3: EWC (Elastic Weight Consolidation) — штрафуем за изменение важных весов
# Считаем Fisher Information Matrix до fine-tuning, штрафуем за отклонение

# Метод 4: Уменьшить learning rate
training_args = TrainingArguments(
    learning_rate=1e-4,  # маленький lr снижает forgetting
    num_train_epochs=1,  # не переобучать
    warmup_ratio=0.03
)
```

**Правило:** LoRA + маленький learning rate + небольшое количество эпох — лучший способ избежать catastrophic forgetting для большинства задач.

---

## Ключевые гиперпараметры fine-tuning

| Параметр | Типичные значения | Влияние |
|----------|-----------------|---------|
| **Learning rate** | 1e-5 – 2e-4 | Выше → быстрее учится, легче забывает |
| **Epochs** | 1–5 | Больше → может переобучиться |
| **Batch size** | 4–32 | Больше → стабильнее градиент |
| **LoRA rank (r)** | 4–64 | Выше → больше параметров, лучше качество, дольше |
| **LoRA alpha** | 2× rank | Scaling factor |
| **Max tokens** | 512–4096 | Зависит от задачи |

```python
# Подбор learning rate через warmup + cosine schedule
training_args = TrainingArguments(
    output_dir="./results",
    num_train_epochs=3,
    per_device_train_batch_size=4,
    gradient_accumulation_steps=4,  # effective batch = 16
    learning_rate=2e-4,
    lr_scheduler_type="cosine",
    warmup_ratio=0.05,
    weight_decay=0.001,
    fp16=True,              # или bf16=True на A100
    evaluation_strategy="steps",
    eval_steps=50,
    save_steps=100,
    load_best_model_at_end=True
)
```

**Как подобрать:** начни с `learning_rate=2e-4`, `r=16`. Если плохо учится — увеличь rank. Если переобучается — уменьши lr и количество эпох.

---

## Как генерировать синтетические данные для fine-tuning?

```python
# Pipeline: сильная модель → синтетические данные → fine-tuning слабой модели
async def generate_synthetic_dataset(
    domain: str,
    target_format: str,
    count: int = 1000
) -> list[dict]:
    examples = []

    # Генерируем разнообразные вопросы по домену
    questions = await gpt4.generate(f"""
        Создай {count} разнообразных вопросов из области "{domain}".
        Включи: простые, сложные, edge cases, вопросы с несколькими вариантами.
        Верни JSON-массив.
    """)

    for question in JSON.parse(questions):
        # Генерируем эталонный ответ
        answer = await gpt4.generate(f"""
            Ответь на вопрос из области {domain} в формате {target_format}.
            Вопрос: {question}
        """)
        examples.append({"input": question, "output": answer})

    return examples

# Важно: проверяй качество синтетических данных!
def filter_quality(examples: list[dict]) -> list[dict]:
    return [
        e for e in examples
        if len(e["output"]) > 50 and      # не пустые ответы
           not contains_refusal(e["output"]) and  # не отказы
           not is_duplicate(e, examples)  # не дубли
    ]
```

---

## Как объединять LoRA адаптеры?

```python
from peft import PeftModel

# Метод 1: Merge into base model (для деплоя без PEFT overhead)
base_model = AutoModelForCausalLM.from_pretrained("base-model")
peft_model = PeftModel.from_pretrained(base_model, "path/to/lora-adapter")

# Объединяем LoRA веса с базовой моделью
merged_model = peft_model.merge_and_unload()
merged_model.save_pretrained("merged-model")

# Метод 2: LoRA Merging — объединение нескольких адаптеров
# Полезно когда есть адаптеры для разных навыков
from peft import LoraConfig, get_peft_model

# Task arithmetic: merged = base + scale1*(adapter1-base) + scale2*(adapter2-base)
def merge_lora_adapters(
    base_model: Model,
    adapters: list[LoRAAdapter],
    scales: list[float]
) -> Model:
    # Взвешенное объединение дельт весов
    merged_delta = sum(
        scale * adapter.delta_weights
        for scale, adapter in zip(scales, adapters)
    )
    return base_model + merged_delta

# Практика: используй mergekit для сложных слияний
# https://github.com/arcee-ai/mergekit
```

**Материалы:**
- [mergekit: Model merging toolkit](https://github.com/arcee-ai/mergekit)
- [TIES-Merging paper](https://arxiv.org/abs/2306.01708)
