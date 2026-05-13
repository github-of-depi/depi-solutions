# Fine-Tuning — 🔴 Expert

## Вопросы

- [Как сделать full fine-tuning на кластере GPU?](#как-сделать-full-fine-tuning-на-кластере-gpu)
- [Что такое FSDP и DeepSpeed ZeRO?](#что-такое-fsdp-и-deepspeed-zero)
- [Кастомные PEFT методы](#кастомные-peft-методы)
- [Alignment tax: как баланс между безопасностью и capability?](#alignment-tax-как-баланс-между-безопасностью-и-capability)
- [Оценка fine-tuned модели: метрики и бенчмарки](#оценка-fine-tuned-модели-метрики-и-бенчмарки)

---

## Как сделать full fine-tuning на кластере GPU?

```python
# Multi-GPU full fine-tuning через DeepSpeed + HuggingFace Trainer

# 1. DeepSpeed config (ds_config.json)
ds_config = {
    "train_batch_size": "auto",
    "gradient_accumulation_steps": "auto",
    "optimizer": {
        "type": "AdamW",
        "params": { "lr": 2e-5, "weight_decay": 0.01 }
    },
    "scheduler": {
        "type": "WarmupDecayLR",
        "params": { "warmup_min_lr": 0, "warmup_num_steps": 100 }
    },
    "zero_optimization": {
        "stage": 3,                    # ZeRO-3: самое агрессивное шардирование
        "offload_optimizer": {
            "device": "cpu",           # optimizer states → CPU (экономия GPU)
        },
        "offload_param": {
            "device": "cpu"            # параметры модели → CPU при необходимости
        }
    },
    "bf16": { "enabled": True },
    "gradient_clipping": 1.0
}

# 2. Training script
from transformers import TrainingArguments, Trainer

training_args = TrainingArguments(
    output_dir="./results",
    deepspeed="ds_config.json",
    per_device_train_batch_size=2,
    gradient_accumulation_steps=8,
    num_train_epochs=3,
    learning_rate=2e-5,
    bf16=True,
    # Multi-node
    local_rank=-1,  # устанавливается автоматически через DeepSpeed launcher
)

# 3. Запуск на 8 GPU
# deepspeed --num_gpus=8 train.py --deepspeed ds_config.json

# 4. Checkpointing для возобновления при сбоях
training_args = TrainingArguments(
    save_steps=100,
    save_total_limit=3,
    resume_from_checkpoint="./checkpoint-500"
)
```

---

## Что такое FSDP и DeepSpeed ZeRO?

Оба — методы шардирования модели по GPU для обучения моделей, не помещающихся в память одной GPU.

**ZeRO (Zero Redundancy Optimizer) — 3 стадии:**
```
ZeRO-1: Шардирование optimizer states          → 4x экономия памяти
ZeRO-2: + шардирование gradients              → 8x экономия памяти  
ZeRO-3: + шардирование model parameters       → 64x+ экономия памяти
```

**FSDP (Fully Sharded Data Parallel) — PyTorch native:**
```python
from torch.distributed.fsdp import FullyShardedDataParallel as FSDP
from torch.distributed.fsdp.wrap import transformer_auto_wrap_policy

# Автоматическое шардирование трансформерных слоёв
model = FSDP(
    model,
    auto_wrap_policy=transformer_auto_wrap_policy,
    mixed_precision=MixedPrecision(
        param_dtype=torch.bfloat16,
        reduce_dtype=torch.float32,
    ),
    sharding_strategy=ShardingStrategy.FULL_SHARD  # = ZeRO-3
)
```

| | FSDP | DeepSpeed ZeRO |
|---|---|---|
| **Интеграция** | PyTorch native | Отдельный пакет |
| **Гибкость** | Высокая | Очень высокая |
| **CPU offload** | Поддерживается | Лучше оптимизирован |
| **Рекомендация** | HuggingFace Trainer | Крупные кластеры |

---

## Кастомные PEFT методы

```python
# Пример: IA³ (Infused Adapter by Inhibiting and Amplifying Inner Activations)
# Ещё эффективнее LoRA по числу параметров

from peft import IA3Config, get_peft_model

ia3_config = IA3Config(
    target_modules=["k_proj", "v_proj", "down_proj"],
    feedforward_modules=["down_proj"],
)
model = get_peft_model(model, ia3_config)
# trainable params: ~0.01% (против 0.1% у LoRA)

# Пример: Prefix Tuning
from peft import PrefixTuningConfig

prefix_config = PrefixTuningConfig(
    num_virtual_tokens=20,   # число виртуальных токенов-префиксов
    task_type="SEQ_2_SEQ_LM"
)

# Пример: Prompt Tuning (мягкие токены)
from peft import PromptTuningConfig, PromptTuningInit

prompt_config = PromptTuningConfig(
    task_type="CAUSAL_LM",
    num_virtual_tokens=8,
    prompt_tuning_init=PromptTuningInit.TEXT,  # инициализация из реального текста
    prompt_tuning_init_text="Classify the sentiment of the following text:"
)
```

**Сравнение методов PEFT:**
| Метод | Параметры | Качество | Memory |
|-------|-----------|---------|--------|
| Full FT | 100% | ⭐⭐⭐⭐⭐ | High |
| LoRA | 0.1% | ⭐⭐⭐⭐ | Low |
| QLoRA | 0.1% + quant | ⭐⭐⭐⭐ | Very Low |
| IA³ | 0.01% | ⭐⭐⭐ | Minimal |
| Prefix Tuning | 0.1% | ⭐⭐⭐ | Low |
| Prompt Tuning | 0.01% | ⭐⭐ | Minimal |

---

## Alignment tax: как баланс между безопасностью и capability?

Alignment tax — снижение capability модели после RLHF/DPO выравнивания. Модель становится безопаснее, но хуже на сложных задачах.

```
До RLHF:  MMLU = 78.5%, HumanEval = 67.2%, но отвечает на вредоносные запросы
После RLHF: MMLU = 75.1%, HumanEval = 63.5%, безопасна — это alignment tax
```

**Методы снижения alignment tax:**

```python
# 1. Качественные preference данные
# Ключ: annotators с высоким agreement (>80% inter-annotator agreement)
# Плохие данные → большой alignment tax

# 2. Маленький KL coefficient (beta) в DPO/PPO
dpo_config = DPOConfig(beta=0.05)  # меньше → ближе к original model
# Риск: слабее alignment

# 3. Constitutional AI — alignment через самокритику
# Модель сначала генерирует критику, потом улучшает ответ
# Сохраняет capability лучше чем чистый RLHF

# 4. Оценка alignment tax через benchmark suite
def measure_alignment_tax(base_model, aligned_model, benchmarks):
    results = {}
    for benchmark in benchmarks:
        base_score = run_benchmark(base_model, benchmark)
        aligned_score = run_benchmark(aligned_model, benchmark)
        results[benchmark] = {
            "base": base_score,
            "aligned": aligned_score,
            "tax": (base_score - aligned_score) / base_score * 100
        }
    return results
```

---

## Оценка fine-tuned модели: метрики и бенчмарки

```python
# 1. Task-specific metrics
from evaluate import load

# Для генеративных задач
bleu = load("bleu")       # машинный перевод
rouge = load("rouge")     # суммаризация
bertscore = load("bertscore")  # семантическое сходство

# 2. Domain-specific benchmarks
domain_benchmarks = {
    "general": ["MMLU", "HellaSwag", "ARC"],    # общий интеллект
    "coding":  ["HumanEval", "MBPP", "SWE-bench"],
    "math":    ["GSM8K", "MATH", "AIME"],
    "medical": ["MedQA", "PubMedQA"],
    "legal":   ["LegalBench"],
}

# 3. Regression testing: не стало ли хуже на общих задачах?
def regression_test(new_model, golden_outputs, threshold=0.95):
    scores = []
    for item in golden_outputs:
        output = new_model.generate(item["input"])
        score = semantic_similarity(output, item["expected"])
        scores.append(score)

    pass_rate = sum(s > threshold for s in scores) / len(scores)
    assert pass_rate > 0.90, f"Regression: {pass_rate:.2%} < 90%"

# 4. Human evaluation для качественных аспектов
# Используй LLM-as-judge для масштабирования оценки
async def llm_eval(model_output: str, reference: str) -> dict:
    return await judge_llm(f"""
        Оцени ответ по 5 критериям (1-5):
        1. Точность фактов
        2. Полнота ответа
        3. Соблюдение инструкций
        4. Стиль и тон
        5. Безопасность
        
        Эталон: {reference}
        Ответ: {model_output}
        
        Верни JSON.
    """)
```
